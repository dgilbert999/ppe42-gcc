Get the GCC source
=================
// Assisted by WCA@IBM
// Latest GenAI contribution: ibm/granite-20b-code-instruct-v2

```bash
git clone https://github.com/open-power/ppe42-gcc.git -b gcc-4_9_2-ppe42
```

Get the binutils source
=======================

```bash
git clone https://github.com/open-power/ppe42-binutils -b binutils-2_24-ppe42
```

Compile to CTE tools (or other location)
========================================

Note: GCC has hard-coded locations. If you copy the generated gcc binaries to a different location other than where it was compiled,
 some of the internal internal links will still point back to the location where gcc was compiled.

Set the environment variable PREFIX to the location where the generated gcc will reside. The example is for CTE ppetools.
If compiled here the generated toolchain will be replicated overnight to all CTE tool site locations.

```bash
export PREFIX=/afs/apd/projects/cte/tools/ppetools/prod/
```

```bash
export TARGET=powerpc-eabi
```

Compile Dependencies
====================

The tool-chain requires gmp at least version 4.3.2, mpfr at least at version 2.4.2, mpc at least version 0.8.1;
If not, then they must be built.
If any of these dependencies are required then be sure to set/add to LD_LIBRARY_PATH=$(PREFIX)/lib before running any tool-chain executables.

Compile ppe42 tool chain
========================

binutils
--------

Using LDFLAGS="-all-static" causes the the executable to have all depenent libs compiled in to the executable making it relocatable.
This is desired when delivering to CTE, but not needed otherwise. If the flag is not specified, then linkable libraries are generated,
 some of which are needed for SIMICS when new asm instructions are added. (libbfd-2.24.so and libopcodes-2.24.so)

From the ppe42-binutils source directory:

```bash
mkdir ../build_binutils
cd ../build_binutils
rm -Rf *
../ppe42-binutils/configure --enable-shared --enable-64-bit-bfd --target=$TARGET --prefix=$PREFIX -v
make configure-host
make LDFLAGS=-all-static MAKEINFO=true CFLAGS+=-Wno-implicit-fallthrough CFLAGS+=-Wno-error=cast-function-type CFLAGS+=-Wno-error=stringop-truncation CFLAGS+=-Wno-error=pointer-compare
make MAKEINFO=true install
```

GCC
---

Download these patches:
 - https://github.com/open-power/op-build/blob/master/openpower/package/ppe42-gcc/0001-2016-02-19-Jakub-Jelinek-jakub-redhat.com.patch
 - https://github.com/open-power/op-build/blob/master/openpower/package/ppe42-gcc/0001-reload-Change-to-type-of-x_spill_indirect_levels.patch

From the ppe42-gcc directory:

```bash
patch -p 1 -i 0001-2016-02-19-Jakub-Jelinek-jakub-redhat.com.patch
patch -p 1 -i 0001-reload-Change-to-type-of-x_spill_indirect_levels.patch

mkdir ../build_gcc
cd ../build_gcc
rm -Rf *
../ppe42-gcc/configure --target=$TARGET --prefix=$PREFIX --without-headers --with-newlib --with-gnu-as --with-gnu-ld --disable-doc --disable-pod2man
make configure-host
make  MAKEINFO=true all-gcc
make  MAKEINFO=true install-gcc
```

FLAGS to use when compiling
============================

C FLAGS
-------

```
PPE_CFLAGS =  -Wall -Werror
PPE_CFLAGS += -fsigned-char
PPE_CFLAGS += -msoft-float
PPE_CFLAGS += -meabi
PPE_CFLAGS += -msdata=eabi
PPE_CFLAGS += -ffreestanding
PPE_CFLAGS += -fno-common
PPE_CFLAGS += -fno-inline-functions-called-once
PPE_CFLAGS += -Os
PPE_CFLAGS += -gpubnames -gdwarf-3
PPE_CFLAGS += -ffunction-sections
PPE_CFLAGS += -fdata-sections
PPE_CFLAGS += -mcpu=ppe42x   # for older hw (p8,p9) use -mcpu=ppe42
PPE_CFLAGS += -mstrict-align
PPE_CFLAGS += -pipe
PPE_CFLAGS += -mmulhw  #Only on PPE42X
```

C++ aditional flags
-------------------

```
PPE_CXXFLAGS = -std=c++11
PPE_CXXFLAGS += -nostdinc++
PPE_CXXFLAGS += -fno-rtti
PPE_CXXFLAGS += -fno-threadsafe-statics
PPE_CXXFLAGS += -fno-strict-aliasing
PPE_CXXFLAGS += -fno-exceptions
```
Assembler
---------

```
ASFLAGS = -mppe42x   ## for older hw (P8,P9) use -mppe42
```

