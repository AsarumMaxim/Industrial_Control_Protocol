# 1. Introduction
The HART (Highway Addressable Remote Transducer) protocol is primarily used in the field of industrial automation communication. It is designed to send and receive digital information while also supporting the transmission of analog signals (such as 4-20 mA signals). This design allows HART devices to transmit both analog signals and digital data simultaneously, providing more flexible and powerful communication capabilities.

The standard HART transmission is an FSK (Frequency Shift Keying) signal superimposed on a 4-20mA signal, with an alternative being the C8PSK (Coherent 8-Phase Shift Keying) signal, which increases the digital transmission rate of HART. Additionally, there are HART communication methods based on RS-485, IP, TDMA, which are not discussed here.

This article only discusses HART FSK.

# 2. Message Format

## 2.1 HART Byte
As the HART protocol has its own unique physical layer communication method, its transmission at the data link layer adopts a transmission method called a HART byte group. Each transmission will send an 11-bit HART byte, and the data parts of multiple HART bytes combine into a valid HART data frame. (This process can be analogously compared to a long string of data being sent through multiple TCP/IP packets)

The format of a HART byte is as follows:

| **Purpose** | Start Bit    | HART Byte                             | Parity Bit | Stop Bit   |
|-------------|--------------|---------------------------------------|------------|------------|
| **Length**  | 1bit         | 8bit                                  | 1bit       | 1bit       |
| **Description** | Fixed at 0, indicating the start | HART data, starting from LSB (Least Significant Bit) to MSB (Most Significant Bit). | Used for data verification | Fixed at 1, indicating the end |

## 2.2 HART Message Format

The structure is as follows:

| **Purpose** | **Length** | **Description**                        |
|-------------|------------|----------------------------------------|
| Preamble    | 5-20 bytes | 5-20 0xFFs, used for the start of transmission, |
| Start Byte  | 1 byte     | Used to identify the start of the packet          |
| Address     | 1 or 5 bytes | Contains master and slave addresses, 1 byte in short frame, 5 bytes in long frame <br> |
| Expansion   | 0-3 bytes  | Used for potential expansion, length indicated by the actual byte |
| Command     | 1 byte     | Indicates the action of this data               |
| Byte Count  | 1 byte     | Size of the status and data part, in bytes       |
| Status      | 2 bytes    | Only present in messages from slave to master, reports errors and status |
| Data        | 0-253 bytes | Not all commands have data bytes, used for holding data |
| Checksum    | 1 byte     | Vertical parity check, used for error detection   |

### 2.2.1 Preamble

The preamble appears at the beginning of every message. It consists of a series of identical bytes, usually consecutive "FF" bytes (11111111 in binary). The main roles of the preamble include:

- Synchronization: The preamble provides a synchronization signal for the receiving device, helping it determine the start position of the data frame. By identifying this series of repeating patterns, the receiver's decoder can synchronize with the sender's data stream, correctly interpreting the subsequent information (such as start bit, address, command, data, etc.).

- Clearing the Line: The continuous preamble helps clear any noise or interference on the communication line, ensuring the clarity and accuracy of data transmission. This "clearing" function is particularly important for industrial devices operating in field environments, which are often filled with electromagnetic interference.

- Receiver Preparation: The preamble also gives the receiving device enough time to prepare for the incoming data. In HART communication, the receiving device (such as a processor or controller) needs to adjust its reception mechanism to accurately decode the incoming information. The presence of the preamble provides a buffer time for this adjustment.

### 2.2.2 Start Byte

Structure:

| **Purpose** | Address Type                           | Number of Extension Bytes | Physical Layer Type                  | Frame Type                                       |
|-------------|----------------------------------------|---------------------------|--------------------------------------|--------------------------------------------------|
| **Length**  | 1bit                                   | 2bit                      | 2bit                                 | 3bit                                              |
| **Description** | 0: Polling - Byte Address (Short Frame) <br>1: Unique - Byte Address (Long Frame) | Usually 00 | 00: Asynchronous (e.g., FSK) <br>11: Synchronous (e.g., C8PSK) | 001: Burst Frame Response <br>010: Master to Field Device <br>110: Field Device to Master |

### 2.2.3 Address

The address part, 1 byte for short frame, 5 bytes for long frame.

Short Frame:

| **Purpose** | Master Sequence     | Burst Mode     | Empty  | Device Address |
|-------------|---------------------|----------------|--------|----------------|
| **Length**  | 1bit                | 1bit           | 2bit   | 4bit           |
| **Description** | 1: Primary Master<br>0: Secondary Master | 1: Yes<br>0: No | Always 00 |                |

