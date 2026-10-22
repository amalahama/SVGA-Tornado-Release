# DI Tornado VGA Palette Architecture & Custom Cockpit (.BT2) Guide

A comprehensive technical reference for digital artists, modders, and technical designers creating custom 800x600 SVGA cockpit panels for **Digital Integration's Tornado & Operation Desert Storm**.

---

## 1. Executive Summary: The "Unaffected Greys" Mystery Explained

If you are painting a custom cockpit in an image editor (Photoshop, GIMP, Aseprite, Paint.NET) and discover that:
* **The cockpit looks great during daytime**, but
* **When the game switches to night (Red Light / Green Light floodlights), some grey panels or bezels turn glowing red while other grey areas remain cold, unlit, or dark grey**,

**Your intuition is 100% correct: the palette contains duplicate greys with different index numbers, and the Tornado engine only applies night lighting to one specific range.**

### The Root Cause:
1. **The Tornado engine is built on a hardware-indexed 256-color VGA DAC palette (`VISUAL/PALETTES.ASM`)**, strictly divided into functional subsystems.
2. **Indices `24` through `111` are reserved for the 3D Outside World** (runway concrete, military bunkers, enemy vehicles, terrain, sky). At night, the 3D rendering pipeline darkens these colors by dividing their RGB values (`shr al, Night`), keeping them **monochrome grey** ($R=G=B$). They are outside the cockpit and **never receive cockpit floodlighting**.
3. **Indices `176` through `255` are the Cockpit Panel Range (`VGA_Panel1`)**. When cockpit floodlights are engaged at night, the engine loads `PANEL_R1.RGB` through `PANEL_R4.RGB` (or `PANEL_G*.RGB` for green light) directly over indices **112 through 255** (with **176..255** mapped to the panel). Every grey in this range is dynamically replaced with a **red-shifted RGB value** (Red gun up to 4× to 6× higher than Green and Blue).
4. **If you painted parts of your cockpit using World Greys (indices 24..28 or 46..49) instead of Panel Greys (indices 192..220)**:
   * During daytime, both look like identical shades of grey on screen.
   * At night, the pixels using **192..220** turn glowing red under the floodlights, while the pixels using **24..28** or **46..49** stay cold grey, creating a patchy, unlit appearance.
5. **The RGB Conversion Trap**: If you edit your cockpit as a 24-bit TrueColor RGB image and use an automated converter, the converter searches the palette starting from index `0`. When it encounters a grey pixel (e.g. RGB `77, 77, 77`), it finds **World Index 48** long before it reaches **Panel Index 201**. As a result, saving as RGB inadvertently converts your cockpit greys into world greys, breaking night illumination!

---

## 2. Complete 256-Color Palette Map (Functional Architecture)

The 256-color palette (`CANONICAL_VGA_PALETTE`) is partitioned into specific hardware zones:

| Index Range | Designation | Count | Daytime Purpose | Behavior Under Night Mode (`Night > 0`) | Allowed in Cockpit .BT2? |
| :---: | :---: | :---: | :--- | :--- | :---: |
| **`0`** | **Transparency Key** | 1 | Pure Black `(0, 0, 0)`. | Remains `(0, 0, 0)`. | **YES (Cutouts ONLY)** |
| **`1..4`** | EGA System Colors | 4 | Red, Yellow, Cyan, White. | Preserved. | **NO** |
| **`5..7`** | External Panel Text | 3 | HUD/External view OSD text. | Remapped via `VGA_EXT_TEXT1/2`. | **NO** |
| **`8..23`** | Special Night Lights | 16 | Runway runway lights, beacons, red/green tracers. | **Untouched by night scaling** (remains bright in the dark). | **NO** |
| **`24..72`** | 3D World (General) | 49 | Concrete runways, buildings, grass, desert sand. | **Darkened mathematically** (`shr al, Night`). Stays monochrome grey. | **NO (Breaks Night Light)** |
| **`73..79`** | 3D World (Features) | 7 | Roads, runways, ground, rivers. | Darkened mathematically. | **NO** |
| **`80..95`** | 3D Hills | 16 | Terrain elevation contours. | Darkened mathematically. | **NO** |
| **`96..111`** | 3D Sky | 16 | Sky gradient from zenith to horizon. | Darkened mathematically. | **NO** |
| **`112..127`** | Moving Map Hills | 16 | Tactical map elevation steps. | Overwritten by `PANEL_xy.RGB`. | Pre-rendered MFD only |
| **`128..141`** | Dynamic Instruments | 14 | Needles, digital counters, HUD green (`133`). | Overwritten by `PANEL_xy.RGB`. | Pre-rendered MFD only |
| **`142..143`** | Extra Panel Colors | 2 | Green CRT indicators. | Overwritten by `PANEL_xy.RGB`. | Pre-rendered MFD only |
| **`144..158`** | Moving Map Symbology | 15 | Waypoints, flight plan lines, airfields. | Overwritten by `PANEL_xy.RGB`. | Pre-rendered MFD only |
| **`159`** | CRT Red Warning | 1 | RWR missile warning, CRT red threat. | Overwritten by `PANEL_xy.RGB`. | Pre-rendered MFD only |
| **`160..175`** | CRT Green Phosphor | 16 | 16 steps of CRT radar/map/TIALD green. | Overwritten by `PANEL_xy.RGB`. | Pre-rendered MFD only |
| **`176..255`** | **COCKPIT PANEL (`VGA_Panel1`)** | **80** | **The entire solid cockpit artwork!** | **Dynamically overwritten by `PANEL_R*.RGB` (Red) or `PANEL_G*.RGB` (Green).** | **YES (100% of solid cockpit)** |

