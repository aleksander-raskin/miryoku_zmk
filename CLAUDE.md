# Miryoku ZMK

ZMK firmware implementation of the Miryoku ergonomic keyboard layout system by Manna Harbour.

## My Chocofi Configuration

Active branch: `chocofi`

| Setting | Value |
|---------|-------|
| Board | nice_nano_v2 |
| Shield | corne_left/right + nice_view_adapter + nice_view |
| Alphas | QWERTY |
| Nav | vi (Vim-style HJKL) |
| Clipboard | Mac (Cmd+Z/X/C/V) |
| Display | Enabled |
| Sleep | 15 min timeout |

### custom_config.h

```c
#define MIRYOKU_ALPHAS_QWERTY
#define MIRYOKU_NAV_VI
#define MIRYOKU_CLIPBOARD_MAC
```

### Mouse Settings (in config/corne.keymap)

```c
&mmv {
    time-to-max-speed-ms = <400>;    // Smooth ramp to max speed
    acceleration-exponent = <1>;      // Smooth acceleration curve
};

&msc {
    time-to-max-speed-ms = <300>;    // Scroll acceleration time
    acceleration-exponent = <0>;      // Constant scroll speed
};
```

### Home Row Mods (Custom Order)

Standard Miryoku: `GUI - ALT - CTRL - SHIFT` (pinky → index)
**My chocofi:**    `GUI - ALT - SHIFT - CTRL` (pinky → index)

```
Left hand:   A=GUI  S=ALT  D=SHIFT  F=CTRL
Right hand:  J=CTRL K=SHIFT L=ALT  '=GUI
```

### NUM Layer (Custom)

Right side has navigation instead of modifiers - useful for editing while entering numbers:

```
[    7    8    9    ]    |  REDO  PASTE  COPY   CUT   UNDO
;    4    5    6    =    |  LEFT  DOWN   UP     RIGHT CAPS
`    1    2    3    \    |  HOME  PGDN   PGUP   END   INS
          .    0    -    |  RET   BSPC   DEL
```

### Building Chocofi Firmware

```bash
# Activate ZMK virtual environment
source ~/.virtualenvs/zmk/bin/activate

# Build left half with Nice!View
cd ~/Projects/chocofi/zmk/app
west build -b nice_nano_v2 -d build/left --pristine -- \
  -DSHIELD="corne_left nice_view_adapter nice_view" \
  -DZMK_CONFIG="$HOME/Projects/miryoku_zmk/config" \
  -DCONFIG_ZMK_STUDIO=n \
  -DCONFIG_ZMK_WIDGET_WPM_STATUS=n \
  -DCONFIG_ZMK_SLEEP=y \
  -DCONFIG_ZMK_IDLE_SLEEP_TIMEOUT=900000 \
  -DCONFIG_ZMK_POINTING=y

# Build right half with Nice!View
west build -b nice_nano_v2 -d build/right --pristine -- \
  -DSHIELD="corne_right nice_view_adapter nice_view" \
  -DZMK_CONFIG="$HOME/Projects/miryoku_zmk/config" \
  -DCONFIG_ZMK_STUDIO=n \
  -DCONFIG_ZMK_WIDGET_WPM_STATUS=n \
  -DCONFIG_ZMK_SLEEP=y \
  -DCONFIG_ZMK_IDLE_SLEEP_TIMEOUT=900000 \
  -DCONFIG_ZMK_POINTING=y
