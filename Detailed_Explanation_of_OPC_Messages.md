OPC (OLE for Process Control) is a standard for industrial communication protocols, facilitating data exchange between devices and systems from different manufacturers. It is primarily used in industrial automation systems. The OPC standard encompasses several specifications, including OPC DA (Data Access), OPC UA (Unified Architecture), OPC HDA (Historical Data Access), and more.

This article focuses on the message formats of OPC UA over TCP and OPC UA Secure Conversation.

# 1. OPC Classic (includes OPC DA, OPC HDA, etc.)
OPC Classic is based on Microsoft's COM/DCOM (Component Object Model/Distributed Component Object Model) technology. Thus, it lacks a unified, standardized "message format" similar to TCP/IP. In OPC Classic, data exchange and communication are facilitated through the COM/DCOM mechanism, meaning data is transmitted as COM objects, not via a specific, fixed format message.

# 2. OPC UA (Unified Architecture)
OPC UA is a more modern protocol, designed to replace OPC Classic, offering a more secure, cross-platform data exchange mechanism. OPC UA defines a comprehensive set of services and information models, enabling its use over different transport layers, such as TCP, HTTP, etc. In the OPC UA communication model, interactions between clients and servers are based on a series of service requests and responses. Each request and response follows the encoding rules defined by OPC UA, allowing serialization into a binary stream or XML.

Common OPC UA messages are mainly divided into two categories: OPC UA over TCP and OPC UA Secure Conversation.

## 2.1 OPC UA over TCP Message Structure
OPC UA over TCP messages include a message header and body, with the main structure as follows:

| **Purpose** | Header | Body |
| ----------- | ------ | ---- |
| **Length**  | 8 byte | Variable |
| **Description** | Controls and describes the message | The actual data to be transmitted, whose content and structure depend on the specific OPC UA service request or response |

### 2.1.1 Message Header
The message header structure is as follows:

| **Purpose** | Message Type | Reserved Section | Message Size |
| ----------- | ------------ | ---------------- | ------------ |
| **Length**  | 3 byte       | 1 byte           | 4 byte       |
| **Description** | Identifies the message type | Set to ASCII code ¡°F¡± for OPC UA link protocol supported values | Total length of the message header + body in bytes |

Message types are divided into four categories:
- HEL: Indicates the body is a Hello message
- ACK: Indicates the body is an Acknowledge message
- ERR: Indicates the body is an Error message
- RHE: Indicates the body is a ReverseHello message

### 2.1.2 Message Body

#### 2.1.2.1 Hello Message
When the message type is HEL, the message body is a Hello message, formatted as follows:

| **Purpose** | **Length** | **Description** |
| ----------- | ---------- | --------------- |
| Protocol Version | 4 byte | Indicates the version of the OPC UA specification used by the sender. Receivers can use this information to determine if they can understand the received message. |
| Receive Buffer Size | 4 byte | Specifies the maximum message size the receiver is prepared to allocate for this connection. It is used for flow control and to prevent the receiver from being overwhelmed by too large messages. The value must be greater than 8192 bytes. |
| Send Buffer Size | 4 byte | Specifies the maximum message size the sender is prepared to use for this connection. This is also part of flow control, ensuring both parties can handle the exchanged data. The value must be greater than 8192 bytes. |
| Maximum Message Size | 4 byte | Specifies the maximum size of the message body allowed by both parties. It is used to prevent performance issues caused by processing too large single messages. A value of 0 indicates no client-imposed limit. |
| Maximum Chunk Count | 4 byte | Specifies the maximum number of chunks into which a response message can be divided. This helps manage the transmission of large amounts of data, ensuring even large messages can be efficiently transferred between both parties. A value of 0 indicates no client-imposed limit. |
| Endpoint URL | Up to 4096 bytes | The URL of the endpoint the client wishes to connect to. If the length exceeds 4096 or the URL identifies a resource that cannot be recognized, the server should return a Bad_TcpEndpointUrlInvalid error message and close the connection. |

The Hello message is an important part of the OPC UA TCP protocol handshake process, allowing clients and servers to exchange basic communication parameters to establish a foundation for more complex interactions.

