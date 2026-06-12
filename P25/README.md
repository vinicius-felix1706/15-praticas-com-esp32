# P25: Debounce temporal de botão



## Objetivo

Explorar o debounce temporal de botão para compreender resposta temporal,
temporização sem bloqueio e o efeito de ajustes de período ou eventos sobre
a previsibilidade do sistema.

## Código-base (original)

```cpp
#include <Arduino.h>

const int BTN_PIN = 18;
const int LED_PIN = 2;
bool estadoLed = false;
bool estadoBotao = HIGH;
bool ultimaLeitura = HIGH;
unsigned long ultimoBounce = 0;
const unsigned long debounceMs = 60;

void setup() {
  pinMode(BTN_PIN, INPUT_PULLUP);
  pinMode(LED_PIN, OUTPUT);
}

void loop() {
  bool leitura = digitalRead(BTN_PIN);

  if (leitura != ultimaLeitura) ultimoBounce = millis();
  if ((millis() - ultimoBounce) > debounceMs) {
    if (leitura != estadoBotao) {
      estadoBotao = leitura;
      if (estadoBotao == LOW) {
        estadoLed = !estadoLed;
        digitalWrite(LED_PIN, estadoLed);
      }
    }
  }
  ultimaLeitura = leitura;
}
```

## Configuração inicial

LED no GPIO 2, botão no GPIO 18 (`INPUT_PULLUP`), debounce de 60 ms, lógica
de alternância por borda (toggle ao pressionar).

## Modificações realizadas

1. **Troca de GPIO** (LED 2 para 21, botão 18 para 19): comportamento
   idêntico, o LED alternou normalmente a cada toque estável.
2. **Alteração do tempo de debounce** (60 ms para 5 ms): com o tempo muito
   curto, alguns toques causaram mais de uma alternância do LED, indicando
   que o ruído mecânico do botão não estava sendo filtrado adequadamente.
   Com o tempo aumentado para 150 ms, o sistema ficou mais robusto contra
   ruído, mas pressionamentos muito rápidos e seguidos passaram a ser
   parcialmente ignorados.
3. **Lógica do botão: borda para ativo em nível** (LED aceso apenas enquanto
   o botão está pressionado, mantendo o debounce): o LED deixou de manter o
   estado após soltar o botão, passando a funcionar como acionamento
   momentâneo em vez de interruptor.

```cpp
if ((millis() - ultimoBounce) > debounceMs) {
  if (leitura != estadoBotao) {
    estadoBotao = leitura;
  }
}
digitalWrite(LED_PIN, estadoBotao == LOW);
```

## Conclusão

O debounce temporal com `millis()` filtra ruídos do botão sem bloquear o
sistema. O tempo de debounce precisa equilibrar a filtragem do ruído mecânico
com a responsividade a toques rápidos. A escolha entre lógica de borda e
nível define se o ajuste do LED é persistente ou temporário, independente do
debounce aplicado.

## Questionamentos

**1. Como o sistema reagiu ao pressionamento rápido ou repetido do botão?**

Com debounce de 60 ms, toques rápidos foram detectados de forma confiável,
uma alternância por toque. Com 5 ms, alguns toques geraram alternâncias
duplicadas devido ao ruído mecânico não filtrado. Com 150 ms, toques muito
seguidos por vezes não foram contabilizados individualmente, sendo tratados
como um único evento.

**2. Em que situação o uso de `delay()` pioraria esta prática em relação à solução atual?**

O uso de `delay()` para implementar o debounce pioraria a prática porque
bloquearia o `loop()` durante o intervalo de espera, impedindo a leitura
contínua do botão e de qualquer outro evento (como um segundo botão ou
sensor) durante esse período, o que não ocorre com a abordagem baseada em
`millis()`.

**3. Que evidência experimental o grupo registrou para justificar sua conclusão?**

A evidência foi a comparação entre os três valores de debounce, 5 ms, 60 ms
e 150 ms, em uma série de toques rápidos no botão. Com 5 ms, o número de
alternâncias do LED foi maior que o número de toques intencionais. Com 60 ms,
o número de alternâncias correspondeu ao número de toques. Com 150 ms, alguns
toques muito próximos não geraram alternância adicional.