```

**Output files:**
- `~/Projects/chocofi/zmk/app/build/left/zephyr/zmk.uf2`
- `~/Projects/chocofi/zmk/app/build/right/zephyr/zmk.uf2`

**Build flags:**
- `CONFIG_ZMK_STUDIO=n` - Disable ZMK Studio (requires extra device config)
- `CONFIG_ZMK_WIDGET_WPM_STATUS=n` - Disable WPM widget (conflicts with Nice!View)
- `CONFIG_ZMK_SLEEP=y` - Enable deep sleep (saves battery)
- `CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=900000` - 15 min sleep timeout
- `CONFIG_ZMK_POINTING=y` - Enable mouse keys

### Quick-Tap Layer-Tap (Thumb Keys)

Tap-then-hold within 200ms to repeat the key instead of activating layer:

| Key | Tap | Hold | Tap+Hold |
|-----|-----|------|----------|
| 31 SPACE | Space | Nav layer | Repeat Space |
| 32 TAB | Tab | Mouse layer | Repeat Tab |
| 33 RET | Enter | Sym layer | Repeat Enter |
| 34 BSPC | Backspace | Num layer | Repeat Backspace |

Defined in `config/corne.keymap`:

```c
qt_lt: quick_tap_layer_tap {
    compatible = "zmk,behavior-hold-tap";
    label = "QT_LT";
    #binding-cells = <2>;
    flavor = "balanced";
    tapping-term-ms = <200>;
    quick-tap-ms = <200>;  // Tap then hold within 200ms = repeat key
    bindings = <&mo>, <&kp>;
};
```

### App & Tab Switching (Mouse Layer)

Hold TAB (position 32) to enter Mouse layer, then use top row for switching:

```
┌──────┬──────┬──────┬──────┬──────┐
│ BOOT │CLOSE │TAB ← │TAB → │ SWAP │
│  0   │  1   │  2   │  3   │  4   │
└──────┴──────┴──────┴──────┴──────┘
```

| Key | Position | Action |
|-----|----------|--------|
| W | 1 | Close tab (Cmd+W) |
| E | 2 | Previous browser tab (Ctrl+Shift+Tab) |
| R | 3 | Next browser tab (Ctrl+Tab) - tri-state, keep pressing |
| T | 4 | App switcher (Cmd+Tab) - tri-state, +Shift to go back |

Defined in `config/corne.keymap`:

```c
/ {
    behaviors {
        swapper: swapper {
            compatible = "zmk,behavior-tri-state";
            label = "SWAPPER";
            #binding-cells = <0>;
            bindings = <&kt LGUI>, <&kp TAB>, <&kt LGUI>;
            ignored-key-positions = <4 12 17>;  // 4=T to cycle, 12/17=Shift for reverse
        };
        tabber: tabber {
            compatible = "zmk,behavior-tri-state";
            label = "TABBER";
            #binding-cells = <0>;
            bindings = <&kt LCTRL>, <&kp TAB>, <&kt LCTRL>;
            ignored-key-positions = <2 3>;  // Allow switching between prev/next
        };
    };

    macros {
        close_tab: close_tab {
            compatible = "zmk,behavior-macro";
            #binding-cells = <0>;
            bindings = <&kp LG(W)>;
        };
        tab_prev: tab_prev {
            compatible = "zmk,behavior-macro";
            #binding-cells = <0>;
            bindings = <&kp LC(LS(TAB))>;
        };
    };
};
```

**Module dependency:** zmk-tri-state is included in `~/Projects/chocofi/zmk/app/west.yml`:
```yaml
- name: zmk-tri-state
  remote: dhruvinsh
  revision: main
```

### ZMK Source Modifications

**Bluetooth icon** (instead of WiFi) - changed `LV_SYMBOL_WIFI` → `LV_SYMBOL_BLUETOOTH` in:
- `~/Projects/chocofi/zmk/app/src/display/widgets/output_status.c`
- `~/Projects/chocofi/zmk/app/boards/shields/nice_view/widgets/status.c` (left half)
- `~/Projects/chocofi/zmk/app/boards/shields/nice_view/widgets/peripheral_status.c` (right half)

## What is Miryoku?

Miryoku is a minimalist, universal keyboard layout designed for compact keyboards (36+ keys). Key design principles:
- **Home row mods**: Modifiers (GUI, Alt, Ctrl, Shift) on home row via hold-tap behavior
- **Layer-tap thumbs**: 6 thumb keys access 6 layers via hold, common keys via tap
- **Colemak-DH default**: Optimized alpha layout (QWERTY and others available)
- **Consistent structure**: Same layout works across all supported keyboards

## Repository Structure

```
miryoku_zmk/
├── config/                    # Keymap files per keyboard
│   ├── corne.keymap          # Corne-specific (includes miryoku)
│   ├── *.keymap              # Other keyboards
│   └── *.conf                # Kconfig overrides (rare)
├── miryoku/                   # Core Miryoku implementation
│   ├── custom_config.h       # YOUR CUSTOMIZATIONS GO HERE
│   ├── miryoku.dtsi          # Main keymap devicetree
│   ├── miryoku.h             # Core includes and macros
│   ├── miryoku_babel/        # Layer definitions
│   │   ├── miryoku_layer_alternatives.h  # All layer layouts
│   │   ├── miryoku_layer_list.h          # Layer names/indices
│   │   └── miryoku_layer_selection.h     # Option → layer mapping
│   ├── mapping/              # Keyboard-specific key mappings
│   │   └── 42/corne.h        # Corne: 42 keys → 36 Miryoku
│   ├── miryoku_behaviors.dtsi   # Hold-tap behaviors
│   ├── miryoku_clipboard.h      # OS-specific clipboard
│   └── miryoku_kludge_*.dtsi    # Optional combos
└── .github/workflows/        # CI build system
    ├── main.yml              # Core build logic
    ├── build-inputs.yml      # Manual trigger with options
    └── outboards/            # External board definitions