#### 2.1.2.2 Acknowledge Message

| **Purpose** | **Length** | **Description** |
| ----------- | ---------- | --------------- |
| Protocol Version | 4 byte | The version of the OPC UA protocol supported by the server |
| Receive Buffer Size | 4 byte | Specifies the maximum message size the receiver is prepared to allocate for this connection. It is used for flow control and to prevent the receiver from being overwhelmed by too large messages. The value must be greater than 8192 bytes. |
| Send Buffer Size | 4 byte | Specifies the maximum message size the sender is prepared to use for this connection. This is also part of flow control, ensuring both parties can handle the exchanged data. The value must be greater than 8192 bytes. |
| Maximum Message Size | 4 byte | Specifies the maximum size of the message body allowed by both parties. It is used to prevent performance issues caused by processing too large single messages. A value of 0 indicates no client-imposed limit. |
| Maximum Chunk Count | 4 byte | Specifies the maximum number of chunks into which a response message can be divided. This helps manage the transmission of large amounts of data, ensuring even large messages can be efficiently transferred between both parties. A value of 0 indicates no client-imposed limit. |

The Acknowledge message provides the basic parameters needed for communication between the client and server, ensuring both parties can efficiently exchange subsequent OPC UA messages. Upon receiving the Acknowledge message, the client adjusts its communication settings accordingly, allowing both parties to begin formal data exchange.

#### 2.1.2.3 Error Message

| **Purpose** | **Length** | **Description** |
| ----------- | ---------- | --------------- |
| Error Code | 4 byte | The numerical code of the error. |
| Reason | Up to 4096 bytes | Detailed description of the error. |

Error codes are updated with version releases. Here's a link to the current version (UA-1.05.03-2023-12-15) error code list on the [official GitHub](https://github.com/OPCFoundation/UA-Nodeset/blob/UA-1.05.03-2023-12-15/Schema/StatusCode.csv).

