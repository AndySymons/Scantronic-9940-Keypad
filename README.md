# Project KELLY - Integrate Scantronic 994x Keypads into ESPHome and Home Assistant

## Objectives 

The objective of this project is to be able to use the Scantronic 994x Keypads in an ESPHome and Home Assistant environment. One or more keypads could be used independently or in conjunction with the Scantronic 9651 hardwired interface -- [project HARVIE](https://github.com/AndySymons/HARVIE-Hardwired-Alarm-Replacement-Board). 

The first phase of the project is to reverse engineer the data communication protocol -- I have no specification or circuit schematic! 
The second phase will be to build the necessary hardware and software components for the ESPHome and Hoem Assistant environment, provisionally expected to include: 
1. A hardware bus interface / line driver to connect the keypad(s) to an ESP32 procesessor
2. An ESPHome Custom Device
2. Integrations that make the Keypad appear to Home Assistant as
    - an alarm keypad compatible with the native [Manual Alarm](https://www.home-assistant.io/integrations/manual/) and with the [Alarmo add-on](https://github.com/nielsfaber/alarmo), if possible including A, B, C, D as buttons for selecting the arming level 
    - an LCD display
    - a [tag reader](https://www.home-assistant.io/integrations/tag/) (model 9440 only)
    - The three LEDs and the backlights as lights
    - the tamper microswitch, panic button, external tamper and exit terminate switches as binary sensors  
    
It is a part-time project, so could take a while; but watch this space, and please share any info you may have on this subject :-)  


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

## PHASE 1 -- reverse-engineering the communications protocol -- Steps and status at 21 September 2024

#### Ph1 Step 1: Data Comm Physical Layer -- Inital inspection (COMPLETED)
With an oscilloscope I can see that Scantronic 9940 Keypad uses a serial protocol a little like I2C but without the ACK bit and running at just 2kHz. It runs at Vcc voltage levels -- nominally 12V but more like 14V on my sample alarm controller using the original power supply (that makes sense because there is a 12V backup battery, which would go up to about 13.8V). 

To reverse engineer the protocol, I plan to connect the two line drivers back to back between the sample control and keypad. The one with optocouplers (fig 4) separates Tx and Rx, so I can see traffic in both directions. I will then use four channels of a cheap logic analyser to get more info on the protocol in both directions. I do not know yet, for example, whether the master or the slave provides the clock signal for keypad responses.

#### Ph1 Step 2: Data Comm Physical Layer -- Bus interface chip options (COMPLETED)
The initial inspection indicates that I will need a bus interface to convert between 3V3 logic levels and the 12-15V required for the long line. 
there are several around, but the one that best meets the present need is the **P82B96 dual bus interface**. One chip has two channels (clock and data) with both directions (Tx and Rx). It works up to 18V and down to 2V. The oututs are open-collector, so can be pulled up to any voltage within the permitted range.
To prevent an output being fed back as an input, and thus locking the bus, non-standard logic levels are used. In brief, the output low level at Sx or Sy is higher than the input low level -- so an output from the chip is not seen as an input, though it will be recognised by 3V3 processors. For full details see the [datasheet](https://www.ti.com/lit/ds/symlink/p82b96.pdf?ts=1726919386924&ref_url=https%253A%252F%252Fwww.google.com%252F).  


#### Ph1 Step 3: Data Comm Physical Layer -- Bus interface -- P82B96 
I built two experimental line drivers with P82B96 bus interface chips, both based on the [Philips application note AN460](https://www.mikrocontroller.net/attachment/13528/AN460_P82B96.pdf) figs 4 and 7. 
1 Simple version (COMPLETED)
THe first is a simple version with only clamping and flyback diodes added. 
![Screenshot 2024-06-19 at 09 42 23](https://github.com/AndySymons/KELLY-Scantronic-9940-Keypad/assets/14819812/3c41f50c-e383-47c4-8a7e-f6d191b13f38)
It works fine. 

2 Optocoupler version (COMPLETED)
I also buit a version with optocouplers.  
![Screenshot 2024-06-19 at 09 42 39](https://github.com/AndySymons/KELLY-Scantronic-9940-Keypad/assets/14819812/53ac75d7-fdbf-422f-aca3-f5815c1d8416)
For optocouplers I am using the PC817, which is adequate for this low frequency application. 

Either bus interface  circuit can be used in the final solution -- I will make up my mind after playing with them for a while. 
The optocoupler solution appears more robust, but takes up more board space and is not strictly necessary because the keypad is powered by the controller, not a separate supply, so Vcc and Ground are shared anyway. 
    

#### Ph1 Step 5: Data Comm Physical Layer -- Line intercept circuit -- (COMPLETED)
In order to analyse the data comm protocol in detail I need to be able to separate the Tx and Rx directions of both channels, clock and data.   
My first idea was to connect two line drivers back-to-back and have one of them be the optocoupler version with separate Tx and Rx. This failed becasue when the line drivers are connected back-to-back, the special logic levels teh prevent feedback and bus lockup also ensure that the signal output of one (Sx, Sy) is not recognised as input by the other (Sx, Sy) -- so signals are not propagated. 
I then tried a modified version of the optocoupler version of the bus interface in which the Sx and Sy work at Vcc = 12-15V. This proves that Sx and Sy can work at 15V, but altough is apparently separated Tx and Rx for each channel, it does not prevent the output being recognised as input, so is of limited use. 

The design finally adopted is a special combination of two P82B96 chips connected one after the other. Inputs and outputs use Sx and Sy, as well as Tx and Ty at Vcc=15V. I assembled these on one board complete with pullup resistors, voltage dividers to provide 3V3 signal levels to the logic analyzer for four channels -- Clock Tx and Rx, Data Tx and Rx -- and a 10-pin box header that is used to connect dirctly to the logic analyzer interface without fidly probes etc. I also included plenty of Test Point eyelets to easily connect an osclloscope or voltmeter. 

#### Ph1 Step 6: Data Comm Link Layer -- Initial logic analyzer (COMPLETED)
I use a generic unbranded Chinese 8-channel logic analyzer interface together with the Sigrok Pulseview (free) software running on an iMac. 
I use channels 0, 2, 4 and 6 simply because these are the easiet to wire up on a prototype board. (Ch 1 is linked to Ch 0, 3 to 2, 5 to 4, and 7 to 6 for the same reason, but the odd channels are not used). 

Initial results confirm the first oscilloscope tests. Data is transmitted in 8-bit packets with no ACK. Lines are normally high. The data line is pulled down to start a new package. 
The clock always copme from the controller (there is no sgnal on clock Rx). 

#### Ph1 Step 7: Data Comm Link Layer -- Logic analyzer custom decode (NEXT UP)
With the setup of the previous step, I tried out some of the decoders provided with Pulseview, such as I2C, PS/2 and UART -- but the results confirmed my visual impression that the Scantronic 994x protocol is none of these. 
To make sense of the content of the transmissions, I propose to build a custom decoder. 
I do not yet know whether LOW represents 1 or 0, and I do not know whether the lowest or highest significant bit is transmitted first -- I will tehrefore need to decode all four combinations to determine which makes the most sense!  

#### Ph1 Step 8: Data Comm Application Layer -- Decodng the commands and responses to/from the keypad (FUTURE PLAN) 
T.b.d. 

## PHASE 2 -- building the hardware and software components 
To be planned. Not yet started. 


---
If you find any of the ideas in this repository useful, please [Buy Me a Coffee](https://buymeacoffee.com/andysymons)