> [!IMPORTANT]
> **The Golden Rule of Cockpit Painting**:
> * **100% of visible solid cockpit panel artwork MUST use indices `176` through `255`**.
> * **The only index below `176` permitted in a cockpit `.BT2` is Index `0`**, which serves strictly as the transparent "window" key.

---

## 3. The Grey Palette Conflict: World Greys vs. Cockpit Panel Greys

The table below demonstrates why using the wrong grey breaks night lighting. 

* 6-bit DAC gun values ($0..63$) are scaled to 8-bit RGB ($0..255$) by multiplying by 4.
* **Night R2** represents Level 2 Red Floodlight (`PANEL_R2.RGB`).

### World Greys (DO NOT USE in Cockpits):
These colors exist in the range `24..49`. Under night mode, they are bit-shifted down, remaining cold, monochrome neutral greys:

| World Index | Equate Name | Day 8-bit RGB | Night R2 RGB | Visual Result Under Red Floodlights |
| :---: | :--- | :---: | :---: | :--- |
| **`24`** | `COL_WHITE1` | `(255, 255, 255)` | `(64, 64, 64)` | Turns dark neutral grey; **does not turn red** |
| **`25`** | `COL_GREY1` | `(203, 203, 203)` | `(51, 51, 51)` | Turns dark neutral grey; **does not turn red** |
| **`26`** | `COL_GREY2` | `(150, 150, 150)` | `(37, 37, 37)` | Turns dark neutral grey; **does not turn red** |
| **`27`** | `COL_GREY3` | `(125, 125, 125)` | `(31, 31, 31)` | Turns dark neutral grey; **does not turn red** |
| **`28`** | `COL_GREY4` | `(97, 97, 97)` | `(24, 24, 24)` | Turns dark neutral grey; **does not turn red** |
| **`46`** | `COL_NEW_GREY1` | `(227, 227, 227)` | `(56, 56, 56)` | Turns dark neutral grey; **does not turn red** |
| **`47`** | `COL_NEW_GREY2` | `(174, 174, 174)` | `(43, 43, 43)` | Turns dark neutral grey; **does not turn red** |
| **`48`** | `COL_NEW_GREY3` | `(77, 77, 77)` | `(19, 19, 19)` | Turns dark neutral grey; **does not turn red** |
| **`49`** | `COL_NEW_GREY4` | `(48, 48, 48)` | `(12, 12, 12)` | Turns dark neutral grey; **does not turn red** |

### Cockpit Panel Greys (MANDATORY for Cockpits):
These colors exist in the range `192..220`. Under night mode, they are overwritten by `PANEL_R*.RGB`, shifting strongly into the red spectrum:

