# Team Members
Veronika Maragulova, Sanket Nadgauda, Vinay Balamourougan, Dakota Survance 

# Bluetooth Lock System
This project includes a prototype of the remote smart door lock. To open the door with this lock, the user would have to know the correct password. If the combination is correct, the LED turns from red to green, and the lock opens up. The LCD screen shows the status of the lock and the current pin entry. In this case, it will go from "locked" to "opened." The camera captures the picture of the user and sends it via email for extra security measures. If the password combination is incorrect, then LCD screen shows "locked" message and the LED stays red.

# Getting Started
These procedures will help you to set up and to run the same project.

# Parts

* 2 Breadboards

<img width="412" alt="Screen Shot 2021-12-03 at 1 56 46 PM" src="https://user-images.githubusercontent.com/87663736/144657700-1cc046e4-9774-4fe5-8b36-40d7c3562623.png">

* Male to Male Jumper Wires 

<img width="412" alt="Screen Shot 2021-12-03 at 1 58 41 PM" src="https://user-images.githubusercontent.com/87663736/144657968-b2be31a6-5b70-4818-b1e9-f3bb93f4fdde.png">


* uLCD-144-G2 128 by 128 Smart Color LCD

<img width="412" alt="Screen Shot 2021-12-03 at 1 35 11 PM" src="https://user-images.githubusercontent.com/87663736/144654948-4e799058-f7ff-4010-a38e-5fe0fa8dddae.png">


* RGB LED

<img width="412" alt="Screen Shot 2021-12-03 at 1 36 05 PM" src="https://user-images.githubusercontent.com/87663736/144655072-f4712c32-0e90-4163-86ce-bd211492e435.png">

* 2 Battery Packs + 6 AA batteries

<img width="412" alt="Screen Shot 2021-12-03 at 1 50 47 PM" src="https://user-images.githubusercontent.com/87663736/144656896-219577b5-89bd-4eda-bc4e-ee393009d142.png">

* RC Servo

<img width="412" alt="Screen Shot 2021-12-03 at 1 48 51 PM" src="https://user-images.githubusercontent.com/87663736/144656670-5152a530-9254-4be3-bcb4-215fc030b8e9.png">

* Adafruit Bluetooth LE UART Friend module

<img width="412" alt="Screen Shot 2021-12-03 at 1 39 41 PM" src="https://user-images.githubusercontent.com/87663736/144655538-829f3ac9-ea62-430d-914a-61b53b4a2cb0.png">


* Raspberry Pi NoIR camera V2

<img width="412" alt="Screen Shot 2021-12-03 at 1 42 07 PM" src="https://user-images.githubusercontent.com/87663736/144655860-717c1974-c8e7-43d2-bcff-8c6efe0c4836.png">


* Raspberry Pi Zero

<img width="412" alt="Screen Shot 2021-12-03 at 1 43 11 PM" src="https://user-images.githubusercontent.com/87663736/144655982-5e7eefcf-4ab9-49a5-b8db-511358620e21.png">

# Hardware Set Up
Below is a table of all the pin outs and wiring details necessary for assembly.

