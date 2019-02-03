TERA protocol data. Reference parser can be found at [tera-data-parser](https://github.com/tera-proxy/tera-data-parser).

## Contained files
`map`
* `protocol.<shareRevision>.map` - Maps packet names to numerical opcodes (randomized by BHS each patch).
* `sysmsg.<majorPatchVersion>.map` - Maps system messages to IDs.

`protocol`
* `<packetName>.<version>.def` - Defines a packet structure.

## `.map` specification
A simple enum format. Each line may contain a single key-value pair seperated by space(s), tab(s) and/or `=`. Additionally supports comments preceded by `#`.
```
# This is a comment
ONE = 1
TWO = 2
THREE = 3 # This is the third value
```

## `.def` specification
Each line may contain the following, in order:
* A series of `-` specifying array nesting.
* A data type (listed below).
* (whitespace)
* A field name.

A `#` and anything after it on the line are treated as comments and should be ignored when parsing.
```
# Some comment
uint64 gameId # Your character

array someArray
- int32 value1
- int32 value2
- array nestedArray
- - int32 value1
- - int32 value2
```

### Fixed Types
* `bool` - 1-byte boolean (0 = false, 1 = true). Common uses: Various
* `byte` - Unsigned 1-byte integer.*
* `int16` - 2-byte signed integer. Common uses: Player stats
* `uint16` - 2-byte unsigned integer.
* `int32` - 4-byte signed integer. Common uses: Various
* `uint32` - 4-byte unsigned integer. Common uses: UniqueID
* `int64` - 8-byte signed integer. Common uses: Money, Date
* `uint64` - 8-byte unsigned integer. Common uses: GameID
* `float` - 4-byte floating point. Common uses: Scale, Speed
* `double` - 8-byte floating point.*
* `angle` - 2-byte rotation (usually represented as radians). Common uses: Direction
* `vec3` - 12-byte vector of 3 `float`s. Common uses: Position
* `vec3fa` - 12-byte vector of 3 `angle`s inefficiently stored as integer `float`s. Common uses: Cosmetic rotation
* `skillid` - 8-byte (patch >=74) or 4-byte (patch <=73) skill ID.
* `customize` - 8-byte array of character customization fields.

\* Rarely used.

### Variable-Length Types
These are automatically hoisted to the top should they appear after other data.
* `string` - Unicode string.
* `array` - An array of objects.
* `bytes` - An array of bytes.

### Meta-Types
* `object` - Lets you group other data in its own namespace.

## Packet data structure
* All data types are little-endian.
* All packets start with `uint16 size`, `uint16 packetID`.
* `string`: `uint16 offset` directly after header, which points to a 0-terminated UCS2 string.
* `bytes`: `uint16 offset`, `uint16 count` directly after header, which points to an array of bytes.
* `array`: `uint16 count`, `uint16 offset` directly after the header, which points to the first `<node>`.
* * `<node>`: `uint16 offset`, `uint16 nextOffset`, followed by data.

## `.def` versioning
**When submitting changes, contributors _must_ leave older versions untouched unless they are trivially backwards compatible. Instead, submit the changed definition as a new file with the version number incremented.**