| **´íÎóÃû**                                                         | **´íÎóÂë**    |
| --------------------------------------------------------------- | ---------- |
| Good                                                            | 0x00000000 |
| Uncertain                                                       | 0x40000000 |
| Bad                                                             | 0x80000000 |
| BadUnexpectedError                                              | 0x80010000 |
| BadInternalError                                                | 0x80020000 |
| BadOutOfMemory                                                  | 0x80030000 |
| BadResourceUnavailable                                          | 0x80040000 |
| BadCommunicationError                                           | 0x80050000 |
| BadEncodingError                                                | 0x80060000 |
| BadDecodingError                                                | 0x80070000 |
| BadEncodingLimitsExceeded                                       | 0x80080000 |
| BadRequestTooLarge                                              | 0x80B80000 |
| BadResponseTooLarge                                             | 0x80B90000 |
| BadUnknownResponse                                              | 0x80090000 |
| BadTimeout                                                      | 0x800A0000 |
| BadServiceUnsupported                                           | 0x800B0000 |
| BadShutdown                                                     | 0x800C0000 |
| BadServerNotConnected                                           | 0x800D0000 |
| BadServerHalted                                                 | 0x800E0000 |
| BadNothingToDo                                                  | 0x800F0000 |
| BadTooManyOperations                                            | 0x80100000 |
| BadTooManyMonitoredItems                                        | 0x80DB0000 |
| BadDataTypeIdUnknown                                            | 0x80110000 |
| BadCertificateInvalid                                           | 0x80120000 |
| BadSecurityChecksFailed                                         | 0x80130000 |
| BadCertificatePolicyCheckFailed                                 | 0x81140000 |
| BadCertificateTimeInvalid                                       | 0x80140000 |
| BadCertificateIssuerTimeInvalid                                 | 0x80150000 |
| BadCertificateHostNameInvalid                                   | 0x80160000 |
| BadCertificateUriInvalid                                        | 0x80170000 |
| BadCertificateUseNotAllowed                                     | 0x80180000 |
| BadCertificateIssuerUseNotAllowed                               | 0x80190000 |
| BadCertificateUntrusted                                         | 0x801A0000 |
| BadCertificateRevocationUnknown                                 | 0x801B0000 |
| BadCertificateIssuerRevocationUnknown                           | 0x801C0000 |
| BadCertificateRevoked                                           | 0x801D0000 |
| BadCertificateIssuerRevoked                                     | 0x801E0000 |
| BadCertificateChainIncomplete                                   | 0x810D0000 |
| BadUserAccessDenied                                             | 0x801F0000 |
| BadIdentityTokenInvalid                                         | 0x80200000 |
| BadIdentityTokenRejected                                        | 0x80210000 |
| BadSecureChannelIdInvalid                                       | 0x80220000 |
| BadInvalidTimestamp                                             | 0x80230000 |
| BadNonceInvalid                                                 | 0x80240000 |
| BadSessionIdInvalid                                             | 0x80250000 |
| BadSessionClosed                                                | 0x80260000 |
| BadSessionNotActivated                                          | 0x80270000 |
| BadSubscriptionIdInvalid                                        | 0x80280000 |
| BadRequestHeaderInvalid                                         | 0x802A0000 |
| BadTimestampsToReturnInvalid                                    | 0x802B0000 |
| BadRequestCancelledByClient                                     | 0x802C0000 |
| BadTooManyArguments                                             | 0x80E50000 |
| BadLicenseExpired                                               | 0x810E0000 |
| BadLicenseLimitsExceeded                                        | 0x810F0000 |
| BadLicenseNotAvailable                                          | 0x81100000 |
| BadServerTooBusy                                                | 0x80EE0000 |
| GoodPasswordChangeRequired                                      | 0x00EF0000 |
| GoodSubscriptionTransferred                                     | 0x002D0000 |
| GoodCompletesAsynchronously                                     | 0x002E0000 |
| GoodOverload                                                    | 0x002F0000 |
| GoodClamped                                                     | 0x00300000 |
| BadNoCommunication                                              | 0x80310000 |
| BadWaitingForInitialData                                        | 0x80320000 |
| BadNodeIdInvalid                                                | 0x80330000 |
| BadNodeIdUnknown                                                | 0x80340000 |
| BadAttributeIdInvalid                                           | 0x80350000 |
| BadIndexRangeInvalid                                            | 0x80360000 |
| BadIndexRangeNoData                                             | 0x80370000 |
| BadIndexRangeDataMismatch                                       | 0x80EA0000 |
| BadDataEncodingInvalid                                          | 0x80380000 |
| BadDataEncodingUnsupported                                      | 0x80390000 |
| BadNotReadable                                                  | 0x803A0000 |
| BadNotWritable                                                  | 0x803B0000 |
| BadOutOfRange                                                   | 0x803C0000 |
| BadNotSupported                                                 | 0x803D0000 |
| BadNotFound                                                     | 0x803E0000 |
| BadObjectDeleted                                                | 0x803F0000 |
| BadNotImplemented                                               | 0x80400000 |
| BadMonitoringModeInvalid                                        | 0x80410000 |
| BadMonitoredItemIdInvalid                                       | 0x80420000 |
| BadMonitoredItemFilterInvalid                                   | 0x80430000 |
| BadMonitoredItemFilterUnsupported                               | 0x80440000 |
| BadFilterNotAllowed                                             | 0x80450000 |
| BadStructureMissing                                             | 0x80460000 |
| BadEventFilterInvalid                                           | 0x80470000 |
| BadContentFilterInvalid                                         | 0x80480000 |
| BadFilterOperatorInvalid                                        | 0x80C10000 |
| BadFilterOperatorUnsupported                                    | 0x80C20000 |
| BadFilterOperandCountMismatch                                   | 0x80C30000 |
| BadFilterOperandInvalid                                         | 0x80490000 |
| BadFilterElementInvalid                                         | 0x80C40000 |
| BadFilterLiteralInvalid                                         | 0x80C50000 |
| BadContinuationPointInvalid                                     | 0x804A0000 |
| BadNoContinuationPoints                                         | 0x804B0000 |
| BadReferenceTypeIdInvalid                                       | 0x804C0000 |
| BadBrowseDirectionInvalid                                       | 0x804D0000 |
| BadNodeNotInView                                                | 0x804E0000 |
| BadNumericOverflow                                              | 0x81120000 |
| BadLocaleNotSupported                                           | 0x80ED0000 |
| BadNoValue                                                      | 0x80F00000 |
| BadServerUriInvalid                                             | 0x804F0000 |
| BadServerNameMissing                                            | 0x80500000 |
| BadDiscoveryUrlMissing                                          | 0x80510000 |
| BadSempahoreFileMissing                                         | 0x80520000 |
| BadRequestTypeInvalid                                           | 0x80530000 |
| BadSecurityModeRejected                                         | 0x80540000 |
| BadSecurityPolicyRejected                                       | 0x80550000 |
| BadTooManySessions                                              | 0x80560000 |
| BadUserSignatureInvalid                                         | 0x80570000 |
| BadApplicationSignatureInvalid                                  | 0x80580000 |
| BadNoValidCertificates                                          | 0x80590000 |
| BadIdentityChangeNotSupported                                   | 0x80C60000 |
| BadRequestCancelledByRequest                                    | 0x805A0000 |
| BadParentNodeIdInvalid                                          | 0x805B0000 |
| BadReferenceNotAllowed                                          | 0x805C0000 |
| BadNodeIdRejected                                               | 0x805D0000 |
| BadNodeIdExists                                                 | 0x805E0000 |
| BadNodeClassInvalid                                             | 0x805F0000 |
| BadBrowseNameInvalid                                            | 0x80600000 |
| BadBrowseNameDuplicated                                         | 0x80610000 |
| BadNodeAttributesInvalid                                        | 0x80620000 |
| BadTypeDefinitionInvalid                                        | 0x80630000 |
| BadSourceNodeIdInvalid                                          | 0x80640000 |
| BadTargetNodeIdInvalid                                          | 0x80650000 |
| BadDuplicateReferenceNotAllowed                                 | 0x80660000 |
| BadInvalidSelfReference                                         | 0x80670000 |
| BadReferenceLocalOnly                                           | 0x80680000 |
| BadNoDeleteRights                                               | 0x80690000 |
| UncertainReferenceNotDeleted                                    | 0x40BC0000 |
| BadServerIndexInvalid                                           | 0x806A0000 |
| BadViewIdUnknown                                                | 0x806B0000 |
| BadViewTimestampInvalid                                         | 0x80C90000 |
| BadViewParameterMismatch                                        | 0x80CA0000 |
| BadViewVersionInvalid                                           | 0x80CB0000 |
| UncertainNotAllNodesAvailable                                   | 0x40C00000 |
| GoodResultsMayBeIncomplete                                      | 0x00BA0000 |
| BadNotTypeDefinition                                            | 0x80C80000 |
| UncertainReferenceOutOfServer                                   | 0x406C0000 |
| BadTooManyMatches                                               | 0x806D0000 |
| BadQueryTooComplex                                              | 0x806E0000 |
| BadNoMatch                                                      | 0x806F0000 |
| BadMaxAgeInvalid                                                | 0x80700000 |
| BadSecurityModeInsufficient                                     | 0x80E60000 |
| BadHistoryOperationInvalid                                      | 0x80710000 |
| BadHistoryOperationUnsupported                                  | 0x80720000 |
| BadInvalidTimestampArgument                                     | 0x80BD0000 |
| BadWriteNotSupported                                            | 0x80730000 |
| BadTypeMismatch                                                 | 0x80740000 |
| BadMethodInvalid                                                | 0x80750000 |
| BadArgumentsMissing                                             | 0x80760000 |
| BadNotExecutable                                                | 0x81110000 |
| BadTooManySubscriptions                                         | 0x80770000 |
| BadTooManyPublishRequests                                       | 0x80780000 |
| BadNoSubscription                                               | 0x80790000 |
| BadSequenceNumberUnknown                                        | 0x807A0000 |
| GoodRetransmissionQueueNotSupported                             | 0x00DF0000 |
| BadMessageNotAvailable                                          | 0x807B0000 |
| BadInsufficientClientProfile                                    | 0x807C0000 |
| BadStateNotActive                                               | 0x80BF0000 |
| BadAlreadyExists                                                | 0x81150000 |
| BadTcpServerTooBusy                                             | 0x807D0000 |
| BadTcpMessageTypeInvalid                                        | 0x807E0000 |
| BadTcpSecureChannelUnknown                                      | 0x807F0000 |
| BadTcpMessageTooLarge                                           | 0x80800000 |
| BadTcpNotEnoughResources                                        | 0x80810000 |
| BadTcpInternalError                                             | 0x80820000 |
| BadTcpEndpointUrlInvalid                                        | 0x80830000 |
| BadRequestInterrupted                                           | 0x80840000 |
| BadRequestTimeout                                               | 0x80850000 |
| BadSecureChannelClosed                                          | 0x80860000 |
| BadSecureChannelTokenUnknown                                    | 0x80870000 |
| BadSequenceNumberInvalid                                        | 0x80880000 |
| BadProtocolVersionUnsupported                                   | 0x80BE0000 |
| BadConfigurationError                                           | 0x80890000 |
| BadNotConnected                                                 | 0x808A0000 |
| BadDeviceFailure                                                | 0x808B0000 |
| BadSensorFailure                                                | 0x808C0000 |
| BadOutOfService                                                 | 0x808D0000 |
| BadDeadbandFilterInvalid                                        | 0x808E0000 |
| UncertainNoCommunicationLastUsableValue                         | 0x408F0000 |
| UncertainLastUsableValue                                        | 0x40900000 |
| UncertainSubstituteValue                                        | 0x40910000 |
| UncertainInitialValue                                           | 0x40920000 |
| UncertainSensorNotAccurate                                      | 0x40930000 |
| UncertainEngineeringUnitsExceeded                               | 0x40940000 |
| UncertainSubNormal                                              | 0x40950000 |
| GoodLocalOverride                                               | 0x00960000 |
| GoodSubNormal                                                   | 0x00EB0000 |
| BadRefreshInProgress                                            | 0x80970000 |
| BadConditionAlreadyDisabled                                     | 0x80980000 |
| BadConditionAlreadyEnabled                                      | 0x80CC0000 |
| BadConditionDisabled                                            | 0x80990000 |
| BadEventIdUnknown                                               | 0x809A0000 |
| BadEventNotAcknowledgeable                                      | 0x80BB0000 |
| BadDialogNotActive                                              | 0x80CD0000 |
| BadDialogResponseInvalid                                        | 0x80CE0000 |
| BadConditionBranchAlreadyAcked                                  | 0x80CF0000 |
| BadConditionBranchAlreadyConfirmed                              | 0x80D00000 |
| BadConditionAlreadyShelved                                      | 0x80D10000 |
| BadConditionNotShelved                                          | 0x80D20000 |
| BadShelvingTimeOutOfRange                                       | 0x80D30000 |
| BadNoData                                                       | 0x809B0000 |
| BadBoundNotFound                                                | 0x80D70000 |
| BadBoundNotSupported                                            | 0x80D80000 |
| BadDataLost                                                     | 0x809D0000 |
| BadDataUnavailable                                              | 0x809E0000 |
| BadEntryExists                                                  | 0x809F0000 |
| BadNoEntryExists                                                | 0x80A00000 |
| BadTimestampNotSupported                                        | 0x80A10000 |
| GoodEntryInserted                                               | 0x00A20000 |
| GoodEntryReplaced                                               | 0x00A30000 |
| UncertainDataSubNormal                                          | 0x40A40000 |
| GoodNoData                                                      | 0x00A50000 |
| GoodMoreData                                                    | 0x00A60000 |
| BadAggregateListMismatch                                        | 0x80D40000 |
| BadAggregateNotSupported                                        | 0x80D50000 |
| BadAggregateInvalidInputs                                       | 0x80D60000 |
| BadAggregateConfigurationRejected                               | 0x80DA0000 |
| GoodDataIgnored                                                 | 0x00D90000 |
| BadRequestNotAllowed                                            | 0x80E40000 |
| BadRequestNotComplete                                           | 0x81130000 |
| BadTransactionPending                                           | 0x80E80000 |
| BadTicketRequired                                               | 0x811F0000 |
| BadTicketInvalid                                                | 0x81200000 |
| BadLocked                                                       | 0x80E90000 |
| BadRequiresLock                                                 | 0x80EC0000 |
| GoodEdited                                                      | 0x00DC0000 |
| GoodPostActionFailed                                            | 0x00DD0000 |
| UncertainDominantValueChanged                                   | 0x40DE0000 |
| GoodDependentValueChanged                                       | 0x00E00000 |
| BadDominantValueChanged                                         | 0x80E10000 |
| UncertainDependentValueChanged                                  | 0x40E20000 |
| BadDependentValueChanged                                        | 0x80E30000 |
| GoodEdited_DependentValueChanged                                | 0x01160000 |
| GoodEdited_DominantValueChanged                                 | 0x01170000 |
| GoodEdited_DominantValueChanged_DependentValueChanged           | 0x01180000 |
| BadEdited_OutOfRange                                            | 0x81190000 |
| BadInitialValue_OutOfRange                                      | 0x811A0000 |
| BadOutOfRange_DominantValueChanged                              | 0x811B0000 |
| BadEdited_OutOfRange_DominantValueChanged                       | 0x811C0000 |
| BadOutOfRange_DominantValueChanged_DependentValueChanged        | 0x811D0000 |
| BadEdited_OutOfRange_DominantValueChanged_DependentValueChanged | 0x811E0000 |
| GoodCommunicationEvent                                          | 0x00A70000 |
| GoodShutdownEvent                                               | 0x00A80000 |
| GoodCallAgain                                                   | 0x00A90000 |
| GoodNonCriticalTimeout                                          | 0x00AA0000 |
| BadInvalidArgument                                              | 0x80AB0000 |
| BadConnectionRejected                                           | 0x80AC0000 |
| BadDisconnect                                                   | 0x80AD0000 |
| BadConnectionClosed                                             | 0x80AE0000 |
| BadInvalidState                                                 | 0x80AF0000 |
| BadEndOfStream                                                  | 0x80B00000 |
| BadNoDataAvailable                                              | 0x80B10000 |
| BadWaitingForResponse                                           | 0x80B20000 |
| BadOperationAbandoned                                           | 0x80B30000 |
| BadExpectedStreamToBlock                                        | 0x80B40000 |
| BadWouldBlock                                                   | 0x80B50000 |
| BadSyntaxError                                                  | 0x80B60000 |
| BadMaxConnectionsReached                                        | 0x80B70000 |
| UncertainTransducerInManual                                     | 0x42080000 |
| UncertainSimulatedValue                                         | 0x42090000 |
| UncertainSensorCalibration                                      | 0x420A0000 |
| UncertainConfigurationError                                     | 0x420F0000 |
| GoodCascadeInitializationAcknowledged                           | 0x04010000 |
| GoodCascadeInitializationRequest                                | 0x04020000 |
| GoodCascadeNotInvited                                           | 0x04030000 |
| GoodCascadeNotSelected                                          | 0x04040000 |
| GoodFaultStateActive                                            | 0x04070000 |
| GoodInitiateFaultState                                          | 0x04080000 |
| GoodCascade                                                     | 0x04090000 |
| BadDataSetIdInvalid                                             | 0x80E70000 |


