# SSH
    login as: biqu
    password: biqu

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

# 40 Pin GPIO
<table style="color:black">
<tr>
    <td rowspan=2 align=center bgcolor=gray>
        Pin
    </td>
    <td colspan=2 align=center bgcolor=#B7DEE8>
        BTT Pi
    </td>
    <td colspan=2 align=center bgcolor=#E6B8B7>
        CB1 eMMC
    </td>
    <td colspan=2 align=center bgcolor=#CCC0DA>
        CB1
    </td>
    <td colspan=2 align=center bgcolor=#D8E4BC>
        CM4
    </td>
    <td colspan=2 align=center bgcolor=#D8E4BC>
        CM4
    </td>
    <td colspan=2 align=center bgcolor=#CCC0DA>
        CB1
    </td>
    <td colspan=2 align=center bgcolor=#E6B8B7>
        CB1 eMMC
    </td>
    <td colspan=2 align=center bgcolor=#B7DEE8>
        BTT Pi
    </td>
    <td rowspan=2 align=center bgcolor=gray>
        Pin
    </td>
</tr>

<tr>
    <td bgcolor=#B7DEE8>
        Signal
    </td>
    <td align=center bgcolor=#B7DEE8>
        Description
    </td>
    <td align=center bgcolor=#E6B8B7>
        Signal
    </td>
    <td align=center bgcolor=#E6B8B7>
        Description
    </td>
    <td align=center bgcolor=#CCC0DA>
        Signal
    </td>
    <td align=center bgcolor=#CCC0DA>
        Description
    </td>
    <td align=center bgcolor=#D8E4BC>
        Signal
    </td>
    <td align=center bgcolor=#D8E4BC>
        Description
    </td>
    <td align=center bgcolor=#D8E4BC>
        Signal
    </td>
    <td align=center bgcolor=#D8E4BC>
        Description
    </td>
    <td align=center bgcolor=#CCC0DA>
        Signal
    </td>
    <td align=center bgcolor=#CCC0DA>
        Description
    </td>
    <td align=center bgcolor=#E6B8B7>
        Signal
    </td>
    <td align=center bgcolor=#E6B8B7>
        Description
    </td>
    <td align=center bgcolor=#B7DEE8>
        Signal
    </td>
    <td align=center bgcolor=#B7DEE8>
        Description
    </td>
</tr>

<tr>
    <td bgcolor=gray>
        1
    </td>
    <td colspan=2 align=center bgcolor=yellow>
        3.3V
    </td>
    <td colspan=2 align=center bgcolor=yellow>
        3.3V
    </td>
    <td colspan=2 align=center bgcolor=yellow>
        3.3V
    </td>
    <td colspan=2 align=center bgcolor=yellow>
        3.3V
    </td>
    <td colspan=2 align=center bgcolor=red>
        5V
    </td>
    <td colspan=2 align=center bgcolor=red>
        5V
    </td>
    <td colspan=2 align=center bgcolor=red>
        5V
    </td>
    <td colspan=2 align=center bgcolor=red>
        5V
    </td>
    <td bgcolor=gray>
        2
    </td>
</tr>

<tr>
    <td bgcolor=gray>
        3
    </td>
    <td bgcolor=#B7DEE8>
        PC3
    </td>
    <td align=center bgcolor=#B7DEE8>
        GPIO67
    </td>
    <td colspan=2 align=center bgcolor=gray>
        NC
    </td>
    <td colspan=2 align=center bgcolor=gray>
        NC
    </td>
    <td align=center bgcolor=#D8E4BC>
        GPIO2
    </td>
    <td align=center bgcolor=#D8E4BC>
        I2C1 SDA
    </td>
    <td colspan=2 align=center bgcolor=red>
        5V
    </td>
    <td colspan=2 align=center bgcolor=red>
        5V
    </td>
    <td colspan=2 align=center bgcolor=red>
        5V
    </td>
    <td colspan=2 align=center bgcolor=red>
        5V
    </td>
    <td bgcolor=gray>
        4
    </td>
</tr>

