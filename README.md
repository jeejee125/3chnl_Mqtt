3 channel mqtt client switch for homeautomation. example usage for lights contoll at openhab

Hardware:

ESP8266 -12 
http://www.dx.com/p/esp-12-esp8266-serial-wifi-wireless-module-w-pcb-antenna-adapter-board-for-arduino-raspberry-pi-379296#.VS9lFt9Pq0w
Arduino nano:
http://www.dx.com/p/arduino-nano-v3-0-81877#.VS9lat9Pq0w

custmo made i/o board chematic and bcp under directory /Eagle

- Product: ![Program Connection diagram](Eagle/3chnlmqtt.png?raw=true)

Sofware :

ESP8266 

https://github.com/tuanpmt/espduino/


Arduino:

modifiet tuanpmt exaple of MQTT client

Example for MQTT client
=======

```c 


/**
 * \file
 *       ESP8266 MQTT Bridge example
 * \
 *       >
 */
#include <SoftwareSerial.h>
#include <espduino.h>
#include <mqtt.h>
int switchPinA = 6;   // switch is connected to pin 2
int switchPinB = 7;
int switchPinC = 8;
int ledPinA = 12;
int ledPinB = 11;
int ledPinC = 10;
int switchPin;
int led1Pin;
int val;                         // variable for reading the pin status
int val2;                       // variable for reading the delayed/debounced status
int abuttonState;                // variable to hold the button state
int bbuttonState;
int cbuttonState;
int lightModea = 0;
int lightModeb = 0;
int lightModec = 0;
SoftwareSerial debugPort(2, 3); // RX, TX
ESP esp(&Serial, &debugPort, 4);
MQTT mqtt(&esp);
boolean wifiConnected = false;

void wifiCb(void* response)
{
  uint32_t status;
  RESPONSE res(response);

  if(res.getArgc() == 1) {
    res.popArgs((uint8_t*)&status, 4);
    if(status == STATION_GOT_IP) {
      debugPort.println("WIFI CONNECTED");
      mqtt.connect("ip.of.mqtt.broker", 1883, false);
      wifiConnected = true;
      //or mqtt.connect("host", 1883); /*without security ssl*/
    } else {
      wifiConnected = false;
      mqtt.disconnect();
    }

  }
}

void mqttConnected(void* response)
{
  debugPort.println("Connected");
  mqtt.subscribe("/home/1/esp01/p1/com"); //or mqtt.subscribe("topic"); /*with qos = 0*/
  mqtt.subscribe("/home/1/esp01/p2/com");
  mqtt.subscribe("/home/1/esp01/p3/com");
  mqtt.publish("/home/1/esp01/p1/com", "data0");
  mqtt.publish("/home/1/esp01/p2/com", "data0");
  mqtt.publish("/home/1/esp01/p3/com", "data0");

}
void mqttDisconnected(void* response)
{

}
void mqttData(void* response)
{
  RESPONSE res(response);

  debugPort.print("Received: topic=");
  String topic = res.popString();
  debugPort.println(topic);

  debugPort.print("data=");
  String data = res.popString();
  debugPort.println(data);
  
  if (topic == "/home/1/esp01/p1/com"){

   if (data == "ON") {                // check if the button is pressed
        if (lightModea == 0) {          // is the light off?
          lightModea = 1;               // turn light on!
          digitalWrite(ledPinA, HIGH);
         // mqtt.publish("/home/1/esp01/p1/state","ON",0,0);
       
        } 
   }
   if (data == "OFF")      {
     if (lightModea == 1){
          lightModea = 0;               // turn light off!
          digitalWrite(ledPinA, LOW);
          // mqtt.publish("/home/1/esp01/p1/state","OFF",0,0);
         }
      }
  }
    if (topic == "/home/1/esp01/p2/com"){

   if (data == "ON") {                // check if the button is pressed
        if (lightModeb == 0) {          // is the light off?
          lightModeb = 1;               // turn light on!
          digitalWrite(ledPinB, HIGH);
         // mqtt.publish("/home/1/esp01/p1/state","ON",0,0);
       
        } 
   }
   if (data == "OFF")      {
     if (lightModeb == 1){
          lightModeb = 0;               // turn light off!
          digitalWrite(ledPinB, LOW);
          // mqtt.publish("/home/1/esp01/p1/state","OFF",0,0);
         }
      }
  }
  if (topic == "/home/1/esp01/p3/com"){

   if (data == "ON") {                // check if the button is pressed
        if (lightModec == 0) {          // is the light off?
          lightModec = 1;               // turn light on!
          digitalWrite(ledPinC, HIGH);
         // mqtt.publish("/home/1/esp01/p1/state","ON",0,0);
       
        } 
   }
   if (data == "OFF")      {
     if (lightModec == 1){
          lightModec = 0;               // turn light off!
          digitalWrite(ledPinC, LOW);
          // mqtt.publish("/home/1/esp01/p1/state","OFF",0,0);
         }
      }
  }
}
void mqttPublished(void* response)
{

}
void setup() {
  Serial.begin(19200);
  debugPort.begin(19200);
  esp.enable();
  delay(500);
  esp.reset();
  delay(500);
  while(!esp.ready());
  pinMode(switchPinA, INPUT);    // Set the switch pin as input
  pinMode(switchPinB, INPUT);    // Set the switch pin as input
  pinMode(switchPinC, INPUT);    // Set the switch pin as input
  pinMode(ledPinA, OUTPUT);
  pinMode(ledPinB, OUTPUT);
  pinMode(ledPinC, OUTPUT);
  debugPort.println("ARDUINO: setup mqtt client");
  if(!mqtt.begin("esp01", "admin", "Isb_C4OGD4c3", 120, 1)) {
    debugPort.println("ARDUINO: fail to setup mqtt");
    while(1);
  }


  debugPort.println("ARDUINO: setup mqtt lwt");
  mqtt.lwt("/lwt", "offline", 0, 0); //or mqtt.lwt("/lwt", "offline");

/*setup mqtt events */
  mqtt.connectedCb.attach(&mqttConnected);
  mqtt.disconnectedCb.attach(&mqttDisconnected);
  mqtt.publishedCb.attach(&mqttPublished);
  mqtt.dataCb.attach(&mqttData);

  /*setup wifi*/
  debugPort.println("ARDUINO: setup wifi");
  esp.wifiCb.attach(&wifiCb);

  esp.wifiConnect("wifiSSID","wifipasswd");


  debugPort.println("ARDUINO: system started");
}

void loop() {
  esp.process();
  if(wifiConnected) {
  switcheesA();
  switcheesB();
  switcheesC();
 // lights();  
 
  }
}

 void switcheesA(){
   val = digitalRead(switchPinA);   // read input value and store it in val
     delay(10);
 //    switchPin = switchPinA;
     val2 = digitalRead(switchPinA);
         
    
           // read the input again to check for bounces
  if (val == val2) {                 // make sure we got 2 consistant readings!
   if (val != abuttonState) {          // the button state has changed!
      if (val == LOW) {         // check if the button is pressed
          led1Pin = ledPinA;
        if (lightModea == 0) {          // is the light off?
          lightModea = 1;               // turn light on!
          digitalWrite(led1Pin, HIGH);
          mqtt.publish("/home/1/esp01/p1/state","ON",0,0);
       
        } else {
          lightModea = 0;               // turn light off!
          digitalWrite(led1Pin, LOW);
           mqtt.publish("/home/1/esp01/p1/state","OFF",0,0);
         }
      }
    }
  }
    abuttonState = val;
    
   
    } 
 
   void switcheesB(){
   val = digitalRead(switchPinB);   // read input value and store it in val
     delay(10);
     switchPin = switchPinB;
     val2 = digitalRead(switchPin);
      
     led1Pin = ledPinB;
    
           // read the input again to check for bounces
  if (val == val2) {                 // make sure we got 2 consistant readings!
   if (val != bbuttonState) {          // the button state has changed!
      if (val == LOW) {                // check if the button is pressed
        if (lightModeb == 0) {          // is the light off?
          lightModeb = 1;               // turn light on!
          digitalWrite(led1Pin, HIGH);
          mqtt.publish("/home/1/esp01/p2/state","ON",0,0);
       
        } else {
          lightModeb = 0;               // turn light off!
          digitalWrite(led1Pin, LOW);
           mqtt.publish("/home/1/esp01/p2/state","OFF",0,0);
         }
      }
    }
  }
    bbuttonState = val;
    
   
    } 
  
    void switcheesC(){
   val = digitalRead(switchPinC);   // read input value and store it in val
     delay(10);
     switchPin = switchPinC;
     val2 = digitalRead(switchPin);
      
     led1Pin = ledPinC;
    
           // read the input again to check for bounces
  if (val == val2) {                 // make sure we got 2 consistant readings!
   if (val != cbuttonState) {          // the button state has changed!
      if (val == LOW) {                // check if the button is pressed
        if (lightModec == 0) {          // is the light off?
          lightModec = 1;               // turn light on!
          digitalWrite(led1Pin, HIGH);
          mqtt.publish("/home/1/esp01/p3/state","ON",0,0);
       
        } else {
          lightModec = 0;               // turn light off!
          digitalWrite(led1Pin, LOW);
           mqtt.publish("/home/1/esp01/p3/state","OFF",0,0);
         }
      }
    }
  }
    cbuttonState = val;
    
   
    } 

```


