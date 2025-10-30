# Ponderada Semáforo Inteligente com Interrupção por Pedestre

## 1. Introdução e Fundamentação Teórica

O projeto visa simular o funcionamento de um semáforo de trânsito em uma via de mão única, com a adição de um botão de solicitação de travessia para pedestres. A principal inovação técnica deste projeto, que o eleva acima de uma simples implementação sequencial, é a utilização do conceito de Máquina de Estados e da função millis() do Arduino.

### 1.1. Programação Não Bloqueante (millis())

Em projetos de microcontroladores, a função delay() é frequentemente utilizada para controlar o tempo. No entanto, ela bloqueia a execução do programa, impedindo que o Arduino realize qualquer outra tarefa (como ler o estado de um botão) durante o tempo de espera.

Para criar um sistema responsivo e interativo, utilizamos a função millis(), que retorna o número de milissegundos desde que a placa Arduino começou a executar o programa atual. Ao invés de parar o código, o programa verifica a diferença entre o tempo atual e o tempo da última mudança de estado.


"A abordagem não bloqueante é crucial para sistemas que exigem a monitorização contínua de entradas (como sensores ou botões) enquanto executam tarefas baseadas em tempo."

1.2. Máquina de Estados Finitos (MEF)

O semáforo é implementado como uma Máquina de Estados Finitos (MEF). O sistema transita entre três estados principais, definidos por variáveis numéricas:

Estado
Descrição
Duração Padrão
0 (Verde)
Sinal aberto para veículos.
4 segundos
1 (Amarelo)
Sinal de transição, alerta para veículos.
2 segundos
2 (Vermelho)
Sinal fechado para veículos, tempo de travessia para pedestres.
6 segundos


A transição entre os estados é controlada por duas condições: o tempo e a interrupção do botão.




2. Hardware: Componentes e Montagem

O projeto requer o Arduino UNO e componentes básicos de sinalização e entrada.

2.1. Lista de Componentes

Componente
Quantidade
Função
Arduino UNO R3
1
Microcontrolador central.
LED Vermelho
1
Sinalização de parada (Veículos).
LED Amarelo
1
Sinalização de atenção (Veículos).
LED Verde
1
Sinalização de passagem (Veículos).
Resistor 220Ω
3
Limitar a corrente para os LEDs (proteção).
Push Button (Botão)
1
Entrada para solicitação de pedestre.
Resistor 10kΩ
1
Resistor pull-down para o botão.
Protoboard
1
Placa de ensaio para montagem.
Jumpers (Fios)
Vários
Conexões elétricas.


2.2. Diagrama de Circuito e Conexões

O diagrama de circuito (baseado na imagem fornecida) detalha as conexões:

Componente
Conexão no Arduino
Tipo de Conexão
LED Vermelho
Pino Digital 10
Saída (OUTPUT)
LED Amarelo
Pino Digital 9
Saída (OUTPUT)
LED Verde
Pino Digital 8
Saída (OUTPUT)
Cátodos dos LEDs
GND (via Resistor 220Ω)
Terra
Botão de Pedestre
Pino Digital 2
Entrada (INPUT)
Botão de Pedestre
5V
Alimentação
Resistor 10kΩ
Pino Digital 2 para GND
Pull-down


Lógica do Botão (Pull-down): O resistor de 10kΩ conectado entre o Pino 2 e o GND garante que a leitura do pino seja LOW (0V) quando o botão não está pressionado. Ao pressionar, o pino é conectado diretamente ao 5V, resultando em uma leitura HIGH (5V).




3. Software: Código Fonte Comentado

O código abaixo implementa a Máquina de Estados e a lógica de interrupção não bloqueante.

C++


// --- Portas dos Componentes ---
const int ledVermelho = 10;
const int ledAmarelo = 9;
const int ledVerde = 8;
const int pinoBotao = 2; // Porta para o botão de pedestre

// --- Variáveis de Controle de Tempo (sem delay) ---
// Define o tempo que cada estado deve durar em milissegundos
unsigned long tempoAnterior = 0;
const long tempoVerde = 4000;   // 4 segundos
const long tempoAmarelo = 2000; // 2 segundos
const long tempoVermelho = 6000; // 6 segundos

// --- Variáveis de Estado ---
int estadoSemaforo = 0; // 0: Verde, 1: Amarelo, 2: Vermelho
bool botaoPressionado = false; // "Memória" para saber se o pedestre pediu para atravessar