<tr>
    <td bgcolor=gray>
        5
    </td>
    <td bgcolor=#B7DEE8>
        PC0
    </td>
    <td align=center bgcolor=#B7DEE8>
        GPIO64
    </td>
    <td colspan=2 align=center bgcolor=gray>
        NC
    </td>
    <td colspan=2 align=center bgcolor=gray>
        NC
    </td>
    <td align=center bgcolor=#D8E4BC>
        GPIO3
    </td>
    <td align=center bgcolor=#D8E4BC>
        I2C1 SCL
    </td>
    <td colspan=2 align=center bgcolor=black>
        <font color=white>GND</font>
    </td>
    <td colspan=2 align=center bgcolor=black>
        <font color=white>GND</font>
    </td>
    <td colspan=2 align=center bgcolor=black>
        <font color=white>GND</font>
    </td>
    <td colspan=2 align=center bgcolor=black>
        <font color=white>GND</font>
    </td>
    <td bgcolor=gray>
        6
    </td>
</tr>

<tr>
    <td bgcolor=gray>
        7
    </td>
    <td bgcolor=#B7DEE8>
        PC7
    </td>
    <td align=center bgcolor=#B7DEE8>
        GPIO71
    </td>
    <td align=center bgcolor=#E6B8B7>
        PI14
    </td>
    <td align=center bgcolor=#E6B8B7>
        GPIO170
    </td>
    <td align=center bgcolor=#CCC0DA>
        PC7
    </td>
    <td align=center bgcolor=#CCC0DA>
        GPIO71
    </td>
    <td align=center bgcolor=#D8E4BC>
        GPIO4
    </td>
    <td align=center bgcolor=#D8E4BC>
        GPCLK0
    </td>
    <td align=center bgcolor=#D8E4BC>
        GPIO14
    </td>
    <td align=center bgcolor=#D8E4BC>
        UART TX
    </td>
    <td align=center bgcolor=#CCC0DA>
        PH0
    </td>
    <td align=center bgcolor=#CCC0DA>
        GPIO224, UART0_TX
    </td>
    <td align=center bgcolor=#E6B8B7>
        PH0
    </td>
    <td align=center bgcolor=#E6B8B7>
        GPIO224, UART0_TX
    </td>
    <td align=center bgcolor=#B7DEE8>
        PH0
    </td>
    <td align=center bgcolor=#B7DEE8>
        GPIO224, UART0_TX
    </td>
    <td bgcolor=gray>
        8
    </td>
</tr>

<tr>
    <td bgcolor=gray>
        9
    </td>
    <td colspan=2 align=center bgcolor=black>
        <font color=white>GND</font>
    </td>
    <td colspan=2 align=center bgcolor=black>
        <font color=white>GND</font>
    </td>
    <td colspan=2 align=center bgcolor=black>
        <font color=white>GND</font>
    </td>
    <td colspan=2 align=center bgcolor=black>
        <font color=white>GND</font>
    </td>
    <td align=center bgcolor=#D8E4BC>
        GPIO15
    </td>
    <td align=center bgcolor=#D8E4BC>
        UART RX
    </td>
    <td align=center bgcolor=#CCC0DA>
        PH1
    </td>
    <td align=center bgcolor=#CCC0DA>
        GPIO225, UART0_RX
    </td>
    <td align=center bgcolor=#E6B8B7>
        PH1
    </td>
    <td align=center bgcolor=#E6B8B7>
        GPIO225, UART0_RX
    </td>
    <td align=center bgcolor=#B7DEE8>
        PH1
    </td>
    <td align=center bgcolor=#B7DEE8>
        GPIO225, UART0_RX
    </td>
    <td bgcolor=gray>
        10
    </td>
</tr>

<tr>
    <td bgcolor=gray>
        11
    </td>
    <td bgcolor=#B7DEE8>
        PC14
    </td>
    <td align=center bgcolor=#B7DEE8>
        GPIO78
    </td>
    <td align=center bgcolor=#E6B8B7>
        PI15
    </td>
    <td align=center bgcolor=#E6B8B7>
        GPIO271
    </td>
    <td align=center bgcolor=#CCC0DA>
        PC14
    </td>
    <td align=center bgcolor=#CCC0DA>
        GPIO78
    </td>
    <td align=center bgcolor=#D8E4BC>
        GPIO17
    </td>
    <td align=center bgcolor=#D8E4BC>
        SPI1 CE1
    </td>
    <td align=center bgcolor=#D8E4BC>
        GPIO18
    </td>
    <td align=center bgcolor=#D8E4BC>
        PCM CLK
    </td>
    <td align=center bgcolor=#CCC0DA>
        PC13
    </td>
    <td align=center bgcolor=#CCC0DA>
        GPIO77
    </td>
    <td align=center bgcolor=#E6B8B7>
        PI7
    </td>
    <td align=center bgcolor=#E6B8B7>
        GPIO263
    </td>
    <td align=center bgcolor=#B7DEE8>
        PC13
    </td>
    <td align=center bgcolor=#B7DEE8>
        GPIO77
    </td>
    <td bgcolor=gray>
        12
    </td>
