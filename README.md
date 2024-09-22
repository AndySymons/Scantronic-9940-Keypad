# Project KELLY - Integrate Scantronic 994x Keypads into ESPHome and Home Assistant

## Objectives 

The objective of this project is to be able to use the Scantronic 994x Keypads in an ESPHome and Home Assistant environment. One or more keypads could be used independently or in conjunction with the Scantronic 9651 hardwired interface -- [project HARVIE](https://github.com/AndySymons/HARVIE-Hardwired-Alarm-Replacement-Board). 

I have no specification or circuit schematic, so the first phase of the project is to reverse engineer the data communication protocol. 
The second phase will be to build the necessary hardware and software components for the ESPHome and Home Assistant environment, provisionally expected to include: 
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

## PHASE 1: reverse-engineering the communications protocol - steps and status at 21 September 2024

#### Ph1 Step 1: Data Comm Physical Layer -- Inital inspection (COMPLETED)
With an oscilloscope I can see that Scantronic 9940 Keypad uses a serial protocol a little like I2C but without the ACK bit and running at just 2kHz. It runs at Vcc voltage levels -- nominally 12V but more like 14V on my sample alarm controller using the original power supply (that makes sense because there is a 12V backup battery, which would go up to about 13.8V). 

To reverse engineer the protocol, I plan to connect the two line drivers back to back between the sample control and keypad. The one with optocouplers (fig 4) separates Tx and Rx, so I can see traffic in both directions. I will then use four channels of a cheap logic analyser to get more info on the protocol in both directions. I do not know yet, for example, whether the master or the slave provides the clock signal for keypad responses.

#### Ph1 Step 2: Bus interface chip options (COMPLETED)
The initial inspection indicates that I will need a bus interface to convert between 3V3 logic levels and the 12-15V required for the long line. 
there are several around, but the one that best meets the present need is the **P82B96 dual bus interface**. One chip has two channels (clock and data) with both directions (Tx and Rx). It works up to 18V and down to 2V. The oututs are open-collector, so can be pulled up to any voltage within the permitted range.
To prevent an output being fed back as an input, and thus locking the bus, non-standard logic levels are used. In brief, the output low level at Sx or Sy is higher than the input low level -- so an output from the chip is not seen as an input, though it will be recognised by 3V3 processors. For full details see the [datasheet](https://www.ti.com/lit/ds/symlink/p82b96.pdf?ts=1726919386924&ref_url=https%253A%252F%252Fwww.google.com%252F).  

#### Ph1 Step 3: Prototype bus interfaces 
I built two experimental line drivers with P82B96 bus interface chips, both based on the [Philips application note AN460](https://www.mikrocontroller.net/attachment/13528/AN460_P82B96.pdf).

##### 1 Simple version (COMPLETED)
The first is a simple version with only clamping and flyback diodes added. 

![Screenshot 2024-09-22 at 15 47 51](https://github.com/user-attachments/assets/20bc9726-3a80-4c4e-b56a-5b894c37c9b1)

The circuit is based on the [Philips application note AN460](https://www.mikrocontroller.net/attachment/13528/AN460_P82B96.pdf) fig 7, for a 15 volt Vcc. I do not include any 3V3 pullup resistors because I assume that I will use a microprocessor with internal pullup resistors.

I built one on a prototype board for testing... 

![KELLY simple line driver prototype](https://github.com/user-attachments/assets/3f0cc24d-a72e-408e-a4e8-a658448bc983)

I cannot use it in earnest until I know more about the data protocol, so to test it, I simply connected it in parallel with the Scantronic 9651 controller and 9940 keypad. 
In place of a microprocessor, I connected the driver to a breadboard wth a 3V3 power supply and 10k pullup resistors. I connected those signals to an oscilloscope.   

![KELLY simple test driver test setup](https://github.com/user-attachments/assets/83a461d6-6636-41be-a8c3-2052d79b886f)

It worked fine. 

##### 2 Optocoupler version (COMPLETED)
I also built a version with optocouplers.  

![Screenshot 2024-09-22 at 16 00 40](https://github.com/user-attachments/assets/247ba720-902e-41af-a131-525ad92fbd4c)

The circuit is based on the [Philips application note AN460](https://www.mikrocontroller.net/attachment/13528/AN460_P82B96.pdf) fig 4, adapted to a 15 volt Vcc. I do not include 3V3 pullup resistors because I assume that I will use a microprocessor with internal pullup resistors. 

For optocouplers I used four PC817s (in one 16-pin DIP socket). These are adequate for this low frequency application. 
* Note: I do not know why R5 is connected to R4 in the application note, and it does not explain. I made a second variant with all pullups connected directly to Vcc and that works fine. 

![KELLY](https://github.com/user-attachments/assets/977f0181-25e2-4ec7-9935-31a9c2f2fc13)

Either bus interface  circuit can be used in the final solution -- I will make up my mind after playing with them for a while. 
The optocoupler solution appears more robust, but takes up more board space and decoupling is not necessary because the keypad is powered by the controller, so Vcc and Ground are shared anyway.


#### Ph1 Step 5: Line intercept circuit (COMPLETED)
In order to analyse the data comm protocol in detail I need to be able to separate the Tx and Rx directions of both channels, clock and data.   
My first idea was to connect two line drivers back-to-back and have one of them be the optocoupler version with separate Tx and Rx. This failed because 
- when the line drivers are connected back-to-back, the special logic levels that prevent feedback and bus lockup also ensure that the signal output of one (Sx, Sy) is not recognised as input by the other (Sx, Sy) -- so signals are not propagated.
- altough Tx and Rx apparently separated, this circuit does not prevent the output being recognised as input, so is of limited use. 

The design finally adopted is a special combination of two P82B96 chips connected one after the other. Inputs and outputs use Sx and Sy, as well as Tx and Ty at Vcc=15V. 

![Screenshot 2024-09-22 at 16 25 06](https://github.com/user-attachments/assets/b7e55feb-baac-4777-a9f9-a3565142d08d)

I assembled these on one board complete with pullup resistors, voltage dividers to provide 3V3 signal levels to the logic analyzer for four channels -- Clock Tx and Rx, Data Tx and Rx -- and a 10-pin box header that is used to connect dirctly to the logic analyzer interface without fiddly probes etc. I also included plenty of Test Point eyelets to easily connect an osclloscope or voltmeter. 
![KELLY (1)](https://github.com/user-attachments/assets/32bd6646-ba2a-4abf-a10d-ae858b08e086)


#### Ph1 Step 6: Logic analyzer setup (COMPLETED)
I use a generic unbranded Chinese 8-channel logic analyzer interface together with the Sigrok Pulseview (free) software running on an iMac. 
I use channels 0, 2, 4 and 6 simply because these are the easiet to wire up on a prototype board. (Ch 1 is linked to Ch 0, 3 to 2, 5 to 4, and 7 to 6 for the same reason, but the odd channels are not used). 

Initial results confirm the first oscilloscope tests. Data is transmitted in 8-bit packets with no ACK. Lines are normally high. The data line is pulled down to start a new package. 
The clock appears to always come from the controller - so far I have not seen any signal on the Clock Rx channel.

My "reverse-engineering workbench" now looks like this ... 
![KELLY reverse-engineering workbench](https://github.com/user-attachments/assets/1bfcd246-55ab-4a1b-8762-9dd195511a62)


#### Ph1 Step 7: Logic analyzer custom decode (NEXT UP)
With the setup of the previous step, I tried out some of the decoders provided with Pulseview, such as I2C, PS/2 and UART -- but the results confirmed my visual impression that the Scantronic 994x protocol is none of these. 
Therefore, to make sense of the content of the transmissions, I propose to build a [custom protocol decoder](https://sigrok.org/wiki/Protocol_decoder_HOWTO). 

I do not yet know whether LOW represents 1 or 0, or whether the lowest or highest significant bit is transmitted first; I will therefore need to decode all four combinations to determine which makes the most sense!
- to be continued. 

#### Ph1 Step 8: Decoding the commands and responses to/from the keypad (FUTURE PLAN) 
Once I can decode the protocol messages, the last step is to determine how the messages are used by the application. I will need to go through all possible commands and responses detailed in the Engineers' [Installation and Programming Guide](https://www.teklib.com/library/scantronic-9651-hardwired-control-panel-installation-and-programming-guide/)

## PHASE 2: Implementation: building the hardware and software components 
To be planned. Not yet started. 


---
If you find any of the ideas in this repository useful, please [Buy Me a Coffee](https://buymeacoffee.com/andysymons)

