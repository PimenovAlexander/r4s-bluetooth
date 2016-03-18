# r4s-bluetooth
Hacking Ready For Sky (R4S) home appliances

This repository holds some data for controlling r4s SkyKettle. 

I welcome suggestions and ideas for simplification and making code more reliable.
Using script wrapper around gatttool is so far the simplest approach. But it is terribly ugly.

Usage: 
*   ./connect.sh auth
*   ./connect.sh status
*   ./connect.sh on
*   ./connect.sh off
   

## Dumps 

* dump/auth.on.off.bin 
    Initiate an auth from application. Hold "+" on kettle. Switch kettle "ON". Press "OFF" just a second before 100C
* dump/auth.bin  
    Initiate an auth from application. do nothing on kettle

## Hardware

* Bluetooth dongle (CSR4.0) - http://ru.aliexpress.com/item/Bluetooth-4-0-Dongles-Mini-USB-2-0-3-0-Bluetooth-Dongle-Adapters-Dual-Mode-adapter/32292553074.html 

Insert dmesg
```
[ 8058.302793] usb 3-13: USB disconnect, device number 5
[ 8059.355045] usb 3-14: USB disconnect, device number 6
[ 8062.905322] usb 3-13: new full-speed USB device number 12 using xhci_hcd
[ 8062.922927] usb 3-13: New USB device found, idVendor=e0ff, idProduct=0002
[ 8062.922928] usb 3-13: New USB device strings: Mfr=1, Product=2, SerialNumber=0
[ 8062.922930] usb 3-13: Product: G3
[ 8062.922930] usb 3-13: Manufacturer: A.....
[ 8062.924416] input: A..... G3 as /devices/pci0000:00/0000:00:14.0/usb3/3-13/3-13:1.0/input/input22
[ 8062.924479] hid-generic 0003:E0FF:0002.0006: input,hidraw0: USB HID v1.00 Mouse [A..... G3] on usb-0000:00:14.0-13/input0
[ 8062.925984] input: A..... G3 as /devices/pci0000:00/0000:00:14.0/usb3/3-13/3-13:1.1/input/input23
[ 8062.926091] hid-generic 0003:E0FF:0002.0007: input,hiddev0,hidraw1: USB HID v1.00 Keyboard [A..... G3] on usb-0000:00:14.0-13/input1
[ 8071.616958] usb 3-10.3: new full-speed USB device number 13 using xhci_hcd
[ 8071.640199] usb 3-10.3: New USB device found, idVendor=0a12, idProduct=0001
[ 8071.640217] usb 3-10.3: New USB device strings: Mfr=0, Product=2, SerialNumber=0
[ 8071.640223] usb 3-10.3: Product: CSR8510 A10
```

## Protocol 

  Protocol had been reversed with the following techique. Dumps were made with "Enable Bluetoog HCI snoop log". 
  Analised with wireshark
  
## Bluetooth level
  
 ```
 [E7:5A:53:79:82:A4]
  Name: RK-M171S
  Alias: RK-M171S [rw]
  Address: E7:5A:53:79:82:A4
  Icon: undefined
  Class: 0x0
  Paired: 0
  Trusted: 0 [rw]
  Blocked: 0 [rw]
  Connected: 0
  UUIDs: [00001800-0000-1000-8000-00805f9b34fb, 00001801-0000-1000-8000-00805f9b34fb, 6e400001-b5a3-f393-e0a9-e50e24dcca9e]

  
  
attr handle = 0x0001, end grp handle = 0x0007 uuid: 00001800-0000-1000-8000-00805f9b34fb
attr handle = 0x0008, end grp handle = 0x0008 uuid: 00001801-0000-1000-8000-00805f9b34fb
attr handle = 0x0009, end grp handle = 0xffff uuid: 6e400001-b5a3-f393-e0a9-e50e24dcca9e

00001800-0000-1000-8000-00805f9b34fb  Generic Access
00001800-0000-1000-8000-00805f9b34fb  Generic Attribute
6e400001-b5a3-f393-e0a9-e50e24dcca9e  UART Service



handle = 0x0002, char properties = 0x0a, char value handle = 0x0003, uuid = 00002a00-0000-1000-8000-00805f9b34fb read write
handle = 0x0004, char properties = 0x02, char value handle = 0x0005, uuid = 00002a01-0000-1000-8000-00805f9b34fb read
handle = 0x0006, char properties = 0x02, char value handle = 0x0007, uuid = 00002a04-0000-1000-8000-00805f9b34fb read

handle = 0x000a, char properties = 0x10, char value handle = 0x000b, uuid = 6e400003-b5a3-f393-e0a9-e50e24dcca9e  notify read
handle = 0x000d, char properties = 0x0c, char value handle = 0x000e, uuid = 6e400002-b5a3-f393-e0a9-e50e24dcca9e  write
```
   
   
No connect 
   0x000c  -> 0x0100
   
   0x000e  -> 55:00:ff:b5:4c:75:b1:b4:0c:88:ef:aa
	      55:<counter>:ff:b5:4c:75:b1:b4:0c:88:ef:aa
	      
	      This changes from one request to the other. Even on one device 
	      1: 55:01:ff:55:3a:57:47:f8:c2:62:4a:aa
	      2: 55:00:ff:b5:4c:75:b1:b4:0c:88:ef:aa
	      
	      55:<counter>:ff:<8 bytes. Seem random. ID? MAC?>:aa
	      
	      
	   <- 55:<counter>:ff:00:aa  
	   
-----------------------------------------------------------------
	      55:0e:ff:55:3a:57:47:f8:c2:62:4a:aa
	      55:0e:ff:01:aa
 
	   
	   
	   
after auth

	   -> 55:21:06:aa 
	      55:<counter>:06:aa 
	         status request
	        
           <- 55:27:06:00:00:28:00:00:0c:00:01:02:00:33:00:00:00:00:00:aa
              55 08 06 00 00 00 00 00 0c 00 00 00 00 3e 00 00 00 00 00 aa
              
               1    2      3  4  5  6  7  8  9  10 11 12            13 14 
              
              55:<counter>:06:00:00:28:00:00:0c:00:01:<heater>:00:<temp?>:00:00:00:00:00:aa
                
              <heater>  
                  00 - off
                  02 - on
              <temp>
                  temperature in C
              

On
          -> 55:0e:05:00:00:28:00:aa
          <- 55:0e:05:01:aa
              
Off 
          -> 55:15:04:aa
          <- 55:15:04:01:aa   
   