Long Frame:

| **Purpose** | Master Sequence     | Burst Mode     | Extended Device Type | Device ID |
|-------------|---------------------|----------------|----------------------|-----------|
| **Length**  | 1bit                | 1bit           | 14bit                | 24bit     |
| **Description** | 1: Primary Master<br>0: Secondary Master | 1: Yes<br>0: No |                      |           |

### 2.2.4 Extension

The extension part is mainly reserved for future use, planning the following roles:
- Device Identification and Classification: Extension fields allow for more detailed identification and classification of devices connected to the system. These fields enable the system to identify the device type, manufacturer, and other relevant information, which is useful for system configuration and troubleshooting.
- Enhanced Device Information: Some extension fields are used to store additional information about device performance, capabilities, and configuration options. This allows operators to more accurately control devices while optimizing the overall performance of the system.
- Improved Diagnostic Capabilities: Extension fields can contain detailed information about the device's status and health, making remote monitoring and diagnostics of devices possible. This helps to identify problems early, reducing system downtime.
- Higher Data Transmission Efficiency: By utilizing extension fields for data transmission, the HART protocol can provide higher data transmission efficiency and greater data capacity while maintaining backward compatibility.
- Support for New Technologies and Features: As process control technology evolves, new monitoring and control requirements emerge. Extension fields make it possible to support these new technologies and features, ensuring the long-term applicability and flexibility of the HART protocol.

### 2.2.5 Command

Commands are divided into three categories: universal, common practice, and device-specific (proprietary commands). Universal commands are those that all devices using the HART protocol must follow, common practice for common devices, and device-specific for certain manufacturers or models.

