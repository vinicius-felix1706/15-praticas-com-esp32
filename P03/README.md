# P03: Botão aciona LED somente enquanto pressionado

| | |
|---|---|
| **Nível** | Fácil |
| **Componentes** | ESP32 DevKit, protoboard, botão tátil, LED vermelho, resistor |
| **Conceitos** | GPIO digital, Serial Monitor, ajuste de parâmetros |

## Objetivo

Explorar o acionamento de um LED por um botão (ativo apenas enquanto
pressionado), consolidando conceitos de GPIO e leitura/escrita digital no
ESP32.

## Código-base (original)

```cpp
#include <Arduino.h>

const int LED_PIN = 2;
const int BUTTON_PIN = 18;

void setup() {
  pinMode(LED_PIN, OUTPUT);
  pinMode(BUTTON_PIN, INPUT_PULLUP);
  Serial.begin(115200);
}

void loop() {
  bool pressionado = digitalRead(BUTTON_PIN) == LOW;
  digitalWrite(LED_PIN, pressionado ? HIGH : LOW);
  Serial.println(pressionado ? "Botao pressionado" : "Botao solto");
  delay(30);
}
```

## Configuração inicial

LED no GPIO 2, botão no GPIO 18 (`INPUT_PULLUP`, pressionado = LOW), leitura a
cada 30 ms.

## Modificações realizadas

1. **Troca de GPIO** (LED 2→21, botão 18→19): funcionamento idêntico ao
   original, confirma que o pino físico não altera a lógica.
2. **Intervalo de leitura** (30 ms → 300 ms): LED passou a responder com
   defasagem perceptível, e cliques muito rápidos às vezes não foram
   detectados.
3. **Lógica nível → borda** (toggle): cada toque agora alterna o LED entre
   ligado/desligado, mantendo o estado mesmo após soltar o botão
   (comportamento de interruptor em vez de momentâneo).



## Questionamentos

**1. Reação a pressionamentos rápidos/repetidos?**
Com 30 ms, detecção confiável; com 300 ms, cliques curtos podem ser
perdidos. Na versão toggle, pressionamentos rápidos alternam o LED várias
vezes, podendo gerar trocas extras sem debounce.

**2. Tipo de sinal usado?**
Digital — `digitalRead`/`digitalWrite` operam apenas com HIGH/LOW, sem PWM ou
variação contínua.

**3. Como tornar mais estável/sensível?**
Estabilidade: implementar debounce (software com `millis()` ou hardware com
capacitor). Sensibilidade: reduzir o intervalo de leitura ou usar `millis()`
no lugar de `delay()`.
