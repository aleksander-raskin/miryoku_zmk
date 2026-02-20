# Miryoku Customization Examples

Real-world customization scenarios for `miryoku/custom_config.h`.

## Example 1: Mac User with QWERTY

```c
// custom_config.h
#define MIRYOKU_ALPHAS_QWERTY
#define MIRYOKU_CLIPBOARD_MAC
```

Result:
- QWERTY alpha layout on BASE layer
- Cmd+Z/X/C/V for clipboard operations
- Default navigation (non-Vim)

## Example 2: Vim User with Colemak-DH

```c
// custom_config.h
#define MIRYOKU_ALPHAS_COLEMAKDH  // This is default, can omit
#define MIRYOKU_NAV_VI
#define MIRYOKU_CLIPBOARD_WIN     // For Linux with Ctrl shortcuts
```

Result:
- Colemak-DH alphas (default)
- Vim-style HJKL navigation on NAV layer
- Ctrl+Z/X/C/V clipboard

## Example 3: Gaming Setup (Tap Layer for Games)

```c
// custom_config.h
#define MIRYOKU_ALPHAS_COLEMAKDH
#define MIRYOKU_TAP_QWERTY        // TAP layer uses QWERTY for games
#define MIRYOKU_CLIPBOARD_WIN
```

Use: Switch to TAP layer (from NAV/MOUSE/MEDIA) for gaming without home row mods.

## Example 4: Flipped Layers (Left-Hand Navigation)

```c
// custom_config.h
#define MIRYOKU_LAYERS_FLIP
#define MIRYOKU_ALPHAS_QWERTY
```

Result - Thumb keys swap:
```
Before: ESC(Med) SPC(Nav) TAB(Mou) | RET(Sym) BSPC(Num) DEL(Fun)
After:  DEL(Fun) BSPC(Num) RET(Sym) | TAB(Mou) SPC(Nav) ESC(Med)
```

## Example 5: Extra Layer as Dvorak

```c
// custom_config.h
#define MIRYOKU_ALPHAS_QWERTY     // Main layer QWERTY
#define MIRYOKU_EXTRA_DVORAK      // Extra layer Dvorak
```

Use: Quick switch between QWERTY and Dvorak via EXTRA key on NAV layer.

## Example 6: 34-Key Board (Thumb Combos)

For boards like Ferris Sweep with only 2 thumb keys per side:

```c
// custom_config.h
#define MIRYOKU_KLUDGE_THUMBCOMBOS
#define MIRYOKU_ALPHAS_COLEMAKDH
```

Result: Two adjacent thumb keys pressed together = third thumb key function.

## Example 7: Inverted-T Navigation

```c
// custom_config.h
#define MIRYOKU_NAV_INVERTEDT
#define MIRYOKU_CLIPBOARD_MAC
```

Result: Traditional inverted-T arrow layout instead of home row arrows.

## Advanced: Custom Layer Definition

To completely override a layer, define it in `custom_config.h`:

```c
// custom_config.h

// Override NUM layer with custom layout
#define MIRYOKU_LAYER_NUM \
&kp LBKT,          &kp N7,            &kp N8,            &kp N9,            &kp RBKT,          U_NA,              &u_to_U_BASE,      &u_to_U_EXTRA,     &u_to_U_TAP,       U_BOOT,            \
&kp SEMI,          &kp N4,            &kp N5,            &kp N6,            &kp EQUAL,         U_NA,              &kp LSHFT,         &kp LCTRL,         &kp LALT,          &kp LGUI,          \
&kp GRAVE,         &kp N1,            &kp N2,            &kp N3,            &kp BSLH,          U_NA,              &u_to_U_NUM,       &u_to_U_NAV,       &kp RALT,          U_NA,              \
U_NP,              U_NP,              &kp COMMA,         &kp N0,            &kp DOT,           U_NA,              U_NA,              U_NA,              U_NP,              U_NP
```

## Kconfig Options

Create `config/corne.conf` (or your keyboard) for ZMK settings:

```ini
# Enable mouse keys
CONFIG_ZMK_POINTING=y

# Increase Bluetooth TX power
CONFIG_BT_CTLR_TX_PWR_PLUS_8=y

# Deep sleep timeout (ms) - default 900000 (15 min)
CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=1800000

# Combo timeout
CONFIG_ZMK_COMBO_MAX_COMBOS_PER_KEY=8

# Enable soft off (for boards that support it)
CONFIG_ZMK_PM_SOFT_OFF=y
```

## Behavior Timing Adjustments

In `miryoku/miryoku.h`, you can adjust:

```c
#define U_TAPPING_TERM 200  // Default: 200ms
```

For faster home row mods, try 175ms. For fewer accidental triggers, try 225ms.

Or use ZMK's global setting in `.conf`:
```ini
CONFIG_ZMK_HOLD_TAP_TAPPING_TERM=200
```

## Combo Timing

For thumb combos kludge, edit `miryoku/miryoku_kludge_thumbcombos.dtsi`:
```c
#define MIRYOKU_KLUDGE_THUMBCOMBOS_TERM 200  // Adjust combo window
```

## Multiple Configurations

For different machines, create separate branches or use CI workflow inputs:

```yaml
# GitHub Actions - Build Inputs workflow
board: nice_nano_v2
shield: corne_left
alphas: QWERTY
clipboard: MAC
nav: vi
```

This generates config dynamically without editing files.
