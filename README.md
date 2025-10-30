# Semáforo Offline

## 1. Objetivo

O objetivo desta atividade foi realizar a montagem física de um semáforo utilizando LEDs e resistores em uma protoboard. Os LEDs devem representar as cores vermelho, amarelo e verde, seguindo o esquema de um semáforo convencional. Além disso, como "ir além", foi feita a implementação de um botão que simula os botões de semáfaro de pedestre, acelerando a mudança de estado ao fechar o semafaro para a travessia de pedestres.

## 2. Metodologia

A atividade foi dividida em duas etapas principais: montagem do circuito e desenvolvimento da lógica de controle.

### 2.1. Circuito Físico

Foi utilizado um microcontrolador Arduino UNO para controlar três LEDs (vermelho, amarelo e verde) que simulam o semáforo. Um Push Button foi adicionado como dispositivo de entrada para permitir o avanço manual do estado.

- Link do vídeo demonstrativo do semáfaro: https://drive.google.com/drive/folders/1GaAipe5WMEKB1amaizIvkqVpxbuep39-?usp=sharing

Segue abaixo o circuito desenvolvido:

<div align="center">

<sub>Figura 1: Circuito do Semáforo com Botão de Controle (Online).</sub> 

<img src="circuito-online.png" style=width:100%>

</div>

<div align="center"> 

<sub>Figura 2: Circuito do Semáforo com Botão de Controle (Offline).</sub> 
  
<img src="circuito-offline.png" style=width:100%> 

</div>

Lista de Componentes e Conexões:

| Componente | Conexão no Arduino | Função |
| :--- | :--- | :--- |
| LED Vermelho | Pino Digital 10 | Sinalização de parada (Veículos). |
| LED Amarelo | Pino Digital 9 | Sinalização de atenção (Veículos). |
| LED Verde | Pino Digital 8 | Sinalização de passagem (Veículos). |
| Push Button | Pino Digital 12 | Entrada para avanço de estado. |
| Resistor 10kΩ | Pino Digital 2 para GND | Resistor pull-down para o botão. |
| Resistor 220Ω | Cátodos dos LEDs para GND | Limitar a corrente e proteger os LEDs. |

### 2.2. Lógica de Controle

O código do microcontrolador, apresentado abaixo, é responsável por gerenciar a transição entre os estados.

C++
```
// --- Portas dos Componentes ---
const int ledVermelho = 10;
const int ledAmarelo = 9;
const int ledVerde = 8;
const int pinoBotao = 12; // Porta para o botão de pedestre

// --- Variáveis de Controle de Tempo (sem delay) ---
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
  pinMode(pinoBotao, INPUT);

  // Inicia o semáforo no estado verde e liga o "cronômetro"
  digitalWrite(ledVerde, HIGH);
  tempoAnterior = millis();
}

void loop() {
  // 1. LER O BOTÃO A TODO MOMENTO
  // Se o botão for pressionado, ativamos nossa "memória"
  if (digitalRead(pinoBotao) == HIGH) {
    botaoPressionado = true;
  }

  // Pega o tempo atual para fazer as comparações
  unsigned long tempoAtual = millis();

  // 2. MÁQUINA DE ESTADOS DO SEMÁFORO (LÓGICA PRINCIPAL)
  switch (estadoSemaforo) {
    case 0: // Estado 0: SINAL VERDE
      digitalWrite(ledVerde, HIGH);
      digitalWrite(ledAmarelo, LOW);
      digitalWrite(ledVermelho, LOW);

      // CONDIÇÃO DE TRANSIÇÃO:
      // Passa para o amarelo se o tempo do verde acabou OU se o botão foi pressionado.
      if (tempoAtual - tempoAnterior >= tempoVerde || botaoPressionado) {
        estadoSemaforo = 1; // Vai para o amarelo
        tempoAnterior = tempoAtual; // Reinicia o cronômetro
        botaoPressionado = false; // "Esquece" o clique, pois já atendemos ao pedido
      }
      break;

    case 1: // Estado 1: SINAL AMARELO
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

    case 2: // Estado 2: SINAL VERMELHO
      digitalWrite(ledVerde, LOW);
      digitalWrite(ledAmarelo, LOW);
      digitalWrite(ledVermelho, HIGH);

      // CONDIÇÃO DE TRANSIÇÃO:
      // Se passaram 6 segundos no vermelho, volta para o verde
      if (tempoAtual - tempoAnterior >= tempoVermelho) {
        estadoSemaforo = 0; // Volta para o verde
        tempoAnterior = tempoAtual; // Reinicia o cronômetro
      }
      break;
  }
}
```

## 3. Estados do Sistema

O sistema opera em um ciclo contínuo de três estados. A tabela abaixo resume os estados e os eventos que causam a transição.

| Estado (estadoSemaforo) | LED Ligado | Duração Padrão | Condição de Transição |
| :--- | :--- | :--- | :--- |
| 0 (Verde) | LED Verde | 4000 ms | Tempo expirado OU Botão Pressionado |
| 1 (Amarelo) | LED Amarelo | 2000 ms | Tempo expirado OU Botão Pressionado |
| 2 (Vermelho) | LED Vermelho | 6000 ms | Tempo expirado OU Botão Pressionado |

## 4. Resultados

A implementação da lógica de avanço de estado resultou em um sistema de semáforo com dupla funcionalidade:

1. Ciclo Automático Robusto: O semáforo realiza o ciclo Verde (4s) -> Amarelo (2s) -> Vermelho (6s) de forma autônoma, utilizando a função millis() para garantir que o programa não seja bloqueado.

2. Controle Manual de Avanço: O botão de controle funciona como um disparador de transição. A cada clique:

• O estado atual é imediatamente finalizado.

• O semáforo avança para o próximo estado no ciclo 

• O cronômetro de tempo para o novo estado é resetado pela função mudarEstado().

## 5. Feedbacks (Samuel Vono)

Parabéns, seu projeto está excelente e recebe uma nota 9/10. A documentação está muito clara e profissional. O grande destaque técnico é o uso correto da função millis() em vez de delay(), criando uma máquina de estados robusta (switch-case) que permite a leitura do botão de forma responsiva. A implementação do botão de pedestre como "ir além" foi uma ótima escolha e a lógica da "memória" do botão (botaoPressionado) foi muito inteligente.

A principal sugestão para o 10/10 é um pequeno refinamento: no código atual, se o pedestre apertar o botão durante o sinal vermelho, o próximo sinal verde será pulado instantaneamente. Para corrigir isso, simplesmente mova a lógica de leitura do botão (if (digitalRead(pinoBotao) == HIGH)) para dentro do case 0 (o estado verde). 






