# MEX Format Specifications
**Version `1.0` | Revision `A` | `DRAFT`**

## Table of Contents

- [Introduction](#introduction)
  - [About MEX](#about-mex)
  - [Features](#features)
- [Terminology](#terminology)
- [MEX](#mex)
  - [Header](#header)
- [Copyright](#copyright)

## Introduction
**MEX** (**M**inimal **EX**cutable) Format is a binary format for storing and
executing small programs. It is designed to be simple, efficient, and easy to parse.

This document describes the format of **MEX** files.

Please look at [**Terminology**](#terminology) for definitions of terms used in this
document.

### About MEX
**MEX** is very minimal. It only contains an (_almost_) bare minimum to execute a program.

**MEX** is designed with `X86_64` in mind. Which makes `X86_64` the **primary target** and **efficient** architecture for **MEX**.

### Features
- Simple and easy to parse for loaders.
- Relocations.
- Memory protection for segments.

## Terminology
In this document, the following terms are used:
- **MEX file** - A file in **MEX** format.
- **Segment** - A contiguous block of memory.
- **Relocation** - A reference to a symbol in another segment.
- **Symbol** - A named reference to a location in memory.
- **Loader** - A program that loads and executes **MEX** files.
- **Executable** - Alias for **MEX file**.

## MEX
This section describes the format of **MEX** files.

### Header
The header of a **MEX** file contains the following fields:
Empty!

## Copyright
Copyright © 2025 TheMonHub.

Everyone is permitted to copy and distribute verbatim copies of this
license document, but changing it is not allowed.