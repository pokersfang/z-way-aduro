# Z-Way-Aduro

The relevant documentation pertains to the ADUROSMART GATEWAY-ESP32-S3R8-V2 hardware.

## Build

Based on the latest [ESP-IDF v5.5](https://github.com/espressif/esp-idf/tree/release/v5.5)

### Initialize environment

```shell
cd ~
git clone --recursive https://github.com/espressif/esp-idf.git
cd esp-idf; git checkout v5.5; git submodule update --init --recursive;
./install.sh
source ./export.sh; cd ..
```

### Start compiling

The firmware source code currently in use is modified based on `z-way-esp32-prod-xiao-zwave.zip` provided by TridentIOT z-way [nightly build](https://github.com/tridentiot/z-way-builds/releases/download/nightly/z-way-esp.tgz). The Z-Wave connection part is exactly the same, the only difference is that Ethernet interface support based on `W5500` is enabled.

```shell
git clone https://github.com/pokersfang/z-way-aduro.git
cd zway-aduro
tar zxvf z-way-esp32-aduro-src.tar.gz
cd upstream/z-way-esp32

idf.py -DZWAY_CMAKE_SUPPORT_ZMATTER_BRIDGE=OFF -DZWAY_CMAKE_ESP32_DEVICE_VARIANT=prod-xiao set-target esp32s3
idf.py -DZWAY_CMAKE_SUPPORT_ZMATTER_BRIDGE=OFF -DZWAY_CMAKE_ESP32_DEVICE_VARIANT=prod-xiao build
```

### Hardware customization

- Rescue Button: GPIO7, active low. Holding it during power-on forces Wi-Fi SoftAP mode and turns the red LED on.
- RGB LEDs are active high: R/G/B use GPIO1/GPIO2/GPIO4. Ethernet down or no IP turns the blue LED on; an acquired Ethernet IP turns the green LED on; disconnecting the cable or losing the IP returns to blue.

## Run

### T32CZ20 (Z-Wave Controller)

#### Install Elcap

```shell
curl -LsSf https://api.tridentiot.com/elcap-installer.sh | bash
```

#### Firmware Flashing

```shell
$ elcap device info
Select a chip target:
1. T32CM11C
2. T32CZ20B
Select target (default: T32CM11C): 2
Select a device:
1. J-Link (USB, Serial: 59610448, COM: )
Select device (default: 59610448): 1
EUI64: 0x5c14eb010000xxxx

$ elcap device erase --all
Select a chip target:
1. T32CM11C
2. T32CZ20B
Select target (default: T32CM11C): 2
Select a device:
1. J-Link (USB, Serial: 59610448, COM: )
Select device (default: 59610448): 1
Chip erase completed successfully.

$ elcap flash --file THIN_GW_DKNCZ20_Shield_zwave_serial_api_controller_1.0_us_lr_debug_signed_combined_aduro.hex
Select a chip target:
1. T32CM11C
2. T32CZ20B
Select target (default: T32CM11C): 2
Successfully flashed 'THIN_GW_DKNCZ20_Shield_zwave_serial_api_controller_1.0_us_lr_debug_signed_combined_aduro.hex'.
```

### ESP32-S3

> **Note:** UART0 is occupied by the CZ20 and cannot be used to flash or debug the ESP32-S3. Flashing and debugging must use the USB Serial/JTAG interface through the board's USB port.

> **Standalone power:** Without a USB host, USB Serial/JTAG input may return immediately. The firmware now waits for a host while keeping Z-Way, Web, Wi-Fi, and Ethernet services running, preventing the previous CLI-exit restart loop. Reconnecting USB restores the CLI.

> **Z-Way ESP Web Tools compatibility:** The Web CLI workflow described in Trident IoT's [Developer-Sandbox](https://github.com/tridentiot/Developer-Sandbox) does not directly apply to this hardware/firmware combination. The current Web Tools terminal cannot use the ESP32-S3 native USB Serial/JTAG console for the CLI, while UART0 is unavailable because it is connected to the CZ20; further Web Tools adaptation may therefore be required. This firmware also uses `I` to display IP/SoftAP information and lowercase `w` to manage Wi-Fi credentials instead of the documented uppercase `W`. The Web interface itself works: hold the Rescue Button during power-on to enter SoftAP mode, connect to that SoftAP, and open `http://<gateway_ip>:8083/`.

```shell
# 1. You need to initialize IDF first, as shown in the example below: 
cd ~/esp-idf; source ./export.sh; cd ~/zway-aduro/upstream/z-way-esp32

# 2. Ensure you have compiled the relevant firmware according to `### Start compiling`.

# 3. Connect the gateway board using a USB cable, and start flashing and debugging by following the commands below to view the debug logs.

idf.py -p /dev/ttyACM0 erase-flash flash monitor

```