| Panel Index | Equate / Purpose | Day 8-bit RGB | Night R2 RGB | Night Visual Appearance | Replaces World Index |
| :---: | :--- | :---: | :---: | :--- | :---: |
| **`192`** | Deepest Shadow / Bezel Rim | `(0, 0, 8)` | `(0, 0, 0)` | Deep black shadow | — |
| **`193`** | Dark Crevice | `(12, 12, 24)` | `(0, 0, 0)` | Deep shadow | — |
| **`194`** | Shadow Grey | `(20, 20, 32)` | `(4, 0, 0)` | Faint warm red shadow | — |
| **`195`** | Dark Bezel Grey | `(28, 28, 44)` | `(8, 0, 0)` | Dim red shadow | — |
| **`196`** | Instrument Housing Low | `(36, 36, 52)` | `(12, 0, 0)` | Subtle red shadow | — |
| **`197`** | Instrument Housing Mid-Low | `(44, 44, 60)` | `(16, 4, 0)` | Low red glow | **`49`** |
| **`198`** | Panel Dark Grey 1 | `(52, 52, 69)` | `(20, 4, 4)` | Warm red-grey metal | — |
| **`199`** | Panel Dark Grey 2 | `(60, 60, 77)` | `(24, 4, 4)` | Warm red-grey metal | — |
| **`200`** | Panel Mid Grey 1 | `(69, 69, 85)` | `(28, 4, 4)` | Red-illuminated metal | — |
| **`201`** | Panel Mid Grey 2 | `(77, 77, 93)` | `(32, 8, 4)` | Red-illuminated metal | **`48`** |
| **`202`** | Panel Mid Grey 3 | `(85, 85, 101)` | `(36, 8, 4)` | Red-illuminated metal | — |
| **`203`** | Panel Mid Grey 4 | `(93, 93, 109)` | `(40, 8, 4)` | Red-illuminated metal | **`28`** |
| **`204`** | Switchplate Grey Low | `(105, 105, 121)` | `(44, 12, 8)` | Distinct red metal sheen | — |
| **`205`** | Switchplate Grey Mid | `(113, 113, 130)` | `(48, 12, 8)` | Distinct red metal sheen | **`27`** |
| **`206`** | Switchplate Grey High | `(121, 121, 138)` | `(52, 12, 8)` | Distinct red metal sheen | — |
| **`207`** | Dial Rim Grey Low | `(130, 130, 146)` | `(56, 12, 8)` | Glowing red metal | — |
| **`208`** | Dial Rim Grey Mid | `(138, 138, 154)` | `(60, 16, 12)` | Glowing red metal | — |
| **`209`** | Dial Rim Grey High | `(146, 146, 162)` | `(64, 16, 12)` | Glowing red metal | — |
| **`210`** | Gauge Bezel Low | `(154, 154, 170)` | `(68, 16, 12)` | Bright red-illuminated bezel | **`26`** |
| **`211`** | Gauge Bezel Mid | `(162, 162, 174)` | `(72, 20, 12)` | Bright red-illuminated bezel | — |
| **`212`** | Gauge Bezel High | `(170, 170, 182)` | `(76, 20, 16)` | Bright red-illuminated bezel | **`47`** |
| **`213`** | Screw / Bolt Head Low | `(178, 178, 190)` | `(80, 20, 16)` | High red metal reflection | — |
| **`214`** | Screw / Bolt Head Mid | `(186, 186, 199)` | `(84, 24, 16)` | High red metal reflection | — |
| **`215`** | Screw / Bolt Head High | `(195, 195, 211)` | `(88, 24, 16)` | Vivid red reflection | — |
| **`216`** | Metal Highlight Low | `(203, 203, 219)` | `(92, 24, 20)` | Intense red highlight | **`25`** |
| **`217`** | Metal Highlight Mid | `(211, 211, 227)` | `(96, 28, 20)` | Intense red highlight | — |
| **`218`** | Metal Highlight High | `(219, 219, 231)` | `(100, 28, 20)` | Intense red highlight | **`46`** |
| **`219`** | Top Ridge Specular | `(227, 227, 239)` | `(104, 28, 20)` | Glinting red specular highlight | — |
| **`220`** | Purest Metal Highlight | `(235, 235, 247)` | `(108, 32, 24)` | Glinting red specular highlight | — |
| **`255`** | Specular Glint / Text White | `(203, 203, 219)` | `(144, 144, 156)`| White text with warm glow | **`24`** |

---

## 4. Full Functional Breakdown of the Cockpit Range (`176..255`)

To ensure your cockpit looks authentic by day and operates correctly by night, adhere to these designated ranges:

### 4.1. Green-Grey Cockpit Base Metal (`176..191`) - 16 Shades
* **Daytime Appearance**: RAF Cockpit Medium Sea Grey with subtle green-grey undertones (`219, 239, 231` down to `24, 56, 89`).
* **Usage**: The main body of the instrument panel, the canopy arch ironwork, and the bulkhead structural framework.
* **Night Behavior**: Responds strongly to red or green floodlights.

### 4.2. Neutral Instrument Greys (`192..220`) - 29 Shades
* **Daytime Appearance**: Perfectly progressive neutral greys from near-black `(0, 0, 8)` to near-white `(235, 235, 247)`.
* **Usage**: Instrument bezels, altimeter/airspeed/VSI casings, MFD button frames, toggle switch collars, screw heads, panel seam lines, and inner shadows.
* **Night Behavior**: Becomes illuminated with vivid crimson/ruby highlights under red floodlights (`PANEL_R*.RGB`).

