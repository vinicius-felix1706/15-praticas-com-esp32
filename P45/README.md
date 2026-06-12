# P45: Fila simples entre sensor e atuador

## Objetivo

Explorar uma fila simples entre sensor e atuador para introduzir a
organização por tasks no ESP32, observando como prioridades, sincronização e
o tamanho da fila alteram a resposta global do sistema.

## Código-base (original)

```cpp
#include <Arduino.h>

QueueHandle_t fila;

void taskProdutor(void *pv) {
  while (true) {
    int valor = analogRead(34);
    xQueueSend(fila, &valor, portMAX_DELAY);
    vTaskDelay(pdMS_TO_TICKS(120));
  }
}

void taskConsumidor(void *pv) {
  pinMode(2, OUTPUT);
  int recebido;
  while (true) {
    if (xQueueReceive(fila, &recebido, portMAX_DELAY)) {
      digitalWrite(2, recebido > 2000);
      Serial.printf("Recebido=%d\n", recebido);
    }
  }
}

void setup() {
  Serial.begin(115200);
  fila = xQueueCreate(5, sizeof(int));
  xTaskCreatePinnedToCore(taskProdutor, "Produtor", 2048, NULL, 1, NULL, 1);
  xTaskCreatePinnedToCore(taskConsumidor, "Consumidor", 2048, NULL, 1, NULL, 1);
}

void loop() { }
```

## Configuração inicial

Potenciômetro no GPIO 34 (ADC), LED no GPIO 2. `taskProdutor` lê o ADC a cada
120 ms e envia para a fila. `taskConsumidor` recebe da fila, acende o LED se
o valor for maior que 2000, e imprime o valor no Serial Monitor. Fila com
capacidade para 5 itens, ambas as tasks com prioridade 1, no core 1.

## Modificações realizadas

1. **Ajuste do período da `taskProdutor`** (120 ms para 20 ms): a fila passou
   a receber novos valores muito mais rápido do que o Serial Monitor
   conseguia exibir. Como `taskConsumidor` processa um item por vez, a fila
   começou a se enchermos com mais facilidade, e o LED passou a refletir
   leituras mais antigas com um atraso perceptível em relação ao valor atual
   do potenciômetro, conforme a fila era esvaziada.

2. **Alteração de prioridade**: a `taskConsumidor` teve sua prioridade
   reduzida para 0, mantendo `taskProdutor` em 1:

```cpp
xTaskCreatePinnedToCore(taskProdutor, "Produtor", 2048, NULL, 1, NULL, 1);
xTaskCreatePinnedToCore(taskConsumidor, "Consumidor", 2048, NULL, 0, NULL, 1);
```

   Com `taskProdutor` em prioridade mais alta e período de 20 ms, a fila
   encheu com mais frequência, fazendo `xQueueSend` aguardar (bloquear) até
   que `taskConsumidor`, de prioridade menor, conseguisse esvaziar espaço.
   Isso introduziu atrasos adicionais na `taskProdutor`, evidenciando que a
   prioridade menor do consumidor limitou a taxa efetiva de produção.

3. **Ajuste do tamanho da fila** (5 para 1 e depois para 20):

```cpp
fila = xQueueCreate(1, sizeof(int));
// ...
fila = xQueueCreate(20, sizeof(int));
```

   Com fila de tamanho 1, cada novo valor produzido só podia ser enviado
   após o consumidor processar o anterior, tornando o sistema mais
   sincronizado, porém com `taskProdutor` frequentemente bloqueada à espera de
   espaço na fila. Com fila de tamanho 20, a `taskProdutor` praticamente
   nunca bloqueou, mas o `taskConsumidor` passou a processar uma sequência
   maior de valores acumulados antes de "alcançar" a leitura mais recente,
   aumentando o atraso entre a leitura real do potenciômetro e a reação do
   LED.

## Conclusão

O período da `taskProdutor` define a taxa de geração de dados, e quando essa
taxa supera a capacidade de consumo, a fila absorve o excesso até atingir sua
capacidade. A prioridade relativa entre produtor e consumidor determina quem
fica bloqueado quando a fila está cheia ou vazia. O tamanho da fila controla
o equilíbrio entre sincronização imediata (fila pequena, mais bloqueios) e
acúmulo de dados (fila grande, maior atraso entre leitura e reação, porém
menos bloqueios).

## Questionamentos

**1. Como a divisão em tasks ajudou ou atrapalhou a compreensão do comportamento do sistema?**

A divisão em tasks ajudou a separar claramente o papel de quem produz dados
(`taskProdutor`) e de quem consome e age sobre eles (`taskConsumidor`). A
parte mais desafiadora foi entender o papel da fila como um buffer
intermediário, cujo tamanho e velocidade de consumo afetam diretamente o
atraso entre a leitura do sensor e a resposta do LED.

**2. Qual alteração teve maior impacto: prioridade, sincronização, fila ou tempo de espera?**

O ajuste do tamanho da fila teve o maior impacto perceptível, pois alterou
diretamente a quantidade de dados acumulados entre produção e consumo, e
consequentemente o atraso entre a leitura do potenciômetro e a reação do LED.
O ajuste de período (tempo de espera) também teve impacto significativo, ao
aumentar a taxa de produção em relação à taxa de consumo.

**3. O sistema ficou mais fluido com fila maior ou menor? O que seus registros indicam?**

Com fila menor (tamanho 1), o sistema ficou mais sincronizado, porém com mais
bloqueios na `taskProdutor`, refletindo quase imediatamente o valor lido pelo
consumidor, mas limitando a taxa de produção à taxa de consumo. Com fila
maior (tamanho 20), a `taskProdutor` quase não bloqueou, mas o
`taskConsumidor` passou a processar uma fila de valores acumulados,
aumentando o atraso entre a leitura real do potenciômetro e a reação do LED.
Os registros indicam que "fluidez" no sentido de produção sem bloqueios
favorece filas maiores, enquanto "fluidez" no sentido de resposta imediata ao
valor atual favorece filas menores.
