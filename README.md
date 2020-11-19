# ESP-IDF InkPlate-6 Low Level classes

This project contains in the `lib/inkplate` folder the low-level classes required to access the InkPlate-6 hardware in an ESP-IDF SDK context. This is an extract from the EPub-InkPlate project that can be looked at as an example of a extensive  application using these classes.

The driver classes are:

- EInk: The e-ink display panel class. (Arduino Inkplate equivalent)
- MCP - MCP23017 16 bits IO expander class. (Arduino Mcp)
- ESP - Some methods similar to Arduino supplied methods, but in an ESP-IDF context
- Wire - Similar to Arduino Wire, but built using i2c functions
- Inkplate6Ctrl - Class to control various modes of usage (like deep sleep, light sleep, sd card mounting, etc.)
- Logging - Some definition for log output
- NonCopyable - Support class to sustain non-copyable singleton.

This project is a working example. You can build it using PlatformIO. It will draw a square and a straight line on the device display.

## ESP-IDF configuration specifics

An InkPlate application requires some functionalities to be properly setup within the ESP-IDF. The following elements have been done for his project and can be used as a reference for other projects:

- **Flash memory partitioning**: the file `partitions.csv` contains the table of partitions required to support the application in the 4MB flash memory. The partitions OTA_0 and OTA_1 have been set to be 1.3MB in size. In the `platformio.ini` file, the line `board_build.partitions=...` is directing the use of these partitions configuration. 
  
- **PSRAM memory management**: The PSRAM is an extension to the ESP32 memory that offer 4MB+4MB of additional RAM. The first 4MB is readily available to integrate to the dynamic memory allocation of the ESP-IDF SDK. To do so, some parameters located in the `sdkconfig` file must be set accordingly. This must be done using the menuconfig application that is part of the ESP-IDF. The following command will launch the application (the current folder must be the main folder of EPub-InkPlate):

  ```
  $ idf.py menuconfig
  ```

  The application will show a list of configuration aspects. To configure PSRAM:

  - Select `Component Config` > `ESP32-Specific` > `Support for external, SPI-Connected RAM`
  - Select `SPI RAM config` > `Initialize SPI RAM during startup`
  - Select `Run memory test on SPI RAM Initialization`
  - Select `Enable workaround for bug in SPI RAM cache for Rev 1 ESP32s`
  - Select `SPI RAM access method` > `Make RAM allocatable using malloc() as well`

  Leave the other options as they are. 

- **ESP32 processor speed**: The processor must be run at 240MHz. The following line in `platformio.ini` request this speed:

    ```
    board_build.f_cpu = 240000000L
    ```
  You can also select the speed in the sdkconfig file:

  - Select `Component config` > `ESP32-Specific` > `CPU frequency` > `240 Mhz`

- **FAT Filesystem Support**: The application requires usage of the micro SD card. This card must be formatted on a computer (Linux or Windows) with a FAT 32 partition. The following parameters must be adjusted in `sdkconfig`:

  - Select `Component config` > `FAT Filesystem support` > `Max Long filename length` > `255`
  - Select `Number of simultaneous open files protected  by lock function` > `5`
  - Select `Prefer external RAM when allocating FATFS buffer`