```

## Layers (10 total)

| Index | Layer  | Access               | Purpose                          |
|-------|--------|----------------------|----------------------------------|
| 0     | BASE   | Default              | Alpha keys + home row mods       |
| 1     | EXTRA  | From NAV/MOUSE/MEDIA | Alternate alpha (default QWERTY) |
| 2     | TAP    | From NAV/MOUSE/MEDIA | Plain alphas, no mods            |
| 3     | BUTTON | Pinky corners        | Mouse buttons + clipboard        |
| 4     | NAV    | Right thumb hold     | Navigation, arrows, clipboard    |
| 5     | MOUSE  | Right thumb hold     | Mouse movement + scroll          |
| 6     | MEDIA  | Left thumb hold      | Media, RGB, Bluetooth            |
| 7     | NUM    | Right thumb hold     | Number pad + symbols             |
| 8     | SYM    | Left thumb hold      | Symbols (shifted numbers)        |
| 9     | FUN    | Left thumb hold      | Function keys F1-F12             |

### Default Thumb Layout (non-flipped)
```
Left hand:   ESC(Media)  SPACE(Nav)  TAB(Mouse)
Right hand:  RET(Sym)    BSPC(Num)   DEL(Fun)
```

## Customization via custom_config.h

Edit `miryoku/custom_config.h` to customize. Add `#define` statements:

### Alpha Layouts
```c
#define MIRYOKU_ALPHAS_QWERTY      // QWERTY
#define MIRYOKU_ALPHAS_COLEMAK     // Colemak
#define MIRYOKU_ALPHAS_COLEMAKDH   // Colemak-DH (default)
#define MIRYOKU_ALPHAS_COLEMAKDHK  // Colemak-DHk
#define MIRYOKU_ALPHAS_DVORAK      // Dvorak
#define MIRYOKU_ALPHAS_WORKMAN     // Workman
#define MIRYOKU_ALPHAS_HALMAK      // Halmak
#define MIRYOKU_ALPHAS_BEAKL15     // BEAKL15
#define MIRYOKU_ALPHAS_AZERTY      // AZERTY
#define MIRYOKU_ALPHAS_QWERTZ      // QWERTZ
```

### Navigation Style
```c
// Default: Home/PgDn/PgUp/End on bottom row, arrows on home row
#define MIRYOKU_NAV_VI         // Vim-style: HJKL arrows
#define MIRYOKU_NAV_INVERTEDT  // Inverted-T arrow cluster
```

### Clipboard (OS-specific)
```c
#define MIRYOKU_CLIPBOARD_MAC  // Cmd+Z/X/C/V
#define MIRYOKU_CLIPBOARD_WIN  // Ctrl+Z/X/C/V (with Ctrl+Y redo)
#define MIRYOKU_CLIPBOARD_FUN  // K_UNDO/K_CUT/K_COPY/K_PASTE
// Default: Shift+Del, Ctrl+Ins, Shift+Ins (CUA)
```

### Layer Flip
```c
#define MIRYOKU_LAYERS_FLIP  // Swap thumb functions between hands
```
With flip, thumb layout becomes:
```
Left hand:   DEL(Fun)    BSPC(Num)   RET(Sym)
Right hand:  TAB(Mouse)  SPACE(Nav)  ESC(Media)
```

### Extra/Tap Layer Alphas
```c
#define MIRYOKU_EXTRA_QWERTY  // Set EXTRA layer to QWERTY
#define MIRYOKU_TAP_QWERTY    // Set TAP layer to QWERTY
```

### Kludges (Optional Combos)
```c
#define MIRYOKU_KLUDGE_THUMBCOMBOS     // Two-thumb combos for 34-key boards
#define MIRYOKU_KLUDGE_TOPROWCOMBOS    // Top row combos
#define MIRYOKU_KLUDGE_BOTTOMROWCOMBOS // Bottom row combos
#define MIRYOKU_KLUDGE_TAPDELAY        // Delayed tap behavior
```