void setup() {
  // Define os pinos dos LEDs como saída
  pinMode(ledVermelho, OUTPUT);
  pinMode(ledAmarelo, OUTPUT);
  pinMode(ledVerde, OUTPUT);
  
  // Define o pino do botão como entrada
  // O resistor pull-down é implementado no circuito físico
  pinMode(pinoBotao, INPUT);

  // Inicia o semáforo no estado verde
  digitalWrite(ledVerde, HIGH);
  tempoAnterior = millis(); // Inicializa o cronômetro
}

void loop() {
  // 1. LEITURA CONTÍNUA DO BOTÃO
  // Esta leitura não bloqueia o programa e é executada a cada ciclo do loop
  if (digitalRead(pinoBotao) == HIGH) {
    // Se o botão for pressionado, a flag de solicitação é ativada
    botaoPressionado = true;
  }

  // Pega o tempo atual para fazer as comparações
  unsigned long tempoAtual = millis();

  // 2. MÁQUINA DE ESTADOS DO SEMÁFORO (LÓGICA PRINCIPAL)
  switch (estadoSemaforo) {
    case 0: // Estado 0: SINAL VERDE (Passagem para veículos)
      digitalWrite(ledVerde, HIGH);
      digitalWrite(ledAmarelo, LOW);
      digitalWrite(ledVermelho, LOW);

      // CONDIÇÃO DE TRANSIÇÃO:
      // Passa para o amarelo se:
      // a) O tempo de 4 segundos acabou (ciclo automático)
      // OU
      // b) O botão foi pressionado (interrupção do pedestre)
      if (tempoAtual - tempoAnterior >= tempoVerde || botaoPressionado) {
        estadoSemaforo = 1; // Vai para o amarelo
        tempoAnterior = tempoAtual; // Reinicia o cronômetro para o próximo estado
        botaoPressionado = false; // Reseta a flag de solicitação do pedestre
      }
      break;

    case 1: // Estado 1: SINAL AMARELO (Alerta de transição)
      digitalWrite(ledVerde, LOW);
      digitalWrite(ledAmarelo, HIGH);
      digitalWrite(ledVermelho, LOW);
      
      // CONDIÇÃO DE TRANSIÇÃO:
      // Se passaram 2 segundos no amarelo, vai para o vermelho
      if (tempoAtual - tempoAnterior >= tempoAmarelo) {
        estadoSemaforo = 2; // Vai para o vermelho
        tempoAnterior = tempoAtual; // Reinicia o cronômetro
      }
      break;

    case 2: // Estado 2: SINAL VERMELHO (Parada de veículos / Travessia de pedestres)
      digitalWrite(ledVerde, LOW);
      digitalWrite(ledAmarelo, LOW);
      digitalWrite(ledVermelho, HIGH);

      // CONDIÇÃO DE TRANSIÇÃO:
      // Se passaram 6 segundos no vermelho, volta para o verde e reinicia o ciclo
      if (tempoAtual - tempoAnterior >= tempoVermelho) {
        estadoSemaforo = 0; // Volta para o verde
        tempoAnterior = tempoAtual; // Reinicia o cronômetro
      }
      break;
  }
}





4. Lógica de Funcionamento e Interrupção

O coração do projeto reside na lógica de transição do estado Verde (0).

1.
Ciclo Automático: O semáforo permanece no estado Verde por 4 segundos. Após esse período, a condição tempoAtual - tempoAnterior >= tempoVerde se torna verdadeira, e a transição para o estado Amarelo (1) é iniciada.

2.
Interrupção por Pedestre: Se o botão for pressionado, a variável botaoPressionado é definida como true. A condição de transição no estado Verde se torna verdadeira (... || botaoPressionado), forçando a mudança imediata para o estado Amarelo, independentemente do tempo restante do ciclo Verde.

3.
Segurança e Fluidez:

•
Se o botão for pressionado durante o Amarelo ou Vermelho, a solicitação é registrada (botaoPressionado = true).

•
Quando o ciclo retorna ao Verde, a solicitação registrada força a transição imediata para o Amarelo, garantindo que o pedestre seja atendido o mais rápido possível no próximo ciclo.

•
A flag botaoPressionado é resetada (false) logo após ser utilizada para iniciar a transição, preparando o sistema para a próxima solicitação.



Esta implementação demonstra um entendimento avançado de programação de microcontroladores, combinando o controle de tempo preciso com a capacidade de resposta a eventos externos.

