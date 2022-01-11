# DUIET-code
The code used in Arduino and Processing for the Assignments:

Assignement 3:
Aruidno:

#include <SPI.h>
#include "RFID.h"
#define SS_PIN 10
#define RST_PIN 9
#define RFID_BITNUM 5
#define SENSOR_NUM 4
#define READ_RATE 200
RFID rfid(SS_PIN, RST_PIN);
int rfidSN[RFID_BITNUM];
int card;
char dataID[RFID_BITNUM] = {'A', 'B', 'C', 'D'}; //data label
//int lastID[SENSOR_NUM] = {0,0,0,0};
long timer = micros(); //timer

int counter = 0;
void setup()
{
  Serial.begin(115200);
  SPI.begin();
  rfid.init();
  for (int i = 0 ; i < RFID_BITNUM ; i++)  rfidSN[i] = -1;
}

void loop()
{
  if (micros() - timer >= (1000000 / READ_RATE) ) { //Timer: send sensor data in every 10ms
    timer = micros();
    if (rfid.isCard()) {  // A tag is detected
      if (rfid.readCardSerial()) { // Read the tag's ID
        for (int i = 0 ; i < RFID_BITNUM ; i++)  rfidSN[i] = rfid.serNum[i];
      }
    }
    else { // No tag is detected.
      for (int i = 0 ; i < RFID_BITNUM ; i++)  rfidSN[i] = -1;
    }

    if (rfidSN[0] < 0) {
      //  Serial.println("No Tag");
    } else {
      if  (rfidSN[0] == 121) {
        card = 1;
        Serial.print("1");
      }
      else if (rfidSN[0] == 25) {
        card = 2;
        Serial.print("2");
      }

      Serial.println();
    }
    rfid.halt(); //Halt the rfid reader
  }
}

void sendDataToProcessing(char symbol, int data) {
  Serial.print(symbol);  // symbol prefix of data type
  Serial.println(data);  // the integer data with a carriage return
}


Processing:

import processing.serial.*;
import processing.sound.*;

Serial myPort;  // Create object from Serial class
String val;     // Data received from the serial port
String command;

void setup()
{
  // I know that the first port in the serial list on my mac
  // is Serial.list()[0].
  // On Windows machines, this generally opens COM1.
  // Open whatever port is the one you're using.
  size(540, 1080);
  String portName = Serial.list()[2]; //change the 0 to a 1 or 2 etc. to match your port
  myPort = new Serial(this, portName, 115200);
  image(loadImage("Duel card game states 0.png"), 0, 0);
}

void draw()
{
  if ( myPort.available() > 0)
  {  // If data is available,
    val = myPort.readStringUntil('\n');         // read it and store it in val
    //println(val); //print it out in the console
    if (val != null) {

      command = val.substring( 0, val.length()-2 );

      println(command, " with length ", command.length());

      if (command.equals("1") == true) {
        image(loadImage("Duel card game states 1.png"), 0, 0);
      } else if (command.equals("2") == true) {
        image(loadImage("Duel card game states 2.png"), 0, 0);
      }
    }
  }
  if (keyPressed) {

    if (key == '2') {
      image(loadImage("Duel card game states 3.png"), 0, 0);
    } else if (key == '3') {
      image(loadImage("Duel card game states 5.png"), 0, 0);
    } else if (key == '1') {
      image(loadImage("Duel card game states 0.png"), 0, 0);
    }
  }
}
