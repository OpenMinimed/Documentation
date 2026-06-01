# Data types and notational conventions

Here we cover a few basics you need to know when reading documentation for GATT services and characteristics.


## Endianness

All multi-byte data is stored in Little-Endian format, i.e. the least significant byte comes first.

For example, a characteristic contains the following 6 bytes `ad de ef be 0d d0` as payload. Assume that its documentation tells us that these represent 3 consecutive 2-byte values `Foo`, `Bar`, `Qux` (in that order). We would then decode that byte stream as follows:

    ad de ef be 0d d0
    ad de .. .. .. ..  Foo: 0xdead
    .. .. ef be .. ..  Bar: 0xbeef
    .. .. .. .. 0d d0  Qux: 0xd00d


## Floating-point types

Two different floating-point types are used in the characteristics. The Bluetooth SIG has different names for them in different documents: _sfloat_ , _medfloat16_/_medfloat32_, … We will use _f16_ for the 16-bit version and _f32_ for the 32-bit version in our documentation. Both are constructed the same way:

    f = m × 10^e

where `m` is the mantissa and `e` is the exponent. Both are encoded in 2's complement. The exponent is stored in the most-significant bits:

    f16:                     eeee mmmm mmmm mmmm
    f32: eeee eeee mmmm mmmm mmmm mmmm mmmm mmmm

For example:

    0xf90f     =   -1777 × 10^-1 = -177.7
    0xf82625a0 = 2500000 × 10^-8 =    0.025


## Bitfields

Bits are numbered from the least significant bit, starting with 0.

For example, bits 1, 2, 4 are set in byte `0x16`:

           bit 7   bit 0
           |       |
    0x16 = 0001 0110