</tr>

<tr>
    <td bgcolor=gray>
        13
    </td>
    <td bgcolor=#B7DEE8>
        PC12
    </td>
    <td align=center bgcolor=#B7DEE8>
        GPIO76
    </td>
    <td align=center bgcolor=#E6B8B7>
        PI6
    </td>
    <td align=center bgcolor=#E6B8B7>
        GPIO262
    </td>
    <td align=center bgcolor=#CCC0DA>
        PC12
    </td>
    <td align=center bgcolor=#CCC0DA>
        GPIO76
    </td>
    <td align=center bgcolor=#D8E4BC>
        GPIO27
    </td>
    <td align=center bgcolor=#D8E4BC>
        <br />
    </td>
    <td colspan=2 align=center bgcolor=black>
        <font color=white>GND</font>
    </td>
    <td colspan=2 align=center bgcolor=black>
        <font color=white>GND</font>
    </td>
    <td colspan=2 align=center bgcolor=black>
        <font color=white>GND</font>
    </td>
    <td colspan=2 align=center bgcolor=black>
        <font color=white>GND</font>
    </td>
    <td bgcolor=gray>
        14
    </td>
</tr>

<tr>
    <td bgcolor=gray>
        15
    </td>
    <td bgcolor=#B7DEE8>
        PC10
    </td>
    <td align=center bgcolor=#B7DEE8>
        74
    </td>
    <td align=center bgcolor=#E6B8B7>
        PI4
    </td>
    <td align=center bgcolor=#E6B8B7>
        GPIO260
    </td>
    <td align=center bgcolor=#CCC0DA>
        PC10
    </td>
    <td align=center bgcolor=#CCC0DA>
        GPIO74
    </td>
    <td align=center bgcolor=#D8E4BC>
        GPIO22
    </td>
    <td align=center bgcolor=#D8E4BC>
        <br />
    </td>
    <td align=center bgcolor=#D8E4BC>
        GPIO23
    </td>
    <td align=center bgcolor=#D8E4BC>
        <br />
    </td>
    <td align=center bgcolor=#CCC0DA>
        PC11
    </td>
    <td align=center bgcolor=#CCC0DA>
        GPIO75
    </td>
    <td align=center bgcolor=#E6B8B7>
        PI5
    </td>
    <td align=center bgcolor=#E6B8B7>
        GPIO261
    </td>
    <td align=center bgcolor=#B7DEE8>
        PC11
    </td>
    <td align=center bgcolor=#B7DEE8>
        GPIO75
    </td>
    <td bgcolor=gray>
        16
    </td>
</tr>

<tr>
    <td bgcolor=gray>
        17
    </td>
    <td colspan=2 align=center bgcolor=yellow>
        3.3V
    </td>
    <td colspan=2 align=center bgcolor=yellow>
        3.3V
    </td>
    <td colspan=2 align=center bgcolor=yellow>
        3.3V
    </td>
    <td colspan=2 align=center bgcolor=yellow>
        3.3V
    </td>
    <td align=center bgcolor=#D8E4BC>
        GPIO24
    </td>
    <td align=center bgcolor=#D8E4BC>
        <br />
    </td>
    <td align=center bgcolor=#CCC0DA>
        PC9
    </td>
    <td align=center bgcolor=#CCC0DA>
        GPIO73
    </td>
    <td align=center bgcolor=#E6B8B7>
        PI3
    </td>
    <td align=center bgcolor=#E6B8B7>
        GPIO259
    </td>
    <td align=center bgcolor=#B7DEE8>
        PC9
    </td>
    <td align=center bgcolor=#B7DEE8>
        GPIO73
    </td>
    <td bgcolor=gray>
        18
    </td>
</tr>

