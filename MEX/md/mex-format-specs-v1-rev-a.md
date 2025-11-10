# DRAFT
TODO: apparently I was gone too far and string table is reductant and serve no use
TODO: add symbol table number/size at the header because I forgot and now it's impossible to load
# MEX Format Specifications
**Version `1.0` | Revision `A` | `9/10/2025`**

## Foreword
This document describes the format of **MEX** files and basic concepts related to it.

Please look at [**Terminology**](#12-terminology) for definitions of terms used in this
document.

## Table of Contents

- **1: [Introduction](#1-introduction)**
- **2: [MEX Header](#2-mex-header)**
- **3: [MEX Segment Header](#3-mex-segment-header-table)**
- **4: [MEX String Table](#4-mex-string-table)**
- **5: [MEX Symbol Table](#5-mex-symbol-table)**
- **6: [MEX Relocation Header](#6-mex-relocation-header-table)**
- **Appendix: [Copyright](#copyright)**

---

## 1. Introduction
**MEX** (**M**inimal **EX**cutable) Format is a binary format for storing and
executing small programs. It is designed to be simple, efficient, and easy to parse.

### 1.1 About MEX
**MEX** is very minimal. It only contains an (_almost_) bare minimum to execute a program.

**MEX** is based on the **ELF** format but is much simpler and easier to parse.

**MEX** is designed to be minimal while having more features compared to **flat binary** such as:
- Relocation
- Segmentation
- Zero-initialized Data Segment
- Memory Protection (Read/Write/Execute)
- Endianness support (Little/Big)
- 32-bit and 64-bit support

### 1.2 Terminology
In this document, the following terms are used:

| Term                 | Description                                                                               |
|----------------------|-------------------------------------------------------------------------------------------|
| **MEX**              | A file in **MEX** format.                                                                 |
| **Data**             | The actual bytes contained in a field of a MEX file structure.                            |
| **Segment**          | A contiguous block of memory.                                                             |
| **Relocation**       | A reference to a symbol in another segment.                                               |
| **Symbol**           | A named reference to a location in memory.                                                |
| **Virtual Address**  | The virtual address of a location in memory.                                              |
| **Address**          | Alias for **Virtual Address**.                                                            |
| **Physical Address** | The physical address of a location in memory.                                             |
| **Loader**           | A program that loads and executes **MEX** files.                                          |
| **Linker**           | A program that produces **MEX** files.                                                    |
| **Endian**           | The byte order of multi-byte data types.                                                  |
| **Little Endian**    | The byte order of multi-byte data types where the least-significant byte is stored first. |
| **Big Endian**       | The byte order of multi-byte data types where the most-significant byte is stored first.  |
| **Enumerations**     | A set of named values.                                                                    |
| **Enum**             | Alias for **Enumeration**.                                                                |
| **32-bit Header**    | A header structure for **MEX** file with 32-bit file class.                               |
| **64-bit Header**    | A header structure for **MEX** file with 64-bit file class.                               |
| **Zero-initialized** | Data that is initialized to zero at runtime.                                              |

### 1.3 Data Types
**MEX** uses the following data types:

| Name            | Size | Description               |
|-----------------|------|---------------------------|
| `uint8_t`       | 1    | 8-bit unsigned integer.   |
| `uint16_t`      | 2    | 16-bit unsigned integer.  |
| `uint32_t`      | 4    | 32-bit unsigned integer.  |
| `int32_t`       | 4    | 32-bit signed integer.    |
| `uint64_t`      | 8    | 64-bit unsigned integer.  |
| `int64_t`       | 8    | 64-bit signed integer.    |

All **data** must be padded and aligned to their natural size. As an example, `uint32_t` must be padded to 4 bytes.

All **offsets** and **sizes** are stored in bytes.

All **offsets** are relative to the beginning of the file.

All unspecified **enum** are reserved for future use.

### 1.4 File Format
A **MEX** file consists of a header, followed by a segment header table, a relocation header table, and the segments
themselves.

The following is the structure of a **MEX** file:

| MEX File                |
|-------------------------|
| MEX Header              |
| Segment Header Table    |
| String Table            |
| Symbol Table            |
| Relocation Header Table |
| Segment 1               |
| Segment 2               |
| ...                     |

Each **Header** has their own 32-bit and 64-bit versions.

---

## 2. MEX Header
The **MEX** header is the first thing in a **MEX** file. It contains information about the file and the segments it
contains.

The **MEX** header is always at the beginning of the file (offset `0`).

### 2.1 Contents of the MEX Header
The header of a **MEX** file contains the following fields:

#### 32-bit Header
| Offset | Size | Field            | Description                        |
|--------|------|------------------|------------------------------------|
| 0      | 8    | MEX Ident        | MEX identification structure.      |
| 8      | 4    | Entry Point      | Entry point of the program.        |
| 12     | 2    | Segment Count    | Number of segments in the file.    |
| 14     | 2    | Relocation Count | Number of relocations in the file. |
| 16     |      |                  | End of The Header (Size)           |

#### 64-bit Header
| Offset | Size | Field            | Description                        |
|--------|------|------------------|------------------------------------|
| 0      | 8    | MEX Ident        | MEX identification structure.      |
| 8      | 8    | Entry Point      | Entry point of the program.        |
| 16     | 2    | Segment Count    | Number of segments in the file.    |
| 18     | 2    | Relocation Count | Number of relocations in the file. |
| 20     |      |                  | End of The Header (Size)           |

#### MEX Identification
The identification structure is described in the [**MEX Identification**](#22-mex-identification) section.

#### Entry Point
The entry point of the program. Contains the virtual address of the first instruction to execute.

#### Segment Count
The number of segment header table entries in the file.

#### Relocation Count
The number of relocation header table entries in the file.

#### C Examples
```c++
// 32-bit header
typedef struct {
    MexIdent ident;
    uint32_t entry_point;
    uint16_t segment_count;
    uint16_t relocation_count;
} MexHeader32;

// 64-bit header
typedef struct {
    MexIdent ident;
    uint64_t entry_point;
    uint16_t segment_count;
    uint16_t relocation_count;
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
| 8      |      |            | End of Identification (Size)     |

#### Magic
The magic number is a 4-byte array of `0x7F`, `M`, `E`, `X` in order. Where `M`, `E`, and `X` are ASCII characters.

```text
{ 0x7F, 'M', 'E', 'X' }
```

#### Endianness
The endianness of the file. Can be either `0x00` for little endian or `0x01` for big endian.

| Value | Endianness    |
|-------|---------------|
| 0x00  | Little Endian |
| 0x01  | Big Endian    |

#### Version
The version of the **MEX** format. This document describes version `0x01`. Any field after this field is not guaranteed
to be compatible with other versions.

#### File Class
The file class of the **MEX** file. Can be either `0x00` for 32-bit or `0x01` for 64-bit.

| Value | File Class |
|-------|------------|
| 0x00  | 32-bit     |
| 0x01  | 64-bit     |

#### Padding
Padding to align the structure to 8 bytes. This byte must be `0x00`.

#### C Examples
```c++
typedef struct {
    uint8_t magic[4] = {0x7F, 'M', 'E', 'X'};
    uint8_t endianness;
    uint8_t version;
    uint8_t file_class;
    uint8_t padding;
} MexIdent;
```

---

## 3. MEX Segment Header Table
Segment header table is an array of segment header table entry. Each segment header describes a segment in the file.
The segment header table is located **immediately** after the **MEX** header.

### 3.1 Segment Header Table Entry
The segment header table entry is described in the following table:

#### 32-bit Header
| Offset | Size | Field             | Description                              |
|--------|------|-------------------|------------------------------------------|
| 0      | 1    | Segment Type      | Type of the segment.                     |
| 1      | 3    | Padding           |                                          |
| 4      | 4    | Segment Offset    | Offset of the segment in the file.       |
| 8      | 4    | Segment Size      | Size of the segment in the file.         |
| 12     | 4    | Segment Virtual   | Starting virtual address of the segment. |
| 16     | 4    | Segment Alignment | Alignment of the segment.                |
| 20     |      |                   | End of Segment Header (Size)             |

#### 64-bit Header
| Offset | Size | Field             | Description                              |
|--------|------|-------------------|------------------------------------------|
| 0      | 1    | Segment Type      | Type of the segment.                     |
| 1      | 7    | Padding           |                                          |
| 8      | 8    | Segment Offset    | Offset of the segment in the file.       |
| 16     | 8    | Segment Size      | Size of the segment in the file.         |
| 24     | 8    | Segment Virtual   | Starting virtual address of the segment. |
| 32     | 8    | Segment Alignment | Alignment of the segment.                |
| 40     |      |                   | End of Segment Header (Size)             |

#### Segment Type
Type of the segment. Must be one of the following enum values:

| Value | Type          |
|-------|---------------|
| 0x00  | Null          |
| 0x01  | Read          |
| 0x02  | Write         |
| 0x03  | Execute       |
| 0x04  | Write/Execute |
| 0x05  | Custom        |
| 0x06  | Zero-init     |

```c++
typedef enum {
    kMexSegmentTypeNull = 0x00,
    kMexSegmentTypeRead = 0x01,
    kMexSegmentTypeWrite = 0x02,
    kMexSegmentTypeExec = 0x03,
    kMexSegmentTypeWriteExec = 0x04,
    kMexSegmentTypeCustom = 0x05,
    kMexSegmentTypeZeroInit = 0x06,
    kMexSegmentTypeZeroInitExec = 0x07,
} MexSegmentType;
```

- **Null**: This segment is not used and must be ignored.
- **Read**: This segment is read-only. **(R)**
- **Write**: This segment is read-write. **(RW)**
- **Execute**: This segment is read-execute. **(RX)**
- **Write/Execute**: This segment is read-write-execute. **(RWX)**
- **Custom**: This segment is custom defined by the program/loader, whose format and meaning is determined solely by the
  program/loader.
- **Zero-init**: This segment is zero-initialized. **(RW)**
- **Zero-init-exec**: This segment is zero-initialized and read-write-execute. **(RWX)**

#### Padding
Padding to align the structure to 4 or 8 bytes respectively to their file class. These bytes must be `0x00`.

#### Segment Offset
Offset of the segment in the file.

#### Segment Size
Size of the segment in the file.

#### Segment Virtual
Starting virtual address of the segment.

#### Segment Alignment
Specifies the required alignment of the segment in both the file and memory.
Values 0 or 1 indicate that no alignment is required.
Otherwise, this value must be a positive power of two.

Both the segment’s file offset and its virtual address must be aligned to this boundary:
```text
(segment_offset % segment_align) == 0
(segment_vaddr  % segment_align) == 0
```

The segment’s file offset and virtual address must be congruent modulo the alignment value:
```text
(segment_offset % segment_align) == (segment_vaddr % segment_align)
```

#### C Examples
```c++
// 32-bit segment header table entry
typedef struct {
    uint8_t segment_type;
    uint8_t padding[3];
    uint32_t segment_offset;
    uint32_t segment_size;
    uint32_t segment_virtual;
    uint32_t segment_alignment;
} MexSegmentHeader32;

// 64-bit segment header table entry
typedef struct {
    uint8_t segment_type;
    uint8_t padding[7];
    uint64_t segment_offset;
    uint64_t segment_size;
    uint64_t segment_virtual;
    uint64_t segment_alignment;
} MexSegmentHeader64;
```

---

## 4. MEX String Table
String table is an array of **null-terminated** strings. Each string is referenced by its index in the string table.
The string table is located **immediately** after the **segment header table**.

The first index `0` is reserved and must be a null character (i.e. `'\0'`).

The string table index refer to any byte in the string table.

The index is limited to the integer limit of uint32_t (32-bit) or uint64_t (64-bit) depending on the file class.

The table is stored in the same format as the
[**ELF String Table**](https://gabi.xinuos.com/elf/04-strtab.html#string-table). See
[**ELF String Table**](https://gabi.xinuos.com/elf/04-strtab.html#string-table) for more information.

Examples of string table:

| 0    | 1   | 2   | 3   | 4   | 5   | 6   | 7   | 8   | 9    | 10  | 11  |
|------|-----|-----|-----|-----|-----|-----|-----|-----|------|-----|-----|
| '\0' | 'V' | 'a' | 'r' | 'i' | 'a' | 'b' | 'l' | 'e' | '\0' | 'n' | 'a' |

| 12  | 13  | 14  | 15   | 16  | 17  | 18   | 19  | 20   | 21  | 22  | 23   |
|-----|-----|-----|------|-----|-----|------|-----|------|-----|-----|------|
| 'm' | 'e' | 'x' | '\0' | 'x' | 'x' | '\0' | 'y' | '\0' | 'e' | 'z' | '\0' |

---

| Index | String     |
|-------|------------|
| 0     | _none_     |
| 1     | "Variable" |
| 10    | "name."    |
| 16    | "xx"       |
| 19    | "y"        |
| 21    | "ez"       |

---

## 5. MEX Symbol Table
Symbol table is an array of symbol table entry. Each symbol table entry describes a symbol in the file.
The symbol table is located **immediately** after the **string table**.

Index `0` is reserved and must be a [**null symbol**](#null-symbol).

### 5.1 Symbol Table Entry
The symbol table entry is described in the following table:

#### 32-bit Header
| Offset | Size | Field             | Description                                 |
|--------|------|-------------------|---------------------------------------------|
| 0      | 4    | Symbol Name       | Name of the symbol inside the string table. |
| 4      | 4    | Symbol Value      | Value of the symbol.                        |
| 12     |      |                   | End of Symbol Header (Size)                 |

#### 64-bit Header
| Offset | Size | Field             | Description                                 |
|--------|------|-------------------|---------------------------------------------|
| 0      | 8    | Symbol Name       | Name of the symbol inside the string table. |
| 8      | 8    | Symbol Value      | Value of the symbol.                        |
| 16     |      |                   | End of Symbol Header (Size)                 |

#### Null Symbol
The null symbol is a symbol with a string index of `0` for a name and a value of `0`.

##### C Examples
```c++
// 32-bit null symbol
MexSymbol32 null_symbol_32 = { 0, 0 };

// 64-bit null symbol
MexSymbol64 null_symbol_64 = { 0, 0 };
```

#### Symbol Name
Name of the symbol inside the string table.

This refers to the index of the string in the string table.

#### Symbol Value
The virtual address of the symbol.

#### C Examples
```c++
// 32-bit symbol table entry
typedef struct {
    uint32_t symbol_name;
    uint32_t symbol_value;
} MexSymbol32;

// 64-bit symbol table entry
typedef struct {
    uint64_t symbol_name;
    uint64_t symbol_value;
} MexSymbol64;
```

---

## 6. MEX Relocation Header Table
Relocation header table is an array of relocation header table entry. Each relocation header describes a relocation in
the file.
The relocation header table is located **immediately** after the **symbol table**.

### 6.1 Relocation Header Table Entry
The relocation header table entry is described in the following table:

The entry is stored in the same format as the
[**ELF Relocation Entry**](https://gabi.xinuos.com/elf/06-reloc.html#relocation-entry). See
[**ELF Relocation Entry**](https://gabi.xinuos.com/elf/06-reloc.html#relocation-entry) for more information.

#### 32-bit Header
| Offset | Size | Field             | Description                                     |
|--------|------|-------------------|-------------------------------------------------|
| 0      | 4    | Relocation Offset | Offset to the virtual address to patch          |
| 4      | 4    | Relocation Info   | Contain relocation types and symbol table index |
| 8      | 4    | Relocation Addend | Addend to the virtual address to patch          |
| 12     |      |                   | End of Relocation Header (Size)                 |

#### 64-bit Header
| Offset | Size | Field             | Description                                     |
|--------|------|-------------------|-------------------------------------------------|
| 0      | 8    | Relocation Offset | Offset to the virtual address to patch          |
| 8      | 8    | Relocation Info   | Contain relocation types and symbol table index |
| 16     | 8    | Relocation Addend | Addend to the virtual address to patch          |
| 24     |      |                   | End of Relocation Header (Size)                 |

#### Relocation Offset
Offset to the virtual address to patch.

#### Relocation Info
Relocation types and symbol table index.

To determine the relocation type, the following bit mask is used:

##### 32-bit bitmask
| Bits   | Description          |
|--------|----------------------|
| 0-7    | Relocation Type      |
| 8-31   | Symbol Table Index   |

##### 64-bit bitmask
| Bits   | Description          |
|--------|----------------------|
| 0-31   | Relocation Type      |
| 32-63  | Symbol Table Index   |

---

##### Relocation Type
The relocation type specifies how to interpret and apply the relocation. Relocation types are processor-specific;
descriptions of their behavior appear in the `psABI` supplement

##### Symbol Table Index
The symbol table index is the index of the symbol in the symbol table.

---

#### Relocation Addend
A constant addend used to compute the value to be stored into the relocatable field.

#### C Examples
```c++
// 32-bit relocation header table entry
typedef struct {
    uint32_t relocation_offset;
    uint32_t relocation_info;
    int32_t relocation_addend;
} MexRelocationHeader32;

// 64-bit relocation header table entry
typedef struct {
    uint64_t relocation_offset;
    uint64_t relocation_info;
    int64_t relocation_addend;
} MexRelocationHeader64;
```

---

## Copyright
**Copyright © 2025 TheMonHub**

### Permissions
- You may copy and distribute verbatim copies of this specification document. Modifications of this document itself are
  not permitted.
- Code examples and snippets included in this specification may be copied, distributed, and modified for any purpose,
  without restriction.

### Disclaimer
**This document is provided "as is" without any express or implied warranty. The authors make no representations or
warranties of any kind, express or implied, including, but not limited to, warranties of merchantability, fitness for a
particular purpose, or non-infringement.**