#### 2.1.2.4 ReverseHello Message

| **Purpose** | **Length** | **Description** |
| ----------- | ---------- | --------------- |
| Server URI | Up to 4096 bytes | The ApplicationUri of the server sending the message. |
| Endpoint URL | Up to 4096 bytes | The URL of the endpoint used by the client to establish the SecureChannel. |

For connection-based protocols, such as TCP, the ReverseHello message allows servers behind firewalls, without open ports, to connect to clients and request clients to establish a SecureChannel using the server-created socket.

For message-based protocols, the ReverseHello message allows servers to announce their presence to clients. In this case, the Endpoint URL specifies the server's specific address and any tokens required to access it.

## 2.2 OPC UA Secure Conversation Message Structure
The OPC UA Secure Conversation message format is designed for establishing and maintaining an encrypted and signed communication channel between clients and servers.

| **Purpose** | Header | Security Header | Sequence Header | Payload | Security Footer |
| ----------- | ------ | --------------- | --------------- | ------- | --------------- |
| **Length** | 12 byte | Variable | 8 byte | Variable | Variable |
| **Description** | Controls and describes the message | Contains security-related information. Varies in length based on symmetric/asymmetric security algorithms | Includes a sequence number and request ID | This is the actual application data part, which may be encrypted and/or signed based on the security policy defined in the security header. | (Optional) If the message is signed, this part contains the signature. Not all security policies require a signature. |

