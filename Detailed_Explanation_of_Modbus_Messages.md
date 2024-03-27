[中文版](https://github.com/AsarumMaxim/Industrial_Control_Protocol/blob/main/Modbus%E6%8A%A5%E6%96%87%E8%AF%A6%E8%A7%A3.md)
[English](https://github.com/AsarumMaxim/Industrial_Control_Protocol/blob/main/Detailed_Explanation_of_Modbus_Messages.md)

This article was translated by ChatGPT.

Modbus is a serial communication protocol, originally developed by Modicon (now part of Schneider Electric) in 1979 for use with its PLCs (Programmable Logic Controllers). Modbus has become a widely used communication protocol in the industrial field, especially for monitoring and control systems. The Modbus protocol supports multiple communication methods, including RTU (Remote Terminal Unit mode), TCP/IP, and ASCII (American Standard Code for Information Interchange).

This article mainly introduces the message structures of Modbus RTU, Modbus TCP, and Modbus ASCII.

# 1.Modbus RTU
## 1.1 Introduction
Modbus RTU (Remote Terminal Unit) is a widely used protocol in serial communication, mainly applied between devices in the industrial field. This protocol is based on a master/slave (or client/server) architecture, allowing a master (usually called Master) to communicate with multiple slaves (usually called Slave). In the Modbus RTU protocol, data is transmitted in binary form, making communication more efficient.
## 1.2 Message Format
A typical Modbus RTU message structure is as follows:

| **Purpose** | Device Address                                                 | Function Code         | Data                                | CRC Check           |
| ----------- | ------------------------------------------------------------ | --------------------- | ----------------------------------- | ------------------- |
| **Length**  | 8bit                                                          | 8bit                  | Variable (0 to 252 8bit)            | 16bit               |
| **Description** | Each slave has a unique address. The address range is from 0 to 247. Address 0 is the broadcast address, sending messages to all slaves, but slaves will not respond to broadcast messages. | Used to specify the type of operation the master requires the slave to perform. | The length of the data part is variable, containing the specific parameters of the command, the exact format and length depend on the function code. | Used to check if there is an error in the data during transmission. |
## 1.3 Message Details
### 1.3.1 Device Address
#### 1.3.1.1 Address Range
- In Modbus RTU, each slave is assigned a unique address for identification on the network.
- The address is an 8-bit value, ranging from 0 to 247.
- Typically, address 0 is reserved for broadcast commands, i.e., commands sent to all devices on the network.
- Addresses 248 to 255 are usually reserved for special functions or future use.
#### 1.3.1.2 Address Configuration
- The device address usually needs to be configured before the device is connected to the Modbus network.
- Different device manufacturers may provide different methods for setting these addresses, such as through DIP switches, software interfaces, or directly via Modbus commands.
#### 1.3.1.3 Communication Process
- When the master sends a command to a slave, the master includes the slave's address at the beginning of the message. After receiving the message, the slave first checks the message address; if the message is for itself, it will execute it; otherwise, it will discard it.
- For the broadcast address (0), all devices will execute the command.
#### 1.3.1.4 Address Conflict
- If two or more devices are set to the same address, it will lead to address conflict and consequently, communication failure.
### 1.3.2 Function Code

In the Modbus standard protocol, function codes are divided into three categories: public function codes, custom function codes, and reserved function codes.
- Public function codes are those confirmed by the Modbus Association and have publicly available documentation. They are clearly defined in the documentation to ensure uniqueness.
- Custom function codes are function codes customized by manufacturers (users) and do not guarantee uniqueness.
- Reserved function codes are function codes used when the message format is not given, which are no longer used for public use.
(The difference between public function codes and custom function codes can be roughly compared to the comparison between well-known ports and registered ports in computing.)
#### 1.3.2.1 Reading Function Codes

- **01 (0x01)**: Read Coils - Used to read the current state (ON/OFF) of a group of logical coils.
- **02 (0x02)**: Read Discrete Inputs - Used to read the state (ON/OFF) of a group of discrete inputs.
- **03 (0x03)**: Read Holding Registers - Used to read the binary content of a group of holding registers.
- **04 (0x04)**: Read Input Registers - Used to read the binary content of a group of input registers.

#### 1.3.2.2 Writing Function Codes
- **05 (0x05)**: Write Single Coil - Used to write the state (ON/OFF) of a single logical coil.
- **06 (0x06)**: Write Single Register - Used to write data to a single holding register.
- **15 (0x0F)**: Write Multiple Coils - Used to write the state of a group of logical coils.
- **16 (0x10)**: Write Multiple Registers - Used to write data to a group of holding registers.

#### 1.3.2.3 Diagnostic Function Codes
- **08 (0x08)**: Diagnostic - This group of function codes is used to diagnose the state of the communication link and test and diagnose Modbus devices.
#### 1.3.2.4 Special Function Codes
- **17 (0x11)**: Report Slave ID - Returns information about the device, such as operational status and identification information.
- **22 (0x16)**: Mask Write Register - Allows the user to modify the content of holding registers without changing the content of unspecified bits.
- **23 (0x17)**: Read/Write Multiple Registers - Allows for simultaneous reading and writing operations.
#### 1.3.2.5 Exception Codes
- **Exception Function Codes**: The exception function code is **the normal function code + 0x80**, such as an error in writing a single register returning an exception code of 0x86.
### 1.3.3 Data
The data part mainly contains the specific content of the command, its structure and length depend on the different function codes. This article only lists a few common function codes, and the structure and function of the data part corresponding to them:

#### 1.3.3.1 Reading Holding Registers (Function Code 03)

**Master sends data part**:
The master sends data with function code 03, indicating that it wants to read the contents of 3 registers starting from 0x0032.

| Purpose     | Register Start Address-High | Register Start Address-Low | Number of Registers-High | Number of Registers-Low |
| ----------- | -------------------------- | -------------------------- | ----------------------- | ----------------------- |
| **Length**  | 8bit                       | 8bit                       | 8bit                     | 8bit                     |
| **Example** | 0x00                       | 0x32                       | 0x00                     | 0x03                     |

**Slave replies data part**:
The slave sends data with function code 03, indicating that the total data read is 6 bytes, which are 0x00, 0x01, 0x00, 0x02, 0x00, 0x03, i.e., the data read from the 3 registers is 0x0001, 0x0002, 0x0003.

| Purpose     | Return Byte Count | Register 1 Data-High | Register 1 Data-Low | Register 2 Data-High | Register 2 Data-Low | Register 3 Data-High | Register 3 Data-Low |
| ----------- | ----------------- | -------------------- | ------------------- | -------------------- | ------------------- | -------------------- | ------------------- |
| **Length**  | 8bit              | 8bit                 | 8bit                | 8bit                 | 8bit                | 8bit                 | 8bit                |
| **Example** | 0x06              | 0x00                 | 0x01                | 0x00                 | 0x02                | 0x00                 | 0x03                |
#### 1.3.3.2 Writing a Single Register (Function Code 06)

The data sent by the master and the reply from the slave for function code 06 are the same. For example, if the master wants to write 0x0001 into register 0x0032, the data would be:

|Purpose|Register Address-High|Register Address-Low|Write Data-High|Write Data-Low|
|---|---|---|---|---|
|**Length**|8bit|8bit|8bit|8bit|
|**Example**|0x00|0x32|0x00|0x01|

#### 1.3.3.3 Writing Multiple Registers (Function Code 16)

**Master sends data part**: Starting from register 0x0032, write data to 3 consecutive registers, with the data being 0x0001, 0x0002, 0x0003 respectively.

|Purpose|Register Start Address-High|Register Start Address-Low|Number of Registers-High|Number of Registers-Low|Register 1 Data-High|Register 1 Data-Low|Register 2 Data-High|Register 2 Data-Low|Register 3 Data-High|Register 3 Data-Low|
|---|---|---|---|---|---|---|---|---|---|---|
|**Length**|8bit|8bit|8bit|8bit|8bit|8bit|8bit|8bit|8bit|8bit|
|**Example**|0x00|0x32|0x00|0x03|0x00|0x01|0x00|0x02|0x00|0x03|

**Slave replies data part**: Starting from register 0x0032, data is written to 3 consecutive registers.

|Purpose|Register Start Address-High|Register Start Address-Low|Number of Registers-High|Number of Registers-Low|
|---|---|---|---|---|
|**Length**|8bit|8bit|8bit|8bit|
|**Example**|0x00|0x32|0x00|0x03|

#### 1.3.3.4 Exception Data (Exception Function Codes)

There can be three types of exceptions in Modbus communication:

1. Due to communication faults or other reasons, the slave does not receive the information sent by the master, then the master will process it as a timeout.
2. The slave receives the message, but the message is wrong (CRC check does not pass), the slave will discard the message, and the master will process it as a timeout.
3. The slave receives the message, but the operation requested by the message cannot be implemented (such as the function code does not exist, the register range is not correct, etc.), the slave will return a response message containing the exception code.

| Purpose     | Exception Code |
| ----------- | -------------- |
| **Length**  | 8bit           |
| **Example** | 0x04           |

Common exception codes are as follows

|Exception Code|Name|Cause|
|---|---|---|
|01 (0x01)|Illegal Function Code|The slave does not support the function code.|
|02 (0x02)|Illegal Data Address|There is no corresponding register in the slave.|
|03 (0x03)|Illegal Data Value|Data is out of available range or unavailable.|
|04 (0x04)|Slave Device Failure|The slave has encountered an unknown error.|
|05 (0x05)|Acknowledge|The slave has accepted the command and is processing it, to avoid a timeout error.|
|06 (0x06)|Slave Device Busy|The slave device is processing a long-duration command.|
|07 (0x07)|Negative Acknowledge|The slave cannot perform the command from the master.|
|08 (0x08)|Memory Parity Error|The extended file area failed to pass the consistency check.|

### 1.3.4 CRC Check

The commonly used CRC check in the Modbus RTU protocol uses the CRC-16 algorithm, with the specific polynomial being `0x8005` (or its binary form `1000 0000 0000 0101`), and the initial value is `0xFFFF`.

The basic steps of CRC check are as follows:

1. **Preset**: The CRC register is preset to `0xFFFF`.
2. **Data Input**: All bytes in the message except for the CRC checksum (including device address, function code, and data) are processed in order.
3. **Calculation**: For each byte, from the highest to the lowest bit, perform an XOR operation with the current value of the CRC register. If the result's highest bit is 1, then shift the register value to the left by one bit and perform an XOR operation with `0x8005`; if the highest bit is 0, just shift to the left by one bit. Repeat this process until all 8 bits are processed. Then proceed to the next byte until all bytes are calculated.
4. **Result**: The final value in the CRC register is the CRC checksum, which is usually converted to a Little-Endian format before being attached to the end of the message.

When the recipient receives the message, it will use the same CRC calculation process on the entire message (including the CRC checksum). If the message has not been tampered with, the calculation result should be `0x0000` (considering the inclusion of the CRC code and calculation rules). If the result is not `0x0000`, it indicates that the message may have been tampered with or an error occurred during transmission.

# 2.Modbus TCP

## 2.1 Introduction

Modbus TCP is an extension based on the Modbus RTU protocol, which is a communication protocol used on Ethernet. Compared to Modbus RTU, the main difference in the message format of Modbus TCP is the addition of an MBAP header (Modbus Application Protocol header) at the beginning of the message for transmission over TCP/IP networks.

## 2.2 Message Format

|**Purpose**|**Transaction Identifier**|**Protocol Identifier**|**Length Field**|**Unit Identifier**|**Function Code**|**Data**|
|---|---|---|---|---|---|---|
|**Length**|16bit|16bit|16bit|8bit|8|Variable (0 to 252 8bit)|
|**Description**|Used to identify the correspondence between requests and responses. Each request initiated by the client is assigned a unique transaction identifier, and the server uses the same identifier in its response.|Used to identify the upper-layer protocol, fixed at 0x0000.|Indicates the total length of the unit identifier, function code, and data that follow, in bytes.|Used to identify the remote server on a Modbus gateway when connecting to a Modbus network, specifying the type of operation the master requires the slave to perform.|The length of the data part is variable, containing the specific parameters of the command, the exact format and length depend on the function code.||

The **Transaction Identifier**, **Protocol Identifier**, **Length Field**, and **Unit Identifier** together make up the MBAP header.

## 2.3 Message Details

### 2.3.1 Transaction Identifier

- Used to identify the correspondence between requests and responses. Each request initiated by the client is assigned a unique transaction identifier, and the server uses the same identifier in its response. In an environment with concurrent requests, the transaction identifier is particularly important.
- The transaction identifier is usually generated by the requesting side, through increasing, random, and other different ways of generation.

### 2.3.2 Protocol Identifier

- The protocol identifier is used to identify the upper-layer protocol. In standard Modbus TCP applications, this value is set to 0x0000, indicating the use of the Modbus protocol.
- Although the protocol identifier is usually set to 0, its existence provides Modbus TCP with the possibility of extension. This means that in the future, if necessary, Modbus TCP can support protocols other than Modbus without changing the existing architecture.

### 2.3.3 Length Field

Indicates the total length of the unit identifier, function code, and data that follow, in bytes. For example, if the length field is 0x0008, then the length of the subsequent parts is 8 bytes.

### 2.3.4 Unit Identifier

- In a pure Modbus TCP network, the unit identifier is usually set to 0 or 255. This is because, in such an environment, the IP address is sufficient to distinguish different devices, and the unit identifier does not play a role in distinguishing devices.
- In a Modbus TCP to RTU/ASCII gateway, a Modbus TCP request is sent through the network to a gateway device, which then converts this request into Modbus RTU or ASCII format and sends it to the specified slave via serial communication. In this case, the unit identifier is used to tell the gateway which slave the request should be forwarded to.

### 2.3.5 Function Code

Same as Modbus RTU

### 2.3.6 Data

Same as Modbus RTU

# 3. Modbus ASCII

## 3.1 Introduction

Modbus ASCII (American Standard Code for Information Interchange) message format is a variant of the Modbus protocol that allows devices to communicate in a text-readable format. This format is particularly suitable for applications where speed is not very critical and those that require data to be inspected by human eyes.

## 3.2 Message Format

|**Purpose**|**Start Character**|Device Address|Function Code|Data|**Checksum**|End Character|
|---|---|---|---|---|---|---|
|**Length**|1 ASCII character|2 ASCII characters|2 ASCII characters|Variable (0-504 ASCII characters, should be an even number)|2 ASCII characters|2 ASCII characters|
|**Description**|Begins with a colon (":") character, represented in ASCII code as 0x3A|Each slave has a unique address. The address range is from 0 to 247. Address 0 is the broadcast address, sending messages to all slaves, but slaves will not respond to broadcast messages.|Used to specify the type of operation the master requires the slave to perform.|The length of the data part is variable, containing the specific parameters of the command, the exact format and length depend on the function code.|Used to check if there is an error in the data during transmission.|Each message ends with a carriage return and line feed character (CR LF, ASCII code 0x0D and 0x0A)|

In ASCII mode, each character occupies 10 bits, and the character format is as follows:

|**Purpose**|Start Bit|Data Bits|Parity Bit|Stop Bit|
|---|---|---|---|---|
|**Length**|1bit|7bit|1bit|1bit|
|Note|Marks the beginning of data transmission|The actual data content, i.e., the displayed ASCII code|Used for error detection|Marks the end of data transmission|

## 3.3 Message Details

### 3.3.1 Start Character

Each Modbus ASCII message begins with a colon, indicating the start of a new message to the receiving device.

### 3.3.2 Device Address

Since each byte in Modbus ASCII is represented by two ASCII characters, the device address is no exception. For example, if the device address is 17 (decimal), it will be converted to hexadecimal 11, and then represented in the ASCII message as two characters "11".

### 3.3.3 Function Code

The same as Modbus RTU, that is, the function code's hexadecimal number is displayed using ASCII characters.

### 3.3.4 Data

The same as Modbus RTU, that is, the function code's hexadecimal number is displayed using ASCII characters.

### 3.3.5 Checksum

Modbus ASCII uses a simple checksum mechanism called LRC (Longitudinal Redundancy Check). The purpose of LRC is to ensure the integrity and accuracy of data during transmission. The LRC checksum is calculated by summing the ASCII values of all characters in the message. The steps for calculating LRC are as follows:

1. **Initialize LRC**: The initial value of LRC is 0x00.
2. **Calculate Checksum**: Add up the ASCII values of all characters in the message, except for the starting colon and the ending carriage return and line feed characters (in reality, they are the hexadecimal numbers corresponding to the ASCII character pairs), converting them into bytes (i.e., converting the ASCII character pairs representing hexadecimal numbers into bytes) and then adding them to the LRC. If the sum exceeds the range of a byte (i.e., exceeds 0xFF), only the low 8 bits of the result are kept.
3. **Invert and Add One**: After summing, invert the value of LRC (i.e., 0xFF - LRC), then add 1. The final result is the LRC checksum to be appended to the end of the message when sending.
4. **Processing When Sending the Message**: The calculated LRC checksum needs to be converted into two ASCII characters and appended to the end of the message, followed by the ending carriage return and line feed characters. This way, the receiving side can use the same method to calculate the checksum and compare it with the received checksum to verify the data's integrity and accuracy.

### 3.3.6 End Character

In Modbus ASCII mode, the end character of each message consists of two characters: CR (Carriage Return) and LF (Line Feed). In ASCII encoding, CR has a hexadecimal value of 0x0D, and LF has a hexadecimal value of 0x0A. Therefore, each Modbus ASCII message ends with this character sequence: 0x0D0x0A.

- CR (Carriage Return): In the typewriter era, this operation would move the typewriter head back to the beginning of the line. In computer text files, its function depends on the system but is often used to indicate the end of a line.
- LF (Line Feed): In the typewriter era, this operation would roll the paper up by one line. In computer text files, it is usually used to indicate the start of a new line. In Modbus ASCII protocol, using CR and LF together as the message end character ensures that no matter which operating system the receiving device is on, it can correctly identify the end of the message and proceed with the appropriate processing.
