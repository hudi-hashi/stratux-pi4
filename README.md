# Build a Stratux Europe on a Pi3, Pi4, Pi5 or Pi Zero 2W based on a fresh 64bit RasPiOS Lite Image

- shopping lists:
  - https://github.com/VirusPilot/stratux-pi4/wiki/Shopping-List-v3-TX (active cooling with fan, copper heatsink, TX module)
  - https://github.com/VirusPilot/stratux-pi4/wiki/Shopping-List-v3 (passive cooling with large copper heatsink)
  - https://github.com/VirusPilot/stratux-pi4/wiki/Shopping-List-v2 (active cooling with fan)
  - https://github.com/VirusPilot/stratux-pi4/wiki/Shopping-List-v1 (passive cooling with custum aluminium block)

- both scripts are based on the latest **64bit RasPiOS Lite Bookworm Image**, using **Raspberry Pi Imager** from here: https://www.raspberrypi.com/software/

# stratux-pi4-standard
based on https://github.com/stratux/stratux (**as of February 4, 2025**)

# stratux-pi4-viruspilot
based on my fork https://github.com/VirusPilot/stratux (**as of February 4, 2025**) with the following modifications compared to the "standard" version:
- main/gps.go: enable GPS LED to indicate a valid GPS fix
- main/gen_gdl90.go: increase GDL90 ownship report from 1Hz to 5Hz

## please use these scripts with caution and only on a fresh 64bit RasPiOS Lite Image, because:
- the entire filesystem (except /boot) will be changed to read-only to prevent microSD card corruption
- swapfile will be disabled

## prepare script for Pi3, Pi4, Pi5 or Pi Zero 2W:
- flash latest **64bit RasPiOS Lite Bookworm Image**, using latest **Raspberry Pi Imager** with the following settings:
  - select appropriate hostname
  - enable ssh
  - enable user pi with password
  - **IMPORTANT**: although Stratux is setting up its own WiFi credentials, it is mandatory to `enable and set up WiFi` and in particular `WiFi Country` in the **Raspberry Pi Imager configuration** because otherwise WiFi might not work at all (even more important for Pi Zero 2W so that the Pi connects to your local WiFi network)
- boot your Pi with this image and wait until it is connected to your LAN or WiFi (this may take a few minutes after the first boot)
- please note that the brightness values of the Pi Zero 2W LED are reversed so it will turn off as soon as Stratux has successfully booted

## start build process
login with ssh as `pi` user with the above set password, then:
```
sudo su
```
standard version:
```
cd ~/
apt update
apt full-upgrade -y
sudo bash -c "$(wget -nv -O - https://raw.githubusercontent.com/VirusPilot/stratux-pi4/master/setup-pi4-standard.sh)"
```
viruspilot version:
```
cd ~/
apt update
apt full-upgrade -y
sudo bash -c "$(wget -nv -O - https://raw.githubusercontent.com/VirusPilot/stratux-pi4/master/setup-pi4-viruspilot.sh)"
```
- if you are all set then let the sript **reboot** but if you haven't yet programed your SDRs, now would be a good time before Stratux will be claiming the SDRs after a reboot; please follow the instructions under "Remarks - SDR programming" below for each SDR individually
- after reboot Stratux is providing an unprotected WiFi access point with the SSID "stratux"
- after you have connected with this access point, you can open the Stratux web interface: http://192.168.10.1 and modify the settings according to your needs
- Pi Zero 2W users: Stratux will now no longer connect to your local network unless you change this by switching the Stratux WiFi settings to "AP+Client"

## optional components:
enabling **Persistent logging** on Stratux settings page is required!

- you may now install
  - stratux radar display: https://github.com/TomBric/stratux-radar-display
  - additional maps: https://github.com/stratux/stratux/wiki/Downloading-better-map-data
- if you want to upgrade gloang to the latest version from time to time, you may consider installing https://github.com/stefanmaric/g with the following command: `curl -sSL https://git.io/g-install | sh -s -- -y`

