# vasoDePlantasCaseiroInteligente


//inclui aqui as bibliotecas ESP8266WiFi e PubSubClient
//que tornarão o código funcional.

#include <ESP8266WiFi.h> 
#include <PubSubClient.h>

// a seguir defino as credenciais de wifi.

const char* ssid = "Madalena";
const char* wifi_password = "ehminhaprincesadomar";

// a seguir o servidor mqtt é definido

const char* mqtt_server = "test.mosquitto.org";

// deixei ilustrativamente essas variaveis seguintes
// mas aqui não serão utilizadas.

const char* username = "";
const char* clientId = "";
const char* password = "";

// a seguir o canal criado no Mqtt Explorer para esse trabalho

const char* topic = "channels/2156692/publish/MXIAT7F4WRYU01FF";

// criamos a seguir uma instância do wifi e do MQTT que está
// na biblioteca PubSubClient

WiFiClient wifiClient;
PubSubClient client(wifiClient);

// a seguir definimos também as variáveis do sensor 
// e do LED vermelho que funcionará como atuador simples
// do nosso projeto

const int sensorPin = A0;
const int ledPin = D7;

// no método setup a seguir definimos
// as configurações iniciais que permitirá a comunicação serial e a definição do 
// pino do LED. Aqui é feita a conectação à rede Wi-Fi.

void setup() {
  Serial.begin(115200);

  pinMode(ledPin, OUTPUT);
  digitalWrite(ledPin, LOW);

  WiFi.begin(ssid, wifi_password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connectando ao WiFi...");
  }
  Serial.println("Connectado ao WiFi!");

  client.setServer(mqtt_server, 1883);
}

// Aqui no método loop() a seguir, verifica-se 
// se o cliente MQTT está conectado. Se não estiver, o método reconnect()
// é chamado para se reconectar.

void loop() {
  if (!client.connected()) {
    reconnect();
  }

// Aqui a seguir lê-se
// o valor do sensor analógico conectado ao pino A0 do NodeMcu
// e converte-se o valor para uma string que será publicada no
// topico definido na Plataforma MQTT EXPLORER através de uma
// uma função if else a seguir, e que também exibe no monitor 
// a informação se foi ou não possível enviar a medição. 

  int sensorValue = analogRead(sensorPin);
  String payload = String(sensorValue);
  if (client.publish(topic, payload.c_str())) {
    Serial.println("Medição enviada!");
  } else {
    Serial.println("Falha no envio da medição!");
  }

// Neste if/else seguinte é feito o controle do ATUADOR led, que
// é o atuador mais simples existente em Iot, mas ainda sim, um 
// atuador! Se o valor do sensor for maior que 650, a luz Led 
// vermelha é acionada e permanece ligada, se ficar abaixo, ela desliga,
// e na sequência há uma espera de 1 segundo para recomeçar. 

  if (sensorValue > 650) {
    digitalWrite(ledPin, HIGH);
  } else {
    digitalWrite(ledPin, LOW);
  }

  delay(1000);
}

// o método reconnect() quando há perda da conexão, como o próprio 
// nome sugere. Se for bem sucedida, ele exibe no monitor serial
// uma mensagem de sucesso, senão, ele aponta a falha e continua tentando
// após 5 segundos.

void reconnect() {
  while (!client.connected()) {
    if (client.connect(clientId, username, password)) {
      Serial.println("Connectado ao broker com sucesso!");
    } else {
      Serial.println(" Falha em se conectar. Tentando novamente.");
      delay(5000);
    }
  }
}
