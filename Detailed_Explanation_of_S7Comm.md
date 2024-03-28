

The S7 protocol is a proprietary protocol designed by Siemens for communication with its S7 series PLCs (Programmable Logic Controllers). The S7 protocol is mainly used for communication between Siemens PLCs and between PLCs and other devices. This protocol supports various modes of communication, such as MPI (Multi-Point Interface), PROFIBUS, and Industrial Ethernet, etc. The message structure of the S7 protocol is relatively complex and can be divided into multiple layers.

# 1. Introduction
Compared to the OSI reference model, layers one to four (i.e., the physical layer, data link layer, network layer, transport layer) are the same as conventional TCP/IP communication, which will not be elaborated here. The fifth layer (session layer) is implemented through S7's TPKT, the sixth layer (presentation layer) through COTP, and S7Comm is at the seventh layer (application layer).

The overall structure of the S7 protocol is shown in the diagram below (excluding the TCP/IP part, ditto)

| TPKT Header | COTP Header | S7Comm Header | S7Comm Parameters | Data |
| ----------- | ----------- | ------------- | ----------------- | ---- |

# 2. TPKT
The TPKT header message format is as follows:

| **Purpose** | Version | Reserved | Length |
| ----------- | ------- | -------- | ------ |
| **Length** | 1 byte | 1 byte | 1 byte |
| **Description** | Version information | Reserved (value is 0x00) | Total length of TPKT, COTP, and S7 three-layer protocol (in bytes) |

# 3. COTP
COTP is divided into two types, one is the COTP connection packet, and the other is the COTP function packet.

1. **COTP Connection Packet**: The COTP connection packet is mainly used to establish, maintain, and disconnect the connection of the COTP protocol layer. This process includes steps such as connection request (CR-Connect Request), connection confirmation (CC-Connect Confirm), disconnection request (DR-Disconnect Request), and disconnection confirmation (DC-Disconnect Confirm). Through these connection packets, COTP can establish a stable and reliable session between the communication parties, ensuring the sequential transmission and integrity of data.

2. **COTP Function Packet**: The COTP function packet is used for data transmission and control information exchange on an established COTP connection. This includes the data transfer (DT-Data Transfer) function, through which user data is sent; as well as some control functions, such as flow control, reset, error reporting, etc. The function packet allows COTP to provide efficient and reliable data exchange services on top of the connection.

In summary, **COTP Connection Packet** is mainly used for establishing and managing connections, while **COTP Function Packet** is used for actual data transmission and control operations on these connections.

## 3.1 COTP Connection Packet


| **Purpose** | Length | PDU Type | Destination Reference | Source Reference | Options | Parameters |
| ----------- | ------ | -------- | --------------------- | ---------------- | ------- | ---------- |
| **Length** | 1 byte | 1 byte | 2 bytes | 2 bytes | 1 byte | Variable |
| **Description** | Length of COTP following the length part | Indicates the type of PDU (Protocol Data Unit) | Provided by the receiver during connection establishment, used to identify a unique reference number for the target or session. In communication after connection establishment, this field is used to ensure that packets are delivered to the correct session. | Provided by the initiator during a connection request, used to identify a unique reference number for the source or session. This is also used to ensure that both communication parties exchange data in the correct session. | Depending on the PDU type, may contain additional control information | Additional parameters or control information, the presence and format of this information depend on the PDU type and options field. |

The PDU type has the following options:
- 0x1: ED Expedited Data, expedited data
- 0x2: EA Expedited Data Acknowledgement, expedited data acknowledgment
- 0x4: UD, user data
- 0x5: RJ Reject, rejection
- 0x6: AK Data Acknowledgement, data acknowledgment
- 0x7: ER TPDU Error, TPDU error
- 0x8: DR Disconnect Request, disconnection request
- 0xC: DC Disconnect Confirm, disconnection confirmation
- 0xD: CC Connect Confirm, connection confirmation
- 0xE: CR Connect Request, connection request
- 0xF: DT Data, data transfer

## 3.1 COTP Function Packet


| **Purpose** | Length | PDU Type | Options |
| ----------- | ------ | -------- | ------- |
| **Length** | 1 byte | 1 byte | 1 byte |
| **Description** | Length of COTP following the length part | Indicates the type of PDU (Protocol Data Unit) | Depending on the PDU type, may contain additional control information |

