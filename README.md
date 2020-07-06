##  Partition tables and Bootloader of the ESP32 

The ESP32 chip requires an external flash memory to store programs, data, configuration parameter etc. Let's learn what happens inside the flash memory when a new code is flashed, when a new code is flashed to ESP32 the code is store inside the flash memory. The external memory is connected to the chip via the `SPI bus`, and the supported capacity is up to 16MB. The chip supports 4 x 16 MB of external `QSPI` flash and `SRAM`.

The Flash memory is non-volatile memory, unlike `SRAM` memory. Thus even when ESP32 is switched off the code will be safe int to flash memory.
Most of the modules like ESP32 Wroom use external` Flash-W25Q32` (4M Bytes) for storing the application code. The ESP32 flash memory can contain multiple apps, calibration data, file systems, parameter storage, etc. Hence, it's divided into sections (partitions). The list partitions, their size and position within the flash memory is stored in the memory itself (at address `0x8000`) and it’s called partition table. It's just like the index page of a book but with more details of the content.

[![](https://github.com/iqnev/Partition_tables_and_Bootloader_of_the_ESP32/blob/master/source/4_05.jpg)](https://github.com/iqnev/Partition_tables_and_Bootloader_of_the_ESP32/blob/master/source/4_05.jpg)

The partition table starts at a default offset of 32768 bytes it has a size of 3072 bytes with maximum of 9 to 5 distinct table entries. Each table entry is
32 bytes long. The last 32 bytes are MDF checksum used for checking the integrity of the partition table.
When the ESP32 board is powered up initially the control is given to 4096 bytes position in the flash memory. It is called the bootloader. The responsibility of the bootloader is to select a correct partition to boot based on the partition table then load this code to RAM of the ESP32 chip and finally transfer the memory management to the memory management unit inside the ESP32 chip. This is how the new code is accessed from the flash memory and executed on the ESP32.

[![](https://github.com/iqnev/Partition_tables_and_Bootloader_of_the_ESP32/blob/master/source/5_12.jpg)](https://github.com/iqnev/Partition_tables_and_Bootloader_of_the_ESP32/blob/master/source/5_12.jpg)

------------

1.  Bootloader reads the partition table.
2.  Bootloader selects the correct Boot partition.
3. Bootloader loads the code from the flash memory to the RAM of the ESP32 chip.
4. Bootloader transfers the memory management to MMU of the ESP32 chip.

------------

**What is Bootloader?**

In the ESP32 ROM memory there is a small program, named ** first-stage bootloader**. This program is executed at each reset of the chip. It configures the access to the external flash memory and, if required, stores on it new data coming from the USB port. Once finished, it accesses the flash memory (at address` 0x1000`) and loads and executes the **second-stage bootloader**. The second-stage bootloader reads the partition table at address `0x8000` and searches for app partitions. It decides which application has to be executed based on the content of the otadata partition: if this partition is empty or doesn't exist, the bootloader executes the application stored in the factory partition. This allows to implement an over-the-air (OTA) application update process: you send the new version of your application to the ESP32 chip(the version is stored in a new app partition).
Once the upload is completed, the id of the partition is saved in otadata and the chip is rebooted; the bootloader will execute the new version.
[![](https://github.com/iqnev/Partition_tables_and_Bootloader_of_the_ESP32/blob/master/source/7_04.jpg)](https://github.com/iqnev/Partition_tables_and_Bootloader_of_the_ESP32/blob/master/source/7_04.jpg)

By default, ESP32 uses single factory app with no OTA partition table. You can see the details of this partition below.

**More details:**

The `nvs` partition is a non-volatile storage partition which consists of the bootloader partition table. The `phy_init` consists of default configuration parameters for all wired and wireless communication in the ESP32.
The factory partition consists of the current code which is flashed.

[![](https://github.com/iqnev/Partition_tables_and_Bootloader_of_the_ESP32/blob/master/source/5_57.jpg)](https://github.com/iqnev/Partition_tables_and_Bootloader_of_the_ESP32/blob/master/source/5_57.jpg)


When the factory app OTA definitions mode is enabled in the ESP32 the factory partition is split into three partitions of equal size with name `ota_0`, `ota_1` and `ota_2`.
Also, there is otadata partition - partition points the bootloader to boot correct OTA partition 6.39min.

**For example:**

There is a new version of our code which currently is running in the factory partition of the flash memory. When we sent a new code via Wi-Fi to
the flash memory, the code will first be saved `OTA_0` partition. Now a code integrity check will be done in this OTA partition to confirm whether the code is usable or not. Later the OTA partition data will be updated to point the bootloader to use the `OTA_0` as the boot partition instead of the default factory partition. If by any chance the code in `OTA_0` partition faces some errors the ESP32 automatically rollback to use the default factory partition by resetting the boot flag. If ota data is empty, it will execute the factory app.

[![](https://github.com/iqnev/Partition_tables_and_Bootloader_of_the_ESP32/blob/master/source/7_04.jpg)](https://github.com/iqnev/Partition_tables_and_Bootloader_of_the_ESP32/blob/master/source/7_04.jpg)

You can save a lot of memory if you create a custom partition table. You have to take this fact into your design before implementing the OTA feature in a product. There are now three app partition definitions. The type of the factory app (at `0x10000`) and the next two “OTA” apps are all set to app, but their subtypes are different.

[![](https://github.com/iqnev/Partition_tables_and_Bootloader_of_the_ESP32/blob/master/source/8_22.jpg)](https://github.com/iqnev/Partition_tables_and_Bootloader_of_the_ESP32/blob/master/source/8_22.jpg)
