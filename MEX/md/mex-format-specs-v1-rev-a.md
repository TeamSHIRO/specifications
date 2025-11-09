# MEX Format Specifications
**Version `1.0` | Revision `A` | `DRAFT`**

## Table of Contents

1. [Introduction](#1-introduction)
   1. [About MEX](#11-about-mex)
   2. [Terminology](#12-terminology)
   3. [Data Types](#13-data-types)
   4. [File Format](#14-file-format)
2. [MEX Header](#2-mex-header)
   1. [Contents of the MEX Header](#21-contents-of-the-mex-header)
      - [32-bit Header](#32-bit-header)
      - [64-bit Header](#64-bit-header)
   2. [MEX Identification](#22-mex-identification)
3. [MEX Segment Header](#3-mex-segment-header)
4. [MEX Relocation Header](#4-mex-relocation-header)
5. [**Copyright**](#copyright)

## 1. Introduction
**MEX** (**M**inimal **EX**cutable) Format is a binary format for storing and
executing small programs. It is designed to be simple, efficient, and easy to parse.

This document describes the format of **MEX** files.

Please look at [**Terminology**](#12-terminology) for definitions of terms used in this
document.

### 1.1 About MEX
**MEX** is very minimal. It only contains an (_almost_) bare minimum to execute a program.

**MEX** is based on the **ELF** format but is much simpler and easier to parse.

### 1.2 Terminology
In this document, the following terms are used:
- **MEX File** - A file in **MEX** format.
- **File** - Alias for **MEX File**.


- **Data** - The actual bytes contained in a field of a MEX file structure.


- **Segment** - A contiguous block of memory.
- **Relocation** - A reference to a symbol in another segment.
- **Symbol** - A named reference to a location in memory.


- **Virtual Address** - The virtual address of a location in memory.
- **Address** - Alias for **Virtual Address**.
- **Physical Address** - The physical address of a location in memory.


- **Loader** - A program that loads and executes **MEX** files.
- **Linker** - A program that produces **MEX** files.

### 1.3 Data Types
**MEX** uses the following data types for its c++ examples:
- `uint8_t` - 8-bit unsigned integer.
- `uint16_t` - 16-bit unsigned integer.
- `uint32_t` - 32-bit unsigned integer.
- `uint64_t` - 64-bit unsigned integer.


- `int8_t` - 8-bit signed integer.
- `int16_t` - 16-bit signed integer.
- `int32_t` - 32-bit signed integer.
- `int64_t` - 64-bit signed integer.


- `char` - 8-bit character.
- `unsigned char` - 8-bit unsigned character.

All **Data** must be padded and aligned to their natural size. As an example, `uint32_t` must be padded to 4 bytes.

All **offsets** and **sizes** are stored in bytes.

### 1.4 File Format
A **MEX** file consists of a header and a list of segments.

The following is an example of a **MEX** file:

```text
+-------------------------+
| MEX Header              |
+-------------------------+
| Segment Header Table    |
+-------------------------+
| Segment 1               |
+-------------------------+
| Segment 2               |
+-------------------------+
| ...                     |
+-------------------------+
| Relocation Header Table |
+-------------------------+
```

Please note that the placement of the **segment header table** and the **relocation header table** is not fixed. They can be
placed anywhere in the file as long as the offsets are correctly specified in the header.

## 2. MEX Header
The **MEX** header is the first thing in a **MEX** file. It contains information about the file and the segments it contains.

The **MEX** header is always at the beginning of the file (offset `0`).

### 2.1 Contents of the MEX Header
The header of a **MEX** file contains the following fields:

#### 32-bit Header

| Offset | Size | Field             | Description                           |
|--------|------|-------------------|---------------------------------------|
| 0      | 8    | MEX Ident         | MEX identification structure.         |
| 8      | 4    | Entry Point       | Entry point of the program.           |
| 12     | 4    | Segment Count     | Number of segments in the file.       |
| 16     | 4    | Relocation Count  | Number of relocations in the file.    |
| 20     | 4    | Segment Offset    | Offset to the first segment table.    |
| 24     | 4    | Relocation Offset | Offset to the first relocation table. |
| 28     |      |                   | End of The Header (Size)              |

#### 64-bit Header

| Offset | Size | Field             | Description                                  |
|--------|------|-------------------|----------------------------------------------|
| 0      | 8    | MEX Ident         | MEX identification structure.                |
| 8      | 8    | Entry Point       | Entry point of the program.                  |
| 16     | 8    | Segment Count     | Number of segment headers in the file.       |
| 24     | 8    | Relocation Count  | Number of relocation headers in the file.    |
| 32     | 8    | Segment Offset    | Offset to the first segment header table.    |
| 40     | 8    | Relocation Offset | Offset to the first relocation header table. |
| 48     |      |                   | End of The Header (Size)                     |

#### MEX Identification
The identification structure is described in the [**MEX Identification**](#22-mex-identification) section.

#### Entry Point
The entry point of the program. Contain the virtual address of the first instruction to execute.

#### Segment Count
The number of segment headers in the file.

#### Relocation Count
The number of relocation headers in the file.

#### Segment Offset
The offset to the first segment header table.

#### Relocation Offset
The offset to the first relocation header table.

#### C++ Examples

```c++
// 32-bit header
typedef struct {
    MexIdent ident;
    uint32_t entry_point;
    uint32_t segment_count;
    uint32_t relocation_count;
    uint32_t segment_offset;
    uint32_t relocation_offset;
} MexHeader32;

// 64-bit header
typedef struct {
    MexIdent ident;
    uint64_t entry_point;
    uint64_t segment_count;
    uint64_t relocation_count;
    uint64_t segment_offset;
    uint64_t relocation_offset;
} MexHeader64;
```

### 2.2 MEX Identification

| Offset | Size | Field      | Description                      |
|--------|------|------------|----------------------------------|
| 0      | 4    | Magic      | A 4 bytes array of magic number. |
| 4      | 1    | Endianness | Endianness of the file.          |
| 5      | 1    | Version    | Version of the format.           |
| 6      | 1    | File Class | 32-bit or 64-bit File.           |
| 7      | 1    | Padding    |                                  |
| 8      |      |            | End of Identification (Size).    |

#### Magic
The magic number is a 4-byte array of `0x7F` `M` `E` `X`.

#### Endianness
The endianness of the file. Can be either `0x00` for little endian or `0x01` for big endian.

#### Version
The version of the **MEX** format. This document describes version `0x01`. Any field after this field is not guaranteed to be compatible with other versions.

#### File Class
The file class of the **MEX** file. Can be either `0x00` for 32-bit or `0x01` for 64-bit.

#### Padding
Padding to align the structure to 8 bytes. This byte can be arbitrary.

#### C++ Examples

```c++
typedef struct {
    char    magic[4] = {0x7F, 'M', 'E', 'X'};
    uint8_t endianness;
    uint8_t version;
    uint8_t file_class;
    char    padding;
} MexIdent;
```

## 3. MEX Segment Header
DRAFT EMPTY

## 4. MEX Relocation Header
DRAFT EMPTY

## Copyright
Copyright © 2025 TheMonHub.

Everyone is permitted to copy and distribute verbatim copies of this
license document, but changing it is not allowed.