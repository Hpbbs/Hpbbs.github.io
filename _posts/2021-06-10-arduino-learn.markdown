---
layout: post
title: Arduino 基本概念 和 常用硬件操控
author: Hpbbs
categories: [硬件,Arduino]
---
* toc
{: toc }

Reference: http://www.funduino.de/Arduino-tutorials-08092014.pdf

### What is Arduino

开源，可编程，交互式硬件项目，项目或发明的原型
 Open-source-electronic-prototyping-base for simple used hardware and software
in the field of microcontrolling.

用户：It is mostly used b y artists, designer or
tinkers to realize creative ideas

open source programmable circuit board that can be integrated into a wide variety of makerspace projects both simple and complex

contains: microcontroller sense and control object in physical world

create interactive hardware projects

great platform for prototyping projects and inventions

### Type of Arduino Board type

很多类型的 官方板子
主要是直插适用于原型的，和 触电的适用于体积要求

#### 1. Arduino UNO

1. Reset Button – This will restart any code that is loaded to the Arduino board
2. AREF – Stands for “Analog Reference” and is used to set an external reference
voltage
3. Ground Pin – There are a few ground pins on the Arduino and they all work the
same
4. Digital Input/Output – Pins 0-13 can be used for digital input or output
5. PWM – The pins marked with the (~) symbol can simulate analog output
6. USB Connection – Used for powering up your Arduino and uploading sketches
7. TX/RX – Transmit and receive data indication LEDs
8. ATmega Microcontroller – This is the brains and is where the programs are
stored
9. Power LED Indicator – This LED lights up anytime the board is plugged in a
power source
10.Voltage Regulator – This controls the amount of voltage going into the Arduino
board
11.DC Power Barrel Jack – This is used for powering your Arduino with a power
supply
12.3.3V Pin – This pin supplies 3.3 volts of power to your projects
13.5V Pin – This pin supplies 5 volts of power to your projects
14.Ground Pins – There are a few ground pins on the Arduino and they all work the
same
15.Analog Pins – These pins can read the signal from an analog sensor and convert
it to digital

#### 2. Arduino BreadBoard

扩展用，外接电路用

#### 3. Arduino Shield

add a very specific functionality to your Arduino, you will need to use
a shield. Arduino shields plug into the top of the Arduino board and can add
capabilities such as WiFi, Bluetooth, GPS and much more

#### 4. Arduino Sensors

to sense the world around it

## Program with Arduino

two main part：
1. `void setup()`: 执行一次
2. `void loop()`: 循环执行

### 连接板子

#### IDE 操作
IDE/tools/Board:type_name
IDE/tools/port:serial (PC only choice，only possible for USB driver), 连接板子后，port会出现对应的设备com口

#### 安装USB driver

连接设备后，选择port，会自动推荐安装PC驱动
大多数时候，没法自动识别，需要自己在arduino文件选择driver自己安装
（Click on the unknown device and choose “update
USB driver）

### Programming 注入灵魂

#### Basic Part of sketch

1. Name Variable：非必要
2. setup：必要，只会执行一次
   > Here you are telling the program for example what Pin (slot for cables) should be an input and what should be an output on the boards

   > Defined as Output; Defined as Input
3. loop: 必要
   
   >  continuously repeated by the board. It assimilates the sketch from beginning to end and starts again from the beginning and so on

基础结束，很简单

#### 1. Blinking LED（basic gpio operation）

requirement：
   1. Board with USB cable
   2. led ：use build on pin 13 （for test purpose）

Code：
```c
void setup(){
    pinMode(13, OUTPUT);// 定义13口是输出口
}

void loop(){
    digitalWrite(13, HIGH);
    delay(1000);
    digitalWrite(13, LOW);
    delay(1000);
}
```

Uplaod to Borad:

    click:IDE/upload



#### 2. Blink alternative LED（外置装备连线）

requirement: microcontroller, tow LED(s), two resistor 100 ohm, breadboard, cables

code:

```c
void setup(){
    pinMode(7, OUTPUT);// 初始化 GPIO的控制器
    pinMode(8, OUTPUT);// 7,8 pin 为可输出引脚
}

void loop(){
    // 交替闪烁
    digitalWrite(7, HIGH);
    delay(1000);
    digitalWrite(7, LOW);
    digitalWrite(8, HIGH);
    delay(1000);
    digitalWrite(8, LOW);
}
```

