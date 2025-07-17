# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

PForth is a portable ANS-like Forth interpreter written in ANSI C. It consists of a C kernel providing primitive operations and a Forth dictionary with higher-level words. The project supports both embedded systems (without file I/O) and desktop platforms with full features.

## Build System and Commands

### Using CMake (Recommended)
```bash
cmake .
make
cd fth
./pforth_standalone
```

### Using Unix Makefile
```bash
cd platforms/unix
make all
./pforth_standalone
```

### Build Targets and Process
The build process has 4 distinct stages:
1. **pforth** - Basic executable with minimal C-based dictionary
2. **pforth.dic** - Dictionary file created by running `pforth -i system.fth`
3. **pfdicdat.h** - Dictionary exported as C header via `mkdicdat.fth`
4. **pforth_standalone** - Final executable with embedded dictionary

### Testing
```bash
# Using Unix Makefile
cd platforms/unix
make test

# Using CMake
cd fth
./pforth
include tester.fth
include coretest.fth
```

### Individual test files
```bash
pforth t_corex.fth
pforth t_strings.fth
pforth t_locals.fth
pforth t_alloc.fth
pforth t_floats.fth
pforth t_file.fth
```

## Architecture Overview

### C Kernel (`csrc/`)
- **pf_main.c** - Main entry point and command-line handling
- **pf_core.c** - Core Forth engine and inner interpreter
- **pf_words.c** - Implementation of primitive Forth words
- **pf_mem.c** - Memory management for dictionary and stacks
- **pf_io.c** - Platform-specific I/O abstraction layer
- **pforth.h** - Public API for embedding PForth in applications

### I/O Platform Abstraction
- **stdio/** - Generic platform using standard C I/O
- **posix/** - POSIX-specific implementation with advanced terminal features
- **win32/** - Windows-specific implementations

### Forth Dictionary (`fth/`)
- **system.fth** - Core system words and bootstrap definitions
- **loadp4th.fth** - Master loader that conditionally includes all modules
- **mkdicdat.fth** - Dictionary compiler that generates C header files
- **savedicd.fth** - Dictionary serialization for embedded deployment

### Key Forth Modules
- **locals.fth** - Local variable support
- **floats.fth** - Floating-point arithmetic
- **strings.fth** - String manipulation
- **file.fth** - File I/O operations
- **see.fth** - Word disassembler/decompiler
- **trace.fth** - Execution debugging

## Development Workflow

### Dictionary Development
1. Modify Forth source files in `fth/`
2. Rebuild dictionary: `./pforth -i system.fth`
3. Test changes: `./pforth -d pforth.dic`
4. For embedded: regenerate `pfdicdat.h` and rebuild standalone

### C Kernel Development
1. Modify C source files in `csrc/`
2. Rebuild kernel: `make pforth` (or `make` with CMake)
3. Rebuild dictionary and test

### Cross-Platform Building
- **platforms/unix/** - Standard Unix/Linux/macOS
- **platforms/mingw-crossbuild-linux/** - Windows cross-compilation
- **platforms/zig-crossbuild/** - Zig-based cross-compilation
- **platforms/linux-crossbuild-amiga/** - Amiga cross-compilation

## Key Configuration Options

### Compile-Time Flags
- **PF_SUPPORT_FP** - Enable floating-point support
- **PF_STATIC_DIC** - Embed dictionary in executable
- **PF_NO_MALLOC** - Disable dynamic memory allocation
- **PF_NO_FILEIO** - Disable file I/O for embedded systems
- **PF_EMBEDDED** - Full embedded configuration

### Build Variables
- **WIDTHOPT** - Set to `-m32` for 32-bit builds
- **IO_SOURCE** - Select I/O implementation (posix/stdio/win32)
- **ASAN** - Enable AddressSanitizer for debugging

## Important Implementation Details

### Memory Organization
- **Dictionary** - Two-space design with separate name space and code space
- **Stacks** - Configurable depth parameter and return stacks
- **Cells** - Use `cell_t` (intptr_t) for portability across 32/64-bit systems

### Bootstrap Process
1. C kernel provides ~30 primitive words
2. `system.fth` builds basic Forth infrastructure
3. `loadp4th.fth` conditionally loads extended functionality
4. Dictionary saved as binary file or C header

### Testing Framework
- **tester.fth** - Test framework with `{ ... -> ... }` syntax
- **coretest.fth** - ANS Forth compliance tests
- Individual test files for each major module

## Embedded System Support

PForth is designed for embedded systems with minimal requirements:
- No file I/O required (dictionary can be embedded)
- Configurable memory usage
- Platform-specific I/O abstraction
- Cross-compilation support for various architectures