# Projeto da Matéria de Computação Física e Aplicações - ACH2157

## Descrição

Muitas vezes quando um motociclista necessita realizar uma curva, a seta sinalizadora da moto geralmente não fica razoalvemente visível para os outros condutores. No caso dos ciclistas, nem há uma seta sinalizadora.

Por isso fizemos um colete com setas sinalizadoras para os ciclistas e motociclistas, com o intuito de melhorar a visibilidade deles nas vias.

### Lista de Materiais

| Quantidade | Nome | Link para referência |
| --- | --- | --- |
| 1 | ESP32-WROOM-32 Devkit V1 | https://www.espressif.com/sites/default/files/documentation/esp32-wroom-32_datasheet_en.pdf |
| 1 | fitas de LEDs aRGB | https://pt.aliexpress.com/item/2036819167.html?spm=a2g0o.productlist.main.3.6a0d39dfyRa073&algo_pvid=42e9d96a-c2fd-4721-afd4-f73c281b3c5c&algo_exp_id=42e9d96a-c2fd-4721-afd4-f73c281b3c5c-1&pdp_ext_f=%7B%22sku_id%22%3A%2267389781287%22%7D&pdp_npi=2%40dis%21BRL%2179.72%2159.81%21%21%21%21%21%40211be3d216719178731788378d0781%2167389781287%21sea&curPageLogUid=5lKpX7bm5Spr |
| 1 | Módulo RF YK04 | https://www.faranux.com/product/4-channels-rf-remote-control-module-yk04/ |
| 1 | Sensor de Som Modelo MicNakano | https://github.com/FNakano/CFA/tree/master/projetos/sensorDeSom |
| X | Jumpers variados | --- |
| 2 | Power banks | --- |

Obs: Utilizamos 2 power banks, um para o ESP32 e outro para os LEDs, pois com apenas um os LEDs aRGBs estavam roubando muita energia e o ESP ficou instável)

ESP32:

![esp](/img/esp.jpg)

Sensor de Som Modelo MicNakano:

![mic_nakano](/img/mic_nakano.jpg)

Módulo RF YK04:

![yk04](/img/yk04.jpg)

### Conexões

