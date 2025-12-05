#include <WiFi.h>
#include <WiFiClientSecure.h> // Biblioteca para conexão segura (cadeado)
#include <PubSubClient.h>

// -----------------------------------------------------------------
// 1. CREDENCIAIS (Edite isto com seus dados!)
// -----------------------------------------------------------------

// WiFi
const char* ssid     = "DecoPrincipal";
const char* password = "5545238290";

// MQTT (dados da sua conta HiveMQ Cloud)
const char* mqtt_server = "9dd3751a8eea408ca765c1d3066a96d5.s1.eu.hivemq.cloud"; // URL do servidor
const int   mqtt_port   = 8883;  // Porta segura SSL
const char* mqtt_user   = "BRENOSOUZA"; // Usuário do painel
const char* mqtt_pass   = "Joelma123";  // Senha do painel

// Tópicos MQTT
const char* topic_publicar  = "minha_casa/sensor_distancia";
const char* topic_suscribir = "minha_casa/luz";

// -----------------------------------------------------------------
// 2. HARDWARE (PINOUT)
// -----------------------------------------------------------------
const int trigPin = 5;
const int echoPin = 18;
const int ledPin  = 2;

// -----------------------------------------------------------------
// 3. OBJETOS E VARIÁVEIS GLOBAIS
// -----------------------------------------------------------------
WiFiClientSecure espClient;   // Cliente seguro (TLS)
PubSubClient client(espClient);

long lastMsg = 0; // temporizador sem delay

// Variáveis para medir tempo (em microssegundos)
unsigned long tempoSensor_us = 0;
unsigned long tempoLed_us    = 0;

// -----------------------------------------------------------------
// 4. FUNÇÕES AUXILIARES
// -----------------------------------------------------------------

void setup_wifi() {
  delay(10);
  Serial.print("\nConectando ao WiFi: ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi Conectado!");
}

// Função chamada sempre que chega uma mensagem do broker MQTT
void callback(char* topic, byte* payload, unsigned int length) {

  Serial.println(">>> ENTROU NA CALLBACK MQTT <<<");  // DEBUG VISÍVEL

  // Marca o tempo na chegada da mensagem
  unsigned long inicioLed = micros();

  Serial.print("Mensagem recebida [");
  Serial.print(topic);
  Serial.print("]: ");

  String mensagem = "";
  for (int i = 0; i < length; i++) {
    mensagem += (char)payload[i];
  }
  Serial.println(mensagem);

  // CONTROLE DO LED
  if (mensagem == "ON" || mensagem == "1") {
    digitalWrite(ledPin, HIGH);
    Serial.println("-> AÇÃO: Luz ligada");
  } else if (mensagem == "OFF" || mensagem == "0") {
    digitalWrite(ledPin, LOW);
    Serial.println("-> AÇÃO: Luz desligada");
  }

  // Calcula tempo de resposta do LED
  unsigned long fimLed = micros();
  tempoLed_us = fimLed - inicioLed;
  float tempoLed_s = tempoLed_us / 1000000.0; // converte para segundos

  Serial.print("Tempo de resposta do LED (s): ");
  Serial.println(tempoLed_s, 6);  // 6 casas decimais porque é bem rápido
}

void reconnect() {
  // Loop até conseguir conectar
  while (!client.connected()) {
    Serial.print("Tentando conexão MQTT segura...");

    // ID aleatório para evitar conflitos
    String clientId = "ESP32Client-";
    clientId += String(random(0xffff), HEX);

    // Tentativa de conexão com usuário e senha
    if (client.connect(clientId.c_str(), mqtt_user, mqtt_pass)) {
      Serial.println(" Conectado!");
      // Assina o tópico para receber comandos
      client.subscribe(topic_suscribir);
    } else {
      Serial.print(" Falhou, rc=");
      Serial.print(client.state());
      Serial.println(" tentando novamente em 5 segundos...");
      delay(5000);
    }
  }
}

// -----------------------------------------------------------------
// 5. SETUP E LOOP
// -----------------------------------------------------------------

void setup() {
  Serial.begin(9600);

  // Pinos
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  pinMode(ledPin, OUTPUT);

  // Conexão WiFi
  setup_wifi();

  // Conexão TLS sem verificação de certificado (uso acadêmico)
  espClient.setInsecure();

  client.setServer(mqtt_server, mqtt_port);
  client.setCallback(callback);
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop(); // Mantém a comunicação MQTT ativa

  // Lógica do sensor (a cada 2 segundos)
  long now = millis();
  if (now - lastMsg > 2000) {
    lastMsg = now;

    // Marca início da medição do sensor
    unsigned long inicioSensor = micros();

    // Disparo do HC-SR04
    digitalWrite(trigPin, LOW);
    delayMicroseconds(2);
    digitalWrite(trigPin, HIGH);
    delayMicroseconds(10);
    digitalWrite(trigPin, LOW);

    long duration = pulseIn(echoPin, HIGH);
    int distance = duration * 0.034 / 2;

    // Marca fim e calcula tempo do sensor
    unsigned long fimSensor = micros();
    tempoSensor_us = fimSensor - inicioSensor;
    float tempoSensor_s = tempoSensor_us / 1000000.0;

    Serial.print("Tempo de resposta do SENSOR (s): ");
    Serial.println(tempoSensor_s, 6);

    // Publica distância se estiver em faixa válida
    if (distance > 0 && distance < 400) {
      char msg[10];
      snprintf(msg, 10, "%d", distance);

      Serial.print("Publicando Distância: ");
      Serial.println(msg);

      client.publish(topic_publicar, msg);
    }
  }
}