### 4.3. Annunciator & Warning Red Lights (`221..228`) - 8 Shades
* **Daytime Appearance**: Deep red `(36, 0, 0)` to blazing emergency red `(255, 48, 28)`.
* **Usage**: Fire pull handles, Master Warning lamp, stall warning, engine fire lights.
* **Night Behavior**: **Self-illuminating**. These lamps retain their full, intense red brilliance even in pitch black darkness.

### 4.4. Annunciator Green Indicator (`229`) - 1 Shade
* **Daytime Appearance**: Deep warning green `(0, 28, 0)`.
* **Usage**: Gear down and locked green triangles, RADAR altimeter low-height green flag.
* **Night Behavior**: Retains visible green illumination.

### 4.5. Dynamic Needles & ILS Glide Slope (`230..236`) - 7 Shades
* **Daytime Appearance**: Instrument indicator needles, ILS localizer / glide slope bars.
* **Usage**: Internal simulation rendering needles. (Do not paint static panel needles into the backdrop; the engine draws active needles dynamically).

### 4.6. Caution Amber / Yellow Lamps & Guards (`237..248`) - 12 Shades
* **Daytime Appearance**: Amber/gold warnings `(219, 138, 0)` to bright yellow `(255, 162, 20)`.
* **Usage**: Master Arm switch guard stripes, CWP (Central Warning Panel) caution tiles, canopy unlock warning.
* **Night Behavior**: Self-illuminating amber glow.

### 4.7. Kneepad Paper & Leather Straps (`249..253`) - 5 Shades
* **Daytime Appearance**: Warm beige paper `(140, 124, 108)` and brown leather strap shading.
* **Usage**: Navigator desk clipboard, kneepad flight plan notes, pilot lap straps.

### 4.8. Cockpit Specular White (`255`) - 1 Shade
* **Daytime Appearance**: Crisp white highlight `(203, 203, 219)`.
* **Usage**: Dial graduation marks (tick marks), printed switch labels ("LATE ARM", "JETTISON", "RADAR"), and specular glints on glass.
* **Night Behavior**: Shifts to warm illuminated white (`144, 144, 156`), ensuring dial numbers and text remain legible at night.

---

## 5. The Critical Role of Index 0 (Transparency Key)

In Tornado, color index **`0`** is defined as `(0, 0, 0)` pure black.

> [!CAUTION]
> **Index 0 is NOT just black; it is the 3D Engine Transparency Key!**
> 
> * In `PILOTPAN.BT2` and `FRAMEPAN.BT2`, every pixel set to **Index 0** is skipped by the 2D blitter. The 3D outside world (sky, terrain, clouds, runway lights, target aircraft) is rendered directly through these transparent pixels.
> * If you accidentally paint a piece of the solid cockpit console, canopy strut, or instrument casing using **Index 0**, **the 3D sky and ground will show right through your cockpit!**
> * Inside the MFD (central CRT) and TAB screens, **Index 0** is where the vector radar scan, moving map, and TIALD thermal video are drawn.
> 
> **How to paint solid black in the cockpit:**
> If you need deep black shadows, dark switch cavities, or unlit CRT bezels, **always use Index `192` (`0, 0, 8`) or Index `193` (`12, 12, 24`)**. Never use Index `0` for solid surfaces!

---

## 6. Recommended Artist Workflow & Image Editor Setup

To guarantee 100% fidelity and seamless night lighting, follow these strict workflow rules:

### Rule 1: Always Stay in 8-Bit Indexed ('P') Mode
1. **Never convert your image to 24-bit RGB / 32-bit RGBA.**
2. When exporting from `bt2_converter.py`, the resulting `.PNG` contains the exact 256-color palette.
3. Open this PNG directly in your graphics editor while maintaining **Indexed Color (8-bit)** mode.
4. When saving, export as **8-bit Indexed PNG**.

### Rule 2: Pick Colors by Index Number, Not by Eye
* When drawing cockpit elements, open your editor's **Color Table / Swatch Palette**:
  * For Cockpit Metal: Pick exclusively from **Indices 176 to 191**.
  * For Greys, Bezels, Shadows, Screws, Rims: Pick exclusively from **Indices 192 to 220**.
  * For Switch Labels & Dial Markings: Use **Index 255**.
  * For Amber/Caution Guards: Use **Indices 237 to 248**.
  * For Warning Reds: Use **Indices 221 to 228**.
  * For Window Transparency: Use **Index 0**.