#### 3. Fade LED（输出脉冲电平驱动）

The Arduino is a digital microcontroller，只有 5 volt on 和 5 volt off（+5v -5v）

For example 5 Volts if the LED shines bright, 4 Volts if it shines a bit darker and so on.
THIS DOESN'T WORK ON DIGITAL PINS. But there is an other option. It is called pulse
width modulation (short PWM).

code：

```c

int LED = 9;
int brightness = 0;
int fadingm = 5;

void setup(){
    pinMode(LED, OUTPUT);
}

void loop(){
    analogWrite(LED, brightness);
    brightness += fading;
    delay(25);

    if(brightness == 0 || brightness == 255){
        fading = -fading;
    }
}
```

#### 4. Push button & LED ( 引脚可以读取引脚的电压value)

The digital pins of the microcontroller are not only able to put out voltage, they are also able to read out voltage,

下拉电阻：

>  Now the difficulty: The electrons that are already floating around the pin are only escaping extremely slow. So the microcontroller thinks that the button has been pushed longer than it actually has been. The microcontroller thinks that the button has been pushed until the electrons have escaped completely from the pin. This problem can be fixed by grounding the pin with a 1000 Ohm (1K Ohm) resistor. Now the electrons are able to escape from the pin faster and the microcontroller recognizes that the button only has been pushed briefly. The resistor is called “PULLDOWN”- resistor, because the resistors is always “pulling down” the voltage to 0V.

按下button后，LED点亮5s

Requirement: Arduino, LED(blue), resistor 100 ohm, resister 1000ohm, breadboard, cables, Push button



Code：

```c
int LED=6;
int button=7;
int buttonStatus=0;

void setup(){
  pinMode(LED, OUTPUT);
  pinMode(button, INPUT);//配置引脚输入，可读电压
}

void loop(){
  buttonStatus = digitalRead(button);
  if(buttonStatus == HIGH){
    digitalWrite(LED, HIGH);
    delay(5000); // 由此可见，即使microcontroller空转，引脚的电压是不会改变的，除非重新配置了
    digitalWrite(LED, LOW);
  } else {
    digitalWrite(LED, LOW);
  }
}
```

#### 5. Colour LED （PWM to output a linear constant（比如0~255））



The PWM lets the voltage pulse from +5V to 0V. So the voltage gets turned off and on for only milliseconds. With a really high PWM the 5 V signal nearly gets constant on the pin. With a low PWM it is the other way around and the 5 V signal is barely there (This is only a reduced summary, so you should look it up on the internet, if you need more information). With PWM it is possible to get nearly the same effect as if the voltage would vary. 

引脚号标记有 “~”，说明是 PWM 引脚，比如 3 5 6 9 10 11 号引脚

PWM 使用 `analogWrite()`，来输出PWM

#### 6. Motion Detector

Task：检测到动作的时候，蜂鸣器就会鸣叫

Requirement: Arduino, motion detector, breadboard, cables, pizeo speaker

Content: Read out the voltage values of a motion detector and use them for an output

Motion Detector: The motion detector, also known as PIR sensor, is very simply constructed. Once it has detected a movement, it puts out 5V voltage on a pin. 

​	> 有两种模式，使用jumper来改变，使用 当有动作检测的时候，就会保持 pin 高电压的简单模式

#### 7. Photo Resistor

Task: A LED should light up if it gets dark or rather the photo resistor is covered. 

Required equipment: Arduino/ One LED / Resistor with 200 Ohm / Resistor with 10K Ohm / Breadboard / Cables / Photo resistor 

Learning content: Read out voltage and show out the values via “serial monitor”. 读出引脚电压使用 serial monitor 展示出来

如何读出电压值：

A photo resistor changes its resistance depending on the luminous intensity

如果直接将两个元件串联，会分压

