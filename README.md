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