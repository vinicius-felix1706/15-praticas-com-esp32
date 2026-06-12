# P22: Dois LEDs independentes com millis


## Objetivo

Explorar dois LEDs independentes com `millis()` para entender temporização
sem bloqueio e o efeito de ajustes de período sobre a previsibilidade do
sistema.

## Código-base (original)

```cpp
#include <Arduino.h>

const int LED1 = 2, LED2 = 4;
unsigned long t1 = 0, t2 = 0;
bool e1 = false, e2 = false;

void setup() {
  pinMode(LED1, OUTPUT);
  pinMode(LED2, OUTPUT);
}

void loop() {
  unsigned long agora = millis();

  if (agora - t1 >= 300) { t1 = agora; e1 = !e1; digitalWrite(LED1, e1); }
  if (agora - t2 >= 800) { t2 = agora; e2 = !e2; digitalWrite(LED2, e2); }
}
```

## Configuração inicial

LED1 no GPIO 2 (300 ms), LED2 no GPIO 4 (800 ms), ambos com resistores de
220–330 Ω.

## Modificações realizadas

1. **Troca de GPIO** (LED1 → 21, LED2 → 22): comportamento idêntico, cada
   LED manteve seu ritmo, confirmando que o pino não afeta a lógica.
2. **Redução do período do LED1** (300 ms → 100 ms): LED1 piscou muito mais
   rápido, quase em tremulação, enquanto LED2 (800 ms) seguiu inalterado.
3. **Bloqueio proposital com `delay(1000)`**: ambos os LEDs passaram a
   alternar de forma travada e sincronizada (~1x por segundo),
   independentemente dos períodos configurados.

## Conclusão

`millis()` permite eventos independentes e simultâneos, e alterar o período
de um não afeta o outro. Qualquer `delay()` no `loop()` compromete a
previsibilidade de todos os eventos ao mesmo tempo.

## Questionamentos

**1. Ajuste que mais alterou a previsibilidade?**
A inserção do `delay(1000)`: LED1 (100 ms) e LED2 (800 ms) passaram a
alternar de forma lenta e sincronizada, perdendo a diferença de ritmo entre
eles.

**2. Quando `delay()` pioraria a prática?**
Justamente no objetivo central, manter dois LEDs em ritmos diferentes e
independentes. Com `delay()`, os dois ficariam subordinados ao mesmo
intervalo de bloqueio.

**3. Evidência registrada?**
Sem `delay()`, LED1 e LED2 piscaram em ritmos distintos (100 ms e 800 ms);
com `delay(1000)`, ambos passaram a alternar de forma sincronizada e lenta,
mesmo com os mesmos valores de período no código.
