#include <ESP8266WiFi.h>
#include <LiquidCrystal.h>
#include <DHT.h>

#define DHTPIN A0
#define DHTTYPE DHT11
DHT dht(DHTPIN, DHTTYPE);

LiquidCrystal lcd(12,11,5,4,3,2);

const char* ssid = "YourWiFi";
const char* password = "YourPassword";
const char* server = "http://yourserver.com/update";

const int batteryPin = A1;
float batteryVoltage, SOC, temperature;

void setup() {
  lcd.begin(16,2);
  dht.begin();
  Serial.begin(115200);
  WiFi.begin(ssid, password);
  
  lcd.print("Connecting WiFi");
  while(WiFi.status() != WL_CONNECTED) delay(500);
  lcd.clear();
  lcd.print("WiFi Connected");
  delay(2000);
}

void loop() {
  batteryVoltage = analogRead(batteryPin) * (25.0/1023.0);
  SOC = (batteryVoltage / 25.0) * 100.0;
  temperature = dht.readTemperature();

  lcd.setCursor(0,0);
  lcd.print("SOC:");
  lcd.print(SOC,1);
  lcd.print("%");
  
  lcd.setCursor(0,1);
  lcd.print("Temp:");
  lcd.print(temperature,1);
  lcd.print("C");

  // Send data to server
  if(WiFi.status() == WL_CONNECTED){
    WiFiClient client;
    if(client.connect(server,80)){
      client.print(String("GET /update?SOC=") + SOC + "&Temp=" + temperature + " HTTP/1.1\r\nHost: yourserver.com\r\nConnection: close\r\n\r\n");
      client.stop();
    }
  }
  
  delay(2000);
}