## SkyDemon related Remarks
### connection via Bluetooth LE (experimental support)
- this allows connecting SkyDemon via BLE instead of WiFi: when in range of your Bluetooth device, and with the device powered on, open the Setup menu in SkyDemon and select Connectivity. Then under the Bluetooth heading, choose Add Bluetooth Device. SkyDemon will scan for devices and your device should appear within a second or two. If it does not, check that it is powered on, and that it is a Bluetooth LE device (Bluetooth 4 onwards). You don’t need to pair it with your tablet or phone first. When the Bluetooth device has appeared, select it. You then need to tell SkyDemon what sort of avionics are connected to your device, in case of Stratux choose `NMEA (FLARM or GPS)`.
### connection via WiFi
- WiFi Settings/Stratux IP Address 192.168.10.1 (default): only **GDL90** can be selected and used in SkyDemon
- WiFi Settings/Stratux IP Address 192.168.1.1: both **GDL90** and **FLARM-NMEA** can be selected and used in SkyDemon
- **GDL90** is labeled as "**GDL90 Compatible Device**" under "**Connectivity/All Devices**"
- **FLARM-NMEA** is labeled as "**Air Avionics AT-1**" or "**FLARM with Air Connect**" under "**Connectivity/All Devices**", the "**Air Connect Key**" can be ignored for Stratux Europe
- info for experts: FLARM-NMEA = TCP:2000, GDL90 = UDP:4000 (for FLARM-NMEA, the EFB initiates the connection, for UDP, Stratux will send unicast to all connected DHCP clients)
- more info here: https://github.com/stratux/stratux/wiki/EFB-Configuration#skydemon-using-flarm-nmea-protocol-recommended

## Limitations/Modifications/Issues
- this setup is intended to create a Stratux system, don't use the Pi for any other important stuff as all of your data may be lost during Stratux operation

## Remarks - SDR programming simple mode
- only plug in one SDR and then execute
```
sdr-tool.sh
```

## Remarks - SDR programming expert mode (1)
During boot, Stratux tries to identify which SDR to use for which traffic type (ADS-B, OGN) - this is done by reading the "Serial number" entry in each SDRs. You can check or modify these entries as described below, it is recommended for programming to only plug in one SDR at a time, connect the appropriate antenna and label this combination accordingly, e.g. "868" for OGN.
```
stxstop (stop Stratux from claiming the SDRs)
rtl_eeprom
```
will report something like the following:
```
Current configuration:
__________________________________________
Vendor ID:              0x0bda
Product ID:             0x2838
Manufacturer:           Realtek
Product:                RTL2838UHIDIR
Serial number:          stx:868:0
Serial number enabled:  yes
IR endpoint enabled:    yes
Remote wakeup enabled:  no
__________________________________________
```
This SDR is obviosly programmed for Stratux (stx), OGN (868MHz), and a ppm correction of "0", the ppm can be modified later, see below. If your SDR comes pre-programed (it would be labled with e.g. with "1090") there is no need to program it.

You can program the `Serial number` entry with the following command:
```
rtl_eeprom -s stx:1090:0
```
to prepare it e.g. for ADS-B (1090MHz) use. A reboot is necessary to activate the new serial number.

If for some reasons an error occurs while programming, please consider preparing the SDR with the default values:
```
rtl_eeprom -g realtek
```
At this point you can already test your SDR and receive ADS-B traffic with the following command:
```
rtl_adsb -V
```
Or listen to you favorite FM radio station (my station below is at 106.9MHz) by pluging in a headset and run the following command:
```
rtl_fm -M fm -f 106.9M -s 32000 -g 60 -l 10 - | aplay -t raw -r 32000 -c 1 -f S16_LE
```
## Remarks - SDR programming expert mode (2)
During boot, Stratux furthermore reads the ppm correction from the SDR `Serial number`, e.g. if the `Serial number` is `stx:1090:28` then the ppm used by Stratux is +28. If the appropriate ppm for the SDR is unknown, here are the steps to find out (again it is useful to have only one SDR plugged in to avoid confusion):

`stxstop` (in case Stratux is already running)

`kal -s GSM900` and note donw the channel number with the highest power (e.g. 4)

`kal -b GSM900 -c 4` and note down the average absolute error (e.g. 16.325 ppm)

Once you have found the appropriate ppm (e.g. +16 as in the example above), the SDR `Serial number` needs to be programmed once again:
```
rtl_eeprom -s stx:868:16
reboot
```
For more information on how to use `kal` please visit https://github.com/steve-m/kalibrate-rtl/blob/master/README.md