<tr>
    <td bgcolor=gray>
        19
    </td>
    <td bgcolor=#B7DEE8>
        PH7
    </td>
    <td align=center bgcolor=#B7DEE8>
        GPIO231, SPI1_MOSI
    </td>
    <td align=center bgcolor=#E6B8B7>
        PH7
    </td>
    <td align=center bgcolor=#E6B8B7>
        GPIO231, SPI1_MOSI
    </td>
    <td align=center bgcolor=#CCC0DA>
        PH7
    </td>
    <td align=center bgcolor=#CCC0DA>
        GPIO231, SPI1_MOSI
    </td>
    <td align=center bgcolor=#D8E4BC>
        GPIO10
    </td>
    <td align=center bgcolor=#D8E4BC>
        SPI0 MOSI
    </td>
    <td colspan=2 align=center bgcolor=black>
        <font color=white>GND</font>
    </td>
    <td colspan=2 align=center bgcolor=black>
        <font color=white>GND</font>
    </td>
    <td colspan=2 align=center bgcolor=black>
        <font color=white>GND</font>
    </td>
    <td colspan=2 align=center bgcolor=black>
        <font color=white>GND</font>
    </td>
    <td bgcolor=gray>
        20
    </td>
</tr>

<tr>
    <td bgcolor=gray>
        21
    </td>
    <td bgcolor=#B7DEE8>
        PH8
    </td>
    <td align=center bgcolor=#B7DEE8>
        GPIO232, SPI1_MISO
    </td>
    <td align=center bgcolor=#E6B8B7>
        PH8
    </td>
    <td align=center bgcolor=#E6B8B7>
        GPIO232, SPI1_MISO
    </td>
    <td align=center bgcolor=#CCC0DA>
        PH8
    </td>
    <td align=center bgcolor=#CCC0DA>
        GPIO232, SPI1_MISO
    </td>
    <td align=center bgcolor=#D8E4BC>
        GPIO9
    </td>
    <td align=center bgcolor=#D8E4BC>
        SPI0 MISO
    </td>
    <td align=center bgcolor=#D8E4BC>
        GPIO25
    </td>
    <td align=center bgcolor=#D8E4BC>
        <br />
    </td>
    <td colspan=2 align=center bgcolor=gray>
        NC
    </td>
    <td colspan=2 align=center bgcolor=gray>
        NC
    </td>
    <td align=center bgcolor=#B7DEE8>
        PG13
    </td>
    <td align=center bgcolor=#B7DEE8>
        GPIO205
    </td>
    <td bgcolor=gray>
        22
    </td>
</tr>

<tr>
    <td bgcolor=gray>
        23
    </td>
    <td bgcolor=#B7DEE8>
        PH6
    </td>
    <td align=center bgcolor=#B7DEE8>
        GPIO230, SPI1_CLK
    </td>
    <td align=center bgcolor=#E6B8B7>
        PH6
    </td>
    <td align=center bgcolor=#E6B8B7>
        GPIO230, SPI1_CLK
    </td>
    <td align=center bgcolor=#CCC0DA>
        PH6
    </td>
    <td align=center bgcolor=#CCC0DA>
        GPIO230, SPI1_CLK
    </td>
    <td align=center bgcolor=#D8E4BC>
        GPIO11
    </td>
    <td align=center bgcolor=#D8E4BC>
        SPI0 SCLK
    </td>
    <td align=center bgcolor=#D8E4BC>
        GPIO8
    </td>
    <td align=center bgcolor=#D8E4BC>
        SPI0 CE0
    </td>
    <td colspan=2 align=center bgcolor=gray>
        NC
    </td>
    <td colspan=2 align=center bgcolor=gray>
        NC
    </td>
    <td align=center bgcolor=#B7DEE8>
        PG12
    </td>
    <td align=center bgcolor=#B7DEE8>
        GPIO204
    </td>
    <td bgcolor=gray>
        24
    </td>
</tr>

<tr>
    <td bgcolor=gray>
        25
    </td>
    <td colspan=2 align=center bgcolor=black>
        <font color=white>GND</font>
    </td>
    <td colspan=2 align=center bgcolor=black>
        <font color=white>GND</font>
    </td>
    <td colspan=2 align=center bgcolor=black>
        <font color=white>GND</font>
    </td>
    <td colspan=2 align=center bgcolor=black>
        <font color=white>GND</font>
    </td>
    <td align=center bgcolor=#D8E4BC>
        GPIO7
    </td>
    <td align=center bgcolor=#D8E4BC>
        SPI0 CE1
    </td>
    <td align=center bgcolor=#CCC0DA>
        PG8
    </td>
    <td align=center bgcolor=#CCC0DA>
        GPIO200
    </td>
    <td align=center bgcolor=#E6B8B7>
        PI11
    </td>
    <td align=center bgcolor=#E6B8B7>
        GPIO267
    </td>
    <td align=center bgcolor=#B7DEE8>
        PI9
    </td>
    <td align=center bgcolor=#B7DEE8>
        GPIO265
    </td>
    <td bgcolor=gray>
        26
    </td>
