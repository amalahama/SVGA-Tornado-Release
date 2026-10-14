# SVGA Tornado: Digital Integration Tornado & Desert Storm in High-Resolution 800x600 VESA

[![Release](https://img.shields.io/badge/Release-1.22-brightgreen.svg?style=for-the-badge)](https://github.com/amalahama/SVGA-Tornado-Release)
[![Resolution](https://img.shields.io/badge/Resolution-800x600%20256--Color%20VESA-blue.svg?style=for-the-badge)](https://github.com/amalahama/SVGA-Tornado-Release)
[![Frame Rate](https://img.shields.io/badge/Frame%20Rate-Up%20to%2050%20FPS-green.svg?style=for-the-badge)](https://github.com/amalahama/SVGA-Tornado-Release)
[![Platform](https://img.shields.io/badge/Platform-MS--DOS%20%2F%20DOSBox-orange.svg?style=for-the-badge)](https://github.com/amalahama/SVGA-Tornado-Release)
[![Donate with PayPal](https://img.shields.io/badge/Donate-PayPal-00457C.svg?style=for-the-badge&logo=paypal&logoColor=white)](https://paypal.me/amalahama)

Welcome to the official release repository for the **SVGA High-Resolution Port of Digital Integration's Tornado & Operation Desert Storm**.

This package contains the drop-in replacement archive **`Release 1.22.zip`** with all binary files, high-definition cockpit panels, hardware HOTAS 4-axis flight controls, and art conversion utilities required to upgrade your original copy of Tornado from standard 320x200 Mode 13h to **800x600 256-color VESA SVGA** with fluid **50 FPS flight physics**, tear-free hardware double-buffering, and extensive visual refinements.

---

## Screenshots

<p align="center">
  <img src="SCREENSHOTS/Screenshoot1.png" alt="Screenshot 1" width="49%" />
  <img src="SCREENSHOTS/Screenshoot2.png" alt="Screenshot 2" width="49%" />
</p>
<p align="center">
  <img src="SCREENSHOTS/Screenshoot3.png" alt="Screenshot 3" width="49%" />
  <img src="SCREENSHOTS/Screenshoot4.png" alt="Screenshot 4" width="49%" />
</p>
<p align="center">
  <img src="SCREENSHOTS/Screenshoot5.png" alt="Screenshot 5" width="49%" />
  <img src="SCREENSHOTS/Screenshoot6.png" alt="Screenshot 6" width="49%" />
</p>
<p align="center">
  <img src="SCREENSHOTS/Screenshoot7.png" alt="Screenshot 7" width="98%" />
</p>

---

## Major Enhancements & Features

* **High-Resolution 800x600 VESA Display**:
  Upgrades the visual pipeline from 320x200 to native VESA Mode 103h (800x600, 256 colors), delivering **7.5 times the pixel resolution** of the original 1993/1994 release.
* **Fluid 50 FPS Physics Integration**:
  Unlocks the simulation frame rate from its historical 16-20 FPS cap up to **50 FPS** (`MaxFrameRate = 2`) utilizing 32-bit fractional position and attitude integration.
* **Redesigned High-Definition Cockpits**:
  The 6 in-flight cockpit panel backdrops (`.BT2`) have been redesigned for 800x600 resolution:
  * Pilot Main Instrument Panel (`PILOTPAN.BT2`)
  * Navigator / WSO Tactical Console (`NAVIGPAN.BT2`)
  * Pilot & Navigator Side Panels (`PSIDEPAN.BT2`, `NSIDEPAN.BT2`)
  * Cockpit Canopy Frame (`FRAMEPAN.BT2`)
  * Auxiliary Flight Instrument Panels (`EXTRAPAN.BT2`)
* **Tear-Free Hardware Double Buffering (Zero-Flicker VSYNC)**:
  Uses direct VESA VBE `AX=4F07h` page flipping synchronized with the vertical retrace interval, completely removing screen tearing and CRT raster flicker across all DOSBox flavors including Vanilla DOSBox.
* **Vectorial HUD & Avionics Symbology**:
  Precision-scaled Head-Up Display (HUD), Multi-Function Displays (MFD), look-down moving map, and air-to-air / air-to-ground radar sweeps adapted for high resolution.
* **Hardware HOTAS 4-Axis Throttle & Rudder Support**:
  Full continuous physical throttle lever (dry military thrust & progressive afterburner stages) and rudder pedals/twist grip integration.
* **Enhanced Environmental & Combat Effects**:
  * Multi-stage sky gradient palettes for dawn, day, dusk, and night.
  * Rebuilt high-density night sky starfield matching original visual depth.
* **Dual-Theater Support**:
  Includes standalone executables for both the **European Theater** (`FLIGHT.EXE`) and the **Operation Desert Storm** Gulf War expansion (`DESERT.EXE`).

---

## What's New in Release 1.22

### Manual Keyboard Throttle Control (+/-) Restored
* **Manual Throttle Key Control in Keyboard Mode:** Fixed an issue where the `+` and `-` keys did not control the engine throttle in manual mode when "Keyboard" was selected as the control device in game preferences (`Preferences -> Controls`), if a 4-axis joystick hardware profile (`joysticktype=4axis`) was present in `dosbox.conf`.
* **Device Mode Differentiation:** The simulation now checks the active in-game `ControlDevice`. Hardware joystick throttle lever bypass is strictly limited to dual joystick / HOTAS mode (`CTRL_DEV_JOY2` - Standard Joystick + Rudder Pedals with 4 axes). When playing in Keyboard mode (`Keyboard 1` or `Keyboard 2`) or Single Joystick mode (`CTRL_DEV_JOY1`), `+` and `-` directly increment/decrement manual throttle and reheat.
* **Autothrottle Key Controls:** Adjusting target airspeed (`AutoVias`) with `+` and `-` when autothrottle is engaged (`A` key) continues to work seamlessly across all control schemes.

---

## What's New in Release 1.21

### Autothrottle +/- Speed Control Fix for HOTAS / 4-Axis Controllers
* **Target Airspeed Adjustment Restored:** Fixed an issue where the `+` and `-` keys/buttons did not adjust the demanded autothrottle airspeed (`AutoVias`) when flying with a 4-axis joystick or HOTAS controller (`CTRL_DEV_JOY2` / Standard Joystick + Rudder Pedals).
* **Interrupt Routing Refinement:** Re-routed the 100 Hz timer interrupt table (`KbdCtrlSwitch[CTRL_DEV_JOY2]`) directly to `KbdThrottle`. When autothrottle is engaged (`AutoThrot = 1`), `KbdAutoThrot` now actively processes `+` and `-` inputs (incrementing/decrementing `AutoVias` smoothly), while in manual flight mode (`AutoThrot = 0`), physical throttle lever position continues to modulate engine thrust exclusively without key conflicts.

---

## What's New in Release 1.2

### 1. Vanilla DOSBox Compatibility & Zero-Flicker VSYNC
* Re-engineered VESA double-buffering page flipping routines to call standard VBE `AX=4F07h, BX=0000h` synchronized with hardware CRT status register polling (port 3DAh vertical blanking).
* Eliminates screen tearing and raster flickering on standard **Vanilla DOSBox 0.74 / 0.74-3** and DOSBox ECE without requiring specialized third-party patch builds.

### 2. Native HOTAS 4-Axis Throttle & Rudder Hardware Support
* Exposes full 4-axis flight controller support (Pitch, Roll, Yaw, Throttle):
  * **Axis 1 & 2**: Aileron Roll and Elevator Pitch.
  * **Axis 3**: Rudder Pedals or Joystick Twist Grip.
  * **Axis 4**: Continuous Physical Throttle Lever.
* **Throttle Modulation**:
  * **Dry Military Thrust (0 to 26)**: Moving the physical throttle lever forward from 0% to ~80% travel smoothly and progressively modulates dry engine power.
  * **Reheat / Afterburner Stages (27 to 32)**: Advancing past 80% engages 5 distinct reheat stages with tactile latching.
  * **Autothrottle Integration**: Pressing `A` activates autothrottle to lock current airspeed. Moving the physical throttle lever beyond a widened deadband (+50% hysteresis buffer) automatically disconnects autothrottle back to manual lever control without accidental disconnections.
* **Required DOSBox Configuration (`dosbox.conf`)**:
  ```ini
  [joystick]
  joysticktype=4axis
  timed=true
  autofire=false
  swap34=false
  buttonwrap=false
  ```
* **In-Game Controls Setup**:
  1. Go to the Tornado main menu $\rightarrow$ **Preferences** $\rightarrow$ **Controls**.
  2. Select **Standard Joystick + Rudder Pedals**.
  3. The simulation automatically detects Joystick B, routing Axis 3 to Rudder and Axis 4 to absolute physical Throttle.

### 3. Elimination of Trapped / Ghost Pixels
* **Radar Warning Receiver (RWR)**: Expanded the active refresh redraw rectangle from $120\times 72$ to $120\times 84$, covering the entire CRT green region and upper azimuth threat arc. Threat symbols moving across the top arch no longer leave stuck pixels.
* **Pilot HUD Viewport**: Corrected boundary redrawing along the cockpit canopy frame.

### 4. Cockpit Utility Suite (`tool/` directory inside zip)
* Included **`tool/bt2_converter.py`**: A standalone Python 3 utility for bidirectional conversion between raw `.BT2` cockpit buffers and editable `.PNG` images with exact canonical 256-color palette roundtrip fidelity.
* Included **`tool/README.md`**: A comprehensive technical reference dossier containing the complete coordinate layout, scaling formulas, needle radii, and bounding boxes for all instruments, dials, readouts, and MFDs in 800x600 SVGA.

---

## What's New in Release 1.1

* **Sound Blaster Hardware Initialization Fix**:
  Fixed Sound Blaster sound card detection (`VoiceInit` check) so that clean installations configured with Sound Blaster in `SETUP.INF` boot directly into Demo, Quickstart, and Campaign modes with full AdLib/FM audio without aborting with Error 241.
* **Review Mode 2D Console & 64 KB PREVIEW.BT2 Restoration**:
  Restored the authentic 64,000-byte `PREVIEW.BT2` backdrop in the distribution package. The SVGA engine scales this 2D console dynamically into the bottom 800x120 viewport, fixing panel corruption when upgrading over previous releases.
* **Aircraft Photo Decompression Fix (Review Mode)**:
  Fixed 4-bit delta decompression in `UnpackDeltaPic` by preserving the nibble phase mask (`ah`) and running delta accumulator (`bl`) across rows, eliminating horizontal static and displaying digitized aircraft photos (`.PT2`) in crisp high-definition 800x600.
* **Compass & Cardinal Points Realignment (Explorer & Map Views)**:
  Corrected line and tick mark coordinate scaling ($2.5\times$ horizontal, $3.0\times$ vertical) on the tactical compass, keeping cardinal points ('N', 'S', 'E', 'W') perfectly centered around the compass ring.

---

## Release Package: `Release 1.22.zip`

All replacement files are packaged together into **`Release 1.22.zip`** (also mirrored as `Release1.22.zip`) (~618 KB compressed / 4.7 MB uncompressed):

* **`FLIGHT.EXE`**: European Theater SVGA 800x600 executable (Release 1.22)
* **`DESERT.EXE`**: Desert Storm Theater SVGA 800x600 executable (Release 1.22)
* **`PILOTPAN.BT2`**: Pilot Front Cockpit Panel (800x600 256-color)
* **`NAVIGPAN.BT2`**: Navigator / WSO Tactical Radar Console (800x600)
* **`PSIDEPAN.BT2`**: Pilot Left/Right Side Console (800x600)
* **`NSIDEPAN.BT2`**: Navigator Left/Right Side Console (800x600)
* **`FRAMEPAN.BT2`**: Cockpit Canopy Arch and Glass Frame (800x600)
* **`EXTRAPAN.BT2`**: Auxiliary Instruments & Warning Panel (800x600)
* **`PREVIEW.BT2`**: Authentic Tactical Console Backdrop (320x200, dynamically scaled to 800x120)
* **`DOSBOX_RECOMMENDATIONS.txt`**: Performance, HOTAS controls, and audio tuning guide
* **`tool/`**:
  * **`tool/bt2_converter.py`**: Standalone BT2 $\leftrightarrow$ PNG conversion utility with canonical VGA palette
  * **`tool/README.md`**: Utility instructions and SVGA cockpit instrument coordinate dossier

---

## Installation Guide

### Prerequisites
You need an existing, working installation of **Digital Integration Tornado CD-ROM** (or floppy 1.0e) and **Operation Desert Storm**.

### Step 1: Download `Release 1.22.zip`
Download **`Release 1.22.zip`** from this repository.

### Step 2: Backup Original Files *(Recommended)*
Before extracting, create a backup copy of your original 320x200 files inside your game's `FLIGHT\` folder:
* `FLIGHT.EXE`
* `DESERT.EXE` *(if Desert Storm is installed)*
* All 7 original `.BT2` files (`PILOTPAN.BT2`, `NAVIGPAN.BT2`, `PSIDEPAN.BT2`, `NSIDEPAN.BT2`, `FRAMEPAN.BT2`, `EXTRAPAN.BT2`, `PREVIEW.BT2`)

### Step 3: Extract into `FLIGHT\`
Extract all files from **`Release 1.22.zip`** directly into the `FLIGHT\` directory of your Tornado installation (e.g. `C:\TORNADO\FLIGHT\` or `D:\TORNADO.CD\FLIGHT\`), replacing the existing files when prompted.

### Step 4: Run the Simulation
Launch Tornado through DOSBox using your regular launcher (e.g. `T.BAT`, `GO.BAT`, or direct executable invocation).

---

## Recommended DOSBox Configuration (`dosbox.conf`)

Rendering 800x600 at 50 FPS requires 7.5x the pixel fillrate of the original game. In DOSBox, uncalibrated CPU cycles or default audio buffer sizes can cause audio stuttering (*buffer underrun*) during polygon-heavy scenes (low-altitude flight, complex airbases, horizon gradients).

To achieve **rock-solid 50 FPS** with **crystal-clear, stutter-free audio** and **full 4-axis HOTAS controls**, update your `dosbox.conf` with these settings:

```ini
[cpu]
core=dynamic
cputype=pentium
cycles=fixed 75000
# IMPORTANT: Avoid 'cycles=max'. Unbounded max cycles starves the host audio thread
# and produces audio crackling. A fixed setting between 70000 and 85000 ensures
# rock-solid 50 FPS with consistent timing. If dynamic cycles are preferred, use:
# cycles=max 85%

[mixer]
rate=48000
blocksize=2048
prebuffer=70
# IMPORTANT: 
# - 'rate=48000' matches modern Windows WASAPI native rate, eliminating resampling overhead.
# - 'blocksize=2048' (default is 1024) provides latency headroom during 3D rendering spikes.
# - 'prebuffer=70' (default is 20 ms) is vital: at 50 FPS, one frame takes 20 ms.
#   Setting prebuffer to 70 ms provides a 3-4 frame safety cushion.

[sblaster]
sbtype=sb16
sbbase=220
irq=7
dma=1
hdma=5
sbmixer=true
oplmode=opl3
oplrate=48000
# NOTE: 'oplrate=48000' must match the mixer rate to avoid internal OPL resampling.

[joystick]
joysticktype=4axis
timed=true
autofire=false
swap34=false
buttonwrap=false
# NOTE: Exposes 4-axis hardware controllers (Axis 1=Roll, Axis 2=Pitch, Axis 3=Rudder, Axis 4=Throttle).

[sdl]
output=opengl
# NOTE: Hardware-accelerated video blitting delegates 800x600 scaling to your GPU.

[render]
scaler=none
# NOTE: NEVER use CPU software scalers (such as normal2x, hq) on 800x600 SVGA.
```

### Recommended DOSBox Flavors
1. **Vanilla DOSBox 0.74-3 / ECE**:
   Runs flawlessly with Release 1.2's zero-flicker VSYNC double buffering and tuned `[mixer]`, `[cpu]`, and `[joystick]` configuration.
2. **[DOSBox-Staging](https://dosbox-staging.github.io/) (Highly Recommended)**:
   Includes a modern SDL2 multithreaded audio engine with dynamic rate control that completely prevents audio clicks, pops, and underruns.
3. **[DOSBox-X](https://dosbox-x.com/)**:
   Full-featured emulator with native high-resolution VESA support and dedicated audio thread options (`audiothread=true`).

---

## In-Game Command-Line Switches

You can pass command-line switches to `FLIGHT.EXE` and `DESERT.EXE`:

| Switch | Function | Example |
| :--- | :--- | :--- |
| `/a` | Start mission directly airborne (in flight) | `FLIGHT.EXE /a` |
| `/qs` | Quick Start mission (Airborne + Invulnerability) | `DESERT.EXE /qs` |
| `/sb` | Enable Sound Blaster digitized audio + AdLib FM | `FLIGHT.EXE /a /sb` |
| `/sa` | Enable pure AdLib FM audio (lightest CPU load) | `FLIGHT.EXE /a /sa` |
| `/va` | Fly the Tornado ADV (Air Defence Variant) | `FLIGHT.EXE /va` |
| `/p` | 3D Object / Aircraft model viewer mode | `DESERT.EXE /p` |
| `/nc` | No mid-air or terrain collisions (debugging) | `FLIGHT.EXE /a /nc` |

---

## Support & Donations

If you appreciate this modernization project and would like to support continued preservation, optimization, and reverse-engineering of classic flight simulations, donations are gratefully accepted via PayPal:

<p align="center">
  <a href="https://paypal.me/amalahama">
    <img src="https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif" alt="Donate with PayPal" />
  </a>
</p>

Thank you for keeping one of the greatest military flight simulators ever created flying high!
