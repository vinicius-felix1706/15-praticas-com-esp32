# P41: Primeira task FreeRTOS piscando LED


## Objetivo

Explorar a primeira task FreeRTOS piscando LED para introduzir a organização
por tasks no ESP32, observando como prioridades, sincronização e tempos de
espera alteram a resposta global do sistema.

## Código-base (original)

```cpp
#include <Arduino.h>

void tarefaLed(void *pv) {
  pinMode(2, OUTPUT);
  while (true) {
    digitalWrite(2, !digitalRead(2));
    vTaskDelay(pdMS_TO_TICKS(500));
  }
}

void setup() {
  xTaskCreatePinnedToCore(tarefaLed, "LedTask", 2048, NULL, 1, NULL, 1);
}

void loop() { }
```

## Configuração inicial

LED vermelho no GPIO 2, controlado por uma task chamada `LedTask`, com
prioridade 1, fixada no core 1, alternando o estado a cada 500 ms via
`vTaskDelay`.

## Modificações realizadas

1. **Ajuste do período da task** (500 ms para 100 ms): o LED passou a piscar
   muito mais rápido, quase em tremulação. O `loop()` vazio continuou
   funcionando normalmente, sem impacto visível na task.

2. **Adição de uma segunda task com prioridade mais alta**: foi criada uma
   segunda task (`TaskOcupada`), com prioridade 2, que executa um laço
   ocupado (busy loop) sem `vTaskDelay`:

```cpp
void tarefaOcupada(void *pv) {
  while (true) {
    // laço ocupado, sem vTaskDelay
  }
}

void setup() {
  xTaskCreatePinnedToCore(tarefaLed, "LedTask", 2048, NULL, 1, NULL, 1);
  xTaskCreatePinnedToCore(tarefaOcupada, "TaskOcupada", 2048, NULL, 2, NULL, 1);
}
```

   Com as duas tasks no mesmo core e a `TaskOcupada` com prioridade maior, o
   LED parou de piscar completamente, pois a `LedTask` nunca recebeu tempo de
   CPU. Isso mostrou que prioridades mais altas, sem `vTaskDelay`, podem
   monopolizar o processador.

3. **Mudança de mecanismo: variável compartilhada para fila (Queue)**: a
   segunda task passou a enviar comandos de período via `xQueueSend`, e a
   `LedTask` passou a ler esse valor com `xQueueReceive` (com timeout) antes
   de cada `vTaskDelay`:

```cpp
QueueHandle_t filaPeriodo;

void tarefaLed(void *pv) {
  pinMode(2, OUTPUT);
  unsigned long periodo = 500;
  while (true) {
    unsigned long novo;
    if (xQueueReceive(filaPeriodo, &novo, 0) == pdTRUE) {
      periodo = novo;
    }
    digitalWrite(2, !digitalRead(2));
    vTaskDelay(pdMS_TO_TICKS(periodo));
  }
}
```

   O LED passou a responder a novos valores de período enviados por outra
   task, sem que as duas tasks acessassem diretamente a mesma variável,
   evitando condições de corrida.

## Conclusão

O período de uma task pode ser ajustado livremente, alterando a resposta
visual do LED sem afetar o restante do programa. A prioridade entre tasks
determina quem recebe tempo de CPU, e tasks de alta prioridade sem
`vTaskDelay` podem impedir totalmente a execução de tasks de prioridade
menor. O uso de fila (Queue) para comunicação entre tasks é mais seguro do
que compartilhar variáveis diretamente, pois evita acesso simultâneo aos
mesmos dados.

## Questionamentos

**1. Como a divisão em tasks ajudou ou atrapalhou a compreensão do comportamento do sistema?**

A divisão em tasks ajudou a isolar a responsabilidade de piscar o LED em um
bloco de código independente, facilitando a leitura. Por outro lado, ao
adicionar a segunda task de alta prioridade, ficou mais difícil prever o
comportamento geral sem entender como o agendador do FreeRTOS distribui o
tempo de CPU entre as tasks.

**2. Qual alteração teve maior impacto: prioridade, sincronização, fila ou tempo de espera?**

A alteração de prioridade teve o maior impacto. A simples adição de uma task
de prioridade mais alta sem `vTaskDelay` foi suficiente para travar
completamente o piscar do LED, um efeito muito mais drástico do que a
mudança de período (de 500 ms para 100 ms) ou a troca de variável
compartilhada por fila.

**3. Que cuidado de projeto você adotaria se esta prática fosse levada para um sistema real?**

Sempre incluir `vTaskDelay` (ou outro ponto de bloqueio) em qualquer task de
alta prioridade, para garantir que tasks de prioridade menor também recebam
tempo de CPU, e preferir mecanismos como filas, semáforos ou mutexes para
comunicação entre tasks, em vez de variáveis globais compartilhadas sem
proteção.