* **Do not sample colors from the outside sky or HUD using the Eyedropper tool**, as this may select World Greys (indices 24..28, 46..49).

### Rule 3: Disable Anti-Aliasing on Transparency Boundaries
* When drawing the silhouette of the canopy arch (`FRAMEPAN.BT2`) or windscreen frame (`PILOTPAN.BT2`) against the transparent sky (Index 0):
  * **Turn off Brush Anti-Aliasing and Feathering**.
  * Use a hard-edge 1-pixel pencil tool.
  * Anti-aliasing creates blended intermediate colors that do not exist in the cockpit range, which the converter will map to arbitrary low indices (world/sky colors), creating flickering or discolored outlines around the canopy at night.

---

## 7. Fixing an Existing Cockpit with Rogue Greys (Auto-Sanitizer)

If an artist has already drawn a custom cockpit and inadvertently used low-index World Greys, you do not need to repaint the cockpit by hand!

The script below inspects an 8-bit indexed cockpit PNG, identifies any rogue world greys (indices `24..111`) used on the cockpit panel, and automatically remaps them to the authentic, night-lit Cockpit Panel Greys (`192..220`):

```python
#!/usr/bin/env python3
"""
sanitize_cockpit_palette.py - Remaps rogue World Greys to Cockpit Panel Greys
"""
import sys
from PIL import Image

# Mapping table from World / Low indices to Cockpit Panel indices (176..255)
# Preserves Index 0 (Transparency Key)
REMAP_TABLE = {
    # World Greys -> Panel Greys
    24: 255,  # White (255, 255, 255) -> Specular White (255)
    25: 216,  # Grey 1 (203, 203, 203) -> Panel Highlight Low (216)
    26: 210,  # Grey 2 (150, 150, 150) -> Gauge Bezel Low (210)
    27: 205,  # Grey 3 (125, 125, 125) -> Switchplate Grey Mid (205)
    28: 203,  # Grey 4 (97, 97, 97)   -> Panel Mid Grey 4 (203)
    46: 218,  # New Grey 1 (227)       -> Metal Highlight High (218)
    47: 212,  # New Grey 2 (174)       -> Gauge Bezel High (212)
    48: 201,  # New Grey 3 (77)        -> Panel Mid Grey 2 (201)
    49: 197,  # New Grey 4 (48)        -> Housing Mid-Low (197)
    # HUD / Needle duplicates outside panel range
    128: 211, # Needle grey -> Panel Bezel Mid (211)
    129: 205, # Counter grey -> Switchplate Mid (205)
    138: 201, # Knee ink -> Panel Mid Grey (201)
}

def sanitize_png(input_png, output_png):
    img = Image.open(input_png)
    if img.mode != "P":
        print(f"Error: {input_png} is in mode '{img.mode}', expected indexed 'P'.")
        print("Please convert your image to 8-bit Indexed Color before sanitizing.")
        return False

    palette = img.getpalette()
    pixels = bytearray(img.tobytes())
    remapped_count = 0

    for i in range(len(pixels)):
        val = pixels[i]
        if val in REMAP_TABLE:
            pixels[i] = REMAP_TABLE[val]
            remapped_count += 1

    out_img = Image.frombytes("P", img.size, bytes(pixels))
    out_img.putpalette(palette)
    out_img.save(output_png)
    print(f"Sanitized: {input_png} -> {output_png}")
    print(f"Total rogue pixels corrected to night-lit panel range: {remapped_count:,}")
    return True

if __name__ == "__main__":
    if len(sys.argv) < 3:
        print("Usage: python sanitize_cockpit_palette.py <input.png> <output.png>")
    else:
        sanitize_png(sys.argv[1], sys.argv[2])
```

---

## 8. Summary Checklist Before Converting to `.BT2`

Before generating your final `.BT2` using `bt2_converter.py`, verify this 5-point checklist:

1. [ ] **Image Mode**: The PNG is in **Indexed Color ('P')** mode with 256 colors.
2. [ ] **Dimensions**: In-flight cockpits (`PILOTPAN`, `NAVIGPAN`, `FRAMEPAN`, `PSIDEPAN`, `NSIDEPAN`, `EXTRAPAN`) are exactly **800 x 600 pixels**.
3. [ ] **Byte Size**: Once converted to `.BT2`, the binary file size is exactly **480,000 bytes**.
4. [ ] **Transparency Integrity**: Index `0` is used **only** for open canopy windows and CRT display viewports. No solid panel structure uses Index `0`.
5. [ ] **Index Confinement**: Every solid cockpit pixel uses indices strictly within **`176` through `255`**. No World Greys (`24..28`, `46..49`) are present.