# 4. S7Comm

The message structure is as follows:

| **Purpose** | S7 Header | Parameters | Data |
| ----------- | --------- | ---------- | ---- |
| **Length** | 10 or 12 bytes | Variable | Variable |
| **Description** | Message header | Contains the control information required for specific operations | The data part is optional and depends on the type of operation being performed. |

## 4.1 S7 Header

The message structure is as follows:


| **Purpose** | Protocol ID | Message Type | Redundancy Identification | Protocol Data Unit Reference | Parameter Length | Data Length | Error Type | Error Code |
| ----------- | ----------- | ------------ | ------------------------- | ---------------------------- | ---------------- | ----------- | ---------- | ---------- |
| **Length** | 1 byte | 1 byte | 2 bytes | 2 bytes | 2 bytes | 2 bytes | 1 byte | 1 byte |
| **Description** | Usually 0x32 | Indicates the message type | Used for redundancy management, helps identify and manage possible duplicate packets during communication, ensuring data consistency and accuracy. Usually 0x0000 | This is an identifier used to match requests with their corresponding responses. In complex communication processes, this field ensures that each request receives the correct response. | Length of the parameter field | Length of the data part | Optional, only exists in Ack-Data messages, if an error occurs during transmission, this field will identify the type of error | Optional, only exists in Ack-Data messages, there may be multiple error codes under each error type, indicating the specific reason for the error |

The PDU type has the following values:
- 0x01- Job Request: Requests sent by the master station (e.g., reading/writing memory, reading/writing blocks, starting/stopping devices, communication settings)
- 0x02- Ack: Simple acknowledgment sent by the slave station without a data field (never seen it sent by S300 / S400 devices)
- 0x03- Ack-Data: Acknowledgment with an optional data field, containing a reply to the job request
- 0x07- Userdata: Extension of the original protocol, the parameter field contains request/response id, (used for programming/debugging, SZL reading, security functions, time setting, cyclic reading, etc.)

## 4.2 Parameters, Data

Parameters depend on the message type, the parameter message's first byte is the function code (here we only introduce the message formats for establishing communication, reading data, and writing data)

### 4.2.1 Establishing Communication
A handshake message is sent at the beginning of each session. It is used to negotiate the size of the Ack queue and the maximum PDU length, both parties declare their supported values to ensure successful data transmission. The PDU and Ack queue length fields both follow big-endian data representation.

The message format for sending a request:
S7 header message type 0x01- Job Request, the function code is 0xF0

| **Purpose** | Function Code | Redundancy Data | Ack Queue Size (Caller) | Ack Queue Size (Callee) | Negotiated PDU Length |
| ----------- | ------------ | --------------- | ----------------------- | ----------------------- | --------------------- |
| **Length** | 1 byte | 1 byte | 2 bytes | 2 bytes | 2 bytes |
| **Description** | 0xF0 | 0x00 | | | |

The message format for responding to a request is the same as the request message format, but the S7 header message type is 0x03- Ack-Data.

### 4.2.2 Reading Data

#### 4.2.2.1 Request Message
S7 header message type 0x01- Job Request, the function code is 0x04. The request part only has a parameter part, no data part.


| **Purpose** | Function Code | Item Count | Item 1 | Item 2 | ¡­ | Item n |
| ----------- | ------------ | ---------- | ------ | ------ | - | ------ |
| **Length** | 1 byte | 1 byte | | | | |
| **Description** | 0x04 | | | | | |

Each item's message format is as follows:

| **Purpose** | Variable Specification | Length Specification | Address Specification | Data Specification | Data Length | DB Number | Area | Address |
| ----------- | ---------------------- | -------------------- | --------------------- | ------------------ | ----------- | --------- | ---- | ------- |
| **Length** | 1 byte | 1 byte | 1 byte | 1 byte | 2 bytes | 2 bytes | 1 byte | 3 bytes |
| **Description** | Usually 0x12 | Length of the remaining part of this item | Format of IDS's address specification | Size and variable type of data transfer | Requested data length | If not accessing a DB area, this is 0x0000 | Type of data area | Address of the data to be read |

- Common values for the address specification are as follows:

