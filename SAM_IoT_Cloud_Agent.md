# Using the SAM-IoT as a Cloud Agent for an Application Host Processor

## Application Processor Connections to the SAM-IoT Cloud Agent

A total of 5 wires need to be connected between the application host processor and the SAM-IoT development board.  The Command Line Interface (CLI) requires 2 wires for a UART connection and the Dedicated Telemetry Interface (DTI) requires 3 wires for a uni-directional SPI Master-to-Slave connection.  The SPI pins can be easily accessed using the J20 header of the SAM-IoT board (unfortunately the UART pins are not broken out to either of the 2 headers).

<img src=".//media/image119.png"/>

## Sending Application Processor Telemetry Events

The `telemetry` command is executed using the CLI to update any of the available telemetry variables by using a pre-defined index to select the desired one

```bash
>telemetry <index>,<data>
```
NOTE: The number of characters for the CLI command cannot exceed a total of 80 characters

<img src=".//media/image120.png"/>

## Reading/Writing Application Processor Properties

The `property` command is executed using the CLI to write to any property by using a pre-defined index to select the desired one

```bash
>property <index>,<data>
```
NOTE: The number of characters for the CLI command cannot exceed a total of 80 characters

<img src=".//media/image121.png"/>

## Dedicated Telemetry Interface (DTI)

The DTI is implemented on the SAM-IoT configured as an SPI Slave. The DTI should be used when larger telemetry chunks of data need to be sent which cannot be executed using the CLI due to the limitation of the maximum number of characters that the CLI can process.  The external application processor must be configured for SPI Master mode and then connect to SAM-IoT's header using 3 wires (MOSI, SCK, CS).  The SPI Master must send a valid data frame in the correct format (header + payload) for each telemetry update operation.

For the most reliable performance, the SPI Master should adhere to the following operating parameters:

- Interframe delay = 500 msec minimum
- SCK = 30 kHz maximum
- [Recommendation] Issue the `telemetry 0,0` command on the CLI prior to sending an individual data frame (or at the beginning of a group of consecutive data frames).  This command will ensure that the DTI receive buffer has been properly reset prior to receiving the first byte of the data frame

<img src=".//media/image122.png"/>

## Data Frame Format

Each SPI transaction must follow the format for sending out a valid data frame through the DTI.  A valid data frame must start with a 4-byte header followed by the payload data.  The command byte must be equal to the hex value corresponding to ASCII character 'T' or 't' (signifying that the data frame is to be processed as a `telemetry` command)

<img src=".//media/image123.png"/>

For example, the following data frame will update the `telemetry_Str_1` value with a 1KB array of bytes that begins with "START", ends with "END", and every byte in between with consecutive '1's

<img src=".//media/image124.png"/>

## Set up a Test Platform for the DTI using a PIC-IoT Development Board as the SPI Master

Program a [PIC-IoT development board](https://www.microchip.com/en-us/development-tool/PIC-IoT-WMX#additional-summary) with an existing [PIC-IoT_SPI-Master code example](https://github.com/randywu763/MCHP-IoT_SPI) and make the appropriate jumper connections between the J20 headers on both development boards.  Whenever the SW1 button is pressed on the PIC-IoT board, the red LED will toggle whenever a data frame has been sent out on the SPI bus.  Whenever a valid data frame has been received and properly decoded by the SAM-IoT, its yellow LED will toggle.

NOTE: It is strongly recommended to issue the `telemetry 0,0` command on the CLI prior to sending data frames out to the DTI

<img src=".//media/image125.png"/>

## Data Frames Visualization using the IoT Central Application Template

Select the `SAM-IoT WM v2 Properties & Telemetry Data Buckets` dashboard in the IoT Central Template Application to see the last known received values for the 4 telemetry string variables

<img src=".//media/image126.png"/>

## Visualization using the IoT Central Application Template

See each of the raw data messages received by the IoT Central application by clicking on the device and then selecting the `Raw Data` tab

<img src=".//media/image127.png"/>

<img src=".//media/image128.png"/>

<img src=".//media/image129.png"/>
