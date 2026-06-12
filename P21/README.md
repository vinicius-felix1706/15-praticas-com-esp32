# P21: Pisca-pisca sem delay com millis



## Objetivo

Explorar o pisca-pisca sem `delay()` usando `millis()` para entender
temporização não bloqueante e o efeito de ajustes de período sobre a
previsibilidade do sistema.

## Código-base (original)

```cpp
#include <Arduino.h>

const int LED_PIN = 2;
unsigned long anterior = 0;
unsigned long periodo = 500;
bool estado = false;

void setup() {
  pinMode(LED_PIN, OUTPUT);
}

void loop() {
  unsigned long agora = millis();
  if (agora - anterior >= periodo) {
    anterior = agora;
    estado = !estado;
    digitalWrite(LED_PIN, estado);
  }
}
```

## Configuração inicial

LED vermelho no GPIO 2, período de 500 ms (piscar simétrico 500 ms
ligado/desligado).

## Modificações realizadas

1. **Troca de GPIO + segundo LED** (vermelho → GPIO 21, verde adicional no
   GPIO 22 com período de 300 ms): os dois LEDs piscaram de forma
   independente e simultânea, sem interferência entre si.
2. **Redução do período** (500 ms → 100 ms): o LED vermelho passou a piscar
   muito mais rápido, quase em tremulação contínua, sem afetar o LED verde.
3. **Bloqueio proposital com `delay(1000)`**: mesmo com a lógica de
   `millis()` correta, ambos os LEDs ficaram travados e irregulares,
   atualizando no máximo uma vez por segundo.

## Conclusão

`millis()` permite gerenciar múltiplos eventos temporizados de forma
independente, desde que o `loop()` execute com frequência suficiente.
Qualquer `delay()` no código compromete a previsibilidade de todo o sistema,
mesmo que a lógica temporal esteja correta.

## Questionamentos

**1. Ajuste que mais alterou a previsibilidade?**
A inserção do `delay(1000)`: os LEDs passaram de um piscar fluido e
independente para um piscar "engasgado" e sincronizado, limitado pelo
bloqueio.

**2. Quando `delay()` pioraria a prática?**
Em qualquer cenário com múltiplos eventos independentes (os dois LEDs com
períodos diferentes), pois `delay()` impediria que ambos piscassem
simultaneamente.

**3. Evidência registrada?**
Comparação direta: sem `delay()`, os LEDs piscavam em 100 ms e 300 ms de
forma fluida; com `delay(1000)`, ambos passaram a atualizar de forma lenta e
sincronizada, mesmo mantendo os mesmos valores de período no código.
