# Local Build Guide for Miryoku ZMK

Complete instructions for building Miryoku ZMK firmware locally on macOS.

## Prerequisites

### Option 1: Docker (Recommended - Matches CI exactly)

```bash
# Install Docker Desktop for Mac
brew install --cask docker

# Pull ZMK build image
docker pull zmkfirmware/zmk-build-arm:stable
```

### Option 2: Native Toolchain

```bash
# Install dependencies via Homebrew
brew install cmake ninja gperf python3 ccache qemu dtc wget libmagic

# Install west (Zephyr meta-tool)
pip3 install west

# Install Zephyr SDK
cd ~
wget https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.16.5/zephyr-sdk-0.16.5_macos-x86_64.tar.xz
tar xf zephyr-sdk-0.16.5_macos-x86_64.tar.xz
cd zephyr-sdk-0.16.5
./setup.sh
```

## Directory Setup

```bash
# Create workspace
mkdir -p ~/zmk-workspace && cd ~/zmk-workspace

# Clone miryoku_zmk (this repo)
git clone https://github.com/manna-harbour/miryoku_zmk

# Clone ZMK
git clone -b main --depth 1 https://github.com/zmkfirmware/zmk

# Initialize ZMK
cd zmk
west init -l app
west update
west zephyr-export
```

## Building Firmware

### Basic Build (Corne + Nice!Nano v2)

```bash
cd ~/zmk-workspace/zmk/app

# Build left half
west build -b nice_nano_v2 -d build/left -- \
  -DSHIELD="corne_left" \
  -DZMK_CONFIG="../../miryoku_zmk/config"

# Build right half
west build -b nice_nano_v2 -d build/right -- \
  -DSHIELD="corne_right" \
  -DZMK_CONFIG="../../miryoku_zmk/config"
```

Output files:
- `build/left/zephyr/zmk.uf2`
- `build/right/zephyr/zmk.uf2`

### Build with Nice!View Display

```bash
# Left with display
west build -b nice_nano_v2 -d build/left -- \
  -DSHIELD="corne_left nice_view_adapter nice_view" \
  -DZMK_CONFIG="../../miryoku_zmk/config"

# Right with display
west build -b nice_nano_v2 -d build/right -- \
  -DSHIELD="corne_right nice_view_adapter nice_view" \
  -DZMK_CONFIG="../../miryoku_zmk/config"
```

### Build with Custom Options

Edit `miryoku_zmk/miryoku/custom_config.h` before building:

```c
// Example custom_config.h for QWERTY + Mac clipboard
#define MIRYOKU_ALPHAS_QWERTY
#define MIRYOKU_CLIPBOARD_MAC
#define MIRYOKU_NAV_VI
```

Then build normally - options are picked up automatically.

### Rebuild After Changes

```bash
# Clean and rebuild (recommended after config changes)
west build -b nice_nano_v2 -d build/left --pristine -- \
  -DSHIELD="corne_left" \
  -DZMK_CONFIG="../../miryoku_zmk/config"

# Or just rebuild (faster, for code-only changes)
west build -d build/left
```

## Using Docker

For exact CI environment match:

```bash
cd ~/zmk-workspace

# Start container with workspace mounted
docker run -it --rm \
  -v $(pwd):/workspace \
  -w /workspace \
  zmkfirmware/zmk-build-arm:stable \
  /bin/bash

# Inside container:
cd zmk
west init -l app
west update
west zephyr-export

cd app
west build -b nice_nano_v2 -- \
  -DSHIELD="corne_left" \
  -DZMK_CONFIG="../../miryoku_zmk/config"

# Output will be at zmk/app/build/zephyr/zmk.uf2
# Exit container - file persists in your local filesystem
exit
```

## Flashing Firmware

1. Put keyboard in bootloader mode:
   - Double-tap reset button, OR
   - Hold boot button while connecting USB

2. A drive named `NICENANO` (or similar) appears

3. Copy the `.uf2` file to the drive:
   ```bash
   cp build/left/zephyr/zmk.uf2 /Volumes/NICENANO/
   ```

4. Drive auto-unmounts, keyboard reboots

5. Repeat for other half

## Build Script

Save as `build.sh` in your workspace:

```bash
#!/bin/bash
set -e

WORKSPACE=~/zmk-workspace
MIRYOKU=$WORKSPACE/miryoku_zmk
ZMK=$WORKSPACE/zmk

cd $ZMK/app

# Build both halves
for side in left right; do
  echo "Building $side..."
  west build -b nice_nano_v2 -d build/$side --pristine -- \
    -DSHIELD="corne_$side" \
    -DZMK_CONFIG="$MIRYOKU/config"
  echo "Output: build/$side/zephyr/zmk.uf2"
done

echo "Done! Flash files are in $ZMK/app/build/*/zephyr/zmk.uf2"
```

## Troubleshooting

### "west: command not found"
```bash
pip3 install west
# Or add to PATH: export PATH="$HOME/Library/Python/3.x/bin:$PATH"
```

### "CMake Error: The source directory does not exist"
```bash
# Ensure you're in zmk/app directory
cd ~/zmk-workspace/zmk/app
```

### "devicetree error" or "DTS" errors
```bash
# Clean build directory
rm -rf build/left
west build ...  # rebuild
```

### "Invalid BOARD"
```bash
# List available boards
ls ~/zmk-workspace/zmk/app/boards/arm/
# Common: nice_nano, nice_nano_v2, seeeduino_xiao_ble
```

### Outdated ZMK
```bash
cd ~/zmk-workspace/zmk
git pull
west update
```

## Other Controllers

| Controller | Board Name |
|------------|------------|
| Nice!Nano v1 | `nice_nano` |
| Nice!Nano v2 | `nice_nano_v2` |
| Seeed XIAO BLE | `seeeduino_xiao_ble` |
| nRFMicro | `nrfmicro_13` |
| BlueMicro840 | `bluemicro840_v1` |

## Other Keyboards

Check `miryoku_zmk/config/` for available keymaps:
```bash
ls ~/zmk-workspace/miryoku_zmk/config/*.keymap
```

Common shields:
- `corne_left` / `corne_right`
- `sweep_left` / `sweep_right` (Ferris Sweep)
- `lily58_left` / `lily58_right`
- `sofle_left` / `sofle_right`
- `kyria_left` / `kyria_right`
