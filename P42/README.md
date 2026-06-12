# P42:  Duas tasks com dois LEDs


## Objetivo

Explorar duas tasks com dois LEDs para introduzir a organização por tasks no
ESP32, observando como prioridades, sincronização e tempos de espera alteram
a resposta global do sistema.

## Código-base (original)

```cpp
#include <Arduino.h>

void taskA(void *pv) {
  pinMode(2, OUTPUT);
  while (true) {
    digitalWrite(2, !digitalRead(2));
    vTaskDelay(pdMS_TO_TICKS(300));
  }
}

void taskB(void *pv) {
  pinMode(4, OUTPUT);
  while (true) {
    digitalWrite(4, !digitalRead(4));
    vTaskDelay(pdMS_TO_TICKS(700));
  }
}

void setup() {
  xTaskCreatePinnedToCore(taskA, "TaskA", 2048, NULL, 1, NULL, 1);
  xTaskCreatePinnedToCore(taskB, "TaskB", 2048, NULL, 1, NULL, 1);
}

void loop() { }
```

## Configuração inicial

LED A no GPIO 2 (período de 300 ms), LED B no GPIO 4 (período de 700 ms),
ambas as tasks com prioridade 1, fixadas no core 1.

## Modificações realizadas

1. **Ajuste do período de uma task** (taskA: 300 ms para 50 ms): o LED A
   passou a piscar quase em tremulação contínua, enquanto o LED B continuou
   piscando normalmente a cada 700 ms, sem qualquer alteração no seu ritmo.

2. **Alteração de prioridade entre as tasks**: a prioridade da `taskA` foi
   aumentada para 2, mantendo `taskB` em 1:

```cpp
xTaskCreatePinnedToCore(taskA, "TaskA", 2048, NULL, 2, NULL, 1);
xTaskCreatePinnedToCore(taskB, "TaskB", 2048, NULL, 1, NULL, 1);
```

   Como ambas as tasks usam `vTaskDelay` (cedendo CPU durante a espera), os
   dois LEDs continuaram piscando normalmente em seus respectivos períodos,
   sem disputa perceptível pelo processador. A diferença de prioridade só se
   tornaria crítica se uma das tasks deixasse de ceder CPU (por exemplo, com
   um laço ocupado sem `vTaskDelay`).

3. **Comunicação entre tasks via fila (Queue)**: foi criada uma fila para que
   a `taskB` envie um comando que altera o período da `taskA`:

```cpp
QueueHandle_t filaPeriodo;

void taskA(void *pv) {
  pinMode(2, OUTPUT);
  unsigned long periodo = 300;
  while (true) {
    unsigned long novo;
    if (xQueueReceive(filaPeriodo, &novo, 0) == pdTRUE) {
      periodo = novo;
    }
    digitalWrite(2, !digitalRead(2));
    vTaskDelay(pdMS_TO_TICKS(periodo));
  }
}

void taskB(void *pv) {
  pinMode(4, OUTPUT);
  unsigned long contador = 0;
  while (true) {
    digitalWrite(4, !digitalRead(4));
    contador++;
    if (contador % 5 == 0) {
      unsigned long novoPeriodo = 50;
      xQueueSend(filaPeriodo, &novoPeriodo, 0);
    }
    vTaskDelay(pdMS_TO_TICKS(700));
  }
}

void setup() {
  filaPeriodo = xQueueCreate(5, sizeof(unsigned long));
  xTaskCreatePinnedToCore(taskA, "TaskA", 2048, NULL, 1, NULL, 1);
  xTaskCreatePinnedToCore(taskB, "TaskB", 2048, NULL, 1, NULL, 1);
}
```

   A cada 5 ciclos da `taskB` (cerca de 3,5 segundos), o LED A passou a piscar
   muito mais rápido (50 ms), demonstrando uma task influenciando o
   comportamento de outra de forma segura, sem variáveis compartilhadas
   diretamente.

## Conclusão

Cada task pode ter seu próprio período ajustado independentemente, sem
afetar as demais, desde que todas usem `vTaskDelay` para ceder CPU. A
diferença de prioridade entre tasks só se torna determinante quando alguma
task deixa de ceder o processador. O uso de fila permite que uma task envie
informações para outra de forma controlada, possibilitando comportamentos
coordenados entre LEDs sem acesso direto a variáveis compartilhadas.

## Questionamentos

**1. Como a divisão em tasks ajudou ou atrapalhou a compreensão do comportamento do sistema?**

A divisão em tasks ajudou a isolar cada LED em sua própria lógica e período,
facilitando o entendimento individual de cada comportamento. A parte mais
complexa foi entender a comunicação entre as tasks via fila, que exigiu
acompanhar o fluxo de dados entre `taskB` e `taskA` para compreender o efeito
no LED A.

**2. Qual alteração teve maior impacto: prioridade, sincronização, fila ou tempo de espera?**

O ajuste de tempo de espera (período da taskA) teve o impacto mais visível
e imediato no comportamento do LED A. A alteração de prioridade não teve
impacto perceptível neste caso, pois ambas as tasks cediam CPU com
`vTaskDelay`. A fila teve impacto indireto, mas relevante, pois permitiu que
uma task alterasse o comportamento da outra de forma coordenada.

**3. Que cuidado de projeto você adotaria se esta prática fosse levada para um sistema real?**

Garantir que toda task que compartilha recursos com outras (variáveis, filas,
periféricos) utilize mecanismos seguros de comunicação, como filas,
semáforos ou mutexes, e sempre incluir pontos de cessão de CPU
(`vTaskDelay`) em cada task, evitando que uma task monopolize o processador e
impeça a execução das demais.
