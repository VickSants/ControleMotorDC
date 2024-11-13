
# Controle de Motor DC com Arduino e TB6612

Este projeto implementa o controle de velocidade de um motor DC utilizando Arduino e o driver de motor TB6612. Abaixo estão os detalhes do código e a descrição de cada função e configuração.

## Visão Geral

O objetivo do projeto é controlar a velocidade de um motor DC através de um sinal PWM, com a capacidade de definir a direção de rotação. Um botão de controle liga o sistema, e a velocidade do motor é monitorada através de um sensor analógico.

### Principais Componentes e Bibliotecas

- **Arduino Uno**: Controlador principal.
- **TB6612**: Driver de motor para amplificação do sinal de controle PWM.
- **TimerOne**: Biblioteca de temporizador para amostragem periódica do sinal do motor.

## Código

### Variáveis Globais e Constantes

```cpp
const long nLeituras = 300; // Número de leituras para a coleta de dados
const long periodoAmostragemUs = 10000; // Período de amostragem em microssegundos
bool estadoBotao = false; // Estado do botão de controle do motor
```

### Funções

#### `definirDirecao(bool horario)`

Define a direção de rotação do motor.

- **Parâmetro**: `horario` - `true` define o motor para sentido horário; `false` para anti-horário.
- **Uso**:
  ```cpp
  definirDirecao(true); // Define o sentido horário
  ```

#### `aplicarSinalReferencia(float r)`

Aplica um sinal PWM ao motor.

- **Parâmetro**: `r` - Valor de referência entre `0` e `1`, onde `1` aplica a máxima potência e `0` desliga o motor.
- **Uso**:
  ```cpp
  aplicarSinalReferencia(1.0f); // Motor a 100% de potência
  ```

#### `atualizarEstadoBotao()`

Atualiza o estado do botão de controle do sistema, verificando se ele está pressionado.

- **Uso**:
  ```cpp
  atualizarEstadoBotao();
  ```

#### `lerSaidaSistema()`

Lê o valor do sinal de feedback do motor, representando a velocidade do sistema.

- **Retorna**: Valor analógico entre `0` e `1023`.
- **Uso**:
  ```cpp
  int velocidade = lerSaidaSistema();
  ```

#### `imprimirSaida()`

Imprime no monitor serial o valor lido do sensor de velocidade. Função acionada pelo temporizador configurado.

- **Uso**: Não necessita de chamada direta, pois é acionada pelo `Timer1`.

### `setup()`

Configura os pinos de entrada e saída, ativa o driver TB6612, define a direção inicial do motor e configura o temporizador para o intervalo de amostragem.

```cpp
void setup() {
  pinMode(2, OUTPUT);           // Ativa o pino STBY do TB6612
  pinMode(3, OUTPUT);           // AIN1 TB6612 (direção)
  pinMode(4, OUTPUT);           // AIN2 TB6612 (direção)
  pinMode(5, OUTPUT);           // PWM
  pinMode(6, INPUT_PULLUP);     // Botão de controle
  digitalWrite(2, HIGH);        // Ativa o driver TB6612
  definirDirecao(true);         // Define direção inicial
  Serial.begin(115200);         // Inicializa a comunicação serial
  Timer1.initialize(periodoAmostragemUs); // Configura Timer1
  Timer1.attachInterrupt(imprimirSaida);  // Define a função de interrupção do timer
  Timer1.stop();                // Para o timer inicialmente
}
```

### `loop()`

- Verifica o estado do botão para ativar o motor.
- Aplica sinal de referência máximo ao motor (`100%`).
- Inicia a coleta de dados e o temporizador.
- Após coletar os dados, para o motor.

```cpp
void loop() {
  do {
    atualizarEstadoBotao();
    delay(100); // Aguarda 100 ms para verificar o botão novamente
  } while (!estadoBotao);

  aplicarSinalReferencia(1.0f);  // Inicia motor a 100%
  Timer1.start();                // Inicia coleta de dados
  delay(periodoAmostragemUs * nLeituras / 1000); // Aguarda término das leituras
  Timer1.stop();                 // Para a coleta de dados
  aplicarSinalReferencia(0.0f);  // Desliga o motor
}
```

## Observações

- **Botão de Controle**: O botão ligado ao pino 6 ativa e desativa o motor.
- **PWM**: A intensidade do PWM aplicado ao motor controla sua velocidade.
- **Temporizador**: O Timer1 é utilizado para gerar interrupções periódicas para amostragem de velocidade.

---
