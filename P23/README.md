# P23: Botão não bloqueante altera período do LED



## Objetivo

Explorar um botão não bloqueante que altera o período do LED para entender
resposta temporal, temporização sem bloqueio e o efeito de ajustes de período
ou eventos sobre a previsibilidade do sistema.

## Código-base (original)

```cpp
#include <Arduino.h>

const int LED_PIN = 2;
const int BTN_PIN = 18;
unsigned long ultimoToggle = 0;
unsigned long periodo = 600;
bool estadoLed = false;
bool ultimaLeitura = HIGH;

void setup() {
  pinMode(LED_PIN, OUTPUT);
  pinMode(BTN_PIN, INPUT_PULLUP);
  Serial.begin(115200);
}

void loop() {
  unsigned long agora = millis();
  if (agora - ultimoToggle >= periodo) {
    ultimoToggle = agora;
    estadoLed = !estadoLed;
    digitalWrite(LED_PIN, estadoLed);
  }

  bool leitura = digitalRead(BTN_PIN);
  if (ultimaLeitura == HIGH && leitura == LOW) {
    periodo = (periodo == 600) ? 200 : 600;
  }
  ultimaLeitura = leitura;
}
```

## Configuração inicial

LED no GPIO 2, botão no GPIO 18 (`INPUT_PULLUP`), período inicial de 600 ms,
alternando para 200 ms a cada toque (e voltando a 600 ms no toque seguinte).

## Modificações realizadas

1. **Troca de GPIO** (LED → 21, botão → 19): comportamento idêntico, o LED
   alternou o piscar entre 600 ms e 200 ms a cada toque, normalmente.
2. **Alteração do período principal** (600/200 ms → 1500/50 ms): a diferença
   entre os dois modos ficou muito mais perceptível — de um piscar lento
   (1500 ms) para um piscar quase contínuo (50 ms), tornando o efeito do
   botão muito mais evidente.
3. **Lógica do botão: borda → ativo em nível** (mantém `periodo = 50` apenas
   enquanto pressionado, voltando a 1500 ms ao soltar): o LED só piscou rápido
   durante a pressão contínua do botão, retornando ao ritmo lento
   imediatamente ao soltar — diferente do comportamento original, que mantinha
   o novo período mesmo após soltar o botão.

## Conclusão

A lógica não bloqueante permite que o LED continue piscando normalmente
enquanto o botão é monitorado em paralelo, sem travar o sistema. A diferença
entre os períodos define o quão perceptível é o efeito do botão, e a troca de
lógica (borda → nível) muda completamente a forma como o ajuste é
percebido — de uma alteração persistente para uma alteração temporária.

## Questionamentos

**1. Como o sistema reagiu ao pressionamento rápido ou repetido do botão?**

Na lógica de borda original, cada toque alternou o período de forma
confiável. Pressionamentos muito rápidos e repetidos puderam causar múltiplas
alternâncias seguidas (sem debounce), fazendo o `periodo` oscilar entre 600 e
200 ms várias vezes em sequência. Na lógica ativa em nível, pressionamentos
repetidos resultaram em o LED alternar rapidamente entre os dois ritmos,
acompanhando diretamente o estado do botão.

**2. Em que situação o uso de `delay()` pioraria esta prática em relação à solução atual?**

O uso de `delay()` pioraria a prática porque o LED deixaria de piscar de
forma independente do botão — qualquer `delay()` usado para o piscar do LED
bloquearia a leitura do botão durante esse intervalo, atrasando a detecção do
toque e tornando a resposta do sistema imprevisível.

**3. Que evidência experimental o grupo registrou para justificar sua conclusão?**

A evidência foi a comparação dos três cenários: com `millis()`, o LED piscou
continuamente nos períodos configurados (600/200 ms e depois 1500/50 ms)
enquanto o botão era lido a cada iteração do `loop()`, sem nenhuma pausa
perceptível na resposta ao toque, mesmo durante o piscar do LED.
