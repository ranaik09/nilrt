# NI Linux Real-Time Repository Explanation

## Overview

The **nilrt** repository is the main build system for **NI Linux Real-Time (NILRT)**, a Real-Time Linux distribution created by National Instruments (now Emerson) for embedded hardware devices. This repository uses the OpenEmbedded framework to build packages, images, and installation media for NI's embedded controllers and hardware platforms.

## What is NI Linux Real-Time?

NI Linux Real-Time is a specialized Linux distribution with:
- **Real-Time scheduling** enabled using the `CONFIG_PREEMPT_RT` kernel patch
- **Deterministic performance** for time-critical applications
- Support for **NI embedded hardware** (controllers, CompactRIO, etc.)
- **OpenEmbedded/Yocto-based** build system for customization

It's primarily used in industrial automation, test & measurement, and robotics applications (including FIRST Robotics Competition).

## Repository Purpose

This repository serves multiple purposes:

1. **Build NILRT Distribution** - Creates complete OS images for NI devices
2. **Build Package Feeds** - Compiles IPK packages installable via opkg package manager
3. **Build Custom Kernels** - Enables community developers to build custom Linux kernels
4. **Build Toolchains** - Generates cross-compilation toolchains for developing applications
5. **Community Development** - Allows non-NI developers to build open-source software for their NI Linux RT devices

## Repository Structure

```
nilrt/
├── .github/                    # GitHub workflows and CI configuration
├── CHANGELOG.md               # Consolidated changelog for NILRT releases
├── COPYING.MIT                # License file (MIT license)
├── README.md                  # Main setup and build instructions
├── docker/                    # Docker container definitions for build environment
├── docs/                      # Additional documentation
│   ├── CONTRIBUTING.md        # Contribution guidelines
│   ├── README.kernel.md       # Kernel building documentation
│   └── package-feeds.md       # Package feed structure documentation
├── ni-oe-init-build-env       # Environment initialization script
├── pyrex.ini                  # Pyrex container configuration
├── scripts/                   # Build automation and utility scripts
│   ├── pipelines/            # CI/CD pipeline scripts
│   └── tests/                # Test scripts
└── sources/                   # Git submodules for OpenEmbedded layers
    ├── bitbake/              # BitBake build tool
    ├── openembedded-core/    # Core OpenEmbedded layer
    ├── meta-nilrt/           # NI-specific NILRT layer (main layer)
    ├── meta-openembedded/    # Additional OE layers
    ├── meta-qt5/             # Qt5 support
    ├── meta-security/        # Security-related packages
    ├── meta-virtualization/  # Container/VM support
    └── [other meta layers]   # Various specialized OE layers
```

## Key Components

### 1. OpenEmbedded/Yocto Build System

The repository is based on the **OpenEmbedded** build framework, which uses:
- **BitBake** - Build execution engine that processes recipes
- **Recipes** - Build instructions (`.bb` files) for packages
- **Layers** - Modular collections of recipes and configurations (meta-* directories)
- **Images** - Complete filesystem definitions for bootable systems

### 2. Pyrex Container System

**Pyrex** (from Garmin) is used to containerize the build environment:
- Provides consistent build environment via Docker
- Eliminates "works on my machine" problems
- Transparently wraps BitBake commands
- Configuration in `pyrex.ini`
- Build container defined in `docker/create-build-nilrt.sh`

### 3. Git Submodules

The `sources/` directory contains **14 git submodules**, each representing an OpenEmbedded layer:
- `meta-nilrt` - The core NI-specific layer (most important)
- `openembedded-core` - Base OE framework
- `bitbake` - Build tool
- Other specialized layers for Qt, security, virtualization, etc.

All submodules are synchronized to specific branches (currently `nilrt/master/scarthgap`).

## Development Mainlines

The project maintains **three concurrent development branches**:

1. **`nilrt/master/scarthgap`** - Current x64 development HEAD
2. **`nilrt/master/sumo`** - Current ARM32 development HEAD  
3. **`nilrt-academic/master/sumo`** - Fork for FIRST Robotics Competition (ARM32)

Product release branches are named like: `nilrt/25.8/scarthgap`, `nilrt/23.3/sumo`

## Build Process Overview

### Prerequisites

1. **Docker Engine** (not Docker Desktop) installed on Linux host
2. **Git** for cloning repository and submodules
3. Significant disk space (100s of GB for full builds)

### Setup Steps

```bash
# 1. Clone repository and submodules
git clone https://github.com/ni/nilrt.git
cd nilrt
git checkout nilrt/<release>
git submodule init
git submodule update --remote --checkout

# 2. Build pyrex container
bash ./docker/create-build-nilrt.sh

# 3. Initialize build environment
. ./ni-oe-init-build-env [--org]

# 4. Build packages
bitbake <package-name>
```

### What Can Be Built

1. **Individual Packages**
   ```bash
   bitbake python3 ruby apache2
   ```

2. **Package Feeds**
   ```bash
   bash ../scripts/pipelines/build.core-feeds.sh
   bitbake packagefeed-ni-core
   bitbake packagefeed-ni-extra
   ```

3. **System Images**
   ```bash
   bitbake nilrt-safemode-rootfs      # Safe mode recovery image
   bitbake nilrt-base-system-image    # Main runmode image
   bitbake nilrt-recovery-media       # Bootable USB/CD image (.iso)
   ```

4. **Cross-Compilation Toolchains**
   ```bash
   bash ../scripts/pipelines/build.toolchain.sh         # For Linux hosts
   bash ../scripts/pipelines/build.cross-toolchain.sh   # For Windows hosts
   ```