Translated from [HART Communication Foundation official website](https://www.fieldcommgroup.org/)

| Command Number | Command Description                                      | Type      |
|----------------|----------------------------------------------------------|-----------|
| 0              | Read Unique Identifier associated with the tag           | Universal |
| 1              | Read Primary Variable                                    | Universal |
| 2              | Read Current and Percent of Range                        | Universal |
| 3              | Read Dynamic Variables and Current                       | Universal |
| 6              | Write Polling Address                                    | Universal |
| 7              | Read Loop Configuration                                  | Universal |
| 8              | Read Dynamic Variable Classification                     | Universal |
| 9              | Read Device Variable with Status                         | Universal |
| 11             | Read Unique Identifier associated with the tag           | Universal |
| 12             | Read Message                                             | Universal |
| 13             | Read Tag, Descriptor, and Date                           | Universal |
| 14             | Read Primary Variable Sensor Information                 | Universal |
| 15             | Read Device Information                                  | Universal |
| 16             | Read Final Assembly Number                               | Universal |
| 17             | Write Message                                            | Universal |
| 18             | Write Tag, Descriptor, and Date                          | Universal |
| 19             | Write Final Assembly Number                              | Universal |
| 20             | Read Long Tag                                            | Universal |
| 21             | Read Unique Identifier associated with the Long Tag      | Universal |
| 22             | Write Long Tag                                           | Universal |
| 38             | Reset Configuration Change Flag                          | Universal |
| 48             | Read Additional Device Status                            | Universal |
| 33             | Read Device Variable                                     | Common Practice |
| 34             | Write Damping Value of Primary Variable                  | Common Practice |
| 35             | Write Range Values of Primary Variable Data written with command 35 updates the settings for 4mA and 20mA in the menu | Common Practice |
| 36             | Set Upper Range Value of Primary Variable                | Common Practice |
| 37             | Set Lower Range Value of Primary Variable                | Common Practice |
| 40             | Enter/Exit Fixed Current Mode                            | Common Practice |
| 41             | Perform Self-Test                                        | Common Practice |
| 44             | Write Units of Primary Variable                          | Common Practice |
| 45             | Trim Loop Current Zero                                   | Common Practice |
| 46             | Trim Loop Current Gain                                   | Common Practice |
| 47             | Write Transfer Function of Primary Variable              | Common Practice |
| 54             | Read Device Variable Information                         | Common Practice |
| 59             | Write Pre-amble Size                                     | Common Practice |
| 71             | Lock Device                                              | Common Practice |
| 76             | Read Device Lock Status                                  | Common Practice |
| 140            | Write Field Statistics Overwrite the device's recorded maximum and minimum values | Device Specific |
| 141            | Read Field Statistics Read the current maximum and minimum values from the device | Device Specific |
| 144            | Write Switch 1 Configuration Write Switch 1 mode, setpoint, deadband, latch settings, and delay settings | Device Specific |
| 145            | Read Switch 1 Configuration Read Switch 1 mode, setpoint, deadband, latch settings, and delay settings | Device Specific |
| 221            | Enable/Disable Write Protection, Modify Password Allows enabling/disabling write protection mode and allows editing of the device password | Device Specific |
| 222            | Read/Write Protection Status Read the device's write protection status | Device Specific |
| 223            | Write Trip Counter Write a 0 – 9999 unsigned 16-bit integer to the trip counter associated with Switch 1 and Switch 2 | Device Specific |
| 224            | Toggle 1 and 2 Read the value of the trip counter associated with Switch 1 and Switch 2, the value being a 0 – 9999 unsigned 16-bit integer | Device Specific |
| 225            | Manual Reset Reset one or more switches in latched state | Device Specific |
| 226            | Read Switch Latch Status Read the latch status of one or more switches | Device Specific |
| 244            | Write Switch 2 Configuration Write Switch 2 mode, setpoint, deadband, latch settings, and delay settings | Device Specific |
| 245            | Read Switch 2 Configuration Read Switch 2 mode, setpoint, deadband, latch settings, and delay settings | Device Specific |
| 246            | Write Blockage Port Settings                             | Device Specific |
| 247            | Read Blockage Port Settings                              | Device Specific |
| 248            | Write Offset and Range                                   | Device Specific |
| 249            | Read Offset and Range                                    | Device Specific |

### 2.2.6 Status
The status consists of a 1-byte response code and a 1-byte device status code.
#### 2.2.6.1 Response Code

During normal communication, the highest bit is 0, the response code is:

| Response Code | Meaning                                      |
|---------------|----------------------------------------------|
| 0x00          | Execution successful                         |
| 0x02          | Wrong command                                |
| 0x03          | Setting parameter too large                  |
| 0x04          | Setting parameter too small                  |
| 0x05          | Receiving too few data                       |
| 0x06          | Proprietary command error                    |
| 0x07          | In write-protect mode                        |
| 0x08          | 1. Update failed 2. Set to approximate value 3. Delayed response |
| 0x09          | 1. Lower limit value too large 2. Incorrect current mode |
| 0x0a          | 1. Lower limit value too small 2. Invalid local lock |
| 0x0b          | 1. Upper limit value too large 2. Multi-drop mode 3. Invalid device variable code 4. Adjustment out of range 5. Cannot lock locally |
| 0x0c          | 1. Upper limit value too small 2. Invalid unit code 3. Invalid mode selection 4. Invalid slot number |
| 0x0d          | 1. Upper/lower limit value out of range 2. Calculation error 3. Invalid command number |
| 0x0e          | 1. Range too small 2. Setting the lower limit value causes the upper limit value to change and exceed the sensor limit |
| 0x0f          | Invalid analog channel number                |
| 0x10          | Access restricted                            |
| 0x11          | Invalid device variable index                |
| 0x12          | Invalid unit code                            |
| 0x13          | Unreasonable application of device variable  |
| 0x14          | Invalid extended command number              |
| 0x1c          | Unsupported unit code                        |
| 0x20          | Busy                                         |
| 0x21          | Delayed response start                       |
| 0x22          | Delayed response in progress                 |
| 0x40          | Command cannot be executed                   |

During communication failure, the highest bit is 1, the response code is:

| Response Code | Meaning                 |
|---------------|-------------------------|
| 0xc0          | Parity error on receive |
| 0xa0          | Receive buffer data overwrite error |
| 0x90          | Stop bit not received error |
| 0x88          | Checksum error           |
| 0x82          | Receive buffer overflow  |

The specific meaning is affected by different commands, see the foundation's materials.

#### 2.2.6.2 Device Status Code

| Status Code | Meaning                            |
|-------------|------------------------------------|
| 0x80        | Device malfunction                 |
| 0x40        | Configuration parameter changed    |
| 0x20        | Device cold start                  |
| 0x08        | Loop current fixed mode            |
| 0x04        | Loop current saturated             |
| 0x02        | Device variable (not mapped to primary variable) out of limit |
| 0x01        | Primary variable out of limit      |

### 2.2.7 Data

The format of the data part depends on the command, all defined by different commands.
