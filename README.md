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

Since Arya used rpi2040, I was hoping to finally find some use for my [Raspberry Pi Pico](https://www.raspberrypi.com/products/raspberry-pi-pico/) which I bought for no reason, but alas Raspberry Pi Pico only supports 3.3V. Fortunately though, I had a couple [Elite C](https://deskthority.net/wiki/Arduino_Pro_Micro) microcontrollers that do have 5V and all the I2C pins. 

So now I have the touchpad module, I know which pins I need to connect to, and I know which pins on the microcontroller I need to connect my pins to. I have no cluew hat kind of code I would need to get the trackpad signal to cursor movement though. 
The touchpad module did come with a cable that connects to the motherboard. I thought about purchasing [a female breakout board](https://oshpark.com/shared_projects/MjMFKhqM), but the cable was a bit to bulky and would have restricted the final housing a lot. So I decided to ditch the cable it came with, and ordered a [51 pin ZIF breakout board from AliExpress](https://www.aliexpress.us/item/2251832656865723.html). Now it says it will take 2 weeks to arrive, but who knows when I would get it in mail. Once I get the part in mail (I'm thinking next year), I will need to solder the pins and cables. Yay... 

All I can do now is just waiting, and maybe doing some research on the software side of things. 


## 2023/12/18

Parts have arrived.

![parts](images/02_zif_and_breakout.JPG)
- 2 breakout boards for all 51 pins 
- 2 60mm ZIF cables (a little bent, but should be fine)

![pieces](images/03_all_connected.JPG)
![pieces](images/04_breakout_and_elite.JPG)

I connected the board and the breakout board together. I'm not really sure how I am supposed to cram all of these into a compact package. 
Once I have the Elite C code set up and confirm the assembly is working, maybe I could design a smaller PCB with just the few pins I need for the touchpad. In the mean time, breadboard would be good enough for testing purposes. 

Will come back after I solder some pins on both the board and Elite C. 