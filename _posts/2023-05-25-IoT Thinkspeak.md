# 溫溼度測量作業
IoT Thinkspeak.com<br>
## 成果展示<br>
![](https://github.com/kaoethan/MCU-project/blob/main/images/346161727_246609404621194_476624291918512647_n.png?raw=true)<br>
會有曲線是因為我對他吹氣 造成溫度濕度改變<br>
## 方塊圖展示<br>
![](https://github.com/kaoethan/MCU-project/blob/main/images/iot%202.jpg?raw=true)<br>
## 程式
```
/*
 *  This sketch sends data via HTTP GET requests to thingspeak service every 10 minutes
 *  You have to set your wifi credentials and your thingspeak key.
 */
#include <WiFi.h>

#include "DHT.h"

#define DHTPIN 23     // NodeMCU pin D6 connected to DHT11 pin Data
DHT dht(DHTPIN, DHT11, 15);

const char* ssid     = "Test";
const char* password = "tonyikci";


const char* host = "api.thingspeak.com";
const char* thingspeak_key = "9SBGA9RP7ZIOAL9B";

void turnOff(int pin) {
  pinMode(pin, OUTPUT);
  digitalWrite(pin, 1);
}

void setup() {
  Serial.begin(115200);

  // disable all output to save power
  turnOff(0);
  turnOff(2);
  turnOff(4);
  turnOff(5);
  turnOff(12);
  turnOff(13);
  turnOff(14);
  turnOff(15);

  dht.begin();
  delay(10);
  

  // We start by connecting to a WiFi network

  Serial.println();
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);
  
  WiFi.begin(ssid, password);
  
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi connected");  
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

int value = 0;

void loop() {
  delay(5000);
  ++value;

  Serial.print("connecting to ");
  Serial.println(host);
  
  // Use WiFiClient class to create TCP connections
  WiFiClient client;
  const int httpPort = 80;
  if (!client.connect(host, httpPort)) {
    Serial.println("connection failed");
    return;
  }

  String temp = String(dht.readTemperature());
  String humidity = String(dht.readHumidity());
  String url = "/update?key=";
  url += thingspeak_key;
  url += "&field1=";
  url += temp;
  url += "&field2=";
  url += humidity;
  
  Serial.print("Requesting URL: ");
  Serial.println(url);
  
  // This will send the request to the server
  client.print(String("GET ") + url + " HTTP/1.1\r\n" +
               "Host: " + host + "\r\n" + 
               "Connection: close\r\n\r\n");
  delay(10);
  
  // Read all the lines of the reply from server and print them to Serial
  while(client.available()){
    String line = client.readStringUntil('\r');
    Serial.print(line);
  }
  
  Serial.println();
  Serial.println("closing connection. going to sleep...");
  delay(1000);
  // go to deepsleep for 1 minutes
  //system_deep_sleep_set_option(0);
  //system_deep_sleep(1 * 60 * 1000000);
  delay(1*10*100);
}
```
