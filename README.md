# CASA Spack Build Environment for Linux

Spack environment for CASA6 build dependencies on Linux (RHEL 8 / x86_64). Make sure
to have the spack `setup-env.sh` sourced into your shell before running any of the
below commands.

```bash
source ~/path/to/spack_install/share/spack/setup-env.sh
```

or add that line into your `~/.bashrc`.

> **Note:** This build recipe was developed with LLM assistance (Claude) but has
> been debugged and verified to produce a working build through every stage of the
> CASA6 build pipeline (libsakura, casacore, casacpp, casatools, casatasks,
> casashell).

## Prerequisites

- **Spack** (https://spack.io)
- **System GCC** (gcc, g++, gfortran) — RHEL 8 system gcc@8.5.0 is sufficient
- **`spack compiler find`** run once to register the system compiler

All other dependencies (cmake, grpc, protobuf, openmpi, python@3.12, etc.) are
managed by Spack.

Register the system compiler if you haven't already:

```bash
spack compiler find
```

## Setup

Clone this repository:

```bash
git clone <repo-url> /path/to/casa-spack-linux
```

Create and install the environment:

```bash
spack env create casa-dev /path/to/casa-spack-linux/spack.yaml
spack env activate casa-dev
spack concretize
spack install
```

After any changes to `spack.yaml`, recreate the environment:

```bash
spack env rm casa-dev
spack env create casa-dev /path/to/casa-spack-linux/spack.yaml
spack env activate casa-dev
spack concretize
spack install
```

## Usage

Activate the environment before building CASA:

```bash
spack env activate casa-dev
```

Python packages (numpy, pip, build) are not managed by Spack. They are installed
into a venv by the Makefile automatically.

## Makefile

The included `Makefile` is a modified version of the upstream CASA6 Makefile with
the following changes for Spack/Linux compatibility:

- **ccache**: Replaced legacy `-DUseCcache=1` with standard cmake launcher flags
  (`-DCMAKE_C_COMPILER_LAUNCHER=ccache`, `-DCMAKE_CXX_COMPILER_LAUNCHER=ccache`)
  on all cmake targets
- **OpenMPI path**: Removed hardcoded `/usr/lib64/openmpi/bin/` from PATH; Spack
  provides openmpi in the environment PATH
- **No compiler pinning**: Unlike the macOS variant, no `-DCMAKE_C_COMPILER` /
  `-DCMAKE_CXX_COMPILER` overrides are needed — GCC is used consistently for
  C, C++, and Fortran

Copy the Makefile into a clean build directory with the spack env active, then:

```bash
make firstcasa
```

## grpc / protobuf version pinning

Spack's latest grpc/protobuf/abseil-cpp versions do not form a mutually compatible
set with the re2 version that grpc hard-pins. The following versions are known to
work together:

| Package | Version |
|---------|---------|
| grpc | 1.67 (cxxstd=17) |
| protobuf | 26.1 |
| abseil-cpp | 20240116.1 |
| re2 | 2023-09-01 (grpc hard-pin) |

See comments in `spack.yaml` for details.

## Known Issues

### `spack env deactivate` may corrupt PATH

`spack env deactivate` can strip entries from PATH and not restore them on
re-activate. This is a known Spack bug
([spack#48391](https://github.com/spack/spack/issues/48391)).

**Workaround:** Always activate from a fresh shell rather than deactivating and
re-activating:

```bash
# Open a new terminal, then:
source ~/src/spack/share/spack/setup-env.sh
spack env activate casa-dev
```
