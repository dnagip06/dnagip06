//Cài đặt
#define BLYNK_PRINT Serial
#include <Blynk.h>
#include <WiFi.h>
#include <Wire.h>
#include "DHT.h"
#include <WiFiClient.h>
#include <BlynkSimpleEsp32.h>
#include "MAX30100_PulseOximeter.h"
#define REPORTING_PERIOD_MS 1000
char auth[] = "GZOpIuOzDGOrtcc1o-yIBQ_ZBMRWCZr2";
char ssid[] = "HNT";
char pass[] = "123456789";
int tam1=0;
int tam2=0;
int tam3=0;
//
// Nhiệt độ, độ ẩm
#define DHTPIN 25
#define DHTTYPE DHT22
DHT dht(DHTPIN, DHTTYPE);
BlynkTimer timer;
void sendSensor()
{
  float h = dht.readHumidity();
  float t = dht.readTemperature();

  if (isnan(h) || isnan(t)) {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }
   Serial.println("Nhiệt độ");
    Serial.println(t);
  Serial.println("Độ ẩm");
    Serial.println(h);
  Blynk.virtualWrite(V0, t);
  Blynk.virtualWrite(V1, h);
}
//

//Màu
#define S0 14
#define S1 27
#define S2 13
#define S3 12
#define sensorOut 26
int R = 0;
int G = 0;
int B = 0;
void mausac() {
  digitalWrite(S2, LOW);
  digitalWrite(S3, LOW);
  R = pulseIn(sensorOut, LOW);

  digitalWrite(S2, HIGH);
  digitalWrite(S3, HIGH);
  G = pulseIn(sensorOut, LOW);

  digitalWrite(S2, LOW);
  digitalWrite(S3, HIGH);
  B = pulseIn(sensorOut, LOW);
}
//

//Huyết áp
PulseOximeter pox;
float BPM, SpO2;
uint32_t tsLastReport = 0;
void onBeatDetected()
{
  Serial.println("Beat!");
}
//

//Timer

WidgetLED led(V7);
BLYNK_WRITE(V4)
{
  int giatri1 = param.asInt();
  Serial.print(giatri1);
  if (digitalRead(giatri1 == 1))
  {
    tam1=2
  }
  if(tam1 == 2)
  {
    digitalWrite(5, 1);
    led.on();
    Blynk.notify("ĐÃ TỚI GIỜ UỐNG THUỐC LIỀU 01");
    delay(1000);
  }
  if((digitalRead(R<5))&&(digitalRead(G<5))&&(digitalRead(B<5)))
  {
    digitalWrite(5, 0);
    led.off();
  } else
  {
    digitalWrite(5, 1);
    led.on();
  }
  }
  
WidgetLED led1(V8);
BLYNK_WRITE(V5)
{
  int giatri2 = param.asInt();
  if (digitalRead(giatri2 == 1))
  {
    digitalWrite(18, 1);
    led1.on();
    Blynk.notify("ĐÃ TỚI GIỜ UỐNG THUỐC LIỀU 02");
    
  }
}

WidgetLED led2(V9);
BLYNK_WRITE(V6)
{
  int giatri3 = param.asInt();
  if (digitalRead(giatri3 == 1))
  {
    digitalWrite(19, 1);
    led2.on();
    Blynk.notify("ĐÃ TỚI GIỜ UỐNG THUỐC LIỀU 03");
  }
  }
//

void setup()
{
  Serial.begin(115200);
  pinMode(5, OUTPUT);
  pinMode(18, OUTPUT);  
  pinMode(19, OUTPUT);
  pinMode(27, INPUT);
  pinMode(14, INPUT);
  pinMode(12, INPUT);
  pinMode(13, INPUT);
  pinMode(26, INPUT);
  pinMode(25, INPUT);
  pinMode(35, INPUT_PULLUP);
  pinMode(S0, OUTPUT);
  pinMode(S1, OUTPUT);
  pinMode(S2, OUTPUT);
  pinMode(S3, OUTPUT);
  pinMode(sensorOut, INPUT);
  digitalWrite(S0, HIGH);
  digitalWrite(S1, LOW);
  Blynk.begin(auth, ssid, pass, "blynk-server.com", 8080);
  dht.begin();
  timer.setInterval(1000L, sendSensor);
  pox.begin();
  pox.setOnBeatDetectedCallback(onBeatDetected);
  Blynk.notify("HỆ THỐNG KHỞI ĐỘNG THÀNH CÔNG");
}

void loop()
{
  mausac();  
  Blynk.run();
  timer.run();
  pox.update();
  BPM = pox.getHeartRate();
  SpO2 = pox.getSpO2();
    Serial.print("SPO2 = ");
    Serial.print(SpO2);
    Serial.print("BPM = ");
    Serial.println(BPM);
  {
    Blynk.virtualWrite(V2, BPM);
    Blynk.virtualWrite(V3, SpO2);
  }
}