| Pino do NodeMCU | Pino do Motor | Pino das pilhas |
| --- | --- | --- |
| 25 | Laranja | --- |
| GND | Marrom | Preto (Negativo das pilhas) |
| --- | Vermelho | Vermelho (Positivo das pilhas |

**Nota**: Vale ressaltar que o GND da fita de LEDs tem que estar junto com o GND do ESP32, se não as cores da fita não irão funcinar corretamente.

## Como montar o dispositivo físico

Fazer as conexões listadas, transferir o código `sketch_oct14a.ino` para o ESP32, e ligar o ESP32 e os LEDs com os power banks.

## Arquitetura e organização

Figura 1 - Feito usando yEd, arquivo-fonte da figura em /docs/Rede.graphml:

![rede](/docs/Rede.png)

O dispositivo (digitalLocker) conecta-se ao ponto de acesso wi-fi como um cliente wi-fi e obtém um endereço IP local. Através do navegador, outros dispositivos podem navegar (fazer requisições HTTP) para o endereço IP e receberão como resposta uma página web contendo dois botões. Clicar nos botões causa o envio de uma nova requisição (HTTP:GET) que, quando recebida pelo digitalLocker, causa o giro do servo motor e o envio da resposta para o navegador.

O dispositivo pode ser visto como a interconexão do motor com o modem wifi (embutido no controlador) e o controlador. A interface entre o programador e o hardware do controlador é feita através de Micropython. O programa `digitalLocker.py` contém os comandos para conectar ao wifi (como cliente), funcionar como um servidor web e controlar o motor. Uma ilustração é apresentada na figura 2.

Figura 2- Feito usando yEd, arquivo-fonte da figura em /docs/layerModel.graphml:

![camadas](/docs/layerModel.png)


## Como usar o programa

Para executar `digitalLocker.py` no Node, este deve estar carregado com Micropython. Instruções sobre como carregar Micropython neste [link externo](https://github.com/FNakano/CFA/tree/master/programas/Micropython). Depois de carregar, ou transferir o programa ou executá-lo usando, por exemplo WebREPL (instruções neste [link externo]()https://github.com/FNakano/CFA/tree/master/programas/Micropython/webREPL), ou o método que preferir. No exemplo, uso Thonny e envio `digitalLocker.py` para o Node. No arquivo é definida a função `startServer()`. Desta forma, no REPL, digitar `import digitalLocker` para importar a função e digitar `digitalLocker.startServer()` para iniciar o servidor. Isto é mais cômodo que executar os comandos um por um, seja digitando, seja com copy-paste.

## Como o motor funciona

![Quando tiver vídeo da operação com navegador, transferir este para a explicação do servo.](./docs/output.gif)

O servomotor é um motor em que o eixo gira menos de uma volta e o ângulo de giro do eixo pode ser controlado. O motor específico desta montagem permite ângulos entre zero e 180 graus (especificação técnica: http://www.datasheet-pdf.com/PDF/SG90-Datasheet-TowerPro-791970). Este motor recebe energia pelos fios marrom (GND) e vermelho (VCC). A tensão de alimentação pode ser algo entre 4V e 7.2V. Nesta montagem será 6V. O fio laranja conduz o sinal de controle para o motor.

O sinal de controle é um trem de pulsos de 20ms (50Hz), com duração do patamar em nível 1 variando entre 1 e 2ms. O ângulo de giro é proporcional à duração do patamar em nível 1. Por exemplo, com pulsos de 1,5ms durante o intervalo de 20ms, o ângulo de giro é de 90 graus (aproximadamente); com pulsos de 2ms, o ângulo é de 180 graus (https://www.engineersgarage.com/servo-motor-sg90-9g-with-89c51-microcontroller/). Esse tipo de sinal pertence à categoria dos sinais em *Pulse Width Modulation* (PWM).

Um sinal PWM é especificado pela frequência e pelo ciclo de carga (*duty-cycle*). O ciclo de carga é o percentual do tempo em que o sinal fica em nível 1 comparado com o período todo do sinal. Por exemplo, um sinal de 50Hz tem período de 20ms. Se o ciclo de carga for 20%, durante 20% desse período (ié 4ms), o sinal fica em nível 1 e o restante do tempo (16ms) fica em nível zero. Se o ciclo de carga for 50%, o patamar 1 dura 10ms e o patamar zero dura 10ms.

## Como enviar comandos para o motor

O ESP32 tem geradores PWM com frequência e ciclo de carga (*duty-cycle*) ajustáveis. O ciclo de carga é codificado como um inteiro entre 0 e 1023, que correspode (linearmente) ao ciclo de carga de zero até 100%. Como o motor responde a um sinal de duração de 1-2ms, o comando do ESP vai de aprox. 50-100.

Para usá-los com Micropython, digitando o programa abaixo, o servo é colocado em um ângulo perto de zero. `motor.duty()` pode ser executados com outros valores, por exemplo, 60, 100, 120, para diferentes ângulos.

```python
from machine import Pin, PWM
p25 = Pin(25, Pin.OUT)         # configura o pino 25 como saída
motor = PWM(p25, freq=50)      # configura o pino 25 como PWM a 50Hz
motor.duty(40)                 # o patamar 1 dura 40/1024 do período 
```
Fonte: https://docs.micropython.org/en/latest/esp8266/tutorial/pwm.html#control-a-hobby-servo

## Como o programa foi feito

```python
p25 = Pin(25, Pin.OUT)
motor = PWM(p25, freq=50)
motor.duty(40)
```
Fonte: https://docs.micropython.org/en/latest/esp8266/tutorial/pwm.html#control-a-hobby-servo

```python
s.bind(('', 3000))   # bind to port 3000 https://docs.micropython.org/en/latest/library/socket.html#socket.socket.bind
s.listen(2)  # allow for 2 connection before refusing https://docs.micropython.org/en/latest/library/socket.html#socket.socket.listen
```

Uma requisição GET é feita (pelo navegador) concatenando, à URL (que endereça a requisição para o servidor), os parâmetros, no formato `?<id>=<valor>` no texto da requisição.

```python
    request = conn.recv(1024)  # get bytes https://docs.micropython.org/en/latest/library/socket.html#socket.socket.recv
    request = str(request)     # convert to string
```

Sobre o texto (string) da requisição, busca-se o parâmetro do GET:

```python
locker_on = request.find('/?locker=on') # find get request text https://www.w3schools.com/python/ref_string_find.asp
```

Em função do valor, o eixo do motor é girado a mais ou a menos:

```python
    if locker_on == 6:
        print('LOCKER ON')
        motor.duty(110)
        locker_state = "ON"
    if locker_off == 6:
        print('LOCKER OFF')
        motor.duty(40)
        locker_state = "OFF"
```

Ao final, a página é reenviada (não precisaria), para permitir um novo comando abre/fecha, junto com um código de resposta `HTTP-200` que significa OK (https://developer.mozilla.org/en-US/docs/Web/HTTP/Status).

## Referências

https://randomnerdtutorials.com/esp32-esp8266-micropython-web-server/
https://docs.micropython.org/en/latest/esp8266/tutorial/network_tcp.html#http-get-request


## Colaborar usando github (meta)

A maneira indicada pelo mantenedor do github para colaborar com projetos hospedados nele é através de bifurcação e pull request: https://stackoverflow.com/questions/32750228/how-to-contribute-to-someone-elses-repository.

É possível ser colaborador do projeto e fazer pull/push (https://stackoverflow.com/questions/42619669/how-to-make-branch-from-friends-repository), mas isto pode gerar confusão (quando fiz isso com outra pessoa eu a confundi e ela acabou reiniciando o repositório - `git init` - não foi um bom resultado).

Quando procurei essa informação, nas postagens, encontrei um material interessante sobre a organização (interna) do git: https://stackoverflow.com/tags/git/info

Sobre bifurcação: https://docs.github.com/pt/get-started/quickstart/fork-a-repo, https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/about-forks.