### 2.2.1 Message Header

| **Purpose** | Message Type | Is Final | Message Size | Secure Channel ID |
| ----------- | ------------ | -------- | ------------ | ----------------- |
| **Length** | 3 byte | 1 byte | 4 byte | 4 byte |
| **Description** | Identifies the message type | A one-byte ASCII code indicating if this is the last chunk of the message. | The length from the start of the message header, in bytes | The unique identifier for the SecureChannel assigned by the server |

#### 2.2.1.1 Message Type
There are three main message types:
- MSG: Messages encrypted with channel-related keys
- OPN: Messages that open a secure channel
- CLO: Messages that close a secure channel

#### 2.2.1.2 Is Final

A one-byte ASCII code indicating whether the MessageChunk is the last chunk of the message.

Defined values are:
- C: Intermediate chunk.
- F: Final chunk.
- A: Final chunk (used when an error occurs, and the message is aborted).

This field is only meaningful for messages of type MSG. For other message types, this field is always ¡°F¡±.

#### 2.2.1.3 Message Size
The length from the start of the message header, in bytes

#### 2.2.1.4 Secure Channel ID

The unique identifier for the SecureChannel assigned by the server. If the server receives a secure channel ID it cannot recognize, it should return the corresponding transport layer error.

### 2.2.2 Security Header
Defines which encryption operations were applied to the message. There are two versions: the asymmetric algorithm security header and the symmetric algorithm security header.

