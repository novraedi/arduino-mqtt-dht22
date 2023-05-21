# arduino-mqtt-dht22

#include <WiFi.h>
#include <PubSubClient.h>
#include <Adafruit_Sensor.h>
#include <DHT.h>

// Konfigurasi WiFi
const char* ssid = "GalaxyA50s";
const char* password = "novra1234";

// Konfigurasi MQTT
const char* mqttServer = "54.255.164.23";
const int mqttPort = 1883;
const char* mqttUser = "novan";
const char* mqttPassword = "123";

// Konfigurasi sensor suhu DHT11
#define DHT_PIN 2
#define DHT_TYPE DHT22
DHT dht(DHT_PIN, DHT_TYPE);

// Objek WiFi dan MQTT
WiFiClient espClient;
PubSubClient client(espClient);

void setup() {
  // Inisialisasi Serial Monitor
  Serial.begin(115200);

  // Menghubungkan ke jaringan WiFi
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
  Serial.println("Connected to WiFi");

  // Mengatur server MQTT dan koneksi
  client.setServer(mqttServer, mqttPort);
  client.setCallback(callback);

  // Inisialisasi sensor suhu DHT11
  dht.begin();
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  // Membaca suhu dari sensor DHT11
  float temperature = dht.readTemperature();

  // Mengirimkan suhu ke broker MQTT
  String temperatureString = String(temperature);
  char messageBuffer[50];
  temperatureString.toCharArray(messageBuffer, 50);
  client.publish("test/iot", messageBuffer);

  delay(5000); // Menunda pengiriman selama 5 detik
}

void callback(char* topic, byte* payload, unsigned int length) {
  // Fungsi callback untuk menerima pesan MQTT (tidak digunakan dalam contoh ini)
}

void reconnect() {
  while (!client.connected()) {
    Serial.println("Connecting to MQTT Broker...");
    if (client.connect("ESP32Client", mqttUser, mqttPassword)) {
      Serial.println("Connected to MQTT Broker");
    } else {
      Serial.print("Failed, rc=");
      Serial.print(client.state());
      Serial.println(" Retrying in 5 seconds...");
      delay(5000);
    }
  }
}
