# WPI ECE 2049 STM32 Adafruit Graphics Library

This repository contains the STM32 project for testing an STM32 port of the [Adafruit GFX](https://github.com/adafruit/Adafruit-GFX-Library) and the [Adafruit ST7735](https://github.com/adafruit/Adafruit-ST7735-Library) libraries designed specifically for the WPI ECE 2049 course. This project is not an original project, rather it is an adaptation of these libraries as they are written and published online. The Adafruit GFX library is distributed with the BSD license while the Adafruit ST7735 library is distributed with the MIT license, both of which allow redistribution of their code within projects given the original copyright notice is included. As such, the original BSD license for the Adafruit GFX library is included in the [LICENSE](LICENSE) file while the original MIT license for the Adafruit ST7735 library is included in the [LICENSE_MIT](LICENSE_MIT) file.

## Installation Instructions

To install and set up this library into a custom project, the following steps should be taken:

1. Download the latest release of the library files from the [Releases Page](https://github.com/WPI-ECE2049-Reimagined-MQP/ECE2049-STM32-Adafruit-Graphics-Library/releases) on the right, or create a clone of the repository using Git.

2. The following folders/files will need to be copied into your project. Please copy them into the same Core/Inc or Core/Src folder that appears in your STM32 project. Note that the file structure in the zipped release file is the same as it appears in the cloned repository for uniformity between any chosen method from Step 1:
    * Core/Inc/graphics (folder)
    * Core/Inc/dwt_module.h (file)
    * Core/Src/graphics (folder)
    * Core/Src/dwt_module.c (file)

3. After copying the files into their spots, open your project's .ioc file in STM32CubeMX program, or create one using the Board selector menu. This library is designed for use in the NUCLEO-H533RE development board without TrustZone activated. 

4. Ensure that the *__instruction cache__* (the *__ICACHE__* module) found in the "System Core/ICACHE" dropdown in the STM32CubeMX program is enabled and set to "2-ways set associative cache" for increased performance

5. Enable the *__I2C1__* peripheral. Note: you *__MUST__* enable the I2C1 peripheral since that peripheral connects to the associated clock and data line pins on the Arduino header pinouts, which is required by the Adafruit 1.8" TFT Display Shield.
    <ol type="a">
        <li>Expand the "Connectivity" dropdown menu in the STM32CubeMX program</li>
        <li>Click on the "I2C1" peripheral in that expanded drop down and set the "I2C" dropdown in the "Mode" panel to "I2C"</li>
        <li>In the "Configuration" panel just below the "Mode" panel, select the "NVIC Settings" pane and check the box for "I2C1 Event interrupt" and "I2C1 Error interrupt". This will enable interrupts for the peripheral that allow proper functionality</li>
        <li>In the same configuration panel, select the "GPIO Settings" pane and ensure that two pins are visible, "I2C1_SCL" and "I2C1_SDA" connected to GPIO pins PB6 and PB7, respectively. Click on each entry and set both GPIO pins to use a pull-up resistor and select "Very High" for the "Output Speed" for the pin. If you can't see the dropdown settings appear underneath the pins, you might have to scroll down within the panel or expand the amount of vertical space in the panel</li>
        <li>Enable GPDMA for sending and receiving data via I2C:
            <ol>
                <li>Expand the "System Core" dropdown and select the "GPDMA1" (General Purpose DMA 1) peripheral</li>
                <li>For Channel 6 in the "Mode" panel, select from the dropdown "Standard Request Mode"</li>
                <li>After selecting the mode from the dropdown, select the "CH6" pane in the "Configuration" panel below the "Mode" panel. </li>
                <li>Ensure that "Circular Mode" is disabled</li>
                <li>Set the dropdown for "Request" to "I2C1_TX". You might have to scroll through the options to find it. If the option is greyed out, then you must enable the I2C peripheral.
                <li>Set the dropdown for "Direction" to "Memory To Peripheral"</li>
                <li>Under the "Source Data Setting" section, enable "Source Address Increment After Transfer"</li>
                <li>For both the "Source Data Setting" and "Destination Data Setting" sections, ensure that "Data Width" is set to Byte, "Burst Length" is set to 1, and the "Allocated Port for Transfer" is set to Port 0</li>
                <li>Keep the default for all other settings</li>
                <li>For Channel 5 in the "Mode" panel, select from the dropdown "Standard Request Mode" to enabled it</li>
                <li>Set the "Request" dropdown to "I2C1_RX" and set the "Direction" dropdown to "Peripheral to Memory"</li>
                <li>Set the "Destination Address Increment After Transfer" setting in the <b><em>Destination Data Setting</em></b> section to "Enabled", NOT the "Source Data Setting" section</li>
                <li>All other settings should be left to their default values</li>
            </ol>
        </li>
    </ol>
6. Enable the *__SPI2__* peripheral. While the number of the peripheral is not as important for this library, these steps operate on the assumption that the SPI2 peripheral is used. Note: If using the BSP driver for serial *printf()* support and on-board user button and LED control, the SPI1 peripheral will be unavailable.
    <ol type="a">
        <li>Expand the "Connectivity" dropdown and select the SPI2 peripheral</li>
        <li>Select from the "Mode" dropdown "Half-Duplex Master". Do not enable the "Hardware NSS" or the "Hardware RDY" signals. Full duplex is not required for this shield unless the SD card slot is enabled, which is currently not supported by this library port.</li>
        <li>Ensure that the "Frame Format" is set to "Motorola" and set the "Data Size" setting to "8 bits" with the "First Bit" setting set to "MSB First"</li>
        <li>Set the prescaler to 2. This will affect the baud rate shown below, which will be set to the correct value when the clock rates are adjusted in a later step</li>
        <li>Ensure that "Clock Polarity (CPOL)" is set to "Low" and that "Clock Phase (CPHA)" is set to "1 Edge"</li>
        <li>Keep the rest of the settings set to their defaults</li>
        <li>Select the "NVIC Settings" pane in the same configuration window and check the box for "SPI2 Global Interrupts"</li>
        <li>Select the "GPIO Settings" pane in the same configuration window and ensure that two pin entries appear named "SPI2_SCK" and "SPI2_MOSI". Ensure that no pullup or pulldown resistors are attached to the pins and set their "Maximum Output Speed" to "Very High"</li>
        <li>Enable GPDMA for SPI transmission</li>
        <ol>
            <li>Expand the "System Core" dropdown and select the GPDMA1 peripheral from the list</li>
            <li>For Channel 7 in the "Mode" panel, select from the dropdown "Standard Request Mode"</li>
            <li>Select the "CH7" pane in the "Configuration" panel below the "Mode" panel</li>
            <li>In the "Request" dropdown, select "SPI2_TX/I2S2_TX" to attach the SPI transmission interface to this channel</li>
            <li>Set the "Prority" to "High" and ensure that the "Direction" dropdown is set to "Memory to Peripheral"/li>
            <li>In the "Source Data Setting" section, set the "Source Address Incrememnt After Transfer" to "Enabled" and ensure that data width is set equal to "Byte". All other settings can remain set to their defaults</li>
        </ol>
    </ol>