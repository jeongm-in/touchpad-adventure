# touchpad-adventure

I've been obsessed with the [Framework Laptop](https://frame.work/) lately, especially after they announced the AMD model. However, I could not justify the purchase because I have a perfectly fine 2018 LG Gram. 
One day I was browing Framework's parts shop and saw this [touchpad module](https://frame.work/products/touchpad-kit?v=FRANFT0001). I am also obsessed with touchpads, and I really like Apple's Trackpads. However, buying a MacBook is even more unjustfiable (although M series chips are pretty intriguing...). Problem is, there is absolutely no market for decent stand alone touch pad input devices out there. So I thought, why not make one myself? 

I impulse bought the touchpad module without much thinking (this was much easier to justify. After all, it's not nearly $1000). After placing my order, I looked up the data sheets and learned that this particular touchpad module uses I2C. Good news is, I have a lot of Arduino and Arduino compatible microcontrollers lying around in my drawer from my custom keyboard phase (everyone goes through one, right?). Though I absoluetly had no I dea how I will connect the part and the microcontroller, but surely somebody out there would have done something neat with the touchpad module. 

Apparently touchpad is really niche! I found exactly "two" posts, [one from reddit](https://www.reddit.com/r/framework/comments/10yy99y/could_i_build_a_stand_alone_touchpad_using_the/) and [another from Framework forum](https://community.frame.work/t/framework-magic-trackpad/19453) about making a stand alone touchpad input device from Framework's touchpad kit. And none of them was step by step guide on how to make one yourself. I did find [a cool PCB made by Arya](https://www.tindie.com/products/crimier/framework-input-cover-controller/) that connects to the entire keyboard module (keyboard + touchpad + fingerprint sensor) using rpi2040 and turns it into a stand alone keyboard. While it was really cool, this was a keyboard and fingerprint sensor more than what I was trying to make.

So I started actually reading through the documentation to figure out what exactly did I sign myself up to. 

Framework's touchpad module has multiple female connectors that connect to Fingerprint sensor and keyboard. It then connects to the motherboard directly via a 51 pin ZIF female connector to 10156001-051100LF male connector. I think this means I can also order a fingerprint module, connect to the touchpad module, somehow figure out the wiring, ????, and profit, I have a touchpad & fingerprint reader. But I have a very sweaty palm and have a lot of trouble with finger print readers, so I'll reserve that for another time. 

For the I2C though, I need 4 connections: SCL (Clock), SDA (Data), Power Supply (5V), and GND (Ground). According to the [Framework official documentation](https://github.com/FrameworkComputer/Framework-Laptop-13/tree/main/Touchpad), each pin is connected to pins 34, 33, 32, and several (17, 21, 38, 15, 16, 25, 27). So the touchpad module requires 5V.

| Pin | Conn | Signal | Notes|
|-----|----- | ------ | ----- |
| 33 | 17 | GND |  |
| 34  | 34   | TP_SCL | Touchpad I2C SCL -- Clock Line|
| 35 | 18 | TP_INT | Touchpad interrupt GPIO | 
| 36 | 33 | TP_SDA | Touchpad I2C SDA -- Data Line | 
| 38 | 32 | 5VS | 5V, Connected to 5VS rail through a 0 ohm resistor |
| 39 | 20 | 5VALW | Fingerprint 5V | 
| 40 | 31 | USB_N | Fingerprint USB | 
| 41 | 21 | GND |  |
| 42 | 20 | USB_P | Fingerprint USB | 
| 37 | 19 | TP_BOARD_ID | |
| 26 | 38 | GND |  |
| 29 | 15 | GND |  |
| 31 | 16 | GND |  |
| 49 | 25 | GND |  | 
| 48 | 27 | FPR_LED_COM | 5V, Fingerprint LED |

Since Arya used rpi2040, I was hoping to finally find some use for my [Raspberry Pi Pico](https://www.raspberrypi.com/products/raspberry-pi-pico/) which I bought for no reason, but alas Raspberry Pi Pico only supports 3.3V. Fortunately though, I had a couple [Elite C](https://deskthority.net/wiki/Elite-C) microcontrollers that do have 5V and all the I2C pins. 

So now I have the touchpad module, I know which pins I need to connect to, and I know which pins on the microcontroller I need to connect my pins to. I have no cluew hat kind of code I would need to get the trackpad signal to cursor movement though. 
The touchpad module did come with a cable that connects to the motherboard. I thought about purchasing [a female breakout board](https://oshpark.com/shared_projects/MjMFKhqM), but the cable was a bit to bulky and would have restricted the final housing a lot. So I decided to ditch the cable it came with, and ordered a [51 pin ZIF breakout board from AliExpress](https://www.aliexpress.us/item/2251832656865723.html). Now it says it will take 2 weeks to arrive, but who knows when I would get it in mail. Once I get the part in mail (I'm thinking next year), I will need to solder the pins and cables. Yay... 

All I can do now is just waiting, and maybe doing some research on the software side of things. 


## 2023/12/18

Parts have arrived.

![parts](images/02_zif_and_breakout.JPG)
- 2 breakout boards for all 51 pins 
- 2 60mm ZIF cables (a little bent, but should be fine)

![pieces1](images/03_all_connected.JPG)
![pieces2](images/04_breakout_and_elite.JPG)

I connected the board and the breakout board together. I'm not really sure how I am supposed to cram all of these into a compact package. 
Once I have the Elite C code set up and confirm the assembly is working, maybe I could design a smaller PCB with just the few pins I need for the touchpad. In the mean time, breadboard would be good enough for testing purposes. 

Will come back after I solder some pins on both the board and Elite C. 

## 2023/12/20
Soldered the parts. Pardon the sloppy solder job!

![soldered](images/05_soldered_dev.jpg)

On a hindsight, I should have soldered the pins on the opposite side for the ZIF breakout board. Good thing I have extra. 

While all four pins "should" be connected correctly, my computer was unable to recognize Elite C when I plugged it via USB cable. Nothing showed up under PORT for any of the IDEs I tried! I think I'm missing some driver because QMK Toolbox could recognize Elite C just fine.

Need to figure out a way to work with ATMega32U4 next. Also my head hurts from all the soldering fume I inhaled earlier. 

## 2023/12/22

Turns out Elite-C is not compatible with Arduino IDE out of the box. Makes sense because this board was made for drop-in replacement for Pro Micro for keyboard folks. 


# 2023/12/24 
After digging around, I learned that it is possible to convert an Arduino-compatible board (like my Elite-C) to be recognized by Arduino IDE by flashing Arduino compatible bootloader. This can be achieved using special boards called programmer, but I could also use another Arduino to flash another board. Arduino IDE has built in example called "Arduino as ISP" that allows one Arduino to serve as programmer for another board (Following this official guide: [Arduino as ISP and Arduino Bootloaders](https://support.arduino.cc/hc/en-us/articles/4408887452434-Flash-USB-to-serial-firmware-in-DFU-mode))

Luckily I had an Arduino Uno from college, so I uploaded Arduino as ISP sketch and wired up my Elite C. After what felt like an eternity, Arduino Uno reported it had finished its job. My Elite C should now be recognized as Arduino Leonardo. Indeed, when I plugged in Elite C, it was immediately available in Arduino IDE. Now I should be able to write Arduino code and upload to my Elite C. 

I did a quick test to see if things are actually working by uploading the classic "Blink" sketch. Upload was seemingly successful, but the blue LED on the board was not blinking. After some frantic research, I found out that Elite-C does not have any onboard LED other than the Power indicator LED. Instead I decided to wire potentiometer and print out the results to the serial port. Thankfully I saw text printed to the console changing values as I turned the dial. Things were looking good.  

While I could have found a way to write code in C and directly upload whatever was compiled to the Elite C, I figured it would be much easier to use Arduino as I have 0 experience with directly interfacing with a microcontroller. 

# 2023/12/30

It's been a rough week, since I wasn't able to get the I2C connection working with the Arduino for a week. I made many mistakes along the way, such as connecting the ZIF cable backwards (!) and adding unnecessary pull up resistors. Using the [I2C scanner sketch](https://playground.arduino.cc/Main/I2cScanner/), I finally figured out the pin connections and the device ID of the touchpad. 

| Pin | Elite C | Breakout board |
| --- | --- | --- |
| SDA | D2 | 36 |
| SCL | D3 | 34 | 
| GND | GND | 33 |
| 5V | VCC | 40 |

And no need for any additional resistors. 

Reported I2C address is: 

```
I2C device found at address 0x2C  ! (44)
I2C device found at address 0x33  ! (51)
done
```

Not sure why there are two addresses reported though. 

Also just in case, the device address on an actual framework laptop is `/devices/pci0000:00/0000:00:15.3/i2c_designware.2/i2c-2/i2c-PIXA3854:00/0018:093A:0274.0004/input/input9`

Got a reading from address 0x2C, getting "37" whenever I  do anything


# 1/5/2024 
New year, but not much progress. Cannot quite figure out how to get a meaningful reading out of the touchpad without official data sheet available from the manufacturer. I did reach out to PixArt through their support portal, but my request for data sheet was declined. They said they just provided the IC, and Framework took  care of everything else such as firmware and the part development. 

Anyways tried dumping some data out of the registers. 

```
passionfruit@raspberrypi:~ $ i2cdetect -y 1
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:                         -- -- -- -- -- -- -- --
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
20: -- -- -- -- -- -- -- -- -- -- -- -- 2c -- -- --
30: -- -- -- 33 -- -- -- -- -- -- -- -- -- -- -- --
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
70: -- -- -- -- -- -- -- --
```

Outputs of i2cdump on each device:

```
passionfruit@raspberrypi:~ $ i2cdump -y 1 0x2c
No size specified (using byte-data access)
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f    0123456789abcdef
00: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
10: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
20: 1e 06 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ??..............
30: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
40: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
50: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
60: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
70: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
80: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
90: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
a0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
b0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
c0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
d0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
e0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
f0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
```

```
passionfruit@raspberrypi:~ $ i2cdump -y 1 0x33
No size specified (using byte-data access)
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f    0123456789abcdef
00: 00 00 00 00 00 00 00 00 00 00 01 00 00 00 1c 18    ..........?...??
10: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
20: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
30: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
40: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
50: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
60: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
70: 09 00 00 00 00 00 00 00 00 00 00 00 00 01 00 06    ?............?.?
80: 00 00 00 00 00 00 00 00 00 00 01 00 00 00 1c 18    ..........?...??
90: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
a0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
b0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
c0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
d0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
e0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
f0: 09 00 00 00 00 00 00 00 00 00 00 00 00 01 00 06    ?............?.?
```

Values all stayed the same whether a finger was touching the pad or not.

```
passionfruit@raspberrypi:/sys/bus/i2c/devices $ i2cdump -y 1 0x33
No size specified (using byte-data access)
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f    0123456789abcdef
00: 00 00 00 00 00 00 00 00 00 00 01 00 00 00 1c 18    ..........?...??
10: 74 16 00 00 00 00 00 00 00 00 00 00 00 00 00 00    t?..............
20: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
30: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
40: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
50: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
60: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
70: c1 02 00 00 00 00 4d 01 00 00 00 00 00 01 00 06    ??....M?.....?.?
80: 00 00 00 00 00 00 00 00 00 00 01 00 00 00 1c 18    ..........?...??
90: 74 16 00 00 00 00 00 00 00 00 00 00 00 00 00 00    t?..............
a0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
b0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
c0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
d0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
e0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
f0: c1 02 00 00 00 00 4d 01 00 00 00 00 00 01 00 06    ??....M?.....?.?
```

I noticed some changes in the values within the dump, but honestly I'm not sure how to proceed further. I did order the framework input cover thing mentioned earlier, so I will try and see if that leads me anywhere. At this point my only option seems to be  reverse engineering the thing. 


# 3/1/2024
Brought the set up to 2600 meet up, and got some help on how to dump things out of the register and format so that it prints out pretty. But we couldn't quite figure out how to get actual data read out from either of the registers. 

Still stuck?  

# 4/20/2024
Was scrolling hackaday and found an article titled [Human-Interfacing devices: HID over I2C](https://hackaday.com/2024/04/17/human-interfacing-devices-hid-over-i2c/). From 3 days ago. Clicked on it, and it's written by no one other than Arya, and contained whole lot of information on how to work with HID over I2C device. It turns out, I got the device ID 0x2c correct. All I had to do was read blocks out of the register 0x20 on this device on 0x2c. This was not possible through Raspberry Pi 3 I was using, as block read is not supported.

```
$ i2cdetect -F 1
Functionalities implemented by /dev/i2c-1:
I2C                              yes
SMBus Quick Command              yes
SMBus Send Byte                  yes
SMBus Receive Byte               yes
SMBus Write Byte                 yes
SMBus Read Byte                  yes
SMBus Write Word                 yes
SMBus Read Word                  yes
SMBus Process Call               yes
SMBus Block Write                yes
SMBus Block Read                 no
SMBus Block Process Call         no
SMBus PEC                        yes
I2C Block Write                  yes
I2C Block Read                   yes
```

Arya was using Circuit Python in the article, so I brought out Raspberry Pi Pico and installed Circuit Python. Only catch was that Pi Pico is 3.3V only, so I figured I could power the touchpad using the 5V pins from Raspberry Pi 3. 

However, when I loaded i2c, Circuit Python complained about missing pull up resistors.

```
import block
i2c = block.I2C() # This has to be something else, but forgot to write it down. 
```

That meant I had to use the power from the Raspberry Pi Pico, and then connect SDA/SCL with the power using resistors. Would touchpad work with 3.3V though? 

So I just pulled out the Elite C and wired out the breakout board, and ran the old script I had, which read a byte from 0x20 on I2c device 0x2c which always returns 37. 

Then it occurred to me, what if I read the entire block from 0x20 like Arya was doing? So I modified the script a little, and I actually was able to read meaningful data out of the touchpad.  

```
#include <Wire.h>  

#define ADDR 0x2C

void setup() {
  Wire.begin();  
  Serial.begin(115200);
  Serial.print("init");
}


void loop() {

  Wire.requestFrom(ADDR, 16);
  while(Wire.available()){
    char strBuf[4];
    byte c = Wire.read();
    sprintf(strBuf, "%02x", c);
    Serial.print(strBuf);
    Serial.print(" ");
  }

  Serial.print("\n");
  
  delay(1000); 
}
```

Some sample output
```
25 00 01 03 00 23 02 b4 01 00 00 00 00 00 00 00 
25 00 01 03 00 24 02 b4 01 00 00 00 00 00 00 00 
25 00 01 03 00 28 02 b4 01 03 01 48 03 dd 01 00 
25 00 01 03 00 2e 02 b4 01 03 01 49 03 dc 01 00 
25 00 01 03 00 35 02 b5 01 03 01 4c 03 dc 01 00 
25 00 01 03 00 fd 01 a7 01 03 01 07 03 dd 01 00 
25 00 01 03 00 47 02 8a 01 03 01 42 03 ec 01 00 
25 00 01 03 00 63 02 99 01 03 01 51 03 09 02 00 
25 00 01 03 00 1c 02 a8 01 03 01 0a 03 12 02 00 
25 00 01 03 00 e4 01 e7 01 03 01 0e 03 4d 02 00 
25 00 01 03 00 b4 01 0c 02 03 01 2b 03 5b 02 03 
00 00 01 01 00 f3 02 27 01 00 00 00 00 00 00 00 
00 00 01 01 00 f3 02 27 01 00 00 00 00 00 00 00 
00 00 01 01 00 f3 02 27 01 00 00 00 00 00 00 00 
00 00 01 01 00 f3 02 27 01 00 00 00 00 00 00 00 
00 00 01 01 00 f3 02 27 01 00 00 00 00 00 00 00 
00 00 01 01 00 f3 02 27 01 00 00 00 00 00 00 00 
```

Obviously, `00 00 01 01 00 f3 02 27 01 00 00 00 00 00 00 00` is the "idle" state.

I would need to read more on the spec, but it looks like these values are the HID descriptor, so I should be able to further decode what this byte soup means. Regardless, a huge progress. 


## Single finger
```
25 00 01 03 00 a1 01 95 01 03 01 bb 02 e5 01 03 
25 00 01 03 00 a5 01 8d 01 03 01 f1 02 ba 01 03 
25 00 01 03 00 de 01 63 01 03 01 31 03 93 01 03 
25 00 01 01 00 de 01 63 01 01 01 31 03 93 01 01 
00 00 01 01 00 de 01 63 01 01 01 31 03 93 01 01 
00 00 01 01 00 de 01 63 01 01 01 31 03 93 01 01 
25 00 01 03 00 a3 02 92 00 00 00 00 00 00 00 00 
25 00 01 03 00 a3 02 92 00 00 00 00 00 00 00 00 
25 00 01 03 00 a3 02 92 00 00 00 00 00 00 00 00 
25 00 01 03 00 a3 02 92 00 00 00 00 00 00 00 00 
25 00 01 03 00 a3 02 a4 00 00 00 00 00 00 00 00 
25 00 01 03 00 a9 02 3f 01 00 00 00 00 00 00 00 
25 00 01 03 00 b5 02 f8 01 00 00 00 00 00 00 00 
25 00 01 01 00 b5 02 f8 01 00 00 00 00 00 00 00 
25 00 01 03 01 ea 02 0d 00 00 00 00 00 00 00 00 
25 00 01 03 01 0e 03 12 01 00 00 00 00 00 00 00 
25 00 01 01 01 0e 03 12 01 00 00 00 00 00 00 00 
25 00 01 03 00 43 03 e6 00 00 00 00 00 00 00 00 
25 00 01 01 00 43 03 e6 00 00 00 00 00 00 00 00 
25 00 01 03 01 e2 03 01 00 00 00 00 00 00 00 00 
25 00 01 03 01 e1 03 f0 00 00 00 00 00 00 00 00 
25 00 01 01 01 e1 03 f0 00 00 00 00 00 00 00 00 
25 00 01 03 00 84 03 2c 00 00 00 00 00 00 00 00
```

## Two fingers
```
00 00 01 01 00 7f 03 9c 01 00 00 00 00 00 00 00 
00 00 01 01 00 7f 03 9c 01 00 00 00 00 00 00 00 
25 00 01 03 00 51 02 4d 00 00 00 00 00 00 00 00 
25 00 01 03 00 51 02 4d 00 00 00 00 00 00 00 00 
25 00 01 03 00 51 02 4d 00 00 00 00 00 00 00 00 
25 00 01 03 00 51 02 4d 00 03 01 71 03 94 00 00 
25 00 01 03 00 67 02 a0 00 03 01 87 03 e9 00 00 
25 00 01 01 00 67 02 a0 00 01 01 87 03 e9 00 00 
25 00 01 03 02 3b 02 10 00 03 03 c1 03 39 00 00 
25 00 01 03 02 69 02 fd 00 03 03 d1 03 3b 01 00 
25 00 01 01 02 69 02 fd 00 01 03 d1 03 3b 01 00 
25 00 01 03 00 1d 02 6f 02 03 01 46 03 cd 02 00 
25 00 01 03 00 35 02 e9 01 03 01 70 03 3a 02 00 
25 00 01 03 00 22 02 42 01 01 01 70 03 3a 02 00 
25 00 01 01 00 22 02 42 01 00 00 00 00 00 00 00 
25 00 01 03 00 00 02 ae 00 03 01 70 03 dd 00 00 
25 00 01 03 00 00 02 af 00 03 01 70 03 de 00 00 
```

## Three fingers 
```
00 00 01 01 00 03 02 b7 00 01 01 71 03 e5 00 00 
00 00 01 01 00 03 02 b7 00 01 01 71 03 e5 00 00 
25 00 01 03 00 c0 02 0a 00 03 01 9a 01 76 00 00 
25 00 01 03 00 c0 02 0a 00 03 01 9a 01 76 00 03 
25 00 01 03 00 c0 02 0a 00 03 01 9a 01 76 00 03 
25 00 01 03 00 c0 02 0a 00 03 01 9a 01 76 00 03 
25 00 01 01 00 c0 02 0a 00 01 01 9a 01 76 00 01 
25 00 01 03 00 cf 00 99 00 00 00 00 00 00 00 00 
25 00 01 03 00 cf 00 99 00 00 00 00 00 00 00 00 
25 00 01 03 00 cf 00 99 00 03 01 e8 01 0a 00 00 
25 00 01 03 00 cf 00 99 00 03 01 e8 01 0a 00 00 
25 00 01 01 00 cf 00 99 00 01 01 e8 01 0a 00 00 
25 00 01 03 00 d4 01 d9 00 00 00 00 00 00 00 00 
25 00 01 03 00 d4 01 d9 00 03 01 54 03 30 01 03 
25 00 01 03 00 d4 01 d9 00 03 01 54 03 30 01 03 
25 00 01 03 00 d4 01 d9 00 03 01 54 03 30 01 03 
25 00 01 01 00 d4 01 d9 00 01 01 54 03 30 01 01 
00 00 01 01 00 d4 01 d9 00 01 01 54 03 30 01 01 
00 00 01 01 00 d4 01 d9 00 01 01 54 03 30 01 01 
00 00 01 01 00 d4 01 d9 00 01 01 54 03 30 01 01
```

Apparently something is not right with how I'm printing the bytes. Should tweak the code until I get the right HID descriptor out.


# 10/2/2024 

| Name | FT 260 | Dev board | Touchpad | 
|---|---|---|---|
| SCL | DIO5 | IO0 | 34 | 
| SDA | DIO6 | IO1 | 36 |
| INT | DIO8 | IO3 | 35 |
| 5V | - | - | 40 | 
| GND | - | - | 33 |
