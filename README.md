# ESP-IDF InkPlate-6 Low Level classes

This project contains, in the `lib/inkplate` folder, the low-level classes required to access the InkPlate-6 hardware in an ESP-IDF SDK context. This is an extract from the EPub-InkPlate project that I'm currently developing. You can look at it as an example of an extensive application using all these classes. The classes were first retrieved from the Arduino library made by e-Radionica and refactored to be inline with my own coding practices. They have not been packaged as a component (in the ESP-IDF context) as they are being used in a PlatformIO application (they could be easily transformed as such, adding the appropriate CMakeLists.txt). 

Beware that these classes are **not** re-entrant. That means that it is not possible to use them in a multi-thread context without proper mutual exclusion access control. 

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

- **FAT Filesystem Support**: If the application requires usage of the micro SD card. This card must be formatted on a computer (Linux or Windows) with a FAT 32 partition. The InlPlate6Ctrl class requires this in its current configuration. The following parameters must be adjusted in `sdkconfig`:

  - Select `Component config` > `FAT Filesystem support` > `Max Long filename length` > `255`
  - Select `Number of simultaneous open files protected  by lock function` > `5`
  - Select `Prefer external RAM when allocating FATFS buffer`

- **Flash memory partitioning (optional)**: the file `partitions.csv` contains the table of partitions definition for the 4MB flash memory, required to support the largest applications size in a OTA context. The partitions OTA_0 and OTA_1 have been set to be 1.3MB in size. In the `platformio.ini` file, the line `board_build.partitions=...` is directing the use of these partitions configuration. 
 