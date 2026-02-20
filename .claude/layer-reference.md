# Miryoku Layer Reference

Detailed reference for all Miryoku layers and their key positions.

## Layer Structure

All layers use a 10-column × 4-row format (40 positions):
- Row 0: Top row (10 keys)
- Row 1: Home row (10 keys)
- Row 2: Bottom row (10 keys)
- Row 3: Thumb row (6 usable + 4 unused)

```
Positions:
00 01 02 03 04 | 05 06 07 08 09   (top)
10 11 12 13 14 | 15 16 17 18 19   (home)
20 21 22 23 24 | 25 26 27 28 29   (bottom)
30 31 32 33 34 | 35 36 37 38 39   (thumb)
    └─unused─┘               └─unused─┘
```

## BASE Layer (Default: Colemak-DH)

```
Q      W      F      P      B      |  J      L      U      Y      '
A/GUI  R/ALT  S/CTL  T/SFT  G      |  M      N/SFT  E/CTL  I/ALT  O/GUI
Z/BTN  X/RALT C      D      V      |  K      H      ,      ./RALT //BTN
              ESC    SPC    TAB    |  RET    BSPC   DEL
              MED    NAV    MOU    |  SYM    NUM    FUN
```

Home row mods (hold = modifier, tap = letter):
- A = GUI, R = Alt, S = Ctrl, T = Shift (left)
- N = Shift, E = Ctrl, I = Alt, O = GUI (right)

Corner keys (Z, /) access BUTTON layer on hold.

## NAV Layer (Hold: Space/Right-inner thumb)

```
BOOT   TAP    EXTRA  BASE   -      |  REDO   PASTE  COPY   CUT    UNDO
GUI    ALT    CTRL   SHIFT  -      |  CAPS   LEFT   DOWN   UP     RIGHT
-      RALT   NUM    NAV    -      |  INS    HOME   PGDN   PGUP   END
              -      -      -      |  RET    BSPC   DEL
```

Navigation alternatives:
- **VI style** (`MIRYOKU_NAV_VI`): HJKL arrows like Vim
- **Inverted-T** (`MIRYOKU_NAV_INVERTEDT`): Arrow cluster in T shape

## NUM Layer (Hold: Backspace/Right-middle thumb)

```
[      7      8      9      ]      |  -      BASE   EXTRA  TAP    BOOT
;      4      5      6      =      |  -      SHIFT  CTRL   ALT    GUI
`      1      2      3      \      |  -      NUM    NAV    RALT   -
              .      0      -      |  -      -      -
```

## SYM Layer (Hold: Return/Left-inner thumb)

```
{      &      *      (      }      |  -      BASE   EXTRA  TAP    BOOT
:      $      %      ^      +      |  -      SHIFT  CTRL   ALT    GUI
~      !      @      #      |      |  -      SYM    MOUSE  RALT   -
              (      )      _      |  -      -      -
```

## FUN Layer (Hold: Delete/Left-outer thumb)

```
F12    F7     F8     F9     PSCRN  |  -      BASE   EXTRA  TAP    BOOT
F11    F4     F5     F6     SLCK   |  -      SHIFT  CTRL   ALT    GUI
F10    F1     F2     F3     PAUSE  |  -      FUN    MEDIA  RALT   -
              APP    SPC    TAB    |  -      -      -
```

## MEDIA Layer (Hold: Escape/Left-middle thumb)

```
BOOT   TAP    EXTRA  BASE   -      |  RGBTOG RGBEFF RGBHUI RGBSAI RGBBRI
GUI    ALT    CTRL   SHIFT  -      |  EPTOG  PREV   VOLDN  VOLUP  NEXT
-      RALT   FUN    MEDIA  -      |  OUTTOG BT0    BT1    BT2    BT3
              -      -      -      |  STOP   PP     MUTE
```

Bluetooth profiles: BT0-BT3 for pairing 4 devices

## MOUSE Layer (Hold: Tab/Right-outer thumb)

```
BOOT   TAP    EXTRA  BASE   -      |  REDO   PASTE  COPY   CUT    UNDO
GUI    ALT    CTRL   SHIFT  -      |  -      MS_L   MS_D   MS_U   MS_R
-      RALT   SYM    MOUSE  -      |  -      WH_L   WH_D   WH_U   WH_R
              -      -      -      |  BTN2   BTN1   BTN3
```

Mouse keys require `CONFIG_ZMK_POINTING=y` in Kconfig.

## BUTTON Layer (Hold: Z or / keys)

```
UNDO   CUT    COPY   PASTE  REDO   |  REDO   PASTE  COPY   CUT    UNDO
GUI    ALT    CTRL   SHIFT  -      |  -      SHIFT  CTRL   ALT    GUI
UNDO   CUT    COPY   PASTE  REDO   |  REDO   PASTE  COPY   CUT    UNDO
              BTN3   BTN1   BTN2   |  BTN2   BTN1   BTN3
```

Mirrored for use with either hand.

## EXTRA Layer (Default: QWERTY)

Same structure as BASE but with alternate alpha layout.
Access via: NAV/MOUSE/MEDIA layer → top row EXTRA key.

## TAP Layer

Plain alphas without home row mods - useful for gaming or rapid typing.
Access via: NAV/MOUSE/MEDIA layer → top row TAP key.

## Key Symbols Reference

| Symbol | Meaning |
|--------|---------|
| GUI | Super/Win/Cmd |
| ALT | Alt/Option |
| CTL | Control |
| SFT | Shift |
| RALT | Right Alt (AltGr) |
| BTN | Button layer |
| - | Not available |
| MS_* | Mouse movement |
| WH_* | Mouse wheel |
| BTN* | Mouse buttons |
| BT* | Bluetooth profile |
| BOOT | Bootloader |
