# GATT Streaming

Some services (e.g. [History And Trace](hat-service.md)) need to transfer large amounts of data, a lot more than fits into a single ATT MTU. This is why the pump negotiates an ATT MTU size of 184 bytes after establishing the BLE connection: larger MTUs increase the throughput.

Since the data in question often exceeds even that MTU size, it is broken up into multiple chunks for the transfer. Each chunk is prefixed with one byte indicating the chunk index. Counting starts at 0.

The services employing this scheme usually contain at least a _Control Point_ characteristic and a _Data_ characteristic. The _Control Point_ is used to specify what is to be send/received while the _Data_ transfers the actual chunks.

The chunks are not simply sent all in sequence, though. Instead the pump will usually send a single one or a couple of them and then wait for the other side to request more. These requests work by writing the next expected chunk index + 0x80 to the _Data_ characteristic.

> [!NOTE]
> Some of the characteristics repeat sending the most recently sent chunk every couple of seconds until the next ones are requested. Always inspect the chunk index to see what you actually received.


## An example

We want to retrieve a certificate through the pump's [Certificate Management Service](cm-service.md). Suppose that the certificate data is `00 01 02 … ff`, so 256 bytes in total. Also suppose that the maximum ATT MTU size is 50 bytes, so the pump has to split up the data into chunks of 49 bytes (1 byte is reserved for the chunk index).

An exchange with the pump could look at follows:

1. Subscribe to the _Control Point_ characteristic

2. Send the _Get Certificate_ command on the _Control Point_ (CP):

    ```
    TX(CP):    00
    ```

3. Subscribe to the _Data_ characteristic

4. Receive _Data_ notifications:

    ```
    RX(Data):  00 00 01 02 … 30
    RX(Data):  01 31 33 34 … 61
    ```

5. Request the next chunk(s), starting at index 2, by writing to _Data_:

    ```
    TX(Data):  82
    ```

6. Receive _Data_ notifications:

    ```
    RX(Data):  02 62 01 02 … 92
    RX(Data):  03 93 97 98 … c3
    ```

7. Request the next chunk(s), starting at index 4, by writing to _Data_:

    ```
    TX(Data):  84
    ```

8. Receive _Data_ notifications:

    ```
    RX(Data):  04 c4 01 02 … f4
    RX(Data):  05 f5 f6 f7 f8 f9 fa fb fc fd fe ff
    ```

9. Receive _Control Point_ indication:

    ```
    RX(CP):    03 00 01
    ```

    This translates to _Opcode=Response_, _Request Opcode=Get Certificate_, _Response Code=Success_. The pump tells us that the transfer is complete, i.e. there are no more chunks to receive.

To reassemble the data we order the chunks by their chunk index (first byte), strip that byte from each of the chunks and then concatenate the remaining data.

> [!NOTE]
> You should not make any assumptions about the number of chunks the pump will send. You may get more or less than the 2 chunks per request chosen in this example.