## Package Feeds Structure

NILRT distributes packages through structured feeds:

```
${release}/
├── ${arch}/
│   ├── main/      # Core Feed - Required packages
│   └── extra/     # Extra Feed - Community packages (unsupported)
├── ni-main/       # NI proprietary packages (LabVIEW-agnostic)
└── ni-lv${ver}/   # NI proprietary packages (LabVIEW-specific)
```

- **Core Feed** (`main/`) - Required for images, officially supported
- **Extra Feed** (`extra/`) - Community packages, no support guarantees
- **NIFeeds** - NI proprietary software (LabVIEW runtime, drivers, etc.)

## Supported Architectures

- **x64 (x86_64)** - Intel/AMD 64-bit processors (current focus)
- **ARM32 (armv7)** - ARMv7 processors (older NI hardware)
  - Note: ARM support limited to kernel 4.14 and below
  - Zynq-based platforms (CompactRIO, roboRIO, etc.)

## Key Workflows

### For NI Employees (with `--org` flag)
- Access to internal NI corporate network resources
- Can build from `nilrt/master/*` branches
- Access to `ni-org.conf` configuration snippet

### For Community Developers
- Build packages for open-source projects
- Build custom kernels and modules
- Create custom distributions
- Install packages to existing NILRT devices
- **Cannot** access NI internal feeds without published release versions

### For FIRST Robotics
- Special `nilrt-academic/master/sumo` branch
- ARM32 support for roboRIO controllers
- Used in FRC competitions

## Important Files

| File | Purpose |
|------|---------|
| `README.md` | Main documentation with setup instructions |
| `CHANGELOG.md` | Release history and changes |
| `ni-oe-init-build-env` | Environment setup script (must be sourced) |
| `pyrex.ini` | Pyrex container configuration |
| `.gitmodules` | Defines all OpenEmbedded layer submodules |
| `docs/README.kernel.md` | Detailed kernel building guide |
| `docs/package-feeds.md` | Package feed structure explanation |
| `docs/CONTRIBUTING.md` | Contribution guidelines and maintainer info |

## Build Artifacts Location

After building, artifacts appear in `build/tmp-glibc/deploy/`:

- **IPK packages**: `tmp-glibc/deploy/ipk/...`
- **Images**: `tmp-glibc/deploy/images/x64/`
  - `nilrt-safemode-rootfs-x64.tar.gz`
  - `nilrt-base-system-image-x64.tar`
  - `nilrt-recovery-media-x64.iso`
- **Toolchains**: `tmp-glibc/deploy/sdk/`
  - `oecore-x86_64-core2-64-toolchain-9.2.sh`

## Community Resources

- **Source Code**: https://github.com/ni/nilrt
- **Documentation**: https://nilrt-docs.ni.com
- **Discussion Forum**: https://forums.ni.com/t5/NI-Linux-Real-Time-Discussions/bd-p/7111
- **Bug Reports**: https://github.com/ni/nilrt/issues
- **Security Issues**: ni-psirt@emerson.com

## Project Maintainers

- Alex Stewart (alex.stewart@emerson.com)
- Chaitanya Vadrevu (chaitanya.vadrevu@emerson.com)
- NI RTOS Team (RTOS@emerson.com)

## License

The repository is licensed under the **MIT License** (see `COPYING.MIT`).

## Common Use Cases

### 1. Building a Custom Kernel
Community developers can build modified kernels with custom configurations or patches. See `docs/README.kernel.md` for detailed instructions.

### 2. Adding Open-Source Packages
Create BitBake recipes in `meta-nilrt` layer or overlay layers to add new open-source software packages to NILRT.

### 3. Creating Custom Images
Modify image recipes to create custom NILRT distributions with specific package sets.

### 4. Cross-Compiling Applications
Build the SDK toolchain to develop and cross-compile applications on a host machine for deployment to NILRT targets.

### 5. Contributing Fixes
Submit pull requests to fix bugs, add features, or update packages in the NILRT distribution.

## Build System Notes

- **Sstate Cache**: OpenEmbedded uses shared state cache to speed up rebuilds
- **Build Time**: Full builds can take several hours depending on hardware
- **Disk Space**: 100s of GB required for complete builds
- **Parallelization**: BitBake automatically parallelizes builds
- **Reproducibility**: Pyrex containers ensure consistent build environments

## Development Model

NILRT follows a **mainline-branch model**:

1. **Mainline branches** (`nilrt/master/*`) - Active development, open for contributions
2. **Product branches** (`nilrt/25.8/*`) - Stable releases, only critical fixes
3. **Academic fork** (`nilrt-academic/*`) - Special branch for educational use

## Integration with NI Ecosystem

NILRT devices integrate with:
- **NI MAX** (Measurement & Automation Explorer) - Device configuration tool
- **LabVIEW** - Graphical programming environment
- **NI Hardware** - CompactRIO, PXI, roboRIO controllers
- **NI Drivers** - Distributed via NIFeeds (proprietary)

## Summary

The **nilrt** repository is a comprehensive, professional-grade build system for creating embedded Linux distributions. It leverages industry-standard OpenEmbedded/Yocto tools, containerized build environments (Pyrex), and a modular layer architecture to support both NI's commercial products and a thriving community of developers working with NI hardware platforms. Whether you're building custom kernels, adding open-source packages, or creating complete custom distributions, this repository provides the tools and infrastructure needed for embedded Linux development on NI hardware.
