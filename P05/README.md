# P05: Semáforo de Três LEDs (ESP32)

## Objetivo

Explorar um semáforo de três LEDs para consolidar conceitos de GPIO e
escrita digital no ESP32, observando como mudanças no código ou no circuito
afetam o comportamento físico do sistema.

## Código-base (original)

```cpp
#include <Arduino.h>

const int LED_R = 2;
const int LED_Y = 4;
const int LED_G = 5;

void setup() {
  pinMode(LED_R, OUTPUT);
  pinMode(LED_Y, OUTPUT);
  pinMode(LED_G, OUTPUT);
}

void loop() {
  digitalWrite(LED_G, HIGH);
  delay(2000);
  digitalWrite(LED_G, LOW);

  digitalWrite(LED_Y, HIGH);
  delay(700);
  digitalWrite(LED_Y, LOW);

  digitalWrite(LED_R, HIGH);
  delay(2000);
  digitalWrite(LED_R, LOW);
}
```

## Configuração inicial

- **LED Verde:** GPIO 5: 2000 ms aceso
- **LED Amarelo:** GPIO 4: 700 ms aceso
- **LED Vermelho:** GPIO 2: 2000 ms aceso
- **Ciclo total:** 4700 ms (verde → amarelo → vermelho → repete)
- **Componentes:** 3 LEDs (vermelho, amarelo, verde) + resistores de
  220–330 Ω, cada um ligado ao seu GPIO e ao GND

## Modificações realizadas

### 1. Troca de GPIO (LED Verde 5 → 21)

```cpp
const int LED_R = 2;
const int LED_Y = 4;
const int LED_G = 21;
```

**Efeito observado:** o ciclo do semáforo permaneceu idêntico, verde por
2 s, amarelo por 0,7 s e vermelho por 2 s, na mesma sequência. Isso confirma
que a lógica de temporização depende apenas do código, não do pino físico
utilizado, desde que seja um GPIO digital válido.

### 2. Alteração dos tempos (amarelo 700 ms → 300 ms, vermelho 2000 ms → 3000 ms)

```cpp
digitalWrite(LED_Y, HIGH);
delay(300);
digitalWrite(LED_Y, LOW);

digitalWrite(LED_R, HIGH);
delay(3000);
digitalWrite(LED_R, LOW);
```

**Efeito observado:** o amarelo passou a ser quase imperceptível (um "flash"
muito rápido entre o verde e o vermelho), enquanto o vermelho ficou
visivelmente mais longo, alterando bastante o ritmo geral do semáforo,
muito mais tempo "parado" (vermelho) e quase nenhuma transição perceptível
(amarelo).

### 3. Acréscimo de um segundo evento sincronizado (buzzer no amarelo)

Foi adicionado um buzzer no GPIO 19, que soa durante o tempo em que o LED
amarelo está aceso, sinalizando a transição de forma sonora:

```cpp
const int LED_R = 2;
const int LED_Y = 4;
const int LED_G = 21;
const int BUZZER_PIN = 19;

void setup() {
  pinMode(LED_R, OUTPUT);
  pinMode(LED_Y, OUTPUT);
  pinMode(LED_G, OUTPUT);
  pinMode(BUZZER_PIN, OUTPUT);
}

void loop() {
  digitalWrite(LED_G, HIGH);
  delay(2000);
  digitalWrite(LED_G, LOW);

  digitalWrite(LED_Y, HIGH);
  digitalWrite(BUZZER_PIN, HIGH);
  delay(300);
  digitalWrite(LED_Y, LOW);
  digitalWrite(BUZZER_PIN, LOW);

  digitalWrite(LED_R, HIGH);
  delay(3000);
  digitalWrite(LED_R, LOW);
}
```

**Efeito observado:** o buzzer soou exatamente durante o intervalo do LED
amarelo, criando um alerta sonoro sincronizado com a transição visual. Isso
tornou a fase de transição muito mais perceptível, mesmo com o tempo do
amarelo reduzido para apenas 300 ms, o som compensou a curta duração visual.

## Conclusão

O comportamento do semáforo é definido principalmente pela sequência e pelos
tempos programados no código, sendo o GPIO apenas uma interface configurável
que não altera a lógica de funcionamento. Alterações nos tempos de cada fase
mudam diretamente a percepção do ritmo do semáforo, podendo tornar fases
praticamente imperceptíveis (como o amarelo em 300 ms). A adição de um evento
sincronizado (buzzer) demonstrou que é possível reforçar uma transição rápida
com um sinal complementar em outro canal (sonoro), mantendo a sincronização
com o estado visual.

## Questionamentos

**1. Que mudança no circuito ou no código gerou o efeito mais perceptível e por quê?**

A alteração dos tempos (amarelo de 700 ms para 300 ms e vermelho de 2000 ms
para 3000 ms) foi a mudança mais perceptível, pois alterou diretamente a
proporção entre as fases do semáforo, o amarelo praticamente desapareceu
visualmente, enquanto o vermelho dominou boa parte do ciclo, mudando a
sensação geral de ritmo do sistema.

**2. Como o ESP32 interpretou o sinal usado nesta prática: digital, analógico ou PWM?**

O sinal é **digital**. Tanto os LEDs quanto o buzzer adicionado são
controlados por `digitalWrite()`, que define apenas os estados `HIGH` ou
`LOW`, sem variação contínua de tensão nem modulação por largura de pulso
(PWM).

**3. Como você explicaria a lógica de estados deste experimento para outra equipe?**

O sistema funciona como uma máquina de estados sequencial e cíclica, com três
estados principais: **Verde** (sinal liberado, LED verde aceso por um tempo
fixo), **Amarelo** (atenção/transição, LED amarelo aceso por um tempo menor,
acompanhado de um alerta sonoro no buzzer) e **Vermelho** (sinal de parada,
LED vermelho aceso pelo maior tempo do ciclo). Ao final do estado Vermelho, o
sistema retorna automaticamente ao estado Verde, repetindo o ciclo
indefinidamente. Cada transição de estado é marcada por `delay()`, que define
quanto tempo o sistema permanece em cada estado antes de avançar para o
próximo.
