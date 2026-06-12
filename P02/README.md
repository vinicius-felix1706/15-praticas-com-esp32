# Prática: LED com Tempos Assimétricos (ESP32)

## Objetivo

Explorar o controle de um LED com tempos assimétricos de ligado/desligado para
consolidar conceitos de GPIO e escrita digital no ESP32, observando como
mudanças no código ou no circuito afetam o comportamento do LED.

## Código-base (original)

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

## Modificações realizadas

### 1. Troca de GPIO (4 → 18)

O LED vermelho foi remontado no GPIO 18, e a constante `LED_PIN` foi
atualizada de 4 para 18:

```cpp
const int LED_PIN = 18;
```

**Efeito observado:** o LED continuou piscando exatamente com o mesmo padrão
(200 ms aceso / 800 ms apagado), confirmando que o comportamento é definido
pela lógica do código e não pelo pino físico utilizado.

### 2. Inversão dos tempos

Os valores de `tempoLigado` e `tempoDesligado` foram invertidos:

```cpp
int tempoLigado = 800;
int tempoDesligado = 200;
```

**Efeito observado:** o LED passou a ficar a maior parte do ciclo aceso, com
apagões breves e rápidos, dando a impressão de uma luz quase constante em vez
do "flash" rápido original.

### 3. Acréscimo de um segundo LED

Foi adicionado um segundo LED (cor amarela) no GPIO 19, ligado em paralelo ao
primeiro com o mesmo padrão de tempos:

```cpp
const int LED_PIN = 18;
const int LED_PIN2 = 19;
int tempoLigado = 800;
int tempoDesligado = 200;

void setup() {
  pinMode(LED_PIN, OUTPUT);
  pinMode(LED_PIN2, OUTPUT);
  Serial.begin(115200);
}

void loop() {
  digitalWrite(LED_PIN, HIGH);
  digitalWrite(LED_PIN2, HIGH);
  delay(tempoLigado);
  digitalWrite(LED_PIN, LOW);
  digitalWrite(LED_PIN2, LOW);
  delay(tempoDesligado);
}
```

**Efeito observado:** os dois LEDs piscaram de forma sincronizada, mas com
brilhos visivelmente diferentes — o LED amarelo apresentou maior luminosidade
percebida do que o vermelho, mesmo recebendo o mesmo sinal e a mesma corrente
limitada pelo resistor, evidenciando a diferença de tensão direta e eficiência
luminosa entre cores de LED.

## Conclusão do grupo

O comportamento temporal do LED é determinado pelas constantes de tempo no
código, e o pino GPIO funciona apenas como uma interface configurável — sua
troca não altera a lógica do programa. Já a escolha do componente físico (cor
do LED) não afeta o tempo, mas influencia diretamente a percepção visual de
brilho e intensidade.

## Questionamentos

**1. Que mudança gerou o efeito mais perceptível, e por quê?**

A inversão dos tempos (`tempoLigado`/`tempoDesligado`) foi a mudança mais
perceptível, pois alterou diretamente a proporção entre o tempo aceso e
apagado dentro do ciclo de 1 segundo, transformando o pisca-pisca rápido em
uma luz quase contínua com apagões breves.

**2. Como o ESP32 interpretou o sinal usado nesta prática?**

O sinal utilizado é **digital**. O comando `digitalWrite()` define o pino em
apenas dois estados, `HIGH` ou `LOW`, sem variação contínua de tensão e sem
modulação por largura de pulso (PWM).

**3. Como tornar o experimento mais estável ou mais sensível?**

Para mais estabilidade, os `delay()` poderiam ser substituídos por uma lógica
baseada em `millis()`, evitando bloquear o `loop()` e permitindo lidar com
outras entradas (botões, sensores) simultaneamente. Para mais sensibilidade,
`digitalWrite()` poderia ser trocado por `analogWrite()` (PWM), permitindo
controlar gradualmente o brilho dos LEDs em vez de apenas ligá-los e
desligá-los.
