# r4s-bluetooth
Hacking Ready For Sky (R4S) home appliances

This repository holds some data for controlling Redmond SkyKettle RK-M171S from GNU/Linux

I welcome suggestions and ideas for simplification and making code more reliable.
Using script wrapper around gatttool is so far the simplest approach. But it is terribly ugly.

Prerequisits:
   You need bluez installed. Version 4.01 is fine

Usage: 
*   ./connect.sh [KETTLE MAC] auth 
*   ./connect.sh [KETTLE MAC] query
*   ./connect.sh [KETTLE MAC] queryone
*   ./connect.sh [KETTLE MAC] keeptemp <temp>
*   ./connect.sh [KETTLE MAC] on
*   ./connect.sh [KETTLE MAC] off

You can get your  [KETTLE MAC] by entering "bt-device -l" 

## Dumps 

You don't need to dump anything, you can just reverse the application itself. Later data had been acquired with decompile method

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

  Protocol had been reversed with the following techique. Dumps were made with "Enable Bluetoog HCI snoop log" Android feature. Analised then with wireshark
  
## Bluetooth level
 
 Device info:
  
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
```

Primary handles

```
attr handle = 0x0001, end grp handle = 0x0007 uuid: 00001800-0000-1000-8000-00805f9b34fb
attr handle = 0x0008, end grp handle = 0x0008 uuid: 00001801-0000-1000-8000-00805f9b34fb
attr handle = 0x0009, end grp handle = 0xffff uuid: 6e400001-b5a3-f393-e0a9-e50e24dcca9e

00001800-0000-1000-8000-00805f9b34fb  Generic Access
00001800-0000-1000-8000-00805f9b34fb  Generic Attribute
6e400001-b5a3-f393-e0a9-e50e24dcca9e  UART Service
```

Detailed capabilites:

```
handle = 0x0002, char properties = 0x0a, char value handle = 0x0003, uuid = 00002a00-0000-1000-8000-00805f9b34fb read write
handle = 0x0004, char properties = 0x02, char value handle = 0x0005, uuid = 00002a01-0000-1000-8000-00805f9b34fb read
handle = 0x0006, char properties = 0x02, char value handle = 0x0007, uuid = 00002a04-0000-1000-8000-00805f9b34fb read

handle = 0x000a, char properties = 0x10, char value handle = 0x000b, uuid = 6e400003-b5a3-f393-e0a9-e50e24dcca9e  notify
handle = 0x000d, char properties = 0x0c, char value handle = 0x000e, uuid = 6e400002-b5a3-f393-e0a9-e50e24dcca9e  write
```
   

### Protocol summary

To start receiving notifications from device on handle 0x000b, you need to write 0x0100 to handle 0x000c (set [Client Characteristic Configuration](https://www.bluetooth.com/specifications/gatt/viewer?attributeXmlFile=org.bluetooth.descriptor.gatt.client_characteristic_configuration.xml) to 0x0001 (big-endian)). 
This needs to be done after every reconnect.
Gatttool doesn't allow you to know when reconnect happend. So I send it every time.

Now you can send commands to handle 0x000e
You will get back the answers from handle 0x000b

#### Command structure

Commands start with 0x55 byte, and end with 0xaa
Second byte is a counter, you should increment it with any new request. I don't know yet what happens when you overflow
Third byte is a command itself
 * 0x01 - request firware version
 * 0x05 - switch the kettle on to keepwarm (on with 0 temperature)
 * 0x04 - switch the kettle off
 * 0x06 - request status
 * 0xFF - authorize
 
 
Next go the parameters.

```
  0x55 0x76 0x06  0xaa
   |    |     |   |
start   |     |  end
    counter   |
        command id
```

As a reply you will recive a sequence that start with 0x55 byte, and end with 0xAA
Second byte is a counter which is the same as in the request it replies to.
Third byte is the command - same as in the request. It depends on the command.
Next goes the data.

#### AUTH
 I can guess you should do the following. Generate a 8byte random ID. This will be your key. 
 I don't know yet if any key is ok. These are - 55:3a:57:47:f8:c2:62:4a, b5:4c:75:b1:b4:0c:88:ef
 
 Send auth command, starting with counter 0

```
 -> 55:<counter>:ff:<8 bytes. Seem random. ID? MAC?>:aa
```

If you are not yet authorized (you need to hold "+" on the kettle for this) the kettle will reply

```
 <- 55:<counter>:ff:00:aa  
```
Send the request again with incrementing counter. Meanwhile hold "+" key. At some point the auth will be passed. And you will get:
   
```
 <- 55:<counter>:ff:01:aa  
```
    
Next time with the same key you should be able to connect from the first attempt.
In the auth state you can issue next commands.

#### FW_VERSION command

```
   ->  55:<counter>:01:aa
reply example
   <-  55:<counter>:01:02:06:aa // ver. 2.6
```

#### BOLING AND HEATING command
```
request examples
   ->  55:<counter>:05:00:00:28:00:aa
   ->  55:<counter>:05:00:00:00:00:aa - just switch on.
   ->  55:<counter>:05:01:00:5f:00:aa - keepwarm at 95
   
       55:<counter>:05:00:00:<keep warm temp>:00:aa
   
reply example   
   <-  55:<counter>:05:01:aa    // 01 status meens OK
```

#### OFF command
```
   ->  55:<counter>:04:aa
reply example   
   <-  55:<counter>:04:01:aa       // 01 status meens OK
```

#### STATUS command
```
   -> 55:<counter>:06:aa 
reply examples:
   <- 55:<counter>:06:00:00:00:00:00:0c:00:00:00:00:51:00:00:00:00:00:aa  - kettle is on in boiling 

   <- 55:<counter>:06:00:00:28:00:00:0c:00:01:02:00:33:00:00:00:00:00:aa  - kettle is on
   <- 55:<counter>:06:00:00:00:00:00:0c:00:00:00:00:3e:00:00:00:00:00:aa  - kettle is off
   <- 55:<counter>:06 00 00 28 00 00 0c 00 01 00 00 64 00 00 00 00 00 aa
   
   <- 55:<counter>:06 01 00 28 00 00 0c 00 01 02 00 64 00 00 00 00 00 aa  - kettle finished boiling ()
   <- 55:<counter>:06 01 00 28 00 00 0c 00 01 02 00 63 00 00 00 00 00 aa  - a while after finished boiling
   
 external num	1     2      3  4            5  6                7  8  9  10 11 12        13 14 
 internal num                  0            1  2               3  4  5  6  7  8         9  10     11  
   	        55:<counter>:06:<keep warm?>:00:<keepwarm temp?>:00:00:0c:00:01:<heater?>:00:<temp>:00:00:00:00:00:aa

```


| Byte ext | Byte int | Action         |
| ---------|----------| ---------------|
| 4        |    0     | Program        |
| 5        |    1     |                |
| 6        |    2     | Target Temperature  |
| 7        |    3     |                |
| 8        |    4     |                |
| 9        |    5     | Remaining hours   |
|10        |    6     | Remaining minutes |
|11        |    7     | Heater on      |
|12        |    8     | State          |
|13        |    9     | Error status   |
|14        |   10     | Current Temp   |

