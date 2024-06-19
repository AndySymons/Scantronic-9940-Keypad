# Project KELLY - Integrate Scantronic 994x Keypads into ESPHome and Home Assistant


## Description of the Scantronic 9940 and 9941 Keypads 

_See also the [Installation Guide](https://www.alertelectrical.com/media/productattachment/0/4429/42999419940rkp.pdf)_

The unit measures 115 x 115 x 24 deep. The lower half is s flap that can be lowered to access the keypad. 

![Screenshot 2024-06-19 at 10 21 45](https://github.com/AndySymons/KELLY-Scantronic-9940-Keypad/assets/14819812/eea9b076-7bbd-4eea-86d1-26fb86a365fe) ![9941EN](https://github.com/AndySymons/KELLY-Scantronic-9940-Keypad/assets/14819812/0adfe867-deed-4dde-827a-0b281ed732ec)

It comprises:  
- Three indicator LEDs top left with icons: exclamation (user alert), spanner (hardware fault), and wavy line (power on, flashes for battery backup)
- A 16 x 2 LCD display with backlight
- Four buttons labelled A, B, C, and D
- Under the flap, a 12-button keypad: numbers 0 to 9 plus tick (enter) and cross (exclude zones). It has a backlight. 
- A tamper microswitch on the back that switches if the unit is removed from the wall
- A sounder inside  
- A tag reader inside the 9940 model

Inside it has connections (from right to left seen from the back): 
- Four to connect to the controller:  "OV, 12V, CLK, DATA"
- A pair "ET" = Exit Terminate (e.g. a switch that when closed confirms alarm arming) 
- A pair "Ext tamper" - for an external tamper switch (bridged if not used)
- A pair "Panic" - for a panic button (bridged if not used) 

Scantronic recommends connection with normal alarm cable. The maximum distance between any kepad and the controller is 200m

## The plan 

1. Build the necesary hardware to connect the keypad to an ESP32 microprocessor (running ESPHome) 
1. Reverse-engineer the protocol between the alarm controller and the keypad using samples of each
1. Code the protocol into an ESPHome Custom Device
2. Build whatever further integrations are necessary to make the Keypad appear to Home Assistant as
    - an alarm keypad compatible with the native [Manual Alarm](https://www.home-assistant.io/integrations/manual/) and with the [Alarmo add-on](https://github.com/nielsfaber/alarmo), if possible including A, B, C, D as buttons for selecting the arming level 
    - an LCD display
    - a [tag reader](https://www.home-assistant.io/integrations/tag/) (model 9440 only)
    - The three LEDs and the backlights as lights
    - the tamper microswitch, panic button, external tamper and exit terminate switches as binary sensors  
    
The kepad could be used standalone or as part of a Scantronic 9651 hardwired alarm replacement -- see [Project HARVIE](https://github.com/AndySymons/HARVIE-Hardwired-Alarm-Replacement-Board)

It is a part-time project, so could take a while; but watch this space, and please share any info you may have on this subject :-)  


## Status at 19 June 2024 

### 1 Line driver hardware  
I am building two line drivers with P82B96 bus interface chips, which can make the voltage conversion between Vcc (up to 15V) and the 3V3 logiv levels used by ESP32 microprocessors. 

My circuits are based on [Philips application note AN460](https://www.mikrocontroller.net/attachment/13528/AN460_P82B96.pdf) figs 4 and 7. 

![Screenshot 2024-06-19 at 09 42 23](https://github.com/AndySymons/KELLY-Scantronic-9940-Keypad/assets/14819812/3c41f50c-e383-47c4-8a7e-f6d191b13f38)

![Screenshot 2024-06-19 at 09 42 39](https://github.com/AndySymons/KELLY-Scantronic-9940-Keypad/assets/14819812/53ac75d7-fdbf-422f-aca3-f5815c1d8416)

For optocouplers I am using the PC817, which is adequate for this low frequency application. 

Either line driver circuit can be used in the final solution -- I will make up my mind after playing with them for a while. the optocoupler solution appears more robust, but takes up more board space and is not strictly necessary because the keypad is powered by the controller, not a separate supply, so Vcc and Ground are shared anyway. 

### 2 Protocol reverse engineering 

With an oscilloscope I can see that Scantronic 9940 Keypad uses a serial protocol a little like I2C but without the ACK bit and running at just 2kHz. It runs at Vcc voltage levels -- nominally 12V but more like 14V on my sample alarm controller using the original power supply (that makes sense because there is a 12V backup battery, which would go up to about 13.8V). 

To reverse engineer the protocol, I plan to connect the two line drivers back to back between the sample control and keypad. The one with optocouplers (fig 4) separates Tx and Rx, so I can see traffic in both directions. I will then use four channels of a cheap logic analyser to get more info on the protocol in both directions. I do not know yet, for example, whether the master or the slave provides the clock signal for keypad responses.

### 3 Custom device 
I read up on how to create a custom devices and set up a development environment by clining the I2C code by way of example to get me started. 
That is all so far! 

### 4 Integration with Home Assistant
Nothing to report yet. 

---
If you find the ideas in this repository useful, please [Buy Me a Coffee](https://buymeacoffee.com/andysymons)

