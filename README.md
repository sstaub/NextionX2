# NextionX2
A new, alternative and universal library to interact with Nextion HMI displays from Arduino and compatible MCUs. The library is mainly based on Thierry's [NextionX](https://github.com/ITEAD-Thierry/NextionX) library and the [EasyNextionLibrary](https://github.com/Seithan/EasyNextionLibrary) from Seithan (return handling).

## General information
To be most universal, this library allows (in opposite to the official library) the use of multiple Nextion HMI displays connected to the same MCU under the condition to have enough hardware or software emulated serial ports (UARTs). 

On an Arduino MEGA, you could for example use the Serial1, Serial2 and Serial3 ports to connect up to 3 Nextion HMIs, while keeping the default Serial port free for debugging in the Serial Monitor of the Arduino IDE. On an Arduino UNO, you might use either the Nextion on Serial and no debugging or the Nextion on a SoftwareSerial port and use Serial for debugging.

The library is written without the use of dynamic memory allocation functions like String to avoid defragmentation of the heap.


## Example

In examples you will find a NextionX2.hmi file. This file file was created for an 3.5 Nextion Enhanced display but can simply modified with the Nextion editor.

![NextionX2 .hmi file](https://github.com/sstaub/NextionX2/blob/main/images/NextionX2.png?raw=true)
NextionX2 HMI demo file

The Nextionx2.ino shows the advantages of the library. The example use the SiftwareSerial libray, so the example runs on an Arduino UNO.

```cpp
#include "Arduino.h"
#include "SoftwareSerial.h"
#include "NextionX2.h"

SoftwareSerial softSerial(2, 3);

NextionComPort nextion;
NextionComponent version(nextion, 0, 1);
NextionComponent number(nextion, 0, 6);
NextionComponent text(nextion, 0, 7);
NextionComponent momentaryButton(nextion, 0, 2);
NextionComponent toggleButton(nextion, 0, 4);
NextionComponent slider(nextion, 0, 8);
NextionComponent checkbox(nextion, 0, 9);
NextionComponent xfloat(nextion, 0, 5);

uint32_t time;
#define TIMER 2000

void ledOn() {
  digitalWrite(LED_BUILTIN, HIGH);
  nextion.rectangleFilled(250, 150, 50, 50, RED);
  }

void ledOff() {
  digitalWrite(LED_BUILTIN, LOW);
  nextion.rectangleFilled(250, 150, 50, 50, BLACK);
  }

void ledToggle() {
  if (digitalRead(LED_BUILTIN)) ledOff();
  else ledOn();
  }

void setup() {
  nextion.begin(softSerial);
  //nextion.debug(Serial); // uncomment for debug
  Serial.begin(9600);
  pinMode(13, OUTPUT);
  version.attribute("txt", "v.1.0.0");
  number.value(5);
  text.text("hello");
  nextion.text(50, 280, 200, 50, 1, WHITE, BLUE, CENTER, MIDDLE, SOLID, "Hello Nextion");
  nextion.picture(320, 100, 0);
  nextion.pictureCropX(320, 160, 50, 50, 0, 0, 0);
  momentaryButton.touch(ledOn);
  momentaryButton.release(ledOff);
  toggleButton.touch(ledToggle);
  time = millis();
  }

void loop() {
  nextion.update();
  if (millis() > time + TIMER) {
    char string[64];
    strcpy(string, text.text());
    int32_t valueNumber = number.value();
    int32_t valueSlider = slider.value();
    Serial.print("Text Field String: ");
    Serial.println(string);
    Serial.print("Number Field Value: ");
    Serial.println(valueNumber);
    Serial.print("Slider Value: ");
    Serial.println(valueSlider);
    Serial.println();
    time = millis();
    }
  }

```

## Documentation

### Graphic Enumarations for text objects

```cpp
enum fill_t { // background fill modes
	CROP,
	SOLID,
	IMAGE,
	NONE
};

enum alignhor_t { // horizontal alignment
	LEFT,
	CENTER,
	RIGHT
};

enum alignver_t { // vertical alignment
	TOP,
	MIDDLE,
	BUTTON
}
```


### Colors

There are some colors predefined

```cpp
#define BLACK      0x0000
#define BLUE       0x001F
#define RED        0xF800
#define GREEN      0x07E0
#define CYAN       0x07FF
#define MAGENTA    0xF81F
#define YELLOW     0xFFE0
#define LIGHT_GREY 0xBDF7
#define GREY       0x8430
#define DARK_GREY  0x4208
#define WHITE      0xFFFF
```

There is also helper function to convert RGB 8bit values to the 16bit 565 format used by Nextion.

```cpp
uint16_t color565(uint8_t red, uint8_t green, uint8_t blue)
```


### *Object Constructor* NextionComPort
```cpp
NextionComPort
```

Creates a display object, this can used for multible displays

**Example**

```cpp
NextionComPort nextion;
```

### *Object Constructor* NextionComponent

```cpp
NextionComponent(NexComm_t &nexComm, uint8_t pageId, uint8_t objectId)
```
- **&nexComm** pointer to NextionComPort you want to use for
- **pageId** the page ID number for the component
- **objectId** the object ID number for the component

Creates a component object

**Example**

```cpp
NextionComponent text(nextion, 0, 7);
```

### *Methods* for NextionComPort

#### begin()
```cpp
void begin(nextionSeriaType &nextionSerial, uint16_t baud = 9600)
```
- **& nextionSerial** pointer to Serial object you want to use
- **baud** the baud rate, standard is 9600


Method to initialize the communication 

**Example**

```cpp
nextion.begin(softSerial); // for use with Softserial
nextion.begin(Serial1); // for use with Serial1 e.g. Arduino MEGA
```

#### debug()
```cpp
void begin(nextionSeriaType &debugSerial, uint16_t baud = 9600)
```
- **& debugSerial** pointer to Serial object you want to use
- **baud** the baud rate, standard is 9600


Method to initialize the communication for debugging

**Example**

```cpp
nextion.debug(Serial);
```

#### update()
```cpp
void update()
```

This must done in the Arduino ```loop()``` function

**Example**

```cpp
void loop() {
  nextion.update();
}
```


