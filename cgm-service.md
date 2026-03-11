# CGM service

* _CGM Specific Ops Control Point_
	* app sends command to the pump
	* pump sends back data in response (indications)
	* SAKE-encrypted in _both_ directions


## CGM Specific Ops Control Point (SOCP)

A command (identified by its _opcode_) is sent by writing to this characteristic. The pump responds by sending an indication for the same characteristic.

If the _E2E-CRC Supported_ bit is set in the _CGM Features_ characteristic (which always seems to be the case for a 780G pump), the _E2E-CRC_ field must be included in all requests. The pump will also add this field in its responses.

The written data must be SAKE-encrypted. The returned data is also SAKE-encrypted.

The pump supports the following standard command as defined in [[CGMS]](#ref-cgms) and [[GSS, sec. 3.46]](#ref-gss):

* _Get Glucose Calibration Value_ (opcode 0x05)

Additionally, the following custom commands are supported:


### Format of custom _Get Calibration Context_ command and response

#### Command structure

Field Name                   |  Data Type   | Size (octets) | Unit
-----------------------------|--------------|---------------|------
Opcode                       | Value 0x81   | 1             | None
Record Number                | u16          | 2             | None
E2E-CRC                      | u16          | 0 or 2        | None

#### Response structure

Field Name                   |  Data Type   | Size (octets) | Unit
-----------------------------|--------------|---------------|------
Response Opcode              | Value 0x82   | 1             | None
Status                       | u8           | 1             | None
Calibration Factor           | u16          | 2             | ???
E2E-CRC                      | u16          | 0 or 2        | None

Bits in the _Status_ field are defined as follows:

Bit | Definition                         | Description
----|------------------------------------|-------------
0   | Change Sensor Needed               |
1   | No Further Calibrations            |


### Format of custom _Read Session Start Time_ command and response

#### Command structure

Field Name                   |  Data Type   | Size (octets) | Unit
-----------------------------|--------------|---------------|------
Opcode                       | Value 0x83   | 1             | None
Session ID                   | u16          | 2             | None
E2E-CRC                      | u16          | 0 or 2        | None

#### Response structure

Field Name                   |  Data Type   | Size (octets) | Unit
-----------------------------|--------------|---------------|------
Response Opcode              | Value 0x84   | 1             | None
Session ID                   | u16          | 2             | None
Date Time                    | see note-1   | 7             | see note-1
Time Zone                    | see note-2   | 1             | see note-2
DST Offset                   | see note-3   | 1             | see note-3
E2E-CRC                      | u16          | 0 or 2        | None

note-1: See [[GSS, sec. 3.79]](#ref-gss) for the definition of this type.

note-2: See [[GSS, sec. 3.255]](#ref-gss) for the definition of this type.

note-3: See [[GSS, sec. 3.86]](#ref-gss) for the definition of this type.


### Format of custom _Read Current Session ID_ command and response

#### Command structure

Field Name                   |  Data Type   | Size (octets) | Unit
-----------------------------|--------------|---------------|------
Opcode                       | Value 0x8c   | 1             | None
E2E-CRC                      | u16          | 0 or 2        | None

#### Response structure

Field Name                   |  Data Type   | Size (octets) | Unit
-----------------------------|--------------|---------------|------
Response Opcode              | Value 0x8d   | 1             | None
Session ID                   | u16          | 2             | None
E2E-CRC                      | u16          | 0 or 2        | None


### Format of custom _Get Sensor Details_ command and response

#### Command structure

Field Name                   |  Data Type   | Size (octets) | Unit
-----------------------------|--------------|---------------|------
Opcode                       | Value 0x90   | 1             | None
E2E-CRC                      | u16          | 0 or 2        | None

#### Response structure

Field Name                   |  Data Type   | Size (octets) | Unit
-----------------------------|--------------|---------------|------
Response Opcode              | Value 0x91   | 1             | None
Flags                        | 8 bit        | 1             | None
Annunciation                 | 16 bit       | 0 or 2        | None
Maximum Calibration Interval | u16          | 0 or 2        | ???
Maximum Sensor Life          | u16          | 0 or 2        | minutes
Sensor Flex Package Version  | u16          | 0 or 2        | ???
Warm-Up Period               | u8           | 0 or 1        | ???
E2E-CRC                      | u16          | 0 or 2        | None

Bits in the _Flags_ field are defined as follows:

Bit | Definition                   | Description
----|------------------------------|-------------
0   | Sensor Details Annunciation  | If this bit is set, field _Annunciation_ is present
1   | Maximum Calibration Interval | If this bit is set, field _Maximum Calibration Interval_ is present
2   | Maximum Sensor Life          | If this bit is set, field _Maximum Sensor Life_ is present
3   | Sensor Flex Package Version  | If this bit is set, field _Sensor Flex Package Version_ is present
4   | Sensor Warm-Up Period        | If this bit is set, field _Warm-Up Period_ is present

Bits in the _Annunciation_ field are defined as follows:

Bit | Definition                         | Description
----|------------------------------------|-------------
0   | Approved Treatment                 |
1   | Disposable                         |
2   | Calibration-Free                   |
3   | Has Calibration Recommended        |
4   | Has Abnormal SG Increase Detection |
5   | Calibration Transfer Supported     |


## References

<a id="ref-cgms"></a>
**[CGMS]**
[Continuous Glucose Monitoring Service](specs/CGMS_v1.0.2.pdf), Bluetooth® Service Specification, v1.0.2

<a id="ref-gss"></a>
**[GSS]**
[GATT Specification Supplement (GSS)](specs/GATT_Specification_Supplement.pdf), Bluetooth® Document, 2025-12-23