#### 2.2.2.1 Asymmetric Algorithm Security Header

| **Purpose** | **Length** | **Description** |
| ----------- | ---------- | --------------- |
| Security Policy URI Length | 4 byte | The length of the security policy URI. This value may be 0 or -1 if no URI is specified. Other negative values are invalid. In bytes. |
| Security Policy URI | Variable | The URI of the security policy used to protect the message. This field is encoded as a UTF-8 string without a null terminator. |
| Sender Certificate Length | 4 byte | The length of the sender's certificate. This value may be 0 or -1 if no certificate is specified. Other negative values are invalid. In bytes. |
| Sender Certificate | Variable | The sender's certificate |
| Receiver Certificate Thumbprint Length | 4 byte | The length of the receiver's certificate thumbprint. This field's value is 20 if encrypted; otherwise, it may be 0 or -1. In bytes. |
| Receiver Certificate Thumbprint | Variable | The receiver's certificate thumbprint. This field should be empty if the message is not encrypted. |

#### 2.2.2.2 Symmetric Algorithm Security Header

| **Purpose** | **Length** | **Description** |
| ----------- | ---------- | --------------- |
| Token ID | 4 byte | The unique identifier of the security token for the secure channel used to protect the message. If the server receives a token ID it cannot recognize, it will return the corresponding transport layer error. |

### 2.2.3 Sequence Header

| **Purpose** | **Length** | **Description** |
| ----------- | ---------- | --------------- |
| Sequence Number | 4 byte | A monotonically increasing sequence number assigned by the sender. |
| Request ID | 4 byte | An identifier assigned by the client to the OPC UA request message. All messages of a request and its corresponding response use the same identifier. |

### 2.2.4 Payload
This is the main body part of the message, containing the actual operation request or response data. The size of the payload is variable, depending on the actual amount of data transmitted.

### 2.2.5 Security Footer
This part is optional and only exists when certain specific security policies are used. It contains additional security information, such as padding data and signatures. The size of the security footer is also variable, depending on the security policy and data used.