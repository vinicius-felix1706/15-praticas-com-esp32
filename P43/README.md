# P43: Task de leitura analógica e task de serial

## Objetivo

Explorar uma task de leitura analógica e uma task de serial para introduzir
a organização por tasks no ESP32, observando como prioridades, sincronização
e tempos de espera alteram a resposta global do sistema.

## Código-base (original)

```cpp
#include <Arduino.h>

volatile int ultimaLeitura = 0;

void taskSensor(void *pv) {
  while (true) {
    ultimaLeitura = analogRead(34);
    vTaskDelay(pdMS_TO_TICKS(100));
  }
}

void taskSerial(void *pv) {
  while (true) {
    Serial.printf("ADC=%d\n", ultimaLeitura);
    vTaskDelay(pdMS_TO_TICKS(500));
  }
}

void setup() {
  Serial.begin(115200);
  xTaskCreatePinnedToCore(taskSensor, "Sensor", 2048, NULL, 1, NULL, 1);
  xTaskCreatePinnedToCore(taskSerial, "Serial", 2048, NULL, 1, NULL, 1);
}

void loop() { }
```

## Configuração inicial

Potenciômetro ligado ao GPIO 34 (ADC). `taskSensor` lê o valor a cada 100 ms
e armazena em `ultimaLeitura`. `taskSerial` imprime esse valor a cada 500 ms.
Ambas as tasks com prioridade 1, no core 1.

## Modificações realizadas

1. **Ajuste do período das tasks** (`taskSensor`: 100 ms para 1000 ms,
   `taskSerial`: 500 ms para 200 ms): com a leitura muito mais lenta que a
   impressão, o Serial Monitor passou a mostrar o mesmo valor de `ADC`
   repetido várias vezes seguidas entre cada nova leitura, evidenciando que
   a `taskSerial` estava lendo a mesma informação repetidas vezes antes da
   próxima atualização.

2. **Alteração de prioridade**: a prioridade da `taskSensor` foi reduzida
   para 0, mantendo `taskSerial` em 1:

```cpp
xTaskCreatePinnedToCore(taskSensor, "Sensor", 2048, NULL, 0, NULL, 1);
xTaskCreatePinnedToCore(taskSerial, "Serial", 2048, NULL, 1, NULL, 1);
```

   Como ambas as tasks usam `vTaskDelay`, não houve diferença perceptível no
   comportamento, ambas continuaram executando normalmente nos seus
   períodos configurados, indicando que a diferença de prioridade não foi
   determinante enquanto as duas tasks cedem CPU regularmente.

3. **Comunicação via fila (Queue) no lugar da variável `volatile`**: a
   variável compartilhada foi substituída por uma fila, garantindo que cada
   leitura seja consumida no máximo uma vez:

```cpp
QueueHandle_t filaLeitura;

void taskSensor(void *pv) {
  while (true) {
    int valor = analogRead(34);
    xQueueOverwrite(filaLeitura, &valor);
    vTaskDelay(pdMS_TO_TICKS(100));
  }
}

void taskSerial(void *pv) {
  int valor;
  while (true) {
    if (xQueuePeek(filaLeitura, &valor, 0) == pdTRUE) {
      Serial.printf("ADC=%d\n", valor);
    }
    vTaskDelay(pdMS_TO_TICKS(500));
  }
}

void setup() {
  Serial.begin(115200);
  filaLeitura = xQueueCreate(1, sizeof(int));
  xTaskCreatePinnedToCore(taskSensor, "Sensor", 2048, NULL, 1, NULL, 1);
  xTaskCreatePinnedToCore(taskSerial, "Serial", 2048, NULL, 1, NULL, 1);
}
```

   O comportamento observado foi equivalente ao da variável `volatile`
   original, já que ambas as tasks acessam um único valor de cada vez. A
   vantagem da fila apareceu na clareza do código, deixando explícito que o
   acesso ao valor é gerenciado pelo FreeRTOS, em vez de depender apenas da
   palavra-chave `volatile` para evitar otimizações indesejadas.

## Conclusão

Os períodos das duas tasks podem ser ajustados de forma independente, mas a
diferença entre eles afeta diretamente a utilidade da informação exibida, uma
`taskSerial` muito mais rápida que a `taskSensor` apenas repete valores
antigos. A prioridade entre tasks que usam `vTaskDelay` regularmente não
altera de forma perceptível o comportamento. A troca de variável `volatile`
por fila não mudou o resultado observado, mas tornou explícito o mecanismo de
compartilhamento de dados entre as tasks.

## Questionamentos

**1. Como a divisão em tasks ajudou ou atrapalhou a compreensão do comportamento do sistema?**

A divisão em tasks ajudou a separar claramente a responsabilidade de leitura
do sensor da responsabilidade de exibição no Serial Monitor. A parte que
exigiu mais atenção foi entender que, com períodos diferentes, a `taskSerial`
pode exibir o mesmo valor várias vezes, o que não seria óbvio apenas olhando
para o código de cada task isoladamente.

**2. Qual alteração teve maior impacto: prioridade, sincronização, fila ou tempo de espera?**

O ajuste de tempo de espera teve o maior impacto perceptível, pois alterou
diretamente a relação entre a frequência de leitura e a frequência de
exibição, mudando a quantidade de valores repetidos no Serial Monitor. A
alteração de prioridade não teve impacto perceptível, e a troca para fila não
alterou o resultado observado, apenas a forma como o compartilhamento de
dados é implementado.

**3. Que cuidado de projeto você adotaria se esta prática fosse levada para um sistema real?**

Garantir que a frequência de leitura de um sensor seja igual ou maior que a
frequência de uso desses dados por outras tasks, evitando processar ou exibir
valores repetidos como se fossem novas leituras, e preferir mecanismos como
filas (`xQueueOverwrite`/`xQueuePeek`) a variáveis `volatile` simples quando
o dado compartilhado precisar de garantias mais claras de consistência.