To create a split voltage we need to connect the photo resistor with a resistor (1-10K Ohm, depending on what kind of photo resistor is used.The resistor should have nearly the same value of resistance as the used photo resistor

 0 Volt corresponds to the number 0 and the highest measurable value 5V corresponds to the number 1023 (0 to 1023 are 1024 numbers = 10 Bit). Example: A voltage of 2,5V is measured, so the microcontroller would give out the value 512 (1024:2)

Serial Monitor：possible to show the read out data from the microcontroller (numbers or text). It is quite useful, because you will not always have an LCD-Monitor connected to show some values. In this sketch we will use the “serial monitor” to get the values shown, which the microcontroller get from the photo resistor.

这个 Serial 应该指的与电脑相连的USB 串口，这样就可以在电脑的串口得到输入的信息，并被显示看到

Code:

```c
int input = A0;
int LED = 10;
int sensorValue = 0;

void setup(){
  Serial.begin(9600); // 配置 与串口通信，使用 serial monitor 读出光感电阻的值
  pinMode(LED, OUTPUT);
}

void loop(){
  delay(50);// 如果频率过高，LED可能不会亮灭，会一直亮，必须delay
  sensorValue = analogRead(input);
  Serial.print("Sensor value =");
  Serial.print(sensorValue);
  
  /* 打入512则点亮LED ，小于则关闭的代码省略 */
}
```



#### 8. Potentiometer 电位器

也是`analogRead()` 来读取模拟量的 引脚的值

#### 9. Temperature measurement

requirement: TMP 36 sensor, ...

Sensor: The sensor has three terminals. 5V, GND, and the pin for the temperature signal. On this pin, the sensor puts out a voltage between 0 and 2.0 volts. 0V = -50 ° C and 2.0V = 150 ° C

The sensor is supposed to be quite precise (±2°C) between -40°C and 150°C, according to the manufacturing. The voltage on this pin must be read out by the microcontroller board and then has to be converted into a temperature value.

很多sensor 如果连接不正确，会直接被电流毁掉



代码编辑：

使用 `analogRead()`由于sensor特点，会有温漂，所以需要收集采样 10 个数据 然后取个均值，同样可以通过 Serial 将值答应在PC terminal上

Sensor pin读出的值需要 映射到温度： “map(a,b,c,d,e)” - This is a “Map command”. With this command it is possible //to change a read out value (a) from one region between (b) and (c) into a //region between (d) and (e).

#### 10. Mesurement of Distance

Task: Measure a distance with the HC-SR04 ultrasonic sensor and show the result on the serial monitor. Required equipment: microcontroller board / cables / Breadboard / HC-SR04 ultrasonic sensor 

 ultrasonic 距离传感器的原理：

Sensor 4 pins: 5V GND echo trigger

The contacts 5V and GND are meant for the power supply. 

Trough the "trigger" pin the sensor gets a short signal (5V) from the microcontroller board to create a sound wave. As soon as the sound wave hits a wall or other objects, it will be reflected and comes back to the ultrasonic sensor. When the sensor detects this returned sound wave, the sensor will send a signal to the Arduino microcontroller trough the "echo" pin. 

The Arduino-board measures the time between the transmission and the return of the sound wave, and converts this time into a distance.(计算 trgger 与 echo 信号的时间差，就是声波的传输时间)



代码逻辑基本差不多，但是需要考虑到很多硬件和物理的特性

code: 使用了 pulseIn： 测量对应引脚 高/低电平持续的时间，并返回

```c

void loop(){
  
digitalWrite(trigger, LOW); //Low voltage on the trigger pin to produce a
//clear signal.
delay(5); //….for 5 milliseconds.
digitalWrite(trigger, HIGH); //Creating the soundwave.
delay(10); //..for 10 milliseconds.
digitalWrite(trigger, LOW); //Stop creating the soundwave.
52 
time = pulseIn(echo, HIGH); //With the command pulseIn (Capital “i” in the
//front of the “n”) the arduino board measures the time between sending and
//receiving the soundwave.
dist = (time/2) / 29.1; //This calculation transforms the measured time into
//the distance in centimeter. (The sound needs 29,1 seconds for one centimeter.
//The time gets divided with two, because we only want to get one distance and
//not the two ways that the soundwave has to take). 
 
  ....
}
```



#### 12. Usage of Infrared Remote （红外设备）

Task: Read out the signal from an infrared remote with an infrared sensor.

重点硬件：infrared receiver

代码：使用了 IRremote.h 的library

```c
IRrecv irrecv(pin);
decode_results results;

void setup(){
  irrecv.enableIRIn(); 
}

void loop(){
  if (irrecv.decode(&results)) { //If data is received...
		Serial.println(results.value, DEC); //..they should show up on the serialmonitor as decimal number (DEC).
		irrecv.resume(); //Receive the next value.
	}
}
```

#### 13. Control a servo (伺服电机)

一样的思路，直接include 对应library，创建代理硬件控制器的c对象，在setup配置pin，并将硬件绑定到对应的pin，在loop逻辑中，按规矩使用

要注意的是，include的库里面，隐藏了对应物理特性，封装出来的只是一个面向对象的功能

CODE:

```c
#include <Servo.h>
Servo servoblue;
void setup(){
  servoblue.attach(8);
}
void loop(){
  servoblue.write(0);
  delay(3000);
  
  servoblue.write(90);
  delay(3000);
  
  servoblue.write(180);
  delay(3000);
  
  servoblue.write(270);
  delay(3000);
  
  servoblue.write(0);
  delay(3000);
}
```



#### 14. LCD Display

LCD硬件连线比较麻烦，应为他的线很多

编码：使用 include<LiquidCrystal.h>

Code:

```c
#include <LiquidCrystal.h>
LiquidCrystal lcd(12,11,6,5,4,3);
void setup(){
	lcd.begin(16,2);
}
void loop(){
  lcd.setCursor(0,0);
  lcd.print(string);
  lcd.setCursor(0,0);
  lcd.print(string);
}
```

#### 15. Relay Shield(开关)

是一种间接控制的工具

ATTENTION: There are two types of relay shields. They differ in the way you get them to switch. Either you have to connect the “IN” pin to 5V+ on the Arduino, or you have to connect the “IN” pin to GND on the Arduino. You can easily find out which type of relay you got. Try connecting the “signal” pin with 5V+ or with GND. The version where the relay switches (you can hear a loud crack) will be the right version. 



#### 16. Stepper 步进电机

requirement: Arduino, stepper with control board, 6 cables

This stepper is especially useful for small projects with the arduino board. The stepper can operate without any external power supply. It can get a quite high torque. (不用额外电源)，One full rotation of the drive shaft can be divided in 2048 separate steps. One little disadvantage of this can be the slow maximum rotation speed. （缺点是 转速比较慢）

code

```c
#include <Stepper.h>
int SPMU = 32;
Stepper myStepper(SPMU, 2,3,4,5);
void setup()
{
	myStepper.setSpeed(500);
}
void loop() {
	myStepper.step(2048);
	delay(500);
	myStepper.step(-2048);
	delay(500);
}
```



#### 17. Moisture sensor(湿度水分传感器)

That means that it can measure the directly adjacent moisture, like Skin moisture or ground humidity, but not the air moisture. One example of use can be the moisture sensor used to measure the ground humidity of a plant.

直接使用AnalogRead就可以读出来

#### 18. RFID Kit

Task: Read out the UID of a RFID tag and display the UID as one contiguous decimal number on the serial monitor.

The RFID (“radio frequency identification”) reader is used to read out a certain code, which is send from a RFID transmitter (also called “RFID tag”) by radio. Each RFID tag has only one unique code. The RFID Kit is useful to realize projects like for example a locking mechanism or other similar projects in which a person should be identified with a tag. 

How does it work? A RFID receiver contains a small copper coil that generates a magnetic field. An RFID transmitter also includes a copper coil that picks up the magnetic field and generates an electrical voltage insinde the transmitter. This voltage is used by a small electronic chip to get it to emit an electrical code by radio. The transmitter directly receives this code and processes it, so that the microcontroller is able to process the received code

Read out and process the data of RFID tags with the Arduino

Equipment: Arduino Uno or MEGA, RFID reader, at lease one RFID tag, breadboard, cables, LED, 200 ohm resistor

连线：

Board: Arduino Uno Arduino Mega MFRC522-READER

Pin: 10 53 SDA

Pin: 13 52 SCK

Pin: 11 51 MOSI

Pin: 12 50 MISO

Pin: nothing nothing IRQ

Pin: GND GND GND

Pin: 9 5 RST

Pin: 3,3V 3,3V 3,3V



从GitHub下载对应的 驱动



Code：

. This program is only destined for UNO R3 microcontroller. If you want to use MEGA2560 or other controllers, you have to adjust the pins.

```c
#include <SPI.h>
#include <MFRC522.h>

...
  
void setup(){
  SPI.begin();
  mfrc522.PCD_init();
}
void loop(){
  mfrc522.PICC_IsNewCardPresent();
  mfrc522.PICC_ReadCardSerial();
}
```

#### 19. I2C Display

Task: Show a text on an I²C LCD Module. 

LCD module 已经提供I2C的接口， I2C的接口连线更少，适用于复杂的项目

Code：

需要下载 I2C display的library

然后使用： Wire 和 LiquidCrystal_I2C 的header

