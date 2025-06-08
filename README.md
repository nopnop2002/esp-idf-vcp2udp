# esp-idf-vcp2udp
VCP to UDP bridge for ESP-IDF.

![Image](https://github.com/user-attachments/assets/6b45c5aa-d51c-4332-bb95-e68989aab820)

ESP-IDF supports VCP hosts.   
Representative VCP devices include Arduino Uno and Arduino Mega.   
ESP-IDF can communicate with VCP devices using the USB port.   
I based it on [this](https://github.com/espressif/esp-idf/tree/master/examples/peripherals/usb/host/cdc/cdc_acm_vcp).   

This project uses the following components.   
Other UART-USB converter chips are not supported.   
- https://components.espressif.com/components/espressif/usb_host_ch34x_vcp   
- https://components.espressif.com/components/espressif/usb_host_cp210x_vcp   
- https://components.espressif.com/components/espressif/usb_host_ftdi_vcp   


# Software requirements
ESP-IDF V5.0 or later.   
ESP-IDF V4.4 release branch reached EOL in July 2024.   


# Hardtware requirements

## ESP32-S2/S3
This project only works with ESP32S2/S3.   
The ESP32S2/S3 has USB capabilities.   

## USB Type-A Femail connector
USB connectors are available from AliExpress or eBay.   
I used it by incorporating it into a Universal PCB.   
![USBConnector](https://github.com/user-attachments/assets/8d7d8f0a-d289-44b8-ae90-c693a1099ca0)

We can buy this breakout on Ebay or AliExpress.   
![usb-conector-11](https://github.com/user-attachments/assets/848998d4-fb0c-4b4f-97ae-0b3ae8b8996a)
![usb-conector-12](https://github.com/user-attachments/assets/6fc34dcf-0b13-4233-8c71-07234e8c6d06)

# Installation
```
git clone https://github.com/nopnop2002/esp-idf-vcp2udp
cd esp-idf-vcp2udp
idf.py set-target {esp32s2/esp32s3}
idf.py menuconfig
idf.py flash
```

# Configuration   
![Image](https://github.com/user-attachments/assets/de63a3cb-b19c-4d0b-88d4-09b9d7dc3c01)
![Image](https://github.com/user-attachments/assets/9d2fb1a7-7d8d-4b36-9db7-47cb8fe99416)

## WiFi Setting
Set the information of your access point.   
![Image](https://github.com/user-attachments/assets/6768397b-03a6-40f9-ba26-860a5a42aef1)

## VCP Setting
Set the information of VCP Connection.   
![Image](https://github.com/user-attachments/assets/9645356a-bd51-412f-9bbc-fe7c7b75b1ed)

## UDP Setting
Set the information of UDP Connection.   
![Image](https://github.com/user-attachments/assets/d77223d4-ed12-491a-a73b-832b6e5fce46)

There are the following four methods for specifying the UDP Address.
- Limited broadcast address   
 The address represented by 255.255.255.255, or \<broadcast\>, cannot cross the router.   
 Both the sender and receiver must specify a Limited broadcast address.   

- Directed broadcast address   
 It is possible to cross the router with an address that represents only the last octet as 255, such as 192.168.10.255.   
 Both the sender and receiver must specify the Directed broadcast address.   
 __Note that it is possible to pass through the router.__   

- Multicast address   
 Data is sent to all PCs belonging to a specific group using a special address (224.0.0.0 to 239.255.255.255) called a multicast address.   
 I've never used it, so I don't know anything more.

- Unicast address   
 It is possible to cross the router with an address that specifies all octets, such as 192.168.10.41.   
 Both the sender and receiver must specify the Unicast address.


# Write this sketch on Arduino Uno.   
You can use any AtMega microcontroller that has a USB port.   

```
unsigned long lastMillis = 0;

void setup() {
  Serial.begin(115200);
}

void loop() {
  while (Serial.available()) {
    String command = Serial.readStringUntil('\n');
    Serial.println(command);
  }

  if(lastMillis + 1000 <= millis()){
    Serial.print("Hello World ");
    Serial.println(millis());
    lastMillis += 1000;
  }

  delay(1);
}
```

Strings from Arduino to ESP32 are terminated with CR(0x0d)+LF(0x0a).   
```
I (1482) VCP: Receiving data through CdcAcmDevice
I (1482) VCP: 0x3fcba0a8   48 65 6c 6c 6f 20 57 6f  72 6c 64 20 36 36 30 30  |Hello World 6600|
I (1492) VCP: 0x3fcba0b8   30 0d 0a                                          |0..|
```

The Arduino sketch inputs data with LF as the terminator.   
So strings from the ESP32 to the Arduino must be terminated with LF (0x0a).   
If the string output from the ESP32 to the Arduino is not terminated with LF (0x0a), the Arduino sketch will complete the input with a timeout.   
The default input timeout for Arduino sketches is 1000 milliseconds.   
If the string received from UDP does not have a LF at the end, this project will add a LF to the end and send it to Arduino.   
The Arduino sketch will echo back the string it reads.   
```
I (78212) VCP: Sending data through CdcAcmDevice
I (78222) VCP: 0x3fcb7930   61 62 63 64 65 66 67 0a                           |abcdefg.|
I (78232) VCP: Receiving data through CdcAcmDevice
I (78232) VCP: 0x3fcba0a8   61 62 63 64 65 66 67 0d  0a                       |abcdefg..|
```

# Wireing   
Arduino Uno connects via USB connector.   
The USB port on the ESP32S2/S3 development board does not function as a USB-HOST.   

```
+---------+  +-------------+  +-----------+
|ESP BOARD|==|USB CONNECTOR|==|Arduino Uno|
+---------+  +-------------+  +-----------+
```

```
ESP BOARD          USB CONNECTOR (type A)
                         +--+
5V        -------------> | || VCC
[GPIO19]  -------------> | || D-
[GPIO20]  -------------> | || D+
GND       -------------> | || GND
                         +--+
```

![Image](https://github.com/user-attachments/assets/7bf405af-b1ec-4c7c-87d1-8bbe176e807b)


# Windows Application   
I used [this](https://sourceforge.net/projects/sockettest/) app.   
This application runs both a UDP-Listener (UDP-Server) and a UDP-Client simultaneously.   

- Listen from ESP32   
	Specify the port number and press the Start Listening button.   
	This application works as a UDP-Listener (UDP-Server) and communicates with the UDP-Client of ESP32.   
	![Image](https://github.com/user-attachments/assets/30bb42e3-6828-4926-bbe5-7f068ef4a052)   
	![Image](https://github.com/user-attachments/assets/c0ef903a-d0a8-4b68-b653-349fc01ac183)   

- Send to ESP32   
	Specify the IP Address of ESP32 and port number and press the Send button.   
	This application works as a UDP-Client and communicates with the UDP-Server of ESP32.   
	![Image](https://github.com/user-attachments/assets/f1d0002b-4f97-4ef4-8bf9-77eec030551e)   
	Instead of an IP address, you can use an mDNS hostname.   
	![Image](https://github.com/user-attachments/assets/5aafff5a-3bab-481d-b65f-98e3dc6fd25e)   


We can use [this](https://www.hw-group.com/software/hercules-setup-utility) app.   
This application runs both a UDP-Listener (UDP-Server) and a UDP-Client simultaneously.   
![Image](https://github.com/user-attachments/assets/62eb14da-fae5-4aa1-90e7-8d4cb81d500b)   


# Python script   
udp-client.py is UDP-Client.   
udp-server.py is UDP-Listener (UDP-Server).   


# Linux Application
I searched for a GUI tool that can be used with ubuntu or debian, but I couldn't find one.   


# Android Application
I used [this](https://play.google.com/store/apps/details?id=com.jca.udpsendreceive) app.   
![Image](https://github.com/user-attachments/assets/17d60387-0a17-49a5-af3d-7e13879495b3)


# Using linux rsyslogd as logger   
We can forward the USB input to rsyslogd on your Linux machine.
```
+-------------+         +-------------+         +-------------+ 
| Arduino Uno |--(USB)->|    ESP32    |--(UDP)->|  rsyslogd   |
+-------------+         +-------------+         +-------------+ 
```

Configure with port number = 514.   
![Image](https://github.com/user-attachments/assets/bf4cadb5-40d8-49ea-a1b9-dee23fa70c63)

The rsyslog server on linux can receive logs from outside.   
Execute the following command on the Linux machine that will receive the logging data.   
Please note that port 22 will be closed when you enable ufw.   
I used Ubuntu 22.04.   

```
$ cd /etc/rsyslog.d/

$ sudo vi 99-remote.conf
module(load="imudp")
input(type="imudp" port="514")

if $fromhost-ip != '127.0.0.1' and $fromhost-ip != 'localhost' then {
    action(type="omfile" file="/var/log/remote")
    stop
}

$ sudo ufw enable
Firewall is active and enabled on system startup

$ sudo ufw allow 514/udp
Rule added
Rule added (v6)

$ sudo ufw allow 22/tcp
Rule added
Rule added (v6)

$ sudo systemctl restart rsyslog

$ ss -nulp | grep 514
UNCONN 0      0            0.0.0.0:514        0.0.0.0:*
UNCONN 0      0               [::]:514           [::]:*

$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
514/udp                    ALLOW       Anywhere
22/tcp                     ALLOW       Anywhere
514/udp (v6)               ALLOW       Anywhere (v6)
22/tcp (v6)                ALLOW       Anywhere (v6)
```

UART input goes to /var/log/remote.   
```
$ tail -f /var/log/remote
Jun  6 11:05:04 Hello World 6721000
Jun  6 11:05:05 Hello World 6722000
Jun  6 11:05:06 Hello World 6723000
Jun  6 11:05:07 Hello World 6724000
Jun  6 11:05:08 Hello World 6725000
Jun  6 11:05:09 Hello World 6726000
Jun  6 11:05:10 Hello World 6727000
Jun  6 11:05:11 Hello World 6728000
Jun  6 11:05:12 Hello World 6729000
Jun  6 11:05:13 Hello World 6730000
Jun  6 11:05:14 Hello World 6731000
Jun  6 11:05:15 Hello World 6732000
```

One advantage of using rsyslogd is that you can take advantage of log file rotation.   
Rotating log files prevents the log files from growing forever.   
The easiest way to rotate logs is to add /var/log/remote to /etc/logrotate.d/rsyslog.   
There are many resources available on the Internet about rotating log files.   
```
$ ls -l /var/log/remote*
-rw-r--r-- 1 syslog syslog    6264 Jun  6 10:35 /var/log/remote
-rw-r--r-- 1 syslog syslog 1219823 Jun  6 10:33 /var/log/remote.1
```
