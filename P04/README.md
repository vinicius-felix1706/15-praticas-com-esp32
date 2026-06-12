# P04: Botão alterna estado do LED

## Objetivo

Explorar o uso de um botão que alterna o estado de um LED para consolidar
conceitos de GPIO e leitura/escrita digital no ESP32, observando como
mudanças no código ou no circuito afetam o comportamento físico do sistema.

## Código-base (original)

```cpp
#include <Arduino.h>

const int LED_PIN = 2;
const int BUTTON_PIN = 18;
bool estadoLed = false;
bool ultimoLeitura = HIGH;

void setup() {
  pinMode(LED_PIN, OUTPUT);
  pinMode(BUTTON_PIN, INPUT_PULLUP);
  Serial.begin(115200);
}

void loop() {
  bool leitura = digitalRead(BUTTON_PIN);
  if (ultimoLeitura == HIGH && leitura == LOW) {
    estadoLed = !estadoLed;
    digitalWrite(LED_PIN, estadoLed);
    Serial.println(estadoLed ? "LED ON" : "LED OFF");
    delay(180); // debounce simples
  }
  ultimoLeitura = leitura;
}
```

## Configuração inicial

- **LED:** GPIO 2
- **Botão:** GPIO 18, configurado como `INPUT_PULLUP` (pressionado = LOW)
- **Debounce:** delay de 180 ms após detectar a borda de descida
- **Componentes:** LED vermelho + resistor de 220–330 Ω, botão tátil ligado
  entre GPIO 18 e GND

## Modificações realizadas

### 1. Troca de GPIO (LED 2 → 21, botão 18 → 19)

```cpp
const int LED_PIN = 21;
const int BUTTON_PIN = 19;
```

**Efeito observado:** o circuito funcionou de forma idêntica ao original — o
LED alternou entre ligado/desligado a cada toque do botão, confirmando que o
comportamento é definido pela lógica do código e não pelos pinos específicos
utilizados.

### 2. Alteração do tempo de debounce (180 ms → 30 ms)

```cpp
delay(30);
```

**Efeito observado:** com o debounce reduzido, o botão se tornou mais
sensível a pressionamentos rápidos, porém em alguns toques o LED alternou de
forma inesperada (mais de uma vez) devido ao "bouncing" mecânico do botão não
ser completamente filtrado, evidenciando que o tempo de debounce original
(180 ms) era mais adequado para evitar leituras espúrias.

### 3. Alteração da lógica do botão: borda → ativo em nível

A lógica foi alterada de "alternância por borda" (toggle) para "ativo em
nível" (LED acende apenas enquanto o botão está pressionado):

```cpp
const int LED_PIN = 21;
const int BUTTON_PIN = 19;

void setup() {
  pinMode(LED_PIN, OUTPUT);
  pinMode(BUTTON_PIN, INPUT_PULLUP);
  Serial.begin(115200);
}

void loop() {
  bool pressionado = digitalRead(BUTTON_PIN) == LOW;
  digitalWrite(LED_PIN, pressionado ? HIGH : LOW);
  Serial.println(pressionado ? "LED ON" : "LED OFF");
  delay(30);
}
```

**Efeito observado:** o LED deixou de manter o estado após soltar o botão. Em
vez de funcionar como um "interruptor" (estado persistente), passou a
funcionar como um acionamento "momentâneo" — aceso apenas durante a pressão,
exigindo contato contínuo para permanecer ligado.


## Questionamentos

**1. Como o sistema reagiu ao pressionamento rápido ou repetido do botão?**

Com o debounce de 180 ms, o sistema respondeu de forma confiável, alternando
o LED uma vez por toque mesmo em pressionamentos rápidos. Com o debounce
reduzido para 30 ms, pressionamentos rápidos ou repetidos por vezes causaram
mais de uma alternância por toque, devido ao bouncing do botão não ser
filtrado adequadamente. Já na versão com lógica ativa em nível,
pressionamentos rápidos e repetidos resultaram em piscadas curtas do LED,
acompanhando diretamente o estado do botão.

**2. Como o ESP32 interpretou o sinal usado nesta prática: digital, analógico ou PWM?**

O sinal é **digital**. A leitura do botão (`digitalRead`) retorna apenas
`HIGH` ou `LOW`, e a escrita do LED (`digitalWrite`) define apenas dois
estados, sem variação contínua de tensão nem modulação por largura de pulso
(PWM).

**3. Se você tivesse de tornar o experimento mais estável ou mais sensível, o que mudaria?**

Para mais estabilidade, seria recomendável manter ou até aumentar levemente o
tempo de debounce (ou implementar debounce por hardware com capacitor),
evitando alternâncias espúrias causadas pelo bouncing do botão. Para mais
sensibilidade na detecção de toques rápidos sem perder estabilidade, o
`delay()` poderia ser substituído por uma lógica baseada em `millis()`,
permitindo verificar o botão com mais frequência enquanto ainda se aplica um
tempo mínimo entre alternâncias válidas.