openhab:

Add the OpenHab Repository

sudo nano /etc/apt/sources.list.d/openhab.list

Insert

 deb http://repository-openhab.forge.cloudbees.com/release/1.6.2/apt-repo/ /

Install Java

echo "deb http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main" | tee /etc/apt/sources.list.d/webupd8team-java.list
echo "deb-src http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main" | tee -a /etc/apt/sources.list.d/webupd8team-java.list
apt-get install oracle-java8-installer

Install OpenHAB

sudo apt-get install openhab-runtime openhab-addon-binding-mqtt openhab-addon-action-mail openhab-addon-binding-bluetooth openhab-addon-binding-serial openhab-addon-binding-weather openhab-addon-persistence-rrd4j


Install Mosquitto

sudo apt-add-repository ppa:mosquitto-dev/mosquitto-ppa
sudo apt-get install mosquitto mosquitto-clients

Start and Test Mosquitto

sudo /etc/init.d/mosquitto start

Configure OpenHAB to use a MQTT Binding

sudo nano /etc/openhab/configurations/openhab_default.cfg

At the very bottom, define a MQTT Broker:

mqtt:broker.url=tcp://192.168.1.100:1883
mqtt:broker.clientId=openhab
Exit, Save,  

restart OpenHAB

sudo /etc/init.d/openhab restart

sudo nano /etc/openhab/configurations/sitemaps/home.sitemap

sitemap demo label="Main Menu"
{
 Frame label="MQTT" {
   Switch item=lamp1 label="Lampa"
   Switch item=lamp2 label="Lampb"
   Switch item=lamp3 label="Lampc"
 }
}

sudo nano /etc/openhab/configurations/items/home.items

Switch lamp1 "Lampa" (all){mqtt=">[broker:/home/1/esp01/p1/com:command:on:ON],>[broker:/home/1/esp01/p1/com:command:off:OFF],<[broker:/home/1/esp01/p1/state:state:default]"}
Switch lamp2 "Lampb" (all){mqtt=">[broker:/home/1/esp01/p2/com:command:on:ON],>[broker:/home/1/esp01/p2/com:command:off:OFF],<[broker:/home/1/esp01/p2/state:state:default]"}
Switch lamp3 "Lampc" (all){mqtt=">[broker:/home/1/esp01/p3/com:command:on:ON],>[broker:/home/1/esp01/p3/com:command:off:OFF],<[broker:/home/1/esp01/p3/state:state:default]"}

sudo /etc/init.d/openhab restart