</tr>

<tr>
    <td bgcolor=gray>
        27
    </td>
    <td bgcolor=#B7DEE8>
        PC2
    </td>
    <td align=center bgcolor=#B7DEE8>
        GPIO66
    </td>
    <td colspan=2 align=center bgcolor=gray>
        NC
    </td>
    <td colspan=2 align=center bgcolor=gray>
        NC
    </td>
    <td align=center bgcolor=#D8E4BC>
        GPIO0
    </td>
    <td align=center bgcolor=#D8E4BC>
        EEPROM SDA
    </td>
    <td align=center bgcolor=#D8E4BC>
        GPIO1
    </td>
    <td align=center bgcolor=#D8E4BC>
        EEPROM SCL
    </td>
    <td align=center bgcolor=#CCC0DA>
        PG7
    </td>
    <td align=center bgcolor=#CCC0DA>
        GPIO199
    </td>
    <td align=center bgcolor=#E6B8B7>
        PI10
    </td>
    <td align=center bgcolor=#E6B8B7>
        GPIO266
    </td>
    <td align=center bgcolor=#B7DEE8>
        PI10
    </td>
    <td align=center bgcolor=#B7DEE8>
        GPIO266
    </td>
    <td bgcolor=gray>
        28
    </td>
</tr>

<tr>
    <td bgcolor=gray>
        29
    </td>
    <td bgcolor=#B7DEE8>
        PC4
    </td>
    <td align=center bgcolor=#B7DEE8>
        GPIO68
    </td>
    <td colspan=2 align=center bgcolor=gray>
        NC
    </td>
    <td colspan=2 align=center bgcolor=gray>
        NC
    </td>
    <td align=center bgcolor=#D8E4BC>
        GPIO5
    </td>
    <td align=center bgcolor=#D8E4BC>
        GPCLK1
    </td>
    <td colspan=2 align=center bgcolor=black>
        <font color=white>GND</font>
    </td>
    <td colspan=2 align=center bgcolor=black>
        <font color=white>GND</font>
    </td>
    <td colspan=2 align=center bgcolor=black>
        <font color=white>GND</font>
    </td>
    <td colspan=2 align=center bgcolor=black>
        <font color=white>GND</font>
    </td>
    <td bgcolor=gray>
        30
    </td>
</tr>

<tr>
    <td bgcolor=gray>
        31
    </td>
    <td bgcolor=#B7DEE8>
        PI5
    </td>
    <td align=center bgcolor=#B7DEE8>
        GPIO261
    </td>
    <td align=center bgcolor=#E6B8B7>
        PI9
    </td>
    <td align=center bgcolor=#E6B8B7>
        GPIO265
    </td>
    <td align=center bgcolor=#CCC0DA>
        PG6
    </td>
    <td align=center bgcolor=#CCC0DA>
        GPIO198
    </td>
    <td align=center bgcolor=#D8E4BC>
        GPIO6
    </td>
    <td align=center bgcolor=#D8E4BC>
        GPCLK2
    </td>
    <td align=center bgcolor=#D8E4BC>
        GPIO12
    </td>
    <td align=center bgcolor=#D8E4BC>
        PWM0
    </td>
    <td align=center bgcolor=#CCC0DA>
        PG9
    </td>
    <td align=center bgcolor=#CCC0DA>
        GPIO201
    </td>
    <td align=center bgcolor=#E6B8B7>
        PI12
    </td>
    <td align=center bgcolor=#E6B8B7>
        GPIO268
    </td>
    <td align=center bgcolor=#B7DEE8>
        PI6
    </td>
    <td align=center bgcolor=#B7DEE8>
        GPIO262
    </td>
    <td bgcolor=gray>
        32
    </td>
</tr>

