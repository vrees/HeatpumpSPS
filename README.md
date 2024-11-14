# HeatpumpSPS
Firmware using Beckhoff SPS to Heatpump

HMI: http://192.168.178.14:1010/  (__SystemAdministrator/isdn98)


# Versionen

## CX5110-0115-9020 IPC:
- Network Adapter: PCT\TCI8254X1
- Feste IP Adresse 192.168.178.14, Subnetz 255.255.255.0, Gateway 192.168.178.1
- TwinCAT Version: 3.1 
- Build 4024.62
- DeviceName: CX-7CCAE0
- Image Version: CX51x0 HPS 6.10a

## Windows 10, EntwicklerPC:
DHCP: Adresse vom Fritzbox Router: immer 192.168.178.180
TE2000 HMI Engineering		1.12.762.46
TC 3.1 4024.62
Local config: C:\ProgramData\Beckhoff\TwinCAT\3.1\Boot

### Beckhoff Support Urls
- CE Image update: https://infosys.beckhoff.com/english.php?content=../content/1033/sw_os/2018982667.html&id=
- Image Download Path: https://download.beckhoff.com/download/software/embPC-Control/CX51xx/CX51x0/CE/TC3
- File: CBxx63_WEC7_HPS_v610a_TC31_B4024.53_v2.zip
- Lizens-File: ./License-Files/TLR_BI_00976901.tclrs
- Copy E:\data\Lizenzen\Beckhoff\TLR_BI_00976901.tclrs  (HDD on WIN10-GAMING)

#### Install HMI Server on CX-5110
CerHost: C:\Users\Viktor Rees\Documents\TcXaeShell\CERHOST.exe
connect to 192.168.178.14: PW: 3377

### Starting the Beckhoff Device Manager 
https://infosys.beckhoff.com/english.php?content=../content/1033/cx51x0_hw/27021604200571915.html&id=
https://192.168.178.14/config/  (Administrator/1)

### Installing HMI Server
https://www.linkedin.com/pulse/twincat-3-hmi-windows-ce-mattias-nilsson

### Access WebServer
https://192.168.178.14:1020/Config  (case sensitive)
http://192.168.178.14:1010/Config

Username: __System/Password
Password: is........8




## TODO:
- Hardening: Diagnose of Analog Status Ionfo .ilke Overrun, Error, ...
- Install HMI on PLC
- Temperatur-Sensor in Boiler
- Quellwassermenge stufenlos steuern - oder Ein-/Ausschalten
- Installation unter Linux VM
