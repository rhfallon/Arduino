*****************************
    Duel RFID Reader Tester with Arduino Uno

Pinout for SparkFun RFID USB Reader Inner
Arduino ----- RFID module
5V            VCC
GND           GND
D2            TX

Pinout for SparkFun RFID USB Reader Outer
Arduino ----- RFID module
5V            VCC
GND           GND
D3            TX

by Brendan Curley, 22 Feb 2016
modified by Rachel Fallon 23/2/16

Inspired by & partially adapted from
https://learn.sparkfun.com/tutorials/sparkfun-rfid-starter-kit-hookup-guide
http://bildr.org/2011/02/rfid-arduino/
https://www.arduino.cc/en/Tutorial/TwoPortReceive

This application comes with no guarantees and 
does not claim to be suitable for any purpose.

******************************/

#include <SoftwareSerial.h>

// Choose pins for SoftwareSerial devices
SoftwareSerial rInnerSerial(19,18); // RX, TX
SoftwareSerial rOuterSerial(17,16); // RX, TX

// For SparkFun's tags, we will receive 16 bytes on every
// tag read, but throw four away. The 13th space will always
// be 0, since proper strings in Arduino end with 0

// These constants hold the total tag length (tagLen) and
// the length of the part we want to keep (idLen)
const int tagLen = 16;
const int idLen = 13;

// Empty arrays to hold a freshly scanned tag
char newInnerTag[idLen];
char newOuterTag[idLen];

void setup() {
  // Open serial communications and wait for port to open:
  Serial1.begin(9600);
   while (!Serial1) {
  Serial2.begin(9600);
   while (!Serial2) {
 
    ; // wait for serial port to connect. Needed for native USB port only
  }

  // Start each software serial port
  rInnerSerial1.begin(9600);
  rOuterSerial2.begin(9600);
}

}
void loop() {
  // Counters for the tag arrays
  int i = 0;
  int j = 0;
  // Variable to hold each byte read from the serial buffer
  int readByte;
  // Flags so we know when a tag is over
  boolean tagInner = false;
  boolean tagOuter = false;

  // By default, the last intialized port is listening.
  // When you want to listen on a port, explicitly select it.
  // Listen to inner reader
  rInnerSerial.listen();

  // This makes sure the whole tag is in the serial buffer before
  // reading, the Arduino can read faster than the ID module can deliver!
  while (rInnerSerial.available() > 0 && tagInner == false) {
    if (rInnerSerial.available() == tagLen) {
      tagInner = true;
    }
    else {
      tagInner = false;
    }
  }

  if (tagInner == true) {
    while (rInnerSerial.available()) {
      // Take each byte out of the serial buffer, one at a time
      readByte = rInnerSerial.read();

      /* This will skip the first byte (2 indicates STX or Start of TeXt) and 
      the last three (ASCII 13 indicates CR or Carriage Return, ASCII 10 indicates 
      LF or LineFeed, and ASCII 3 indicates ETX or End of TeXt) leaving only the 
      unique part of the tag string. It puts the byte into the first space in the 
      array, then steps ahead one spot */
      if (readByte != 2 && readByte != 13 && readByte != 10 && readByte != 3) {
        newInnerTag[i] = readByte;
        i++;
      }

      // If we see ASCII 3, indicating ETX or End of TeXt, the tag is over
      if (readByte == 3) {
        tagInner = false;
      }
    }

    // Print tag from inner reader
    Serial.print("Inner RFID tag: ");
    Serial.println(newInnerTag);
  }

  // Listen to outer reader
  rOuterSerial.listen();

  // This makes sure the whole tag is in the serial buffer before
  // reading, the Arduino can read faster than the ID module can deliver!
  while (rOuterSerial.available() > 0 && tagOuter == false) {
    if (rOuterSerial.available() == tagLen) {
      tagOuter = true;
    }
    else {
      tagOuter = false;
    }
  }
  
  if (tagOuter == true) {
    while (rOuterSerial.available()) {
      // Take each byte out of the serial buffer, one at a time
      readByte = rOuterSerial.read();

      /* This will skip the first byte (2 indicates STX or Start of TeXt) and 
      the last three (ASCII 13 indicates CR or Carriage Return, ASCII 10 indicates 
      LF or LineFeed, and ASCII 3 indicates ETX or End of TeXt) leaving only the 
      unique part of the tag string. It puts the byte into the first space in the 
      array, then steps ahead one spot */
      if (readByte != 2 && readByte != 13 && readByte != 10 && readByte != 3) {
        newOuterTag[j] = readByte;
        j++;
      }

      // If we see ASCII 3, indicating ETX or End of TeXt, the tag is over
      if (readByte == 3) {
        tagOuter = false;
      }
    }

    // Print tag from outer reader
    Serial.print("Outer RFID tag: ");
    Serial.println(newOuterTag);
  }

  // Once tags have been checked, fill them with zeroes
  // to get ready for the next tag read
  for (int c = 0; c < idLen; c++) {
    newInnerTag[c] = 0;
    newOuterTag[c] = 0;
  }
}
