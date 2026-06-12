# Prática: LED com Tempos Assimétricos (ESP32)

## Objetivo

Explorar o controle de um LED com tempos assimétricos de ligado/desligado para
consolidar conceitos de GPIO e escrita digital no ESP32, observando como
mudanças no código ou no circuito afetam o comportamento do LED.

## Código-base

```cpp
#include <Arduino.h>

const int LED_PIN = 4;
int tempoLigado = 200;
int tempoDesligado = 800;

void setup() {
  pinMode(LED_PIN, OUTPUT);
  Serial.begin(115200);
}

void loop() {
  digitalWrite(LED_PIN, HIGH);
  delay(tempoLigado);
  digitalWrite(LED_PIN, LOW);
  delay(tempoDesligado);
}
```

## Configuração inicial

- **Pino:** GPIO 4
- **Tempos:** 200 ms ligado / 800 ms desligado (ciclo de 1 s)
- **Componentes:** LED vermelho + resistor de 220–330 Ω

## Modificações realizadas e efeitos

1. **Troca de GPIO** (4 → outro pino válido): comportamento do LED
   mantido — o tempo é definido pelo código, não pelo pino.
2. **Inversão dos tempos** (800 ms ligado / 200 ms desligado): LED fica mais
   tempo aceso, dando sensação de luz quase constante.
3. **Acréscimo/troca de LED**: permite comparar cores diferentes; o tempo
   continua igual, mas a percepção de brilho muda.

## Conclusão

O comportamento do LED depende principalmente das constantes de tempo no
código. O GPIO é apenas uma interface configurável, e mudanças no componente
afetam mais a percepção visual do que a lógica temporal.

## Questionamentos

**1. Mudança mais perceptível?**
A inversão dos tempos, pois alterou diretamente a proporção
aceso/apagado, mudando o ritmo do pisca-pisca.

**2. Tipo de sinal usado?**
Digital — `digitalWrite()` define apenas HIGH ou LOW, sem variação contínua
ou PWM.

**3. Como tornar mais estável/sensível?**
Usar `millis()` no lugar de `delay()` para não bloquear o `loop()`
(estabilidade) e `analogWrite()`/PWM para controlar o brilho gradualmente
(sensibilidade).