![pin_out_table](https://user-images.githubusercontent.com/82181571/145150355-7e5efb71-6848-43cf-b197-86b43982bf08.jpg)

![led](https://user-images.githubusercontent.com/82181571/145149939-0ffdc11d-b527-4bf1-85b7-6b114f4b1d82.jpg)

A more visual depiction is included with the schematic below:

![0001](https://user-images.githubusercontent.com/82181571/145151013-cd85bb54-9ffe-4226-8307-74941b85d096.jpg)

# Software
The program on the mbed consisted of C++ code and 2 additional libraries (4DGL-uLCD-SE, Servo) for the peripherals. 

The code uses two string variables to keep track of the current pin entered and the desired pin. There is also a boolean flag to keep track of the status of the lock. This flag is used to keep the status printed onto the LCD screen. The currently entered pin is also printed to the LCD screen. 

There is also a switch statement to read inputs from the bluetooth module. The controller pad in the Bluefruit Connect app is used to control the mechanism. The numbers one through 4 are used for entering the pin. The up button checks to see if the pin entered is correct. The entered pin then clears. If the pin is correct, the LED turns from red to green. The right button closes the lock and clears the entered pin if it is not already cleared. The down button simply clears the entered pin. 

Here is the full code file:
```cpp
#include "mbed.h"
#include "rtos.h"
#include "Servo.h"
#include "uLCD_4DGL.h"
#include <string>

uLCD_4DGL uLCD(p9,p10,p11); // serial tx, serial rx, reset pin;
Serial blue(p28,p27);
PwmOut red(p22);
PwmOut green(p23);
Servo myservo(p24);
DigitalOut raspberryConnect(p12);

int main()
{
    myservo = 0.0;
    wait(0.5);
    myservo = 1.0;
    wait(0.5);
    std::string pinCorrect = "1234";
    std::string pin = "";
    
    char bnum=0;
    char bhit=0;
    
    bool open=1;
    
    red = 1;
    
    uLCD.cls();
    uLCD.baudrate(3000000);
    while(1) {
        uLCD.text_width(2);
        uLCD.text_height(2); 
        uLCD.locate(1,2);
        uLCD.color(WHITE);
        uLCD.printf("%s\n", pin);
        if (open) {
            uLCD.text_width(2);
            uLCD.text_height(2); 
            uLCD.locate(1,4);
            uLCD.color(0x00FF00);
            uLCD.printf("Status : Opened\n");
            wait(0.5);
        } else {
            uLCD.text_width(2);
            uLCD.text_height(2); 
            uLCD.locate(1,4);
            uLCD.color(0xFF0000);
            uLCD.printf("Status : Closed\n");
            wait(0.5);
        }
        raspberryConnect = 0;
        if (blue.getc()=='!') {
            if (blue.getc()=='B') { //button data packet
                bnum = blue.getc(); //button number
                bhit = blue.getc(); //1=hit, 0=release
                if (blue.getc()==char(~('!' + 'B' + bnum + bhit))) { //checksum OK?
                    switch (bnum) {
                        case '1': //number button 1
                            if (bhit=='1') {
                                pin.append("1");
                            } else {
                                //add release code here
                            }
                            break;
                        case '2': //number button 2
                            if (bhit=='1') {
                                pin.append("2");
                            } else {
                                //add release code here
                            }
                            break;
                        case '3': //number button 3
                            if (bhit=='1') {
                                pin.append("3");
                            } else {
                                //add release code here
                            }
                            break;
                        case '4': //number button 4
                            if (bhit=='1') {
                                pin.append("4");
                            } else {
                                //add release code here
                            }
                            break;
                        case '5': //button 5 up arrow (check the pin entry)
                            if (bhit=='1') {
                                if (open==1) {
                                    pin = "";
                                } else {
                                    if (pin == pinCorrect){
                                        pin = ""; // reset pin entry when wrong
                                        raspberryConnect = 1;
                                        wait(.5);
                                        raspberryConnect = 0;
                                        green = 1;
                                        red = 0;
                                        open = 1;
                                        myservo = 1.0;
                                        wait(1);
                                        uLCD.cls();
                                    } else{
                                        pin = ""; // reset pin entry when wrong
                                        red = 1;
                                        green = 0;
                                    } 
                                }
                            } else {
                                //add release code here
                            }
                            break;
                        case '6': //button 6 down arrow (reset the pin entry)
                            if (bhit=='1') {
                                uLCD.cls();
                                pin = ""; //add hit code here
                            } else {
                                //add release code here
                            }
                            break;
                        case '8': //button 8 right arrow (close lock)
                            if (bhit=='1') {
                                 red = 1;
                                 green = 0;
                                 open = 0;
                                 myservo = 0.25; //add hit code here
                                 wait(0.5);
                            } else {
                                //add release code here
                            }
                            break;
                        default:
                            break;
                    }
                }
            }
        }
    }
}

```
# Tests

# Demo

