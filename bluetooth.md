# Bluetooth communication

The insulin pumps and any connected devices use Bluetooth Low Energy (BLE) for communication. The pumps act as Central, while the sensors act as Peripheral. This is the only way multiple devices can be connected to a pump at the same time. It also means that any app (such as Medtronic's MiniMed Mobile app or our own scripts) must act as Peripheral when talking to the pump.

Establishing an initial connection roughly works like this: The pump is instructed (by the user) to scan for Peripherals that want to connect to it. The Peripheral sends out advertising packets that are picked up by the pump. These include some information that the pump can use to decide whether this is a suitable device or not. The pump then connects to a selected Peripheral. So it is always the pump that initiates the actual connection.

Both devices then exchange some keys (known as _pairing_ step) and store them for later use. Most importantly, this allows them to reestablish a lost connection without user interaction.

The devices utilize standardized and custom GATT services and characteristics. The interesting data (as payload in specific characteristics) is encrypted using a Medtronic protocol called SAKE (Secure? Authenticated? Key Exchange?).

Please check out our overview of [pump's](pump-services.md) and [app's](app-services.md) GATT services as well as the [Bluetooth SIG's official specifications](https://www.bluetooth.com/specifications/specs/) for more information.


## Advertising details (mobile device)

This is the advertising and connection mode used by mobile devices such as smartphones running Medtronic's MiniMed Mobile app.


### Initial pairing

For the initial connection, the Peripheral must include the following data in its advertising packets:

Data                       | Type | Value
---------------------------|------|-------
16-bit Service Class UUIDs | 0x03 | 0xfe82
Device Name                | 0x09 | "Mobile xxxxxxx"

This specific _Class UUID_ is Medtronic's custom SAKE service.

The _Device Name_ must start with "Mobile " (note the whitespace) and can then be followed by 0 to 7 standard ASCII characters. This is the name the pump will display for confirmation. It serves no other purpose and can basically be random.


### Reconnects

If the pump and a paired Peripheral are disconnected (devices are too far apart, Bluetooth is temporarily disabled etc.), the pump tries to reconnect. It scans for a known Peripheral with the following data in its advertising packets:

Data                       | Type | Value
---------------------------|------|-------
16-bit Service Class UUIDs | 0x03 | 0xfe81

Note that this uses a _different_ UUID than the advertising for pairing. The pump ignores the device name, so there is no need to include it here.

Since the pump does not know if the Peripheral is gone for long, it does not make sense for the pump to spend lots of its battery power on scanning for the Peripheral in short intervals. If the Peripheral's advertising packets are sent only every second or so, chances are high that the pump will miss them if it only scans for them every couple of seconds, too.

> [!NOTE]
> We do not know if the pump actually behaves that way (it is not trivial to measure when and how often the pump is actually scanning), but it would make sense for a battery-powered device.

We can reliably get the pump to reconnect if we let the Peripheral send advertising packets every 150 ms (or faster). Advertising with much longer intervals would usually _not_ get the pump to reconnect.

> [!IMPORTANT]
> The pump seems to only reconnect to devices that are using a Resolvable Private Address (RPA). The initial pairing also works with public addresses though. So if you have trouble with reconnects, this is one of the things to check.


## Advertising details (CareLink device)

This is the advertising and connection mode used by Medtronic's _CareLink Uploader_ in combination with their _Blue Adapter_ (a USB-BLE dongle). It requires the user to input their pump's 6-digit confirmation code on the computer that is trying to connect to the pump.

When told by the user to connect to a CareLink device, the pump scans for Peripherals with the following data in their advertising packets:

Data                       | Type | Value
---------------------------|------|-------
16-bit Service Class UUIDs | 0x03 | 0xfe82
Device Name                | 0x09 | "PC"

To actually get a connection going, the Peripheral also has to answer a following Scan Request from the pump with very specific advertising data in its Scan Response:

```
Length:     12
Type:       0xff   (Manufacturer Specific)
Company ID: 0x01f9 (Medtronic Inc.)
Data:       9-byte hash
```

If any of this is wrong, the pump will not even try to connect.

> [!IMPORTANT]
> In this mode, the pump will only connect to devices that _do not_ use a Resolvable Private Address (RPA).

That hash is computed from the pump's 6-digit confirmation code and the BLE adapter's MAC address as follows:

1. Concat MAC address and confirmation code into an array of 8-bytes. This drops the MAC's uppermost nibble! Use Big-Endian byte order for MAC address and confirmation code.

    ```
    MAC bytes:  aA bB cC dD eE fF  (6 bytes          → 48 bits)
    code bytes: _G hH iI           (6 integer digits → 20 bits)

    -> Ab Bc Cd De Ef FG hH iI
    ```

2. Reverse the order of that array.

3. Compute the 64-bit [FNV-1 hash](https://en.wikipedia.org/wiki/Fowler%E2%80%93Noll%E2%80%93Vo_hash_function) of that array.

    ```
    offset_basis = 0xcbf29ce484222325
    FNV_prime    = 0x100000001b3
    ```

4. Byte-reverse the order of the hash.

5. Prepend byte `0x01` to the hash.

### Example hash computation

```
MAC address:       01:23:45:67:89:ab → 01 23 45 67 89 ab
Confirmation code: "215087" → 215087 (0x3482f) → 03 48 2f

1. concat:  12 34 56 78 9a b3 48 2f
2. reverse: 2f 48 b3 9a 78 56 34 12
3. FNV-1:   32 10 ac 9f 6d 44 f0 4b
4. reverse: 4b f0 44 6d 9f ac 10 32
5. prepend: 01 4b f0 44 6d 9f ac 10 32
```


## MAC addresses

The sensor MACs start with DC:16:A2 (Medtronic Diabetes, https://standards-oui.ieee.org/oui/oui.txt), while the pump use the private OUI 00:23:f7. The lower 3 bytes are the sensor's serial number, converted to hexadecimal. For example:

    CGM GT1122867N → 1122867 = 0x112233
    → Its MAC address should be DC:16:A2:11:22:33.

There seems to be no such relation between a pump's MAC address and its serial number. The addresses look rather random.


## Device names

Pumps use their serial numbers of the form `NGxxxxxxxH` and translate them into the device name `Pump xxxxxxxH`, keeping the 7-digit number the same.