| Hexadecimal Code | Identifier | Description |
| ---------------- | ---------- | ----------- |
| 0x10 | S7ANY | S7-Any pointer similar to address data, e.g., DB1.DBX10.2 |
| 0x13 | PBC-R_ID | PBC's R_ID |
| 0x15 | ALARM_LOCKFREE | Alarm lock/release data set |
| 0x16 | ALARM_IND | Alarm indication data set |
| 0x19 | ALARM_ACK | Alarm acknowledgment message data set |
| 0x1a | ALARM_QUERYREQ | Alarm query request data set |
| 0x1c | NOTIFY_IND | Notification indication data set |
| 0xa2 | DRIVEESANY | Seen in Drive ES Starter, routed through S7 |
| 0xb2 | 1200SYM | S7-1200 symbolic address mode |
| 0xb0 | DBREAD | DB block read, only seen on S7-400 |
| 0x82 | NCK | Sinumerik NCK HMI access |

Common values for data specification are as follows:

| Value | Type | Description | Unit of Length |
| ----- | ---- | ----------- | -------------- |
| 0 | NULL | None | - |
| 3 | BIT | Bit access | Bit |
| 4 | BYTE/WORD/DWORD | Byte/word/dword access | Bit |
| 5 | INTEGER | Integer access | Bit |
| 6 | DINTEGER | Double integer access | Byte |
| 7 | REAL | Real number access | Byte |
| 9 | OCTET STRING | Octet string | Byte |

Common values for data area are as follows:

| Hexadecimal Code | Description |
| ---------------- | ----------- |
| 0x03 | 200 series system information |
| 0x05 | 200 series system flags |
| 0x06 | 200 series analog input |
| 0x07 | 200 series analog output |
| 0x80 | Direct access to peripherals |
| 0x81 | Input (I) |
| 0x82 | Output (Q) |
| 0x83 | Internal flag (M) |
| 0x84 | Data block (DB) |
| 0x85 | Instance data block (DI) |
| 0x86 | Local variable (L) |
| 0x87 | Unknown (V) |
| 0x1c | S7 counter |
| 0x1d | S7 timer (T) |
| 0x1e | Description unclear |
| 0x1f | IEC counter (200 series) |
| 0x20 | IEC timer (200 series) |

#### 4.2.2.2 Response Message

The response message S7 header message type is 0x03- Ack-Data. The response message has both a parameter part and a data part.

Parameter part:

| **Purpose** | Function Code | Item Count |
| ----------- | ------------ | ---------- |
| **Length** | 1 byte | 1 byte |
| **Description** | 0x04 | How many following data parts |

Data part:

| **Purpose** | Return Code | Data Specification | Data Length | Returned Data | Padding Data |
| ----------- | ----------- | ------------------ | ----------- | ------------- | ------------ |
| **Length** | 1 byte | 1 byte | 2 bytes | Variable | Variable |
| **Description** | Indicates the type of return information | Size and variable type of data transfer | Length of subsequent data | Data that was read | If the returned data is less than the data length, fill a portion of 00 until this |

Common return code types:

| Hexadecimal Code | English Description | Chinese Description |
| ---------------- | ------------------- | ------------------- |
| 0x00 | Reserved | Undefined, reserved |
| 0x01 | Hardware error | Hardware error |
| 0x03 | Accessing the object not allowed | Object access not allowed |
| 0x05 | Invalid address | Invalid address, the required address exceeds the limits of this PLC |
| 0x06 | Data type not supported | Data type not supported |
| 0x07 | Data type inconsistent | Date type inconsistent |
| 0x0a | Object does not exist | Object does not exist |
| 0xff | Success | Success |

### 4.2.3 Writing Data
#### 4.2.3.1 Request Message
S7 header message type 0x01- Job Request, the request message format in the parameter part is the same as reading data, but the function code is 0x05, and there is an additional data part:


| **Purpose** | Return Code | Data Specification | Data Length | Write Data | Padding Data |
| ----------- | ----------- | ------------------ | ----------- | ---------- | ------------ |
| **Length** | 1 byte | 1 byte | 2 bytes | Variable | Variable |
| **Description** | Usually 0x00 | Size and variable type of data transfer | Length of subsequent data | Data to be written | If the written data is less than the data length, fill a portion of 00 until this |

#### 4.2.3.2 Response Message

The parameter part is the same as the request message, and the data part is:


| **Purpose** | Return Code |
| ----------- | ----------- |
| **Length** | 1 byte |
| **Description** | Usually 0xff |
