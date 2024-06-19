# KELLY - Scantronic 994x Keypad Integration into ESPHome and Home Assistant

Integrates the Scantronic 9940 or 9441 Keypad into ESPHome and Home Assistant. (They are the same except that 9441 does not have a tag reader).  


The plan is to 
1. Build the necesary hardware to conect the keypad to an ESP32 microprocessor
1. Reverse-engineer the protocol between the alarm controller and the keypad using samples of each
1. Code the protocol into an ESPhome Custom Device
2. Build whatever further integrations are necessary to make the Keypad appear to Home Assistant as (a) an alarm keypad compatible with the native Alarm and with the Alarmo add-on (b) an LED display (c) a tag reader (model 9440 only) (d) switches or sensors for any remainng Scantronic 9940 Keypad features. 

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

