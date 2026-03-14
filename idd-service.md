# Insulin Delivery Service

## Introduction

The Bluetooth SIG actually specifies an official _Insulin Delivery Profile_ (IDP). It defines two roles:

* _Insulin Delivery Sensor_ (in our case: the pump, an _Insulin Delivery Device_ (IDD))
* _Collector_ (in our case: the MiniMed Mobile app)

The _Insulin Delivery Sensor_ provides, among other services, the _Insulin Delivery Service_ (IDS, UUID 0x183A) and associated characteristics. Through them, a device can "control the IDD" and "obtain its status and historical therapy data" [[IDP]](#ref-idp).

For simplicity, we will mostly refer to _pump_ and _app_ instead of _Insulin Delivery Sensor_ and _Collector_ in this documentation.

The pump in question provides an _Insulin Delivery Service_, but it has a custom UUID that differs from the one defined in the standard [[IDS]](#ref-ids). This is a Medtronic custom GATT service nearly all of whose characteristics are SAKE-encoded.

Judging by their names, Medtronic's version of the IDS is actually comprised of the same characteristics as the one specified in the standard, only with Medtronic's own UUIDs; with the exception of the _IDD Record Access Control Point_ (UUID 0x2B27) which is replaced by the _Record Access Control Point_ (UUID 0x2A52). So it seems save to assume that this service is an implementation of the IDS specified in [[IDS]](#ref-ids); maybe only with some non-standard extensions (such as the SAKE encryption).


## IDD Features

This characteristic can be read to determine the supported features of the pump. It is based on the homonymous characteristic defined in [[IDS]](#ref-ids). The only difference being an extended _Flags_ field (due to bit 23 being set) with additional custom Medtronic entries.

If the _E2E-Protection Supported_ bit is set in the _Flags_ field, the optional fields _E2E-Counter_ and _E2E-CRC_ field in the other characteristics of this service must be included and populated with a CRC over the data. See [[IDS, sec. 3.1]](#ref-ids) for the specifics. Note that this goes both ways: The CRC must be included in all our requests, and the pump will also add it in its responses.

The _E2E-Protection Supported_ bit never seems to be set for a 780G pump.

The spec requires the _IDD Features_ to be static during a connection. So reading this characteristic _once_ is sufficient. The features will not change in the middle of a connection. Its structure looks like this:

Field Name                   | Data Type   | Size (octets) | Unit
-----------------------------|-------------|---------------|------
E2E-CRC                      | u16         | 2             | N/A
E2E-Counter                  | u8          | 1             | N/A
Insulin Concentration        | f16         | 2             | IU/mL
Flags                        | ≥ 24 bit    | ≥ 3           | None

Bits in the _Flags_ field are defined as follows (Medtronic's custom extensions marked):

Bit   | Definition                        | Description
------|-----------------------------------|-------------
 0–23 | see [[IDS, sec. 4.4.2]](#ref-ids) |
24    | Reservoir Size 300 IU Supported   | custom extension
25    | Glucose Unit mg/dL Used           | custom extension
26    | LGS Feature Supported             | custom extension
27    | PLGM Feature Supported            | custom extension
28    | HCL Feature Supported             | custom extension
29–30 | ???                               | custom extension; probably unused
31    | Feature Extension 1               | custom extension; If this bit is set, an additional octet is attached (bits 32–39).


## IDD Status Changed

This characteristic can be read to determine various status changes of the pump. It is based on the homonymous characteristic defined in [[IDS]](#ref-ids). The app can also configure this characteristic for indications to automatically receive the status changes when they happen.

The specified characteristic value consists of a single 16-bit flags field. Medtronic extends this to up to 48 bits in their version. They also populate some of the reserved bits with their custom ones. The extension mechanism is very similar to the one in _IDD Feature_: If the highest bit in the current block is _set_, another block of 16 bits is appended, thus extending the flags.

Field Name    | Data Type    | Size (octets) | Unit
--------------|--------------|---------------|------
Flags         | 16–48 bit    | 2–6           | None
E2E-Counter   | u8           | 0 ir 1        | N/A
E2E-CRC       | u16          | 0 or 2        | N/A

Per the spec, the pump is expected to retain the status of a bit of the _Flags_ field until its value is reset by the app through the _Reset Status_ procedure using the characteristic _IDD Status Reader Control Point_.

Bits in the _Flags_ field are defined as follows (Medtronic's custom extensions marked):

Bit   | Definition                               | Description
------|------------------------------------------|-------------
 0    | Therapy Control State Changed            |
 1    | Operational State Changed                |
 2    | Reservoir Status Changed                 |
 3    | Annunciation Status Changed              |
 4    | Total Daily Insulin Status Changed       |
 5    | Active Basal Rate Status Changed         |
 6    | Active Bolus Status Changed              |
 7    | History Event Recorded                   |
 8    | Time In Range Status Changed             | custom extension; marked as reserved in the spec
 9–14 | reserved/unused                          |
15    | Extended Status                          | custom extension; If this bit is set, two additional octets are attached (bits 16–31).
16    | Therapy Algorithm State                  | custom extension
17    | Insulin On Board                         | custom extension
18    | New CGM Measurement                      | custom extension
19    | Sensor EOL                               | custom extension
20    | CGM Calibration                          | custom extension
21    | Sensor Status Message                    | custom extension
22    | Sensor Connectivity State                | custom extension
23    | Display Format Changed                   | custom extension
24    | High/Low Settings Changed                | custom extension
25    | Sensor Changed                           | custom extension
26    | CGM Calibration Context Changed          | custom extension
27    | CGM Time Calibration Recommended Changed | custom extension
28    | Remote Bolus Option Changed              | custom extension
29    | Local UI Interaction Requested           | custom extension
30    | Sensor Warm-up Time Remaining Changed    | custom extension
31    | Extended Status 1                        | custom extension; If this bit is set, two additional octets are attached (bits 32–47).
32    | Sensor Calibration Status Icon Changed   | custom extension
33    | Early Sensor Calibration Time Changed    | custom extension


## References

<a id="ref-idp"></a>
**[IDP]**
[Insulin Delivery Profile](specs/IDP_v1.0.2.pdf), Bluetooth® Profile Specification, v1.0.2

<a id="ref-ids"></a>
**[IDS]**
[Insulin Delivery Service](specs/IDS_v1.0.2-1.pdf), Bluetooth® Service Specification, v1.0.2
