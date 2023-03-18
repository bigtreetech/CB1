
# OS image
* The latest system image is [here](https://github.com/bigtreetech/CB1/releases)
* The source code is [here](https://github.com/bigtreetech/CB1-Kernel)

# V2.3.0 OS Setting
## WIFI Settings
* After the OS writes to the SD card, there is a FAT32 partition named `BOOT`, open `system.cfg` file with `Notpad`, `Notpad++` or `VSCode`.
<br/><img src=Images/system.png width="800"/><br/>
* Set `WIFI_SSID` as your actual wifi name and `WIFI_PASSWD` as your actual wifi password, The space character can be parsed normally without additional escape character.<br/>
For example: `WIFI_SSID="CB1 Tester"`
<br/><img src=Images/wifi.png width="800"/><br/>

## Overlays Settings
* After the OS writes to the SD card, there is a FAT32 partition named `BOOT`, open `BoardEnv.txt` file with `Notpad`, `Notpad++` or `VSCode`.
<br/><img src=Images/BoardEnv.png width="800"/><br/>
* Set as required as shown in the figure below.
    * set `fdtfile` to `sun50i-h616-biqu-emmc` for CB1 eMMC version, set `fdtfile` to `sun50i-h616-biqu-sd` (default value) for CB1 normal version 
    * The default value is `console=display`, This means that the `UART0` of CB1 is used as the debugging port by default. We can use `MobaXterm` to connect to CB1 by UART0 and debug. If klipper wants to use `UART0` to control the motherboard, we need to set it to `console=serial`, now klippe can use `UART0` as `/dev/ttyS0`.
    * CB1 will automatically identify the HDMI resolution, but if your HDMI screen cannot report the resolution through the EDID normally, we can forcibly specify the resolution of CB1 output by uncomment `extraargs=video` and set the actual resolution.<br/>
    For example:<br/>
    BTT-HDMI7 resolution = 1024x600: `extraargs=video=HDMI-A-1:1024x600-24@60`<br/>
    BTT-HDMI5 resolution = 800x480: `extraargs=video=HDMI-A-1:800x480-24@60`<br/>
    * Uncomment `overlays=tft35_spi` to enable TFT35 SPI screen.
    * Uncomment `overlays=mcp2515` to enable MCP2515 spi to canbus module (Theoretically, it can be multiplexed with `tft35_spi` and 'spidev1.2' at the same time, but `mcp2515` needs strong real-time, it is better not to enable other SPI1 features when using `mcp2515`).
    * uncomment `overlays=spidev1_2` to release 'spidev1.2' to user space (For example: adxl345), `spidev1.0` is used by `MCP2515`, `spidev1.1` is used by `tft35_spi`.
    <br/><img src=Images/overlays.png width="800"/><br/>
* NOTE: TFT35 SPI and MCP2515 multiplex a group of SPI1
    ```
    SPI1_CLK=PH6
    SPI1_MISO=PH8
    SPI1_MOSI=PH7
    TFT35_SPI_CS=PC7
    MCP2515_CS=PC11
    MCP2515_IRQ=PC9
    ```

# CB1 eMMC Version
__NOTE: The CB1 eMMC version can also use the SD card as the OS image source, and the priority of the SD card is higher than on-board eMMC, so when using the eMMC, remember not to insert the OS SD card__
1. Download the utility [sunxi-fel](https://github.com/bigtreetech/sunxi-tools) to your computer (Mac OS is not supported) and download the CB1 [driver](https://github.com/bigtreetech/sunxi-tools/raw/master/u-boot-sunxi-cb1-emmc.bin)
   For windows download [sunxi-fel.exe](https://github.com/bigtreetech/sunxi-tools/raw/master/sunxi-fel.exe)
   For linux download [sunxi-fel-aarch64](https://github.com/bigtreetech/sunxi-tools/raw/master/sunxi-fel-aarch64)
   For arm download [sunxi-fel-armhf](https://github.com/bigtreetech/sunxi-tools/raw/master/sunxi-fel-armhf)
2. Push the DIP switch (USB OTG) and (RPI BOOT) to ON to enter BOOT mode.<br/>
    As shown in the following figure is for PI4B_Adapter.<br/>
    For other motherboards, refer to the CM4 eMMC part of the motherboard manual to set the switch.<br/>
    For some motherboards (e.g. Manta-E3EZ, Manta-M8P-V1.1, Manta-M5P) with OTG/UART selector switch for Type-C, we also need to set the switch to OTG mode according to motherboard's manual <br/><img src=Images/eMMC.png width="500"/><br/>
3. USB driver for windows (Linux skip this step): refer to the official website of [AllWinner](https://linux-sunxi.org/FEL/USBBoot#Using_sunxi-fel_on_Windows)
    * Download [Zadig](https://zadig.akeo.ie/) to the rescue
    * Enable `Options->List All Devices` <br/><img src=Images/zadig_list.png width="500"/><br/>
    * Select the USB device to install the driver(most likely will be "unknown"). Make sure the device USB ID is "1F3A:EFE8"). Click `Install Driver` after confirming that the information is correct <br/><img src=Images/zadig_driver.png width="500"/><br/>
4. Open the `Powershell`(windows) or `console terminal`(linux) where you downloaded the sunxi-fel tools and CB1 driver in step 1.
5. Run<br/>
    `.\sunxi-fel.exe -v ver` (windows)<br/>
    `sudo ./sunxi-fel-armhf -v ver` (linux-armhf)<br/>
    `sudo ./sunxi-fel-aarch64 -v ver` (linux-aarch64)<br/>
    to check whether the USB of CB1 is connected normally.<br>
    If you get `ERROR: Allwinner USB FEL device not found!` means that the USB is not recognized. Please recheck whether the driver is installed successfully.<br>
    If you get `AWUSBFEX soc=00001823(H616)` means that CB1 eMMC is ready.
    
    Here on the first line is an example of error you will see if the driver did not install correctly.
    On the second line is an example of what it will look like after zadig has installed the driver correctly.
     <br/><img src=Images/fel_ver.png width="1000"/><br/>
6. Run<br/>
    `.\sunxi-fel.exe uboot .\u-boot-sunxi-cb1-emmc.bin` (windows)<br/>
    `sudo ./sunxi-fel-armhf uboot ./u-boot-sunxi-cb1-emmc.bin` (linux-armhf)<br/>
    `sudo ./sunxi-fel-aarch64 uboot ./u-boot-sunxi-cb1-emmc.bin` (linux-aarch64)<br/>
    to write u-boot to CB1
7. When the uboot is written, the computer will recognize a USB flash disk, and then you can use `balenaEtcher` or `Raspberry Pi Imager` to write the OS image to eMMC. The steps are the same as the SD card version.
8. Refor to [Overlays Settings](https://github.com/bigtreetech/CB1#overlays-settings) to set `fdtfile` to `sun50i-h616-biqu-emmc`

# Note
## Here’s BIGTREETECH! For Makers, by makers!
* We appreciate all of your support to BIGTREETECH! To offer an excellent experience of creation to every makers,We’re devoted to design and produce high-quality and durable accessories!

## How to contact:
### If you have any technical issue,please don’t hesitate contact us:
* BIGTREETECH: service004@biqu3d.com

### Follow us on social media to get more news:
* Facebook: https://www.facebook.com/BIGTREETECH/
* Twitter: https://twitter.com/BigTreeTech
* Instagram: https://www.instagram.com/bigtreetech_official/
* Official Site: https://bigtree-tech.com/

## Purchase link:
* M4P/M8P/CB1: https://www.biqu.equipment/collections/control-board/products/manta-m4p-m8p
