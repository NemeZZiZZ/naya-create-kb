# Host-side maps (static initializers)

`Binding::param1/param2()` resolve action names through runtime maps
filled by static initializers (statics live at page `0x100af0000`;
map u64 = `(param2<<32)|param1`; Vs wire records emit
`[T,08,HIGH32,LOW32]` = `[T,08,p2,p1]`). All tables below are
COMPLETE extractions unless noted.

## Modifier map (+0x98, 27 names)

Byte values = standard HID boot-modifier bits. HID names split on
` + `; extra parts OR their byte into `param1<<24` (= MODMASK,
e.g. `Shift + A` → `0x02070004`).

`0x01`: LCTRL/CTRL/LEFT_CTRL · `0x02`: LSHIFT/SHIFT/LEFT_SHIFT ·
`0x04`: LALT/ALT/LEFT_ALT · `0x08`: LGUI/GUI/META/CMD/LEFT_GUI/
LEFT_META/LMETA/LCMD/LWIN/LEFT_WIN/LEFT_COMMAND (11 aliases) ·
`0x10`: RCTRL/RIGHT_CTRL · `0x20`: RSHIFT/RIGHT_SHIFT ·
`0x40`: RALT (only) · `0x80`: RGUI (only — no RIGHT_ALT/RIGHT_GUI aliases).

## BT map (+0x78, 8 names)

CLEAR→0, NEXT→`0x100000000`, PREV→`0x200000000`,
SELECT_SL→`0x300000000`, DEVICE_n→`0x30000000+n` (family 3 = BT;
DEVICE_1 → wire X=3,Y=1).

## LED map (+0x90, 19 names)

EFFECT_ON_OFF, BREATHE, SOLID, SWIRL, SPEC, EFFECT,
BRIGHTNESS_UP/DOWN, SPEED_UP/DOWN, COLOR_RED/GREEN/BLUE/WHITE/CYAN/
MAGENTA/YELLOW/ORANGE/PINK.

Non-colors: SOLID=`0xd00000000` (p2=13,p1=0 → wire X=13,Y=0 ✓);
BRI_UP/DOWN=(p2=7/8), SPD_UP/DOWN=(p2=9/10), LED_EFFECT=(p2=11).
Colors (all p2=15, packing Y = S|B<<8|H<<16): RED=H0, ORANGE=H30,
YELLOW=H60, GREEN=H120, CYAN=H180, BLUE=H240, MAGENTA=H270, PINK=H300
(all S=100/B=70); WHITE is anomalous raw p1=`0x64` (not packed),
recorded as-is. Statics build x27 as ORANGE then derive every other
color by H-field arithmetic — every delta lands on the canonical hue.

## Mouse map (+0xa8, 15 names — FULLY CLOSED)

`(X,Y)=(fn,signed-delta)`: LEFT=(0,−1), RIGHT=(0,1), UP=(1,−1),
DOWN=(1,1), SCROLL_UP=(4,1), SCROLL_DOWN=(4,−1), SCROLL_LEFT=(6,−1),
SCROLL_RIGHT=(6,1), ZOOM_IN=(8,1), ZOOM_OUT=(8,−1),
M1=(3,1)=LEFT-button, M2=(3,2)=RIGHT-button, M3=(3,4)=MIDDLE,
M4=(3,8), M5=(3,16). fn: 0=H-move, 1=V-move, 3=buttons
(bitmask 1/2/4/8/16), 4=wheel, 6=pan, 8=zoom.
e.g. M1=(3,1) → Vs `0f 08 03 01` byte-exact on the wire.

Second mouse map (+0xb0, 13 keys): same names MINUS ZOOM_IN/ZOOM_OUT —
hypothesis: describe/readback-side index.

## Key map (+0x80, 282 entries)

Plain = `0x0007HHHH` (keyboard page; A=`0x04`, F1–F12=`0x3a–0x45`,
F13–F24=`0x68–0x73`); shifted = `0x02HHHHHH` (MODMASK accumulated via
` + ` names); consumer = `0x000cHHHH` (C_MUTE=`0xe2`,
C_PREVIOUS=`0xb6`); Generic-Desktop = `0x000100HHHH`
(SYSTEM_WAKE_UP=`0x83`, PWR/SLEEP/WAKE). Quirks: DELETE=`0x4c`
(+DEL alias); CLEAR=`0x34` = quote (action-name quirk);
KP_CLEAR=`0xD8` (both sites); INTn/LANGn alias families;
K_LOCK=K_SCREENSAVER=K_COFFEE=`0xF9`; PIPE2=Shift+Non-US-backslash
(ISO pipe ✓); CLEAR2=Shift+NumLock.

Out-map (+0x88, 2 entries): USB_DEVICE→1, BT_OUT→2 (matches T08).

## t19-map (+0xa0, 8 naya-type actions)

TUNE_MODE_L=150, TUNE_MODE_R=151, WINDOWS_OS=200, MAC_OS=201,
SCROLL_DIRECTION_L=300, SCROLL_DIRECTION_R=301,
MODULE_CHARGING=400, MODULE_FORCE_CHARGING=401. Type-19 emits
`[T,04,p1-u32LE]` (T=6 in the `0x3962` bitmask group), e.g.
TUNE_MODE_L → `[KK,06,04,96,00,00,00]`. No genuine wire sample:
the palette MAC_OS resolves host-side to plain HID LGUI.

## Module-gesture map (9 behavior slots)

MOUSE_HORIZONTAL=0, MOUSE_VERTICAL=1, MOUSE_STATIC=2, MOUSE_BUTTONS=3,
MOUSE_SCROLL_VERTICAL=4, STATIC_SCROLL_VERTICAL=5,
MOUSE_SCROLL_HORIZONTAL=6, STATIC_SCROLL_HORIZONTAL=7, STATIC_ZOOM=8.
(Different object than the `0x100af0000` struct.) Vocabulary for the
`30/100b` module-config path.

## +0x70 map — write-only verdict

13 (u32 key, byte val) pairs; 11/13 mirror the static T-table
byte-exact (anomalies 5→2, 9→6). Full `adrp` census: all 14 users sit
inside the fill cluster; every other `add/ldr #0x70` site is
batch-element arithmetic, vtable dispatch, or object-graph walks. NO
reader found via any pattern → write-only from the static viewpoint
(dead type→T override table or debug leftover). `serializeBindingData`
never touches it.

## Renderer action vocabulary (icon registry, 854 names)

Icon names ≠ wire actions (only MO(1)/hold-layer-2 proven on wire).
Highlights: C_ consumer set (12), MB1–MB12, KP_ (20), layer IDs 0..35
(zero-padded icon aliases `00..09`; real IDs 0..35), MO_LAYER_0..35 /
TO_LAYER / TOGGLE_LAYER / HOLD_LAYER families. Gaps: BT_DEVICE_5 icon
with no map entry; LED_GEN/GEN_2 + LED_BRIGHTNESS icons beyond the
19-entry map; richer modifier variants (JIS/MAC, LOPT/ROPT,
LSHFT/RSHFT, RGUI) than the map aliases.
