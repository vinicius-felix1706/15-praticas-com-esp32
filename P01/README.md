# Relatório de Prática — Pisca-LED com ESP32

## Configuração Inicial

| Parâmetro | Valor |
|------------|--------|
| Pino do LED | GPIO 2 |
| Tempo ligado | 500 ms |
| Tempo desligado | 500 ms |
| Resistor | 220 Ω |
| Baud Rate Serial | 115200 |

---

## Modificações Realizadas

### Modificação 1 — Alteração dos Tempos

#### Alteração no Código

```cpp
int tempoLigado = 200;     // era 500
int tempoDesligado = 1000; // era 500
```

#### Efeito Observado

O LED passou a permanecer ligado por menos tempo e apagado por mais tempo. O padrão de piscada tornou-se assimétrico e visualmente diferente da configuração original.

---

### Modificação 2 — Troca de Pino GPIO

#### Alteração no Código

```cpp
const int LED_PIN = 5; // era GPIO 2
```

#### Alteração no Circuito

O fio conectado ao ânodo do LED foi transferido do GPIO 2 para o GPIO 5.

#### Efeito Observado

O funcionamento permaneceu inalterado, demonstrando que o GPIO 5 também pode ser utilizado normalmente como saída digital para acionamento do LED.

---

### Modificação 3 — Adição de um Segundo LED

#### Alteração no Circuito

Foi adicionado um LED verde conectado ao GPIO 4 através de um resistor de 220 Ω.

#### Alteração no Código

```cpp
const int LED_PIN2 = 4;

void setup() {
  pinMode(LED_PIN, OUTPUT);
  pinMode(LED_PIN2, OUTPUT);
}

void loop() {
  digitalWrite(LED_PIN, HIGH);
  digitalWrite(LED_PIN2, LOW);
  delay(tempoLigado);

  digitalWrite(LED_PIN, LOW);
  digitalWrite(LED_PIN2, HIGH);
  delay(tempoDesligado);
}
```

#### Efeito Observado

Os LEDs passaram a funcionar de forma alternada. Enquanto um LED permanece ligado, o outro fica apagado, produzindo um efeito visual de revezamento.

---


# Questionamentos

## 1. Que mudança no circuito ou no código gerou o efeito mais perceptível e por quê?

A adição do segundo LED com funcionamento alternado foi a modificação mais perceptível visualmente. O comportamento deixou de ser apenas um ciclo simples de liga/desliga e passou a apresentar um padrão de revezamento entre dois LEDs.

A alteração dos tempos também produziu uma diferença significativa, pois evidenciou que os períodos ligado e desligado podem ser configurados de forma independente.

---

## 2. Como o ESP32 interpretou o sinal usado nesta prática: digital, analógico ou PWM?

O sinal utilizado foi **digital**.

As funções:

```cpp
digitalWrite(HIGH);
digitalWrite(LOW);
```

controlam apenas dois níveis de tensão:

- **HIGH (3,3 V)** → LED ligado;
- **LOW (0 V)** → LED desligado.

Não houve geração de sinais analógicos nem utilização de PWM (Pulse Width Modulation).

---

## 3. Se você tivesse de tornar o experimento mais estável ou mais sensível, o que mudaria?

### Para aumentar a estabilidade

Substituiria as funções `delay()` pelo uso de `millis()`, permitindo que o ESP32 execute outras tarefas simultaneamente sem bloquear o processador durante os períodos de espera.

### Para aumentar a sensibilidade visual

Reduziria os tempos de acionamento para valores inferiores a 100 ms. Dessa forma, as mudanças de estado ocorreriam mais rapidamente, tornando as diferenças entre configurações mais evidentes durante a observação do experimento.

---

## Considerações Finais

A prática permitiu compreender os conceitos básicos de:

- Saídas digitais no ESP32;
- Controle de LEDs através de GPIOs;
- Temporização utilizando `delay()`;
- Monitoramento por comunicação serial;
- Expansão de circuitos embarcados com múltiplos dispositivos.

Os resultados confirmaram o correto funcionamento do ESP32 no controle de dispositivos digitais simples e demonstraram como pequenas alterações no software podem modificar significativamente o comportamento do sistema.
