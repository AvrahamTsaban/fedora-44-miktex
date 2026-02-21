# Building MiKTeX on Fedora 44 (Development Branch)

This guide summarizes the process of building MiKTeX from source on Fedora 44. Since Fedora 44 is in its pre-beta stage, official MiKTeX repositories are not yet available, and system dependencies (HarfBuzz, ICU, etc.) have versions that differ from MiKTeX’s expected internal defaults.

## The Challenges

1.  **Dependency Mismatch:** The MiKTeX build system defaults to looking for internal library versions (e.g., `-lmiktex-harfbuzz`) which do not exist on Fedora.
2.  **Header Resolution:** Even with system libraries installed, the compiler often fails to locate specific headers like `hb.h` due to non-standard include paths.
3.  **Library Fragmentation:** On Fedora, HarfBuzz is split into several components (e.g., `libharfbuzz` and `libharfbuzz-subset`). MiKTeX’s linker expects these to be unified or named differently.
4.  **Symbol Errors:** Linking fails if `hb_subset` functions are not explicitly linked, as they reside in a separate library file in Fedora.

## Prerequisites

Ensure the following (or newer) development tools are installed:
- **CMake:** 4.2.3
- **GCC-C++:** 16.0.1
- **HarfBuzz-devel:** 12.3.2
- **Libicu-devel:** 77.1
- **Freetype-devel & Graphite2-devel**

## Build Instructions

### 1. Library Naming Workaround (Symlinks)
To satisfy the linker's search for "MiKTeX-branded" libraries, create symbolic links to the existing system libraries:

```bash
sudo ln -s /usr/lib64/libharfbuzz.so /usr/lib64/libmiktex-harfbuzz.so
sudo ln -s /usr/lib64/libharfbuzz-subset.so /usr/lib64/libmiktex-harfbuzz-subset.so
```

### 2. Configure the Build

Use the following `cmake` command to force the use of system libraries, define header paths, and inject the required linker flags:

```bash
rm -rf CMakeCache.txt

cmake -DMIKTEX_LINUX_DIST=fedora \
      -DMIKTEX_USE_SYSTEM_LIBRARIES=ON \
      -DCMAKE_PREFIX_PATH=/usr \
      -DCMAKE_INCLUDE_PATH=/usr/include/harfbuzz \
      -DCMAKE_EXE_LINKER_FLAGS="-lharfbuzz-subset -lharfbuzz -licuuc -licui18n" \
      ../miktex/

```

### 3. Compile

```bash
make

```

### 4. Package as RPM

Ensure you have ownership of the build directory to avoid permission errors during manifest generation:

```bash
sudo chown -R $USER:$USER .
cpack -G RPM

```

## Installation

The resulting RPM package can be installed via DNF:

```bash
sudo dnf install ./miktex-*.rpm

```

# MiKTeX

The [MiKTeX Project Page](https://miktex.org) is the place to go, if
you are new to MiKTeX.

In short: MiKTeX is a modern C/C++ implementation of TeX & Friends for Windows, macOS and Linux. The MiKTeX source code is documented here:
[https://docs.miktex.org/hacking/index.html](https://docs.miktex.org/hacking/index.html)

MiKTeX is also a scalable TeX distribution (["Just enough TeX"](https://miktex.org/kb/just-enough-tex)):

- you have the option to start with MiKTeX executables and some configuration files
- in the course of authoring your documents, only necessary LaTeX packages, fonts etc.
  will be downloaded and installed

## Building

MiKTeX can be built on Windows and Unix-like (including macOS)
systems.  Please consult these HOWTOs for platform-specific build
instructions:

- [https://miktex.org/howto/build-win](https://miktex.org/howto/build-win "Building MiKTeX (Windows)")
- [https://miktex.org/howto/build-unx](https://miktex.org/howto/build-unx "Building MiKTeX (Unix-like)")
- [https://miktex.org/howto/build-mac](https://miktex.org/howto/build-mac "Building MiKTeX (macOS)")

In addition, you can try one of the Dockerized build environments to build MiKTeX:

- [Ubuntu](https://github.com/MiKTeX/docker-miktex-build-ubuntu)
- [Debian](https://github.com/MiKTeX/docker-miktex-build-debian)
- [Fedora](https://github.com/MiKTeX/docker-miktex-build-fedora)
- [openSUSE](https://github.com/MiKTeX/docker-miktex-build-opensuse)

## Deep diving

If you want to understand the MiKTeX source code, have a look at [HACKING.md](HACKING.md).
