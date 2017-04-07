# ArduinoTelemetrie

Arduino Télémétrie Marmott Energies

Wrtitten by Antoine Toenz


Started //  03_04_2017  //

// esp8266_test.ino
//
// Plot LM35 data on thingspeak.com using an Arduino and an ESP8266 WiFi 
// module.
//
// Author: Mahesh Venkitachalam
// Website: electronut.in




// Ajout de librairies 
#include <SoftwareSerial.h>
#include <stdlib.h>


// LED 
int ledPin = 13;
// LM35 analog input
int lm35Pin = 0;

// GLOBALS
int wifiStatus = 0;
bool done = false;
String ssid = "marmott energies";
String key = "79FC371C";
String apiKey = "BSKYZ67DV68B0B9J";

// connect 10 to TX of Serial USB
// connect 11 to RX of serial USB
SoftwareSerial esp8266(10, 11); // RX, TX

// this runs once
void setup()
  {                
  // initialize the digital pin as an output.
  pinMode(ledPin, OUTPUT);    
  // enable debug serial
  Serial.begin(115200); //115200
  // enable software serial
  esp8266.begin(115200); //115200;
  // reset ESP8266-01 (module wifi)
  }

// the loop 
void loop() {
  
  // faire clignoter la LED on board
  digitalWrite(ledPin, HIGH);   
  delay(3000);  
  digitalWrite(ledPin, LOW);

  // read the value from LM35.
  // read 10 values for averaging.
  int val = 0;
  for(int i = 0; i < 10; i++) {
      val += analogRead(lm35Pin);   
      delay(500);
  }

  // convert to temp:
  // temp value is in 0-1023 range
  // LM35 outputs 10mV/degree C. ie, 1 Volt => 100 degrees C
  // So Temp = (avg_val/1023)*5 Volts * 100 degrees/Volt
  float temp = val*50.0f/1023.0f;

  // convert to string
  char buf[16];
  String strTemp = dtostrf(temp, 4, 1, buf);
  
  Serial.println(strTemp);
  

  // TCP connection (TCP>UDP)
  String cmd = "AT+CIPSTART=\"TCP\",\"";
  // Envoie au serveur de thingspeak
  cmd += "184.106.153.149"; // api.thingspeak.com
  Serial.println(cmd);
  // Port 80
  cmd += "\",80";
  esp8266.println(cmd);
   
  if(esp8266.find("Error")){
    Serial.println("AT+CIPSTART error");
    return;
  }
  
  // prepare GET string
  String getStr = "GET /update?api_key=";
  getStr += apiKey;
  getStr +="&field1=";
  getStr += String(strTemp);
  getStr += "\r\n\r\n";

  // send data length
  cmd = "AT+CIPSEND=";
  cmd += String(getStr.length());
  esp8266.println(cmd);
  
  if(esp8266.available())
  {
    while(esp8266.available())
    {
      char q = esp8266.read();
      Serial.write(q);
     }
   }
   
   if(Serial.available())
   {
     delay(1000);
     String command="";
     
     while(Serial.available())
     {
       command+=(char)Serial.read();
      }
      esp8266.println(command);
     }


// Permet de pouvoir changer de API KEY

 if(esp8266.find(">"))
 { 
    esp8266.print(getStr);
  }
else
{ 
    esp8266.println("AT+CIPCLOSE");
    // alert user
    Serial.println("AT+CIPCLOSE");

 /*   while (esp8266.available())
    {
         char c = esp8266.read();
         Serial.write(esp8266.read());
         if (c == '\r') Serial.print('\n');
    }
    */
}
  
  // thingspeak needs 15 sec delay between updates
  delay(16000);
  // delay(16000);  
}
