# P44: Task de botão controla task de LED

## Objetivo

Explorar uma task de botão controlando uma task de LED para introduzir a
organização por tasks no ESP32, observando como prioridades, sincronização
e tempos de espera alteram a resposta global do sistema.

## Código-base (original)

```cpp
#include <Arduino.h>

volatile bool habilitaPisca = false;

void taskBotao(void *pv) {
  pinMode(18, INPUT_PULLUP);
  bool ultima = HIGH;
  while (true) {
    bool leitura = digitalRead(18);
    if (ultima == HIGH && leitura == LOW) {
      habilitaPisca = !habilitaPisca;
    }
    ultima = leitura;
    vTaskDelay(pdMS_TO_TICKS(40));
  }
}

void taskLed(void *pv) {
  pinMode(2, OUTPUT);
  while (true) {
    if (habilitaPisca) digitalWrite(2, !digitalRead(2));
    else digitalWrite(2, LOW);
    vTaskDelay(pdMS_TO_TICKS(300));
  }
}

void setup() {
  xTaskCreatePinnedToCore(taskBotao, "Botao", 2048, NULL, 1, NULL, 1);
  xTaskCreatePinnedToCore(taskLed, "Led", 2048, NULL, 1, NULL, 1);
}

void loop() { }
```

## Configuração inicial

Botão no GPIO 18 (`INPUT_PULLUP`), lido a cada 40 ms, com lógica de borda
(toggle). LED no GPIO 2, piscando a cada 300 ms apenas quando `habilitaPisca`
está ativo. Ambas as tasks com prioridade 1, no core 1.

## Modificações realizadas

1. **Ajuste do período da `taskLed`** (300 ms para 80 ms): com
   `habilitaPisca` ativo, o LED passou a piscar muito mais rápido, quase em
   tremulação. O comportamento do botão (toggle de `habilitaPisca`)
   continuou idêntico, já que o período da `taskBotao` não foi alterado.

2. **Alteração de prioridade**: a `taskBotao` teve sua prioridade aumentada
   para 2, mantendo `taskLed` em 1:

```cpp
xTaskCreatePinnedToCore(taskBotao, "Botao", 2048, NULL, 2, NULL, 1);
xTaskCreatePinnedToCore(taskLed, "Led", 2048, NULL, 1, NULL, 1);
```

   Como ambas as tasks usam `vTaskDelay` regularmente, não houve diferença
   perceptível no comportamento, o botão continuou sendo lido a cada 40 ms e
   o LED piscando conforme o estado de `habilitaPisca`, sem disputa visível
   pelo processador.

3. **Lógica do botão: borda para ativo em nível**: a `taskBotao` foi alterada
   para que `habilitaPisca` seja `true` apenas enquanto o botão está
   pressionado, em vez de alternar com cada toque:

```cpp
void taskBotao(void *pv) {
  pinMode(18, INPUT_PULLUP);
  while (true) {
    habilitaPisca = (digitalRead(18) == LOW);
    vTaskDelay(pdMS_TO_TICKS(40));
  }
}
```

   O LED passou a piscar apenas enquanto o botão estava sendo mantido
   pressionado, parando de piscar e ficando apagado imediatamente ao soltar o
   botão, diferente do comportamento original, que mantinha o piscar ativo
   após soltar até um novo toque.

## Conclusão

O período de cada task pode ser ajustado de forma independente, alterando
diretamente a velocidade do piscar do LED sem afetar a leitura do botão. A
diferença de prioridade entre tasks que usam `vTaskDelay` regularmente não se
mostrou determinante. A escolha entre lógica de borda e nível na `taskBotao`
define se o controle sobre a `taskLed` é persistente (toggle) ou imediato
(enquanto pressionado), mudando completamente a experiência de uso do
sistema.

## Questionamentos

**1. Como o sistema reagiu ao pressionamento rápido ou repetido do botão?**

Na lógica de borda original, cada toque alternou `habilitaPisca` de forma
confiável a cada 40 ms de leitura. Pressionamentos muito rápidos e repetidos,
sem debounce, puderam gerar múltiplas alternâncias seguidas. Na lógica ativa
em nível, pressionamentos repetidos fizeram o LED ligar e parar de piscar
rapidamente, acompanhando diretamente o estado do botão a cada ciclo de
leitura.

**2. Qual alteração teve maior impacto: prioridade, sincronização, fila ou tempo de espera?**

O ajuste de tempo de espera (período da `taskLed`) teve o maior impacto
perceptível, alterando diretamente a velocidade do piscar do LED. A
alteração de prioridade não teve impacto perceptível neste caso, já que ambas
as tasks cedem CPU regularmente com `vTaskDelay`.

**3. Que cuidado de projeto você adotaria se esta prática fosse levada para um sistema real?**

Adicionar debounce na leitura do botão para evitar múltiplas alternâncias
indesejadas em um único toque, e documentar claramente qual task é
responsável por escrever na variável compartilhada (`habilitaPisca`) e qual
apenas lê, evitando que múltiplas tasks escrevam no mesmo dado sem controle.