## Building Firmware

### Via GitHub Actions (Recommended)

1. Fork the repository
2. Go to Actions → "Build Inputs"
3. Click "Run workflow"
4. Fill in:
   - **board**: `nice_nano_v2` (or your controller)
   - **shield**: `corne_left` or `corne_right` (or your keyboard)
   - **alphas**: Select layout (QWERTY, Colemak, etc.)
   - Other options as needed
5. Download artifact (`.uf2` file)

### Local Build

Prerequisites:
- Docker (recommended) OR
- ZMK toolchain (west, Zephyr SDK)

#### Using Docker (mirrors CI)
```bash
# Clone repos
git clone https://github.com/manna-harbour/miryoku_zmk
git clone -b main --depth 1 https://github.com/zmkfirmware/zmk

# Setup ZMK
cd zmk
west init -l app
west update
west zephyr-export

# Build for Corne left half
cd app
west build -b nice_nano_v2 -- \
  -DSHIELD="corne_left" \
  -DZMK_CONFIG="../../miryoku_zmk/config"

# Output: build/zephyr/zmk.uf2
```

#### With custom options
```bash
# Edit miryoku_zmk/miryoku/custom_config.h first, then:
west build -b nice_nano_v2 -- \
  -DSHIELD="corne_left" \
  -DZMK_CONFIG="../../miryoku_zmk/config"
```

#### Using Docker container (exact CI match)
```bash
docker run -it --rm \
  -v $(pwd):/workspace \
  zmkfirmware/zmk-build-arm:stable \
  /bin/bash

# Then run west commands inside container
```

## Chocofi/Corne Notes

- **42 physical keys** → 36 Miryoku keys (outer columns unused)
- Mapping in `miryoku/mapping/42/corne.h`
- Outer columns set to `&none`
- Uses Nice!Nano v2 + Nice!View display
- For split: build both `corne_left` and `corne_right`

### Flashing
1. Double-tap reset button (or hold boot while connecting USB)
2. `NICENANO` drive appears
3. Copy `.uf2` file:
   ```bash
   cp ~/Projects/chocofi/zmk/app/build/left/zephyr/zmk.uf2 /Volumes/NICENANO/
   ```
4. Repeat for right half

## Troubleshooting

### Build Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `Invalid BOARD` | Wrong board name | Check ZMK supported boards |
| `Failed to locate keymap file` | Unsupported keyboard | Check `config/` for `.keymap` |
| `Invalid SHIELD` | Using shield as board | Board = controller, Shield = keyboard |

### Common Issues

**Home row mods feel wrong:**
- Default tapping term is 200ms (`U_TAPPING_TERM`)
- Adjust in `miryoku/miryoku.h` or via Kconfig

**Bluetooth not connecting:**
- Clear bonds: hold outer bottom corners on boot
- Or use `&bt BT_CLR` from Media layer

**Wrong clipboard shortcuts:**
- Set `MIRYOKU_CLIPBOARD_MAC` or `MIRYOKU_CLIPBOARD_WIN`

## File Quick Reference

| Task | File |
|------|------|
| **My layer customizations** | `miryoku/miryoku_babel/miryoku_layer_alternatives.h` (top of file) |
| **My build defaults** | `.github/workflows/build-inputs.yml` |
| Change alphas/options | `miryoku/custom_config.h` |
| Keyboard mapping | `miryoku/mapping/42/corne.h` |
| Hold-tap timing | `miryoku/miryoku.h` (U_TAPPING_TERM) |
| Behaviors | `miryoku/miryoku_behaviors.dtsi` |
| Add Kconfig | `config/corne.conf` (create if needed) |

### Key Customization Locations (chocofi branch)

My home row mod order (GACS → GA**SC**) and custom NUM layer are defined at the **top** of `miryoku/miryoku_babel/miryoku_layer_alternatives.h`:

```c
// Used layers for chocofi
#define MIRYOKU_ALTERNATIVES_BASE_QWERTY \
...
#define MIRYOKU_ALTERNATIVES_NAV_VI \
...
#define MIRYOKU_ALTERNATIVES_NUM \  // Has nav keys on right side
...
// End of used layers for chocofi
```

## Links

- [Miryoku Reference](https://github.com/manna-harbour/miryoku/tree/master/docs/reference)
- [ZMK Documentation](https://zmk.dev/docs)
- [Miryoku ZMK Repo](https://github.com/manna-harbour/miryoku_zmk)