<tr>
    <td bgcolor=gray>
        33
    </td>
    <td bgcolor=#B7DEE8>
        PI14
    </td>
    <td align=center bgcolor=#B7DEE8>
        GPIO270
    </td>
    <td colspan=2 align=center bgcolor=gray>
        NC
    </td>
    <td colspan=2 align=center bgcolor=gray>
        NC
    </td>
    <td align=center bgcolor=#D8E4BC>
        GPIO13
    </td>
    <td align=center bgcolor=#D8E4BC>
        PWM1
    </td>
    <td colspan=2 align=center bgcolor=black>
        <font color=white>GND</font>
    </td>
    <td colspan=2 align=center bgcolor=black>
        <font color=white>GND</font>
    </td>
    <td colspan=2 align=center bgcolor=black>
        <font color=white>GND</font>
    </td>
    <td colspan=2 align=center bgcolor=black>
        <font color=white>GND</font>
    </td>
    <td bgcolor=gray>
        34
    </td>
</tr>

<tr>
    <td bgcolor=gray>
        35
    </td>
    <td bgcolor=#B7DEE8>
        PC6
    </td>
    <td align=center bgcolor=#B7DEE8>
        GPIO70
    </td>
    <td align=center bgcolor=#E6B8B7>
        PI1
    </td>
    <td align=center bgcolor=#E6B8B7>
        GPIO257
    </td>
    <td align=center bgcolor=#CCC0DA>
        PC6
    </td>
    <td align=center bgcolor=#CCC0DA>
        GPIO70
    </td>
    <td align=center bgcolor=#D8E4BC>
        GPIO19
    </td>
    <td align=center bgcolor=#D8E4BC>
        PCM FS
    </td>
    <td align=center bgcolor=#D8E4BC>
        GPIO16
    </td>
    <td align=center bgcolor=#D8E4BC>
        SPI1 CE2
    </td>
    <td colspan=2 align=center bgcolor=gray>
        NC
    </td>
    <td colspan=2 align=center bgcolor=gray>
        NC
    </td>
    <td align=center bgcolor=#B7DEE8>
        PG11
    </td>
    <td align=center bgcolor=#B7DEE8>
        GPIO203
    </td>
    <td bgcolor=gray>
        36
    </td>
</tr>

<tr>
    <td bgcolor=gray>
        37
    </td>
    <td bgcolor=#B7DEE8>
        PC15
    </td>
    <td align=center bgcolor=#B7DEE8>
        GPIO79
    </td>
    <td align=center bgcolor=#E6B8B7>
        PI13
    </td>
    <td align=center bgcolor=#E6B8B7>
        GPIO269
    </td>
    <td align=center bgcolor=#CCC0DA>
        PC15
    </td>
    <td align=center bgcolor=#CCC0DA>
        GPIO79
    </td>
    <td align=center bgcolor=#D8E4BC>
        GPIO26
    </td>
    <td align=center bgcolor=#D8E4BC>
        <br />
    </td>
    <td align=center bgcolor=#D8E4BC>
        GPIO20
    </td>
    <td align=center bgcolor=#D8E4BC>
        PCM DIN
    </td>
    <td align=center bgcolor=#CCC0DA>
        PH10
    </td>
    <td align=center bgcolor=#CCC0DA>
        GPIO234, IR_RX
    </td>
    <td align=center bgcolor=#E6B8B7>
        PH10
    </td>
    <td align=center bgcolor=#E6B8B7>
        GPIO234, IR_RX
    </td>
    <td align=center bgcolor=#B7DEE8>
        PH4
    </td>
    <td align=center bgcolor=#B7DEE8>
        GPIO228
    </td>
    <td bgcolor=gray>
        38
    </td>
</tr>

<tr>
    <td bgcolor=gray>
        39
    </td>
    <td colspan=2 align=center bgcolor=black>
        <font color=white>GND</font>
    </td>
    <td colspan=2 align=center bgcolor=black>
        <font color=white>GND</font>
    </td>
    <td colspan=2 align=center bgcolor=black>
        <font color=white>GND</font>
    </td>
    <td colspan=2 align=center bgcolor=black>
        <font color=white>GND</font>
    </td>
    <td align=center bgcolor=#D8E4BC>
        GPIO21
    </td>
    <td align=center bgcolor=#D8E4BC>
        PCM DOUT
    </td>
    <td align=center bgcolor=#CCC0DA>
        PC8
    </td>
    <td align=center bgcolor=#CCC0DA>
        GPIO72
    </td>
    <td align=center bgcolor=#E6B8B7>
        PI2
    </td>
    <td align=center bgcolor=#E6B8B7>
        GPIO258
    </td>
    <td align=center bgcolor=#B7DEE8>
        PC8
    </td>
    <td align=center bgcolor=#B7DEE8>
        GPIO72
    </td>
    <td bgcolor=gray>
        40
    </td>
</tr>

</table>

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
