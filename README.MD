# Ready to use lv_micropython builds for ESP32
Built from https://github.com/lvgl/lv_micropython

__Information:__
* [Environment setup](#Environment setup)
* [How to clone and compile](#How to clone and compile)
* [How to flash](#How to flash)
## Environment setup

While micropython itself compiles and might (I did not test it) work when using ESP-IDF 4.4,
lv_micropython does not. These firmwares have been compiled under Debian 11 using ESP-IDF 4.2.2.
```
sudo apt-get install git wget flex bison gperf python3 python3-pip python3-setuptools cmake ninja-build ccache libffi-dev libssl-dev dfu-util libusb-1.0-0
git clone -b v4.2.2 --recursive https://github.com/espressif/esp-idf.git esp-idf-v4.2.2
cd esp-idf/
./install.sh esp32
```

## How to clone and compile

```
git clone -j8 --recurse-submodules https://github.com/lvgl/lv_micropython
cd lv_micropython
make -C mpy-cross
cd ports/esp32
make submodules
```
This prepares the project to be built for various ESP32 variants and display options.

Finally, we can build the firmware using:
```
make -C ports/esp32 LV_CFLAGS="-DLV_COLOR_DEPTH=16 -DLV_COLOR_16_SWAP=1" BOARD=GENERIC_SPIRAM V=1
```
In my case I am using either:

```
BOARD=GENERIC_SPIRAM
```
or
```
BOARD=GENERIC
```
Depending on whether or not the board has an external SPI RAM.

Additionally, the LV_CFLAGS are used in this case to set options required for the ILI9341 display driver.

## How to flash

From the ESP-IDF environment, invoke esptool.py with the following parameters to flash your ESP32 device, replacing GENERIC_SPIRAM with GENERIC if needed

```
esptool.py -p PORT -b 460800 --before default_reset --after hard_reset --chip esp32  write_flash --flash_mode dio --flash_size detect --flash_freq 40m 0x1000 build-GENERIC_SPIRAM/bootloader/bootloader.bin 0x8000 build-GENERIC_SPIRAM/partition_table/partition-table.bin 0x10000 build-GENERIC_SPIRAM/micropython.bin
```
