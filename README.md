# Project KELLY - Integrate Scantronic 994x Keypads into ESPHome and Home Assistant

The objective of this project is to be able to use the Scantronic 994x Keypads in an ESPHome and Home Assistant environment. One or more keypads could be used independently or in conjunction with the Scantronic 9651 hardwired interface -- [project HARVIE](https://github.com/AndySymons/HARVIE-Hardwired-Alarm-Replacement-Board). 

I have no specification or circuit schematic, so the first phase of the project is to reverse engineer the data communication protocol. 
The second phase will be to build the necessary hardware and software components for the ESPHome and Home Assistant environment, provisionally expected to include: 
    
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

## Foreseen project products 

| No. | Product                      | Medium                   | Location                    | Status at 23-Sep-24 | Notes                                       |
|-----|------------------------------|--------------------------|-----------------------------|---------|------------------------------------------------|
| 1.  | Design(s) for bus interface (line driver) hardware to connect the keypad(s) to an ESP32 procesessor | Schematic and prototype  | On this Wiki | Complete | Two options available: with and without optocoupling  |
| 2.  | Design(s) for a 'bus intercept' circuit to connect to logic analyzer | Schematic and prototype | On this Wiki | Complete | |
| 3.  | Pulseview protocol decoder   | Python code              | On this Wiki | Pending | Might copy to Sigrok when complete, with the protocol description. |
| 4.  | Description of the protocol  | Wiki (text and diagrams) | On this Wiki | Pending | Might copy to Sigrok when complete, with the protocol decoder.     |
| 5.  | An ESPHome Custom Device     | t.b.d. (as required by ESPHome) | On this Wiki | Future ||
| 6.  | HA keypad integration        | t.b.d. (as required by HA) | On this Wiki | Future | Should be compatible with the native [Manual Alarm](https://www.home-assistant.io/integrations/manual/) and with the [Alarmo add-on](https://github.com/nielsfaber/alarmo), if possible including A, B, C, D as buttons for selecting the arming level |
| 7.  | HA LCD display integration   | t.b.d. (as required by HA) | On this Wiki | Future | 
| 8.  | HA tag reader integration    | t.b.d. (as required by HA) | On this Wiki | Future | Model 9440 only. See [tag reader](https://www.home-assistant.io/integrations/tag/) | 
| 9.  | HA integration of remaining buttons and lights | t.b.d. (as required by HA) | On this Wiki | Future | Three LEDs and the backlights as lights. Tamper microswitch, panic button, external tamper and exit terminate switches as binary sensors | 




---
If you find any of the ideas in this repository useful, please [Buy Me a Coffee](https://buymeacoffee.com/andysymons)

