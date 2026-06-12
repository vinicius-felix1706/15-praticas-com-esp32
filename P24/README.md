# P24: Semáforo não bloqueante por máquina de estados



## Objetivo

Explorar um semáforo não bloqueante por máquina de estados para compreender
resposta temporal, temporização sem bloqueio e o efeito de ajustes de
período ou eventos sobre a previsibilidade do sistema.

## Código-base (original)

```cpp
#include <Arduino.h>

enum Estado { VERDE, AMARELO, VERMELHO };
const int LED_G = 5, LED_Y = 4, LED_R = 2;
Estado estado = VERDE;
unsigned long anterior = 0;

void aplicarSaidas() {
  digitalWrite(LED_G, estado == VERDE);
  digitalWrite(LED_Y, estado == AMARELO);
  digitalWrite(LED_R, estado == VERMELHO);
}

void setup() {
  pinMode(LED_G, OUTPUT);
  pinMode(LED_Y, OUTPUT);
  pinMode(LED_R, OUTPUT);
  aplicarSaidas();
}

void loop() {
  unsigned long agora = millis();
  unsigned long tempoEstado = (estado == AMARELO) ? 700 : 2000;

  if (agora - anterior >= tempoEstado) {
    anterior = agora;
    if (estado == VERDE) estado = AMARELO;
    else if (estado == AMARELO) estado = VERMELHO;
    else estado = VERDE;
    aplicarSaidas();
  }
}
```

## Configuração inicial

LED verde no GPIO 5, amarelo no GPIO 4, vermelho no GPIO 2. Estados VERDE e
VERMELHO duram 2000 ms, AMARELO dura 700 ms.

## Modificações realizadas

1. **Troca de GPIO** (LED verde 5 para 21): comportamento idêntico, o ciclo
   verde, amarelo, vermelho seguiu nos mesmos tempos, confirmando que a
   lógica depende do código e não do pino.
2. **Alteração dos tempos** (amarelo 700 ms para 300 ms, vermelho 2000 ms
   para 3500 ms): o amarelo ficou quase imperceptível e o vermelho passou a
   dominar boa parte do ciclo, alterando bastante a sensação de ritmo do
   semáforo.
3. **Acréscimo de segundo evento** (buzzer no GPIO 19, ativo durante o
   estado AMARELO): o buzzer soou exatamente durante o intervalo do amarelo,
   reforçando a transição mesmo com duração curta de 300 ms.

```cpp
const int BUZZER_PIN = 19;

void aplicarSaidas() {
  digitalWrite(LED_G, estado == VERDE);
  digitalWrite(LED_Y, estado == AMARELO);
  digitalWrite(LED_R, estado == VERMELHO);
  digitalWrite(BUZZER_PIN, estado == AMARELO);
}
```

## Conclusão

A máquina de estados com `millis()` permite alterar a duração de cada estado
de forma independente, sem travar o sistema. O pino GPIO usado não afeta a
lógica. Estados muito curtos podem ficar pouco perceptíveis visualmente, mas
podem ser reforçados por outro canal, como som, sincronizado ao mesmo estado.

## Questionamentos

**1. Qual ajuste temporal mais alterou a previsibilidade do sistema e como isso foi percebido?**

A alteração dos tempos (amarelo para 300 ms e vermelho para 3500 ms) foi a
que mais alterou a previsibilidade. O amarelo passou a ser quase um flash, e
o vermelho dominou o ciclo, mudando significativamente a proporção entre os
estados percebida pelo usuário.

**2. Em que situação o uso de `delay()` pioraria esta prática em relação à solução atual?**

O uso de `delay()` pioraria a prática se fosse necessário monitorar algo em
paralelo, como um botão para forçar uma mudança de estado ou um sensor.
Com `delay()`, o sistema ficaria bloqueado durante cada estado, impedindo
qualquer leitura simultânea, enquanto a máquina de estados com `millis()`
permite adicionar essas verificações sem travar o ciclo do semáforo.

**3. Como você explicaria a lógica de estados deste experimento para outra equipe?**

O sistema é uma máquina de estados cíclica com três estados, VERDE, AMARELO
e VERMELHO. Cada estado tem uma duração própria armazenada em
`tempoEstado`, e a função `aplicarSaidas()` atualiza os LEDs (e o buzzer, na
versão modificada) de acordo com o estado atual. Quando o tempo do estado
atual se esgota, o sistema avança para o próximo estado na sequência e
reinicia a contagem, sem nunca bloquear o `loop()` com `delay()`.
