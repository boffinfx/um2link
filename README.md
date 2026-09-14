# Boffin FX UM2-LINK ASIO Driver (v1.0.0)

[![Platform](https://img.shields.io/badge/Platform-Windows%2010%20%7C%20Windows%2011%20(64--bit)-blue.svg)]()
[![Driver Type](https://img.shields.io/badge/Driver%20Type-ASIO%202.2%20Wrapper-brightgreen.svg)]()
[![Architecture](https://img.shields.io/badge/Architecture-x64%20%2F%20x86-orange.svg)]()
[![License](https://img.shields.io/badge/License-Freeware-lightgrey.svg)]()

The **Boffin FX UM2-LINK** is a dedicated, self-contained, 64-bit user-mode **ASIO 2.2** driver wrapper engineered from the ground up in C++. It natively bridges incoming Digital Audio Workstation (DAW) audio buffer requests directly to hardware endpoints via Windows WASAPI Exclusive mode.

By bypassing the sluggish standard Windows audio subsystem, UM2-LINK unlocks true ultra-low latency, bit-perfect signal purity, and stable performance for budget class-compliant USB audio interfaces.

---

## 🚀 Key Features

* **⚡ Ultra-Low Round-Trip Latency:** Drops tracking latency from the standard ~45ms down to an imperceptible **5ms – 15ms**.
* **🎛️ Bit-Perfect Signal Purity:** Tunnels directly underneath the Windows audio engine/mixer to capture pure, uncompressed hardware audio without OS-level noise gates, DSP filters, or harmonic distortion.
* **🧵 MMCSS Kernel Priority:** Integrates with the Windows Multimedia Class Scheduler Service (`avrt.dll`) to assign audio streaming threads critical "Pro Audio" priority, actively preventing buffer underruns, clicks, and pops.
* **🔄 Clock Drift Stabilization:** Automatically locks and synchronizes virtual ADC (Input) and DAC (Output) data packets to eliminate timing drift and buffer jitter.
* **💎 32-Bit Float Processing:** Native floating-point processing eliminates CPU-heavy bit-depth conversion artifacts and preserves dynamic headroom.
* **🛡️ Zero-Bloat Defensive Architecture:** Statically linked monolithic binary featuring safe pointer checks to ensure clean DAW initialization and teardown without segmentation faults.

---

## 🎛️ Hardware Compatibility

Optimized specifically for **Texas Instruments PCM2902 / PCM2902C** and compatible **Coolaudio** CODEC chipsets (`VID: 08BB, PID: 2902` / *"USB Audio CODEC"*):

### Audio Interfaces & Mixers
* **Behringer:** U-Phoria UM2, U-Control UCA202
* **Alesis:** MultiMix 4 USB *(Original non-FX version)*
* **ADS Tech:** Instant Music RDX-150

### Phono Preamps & Vinyl Tools
* **Behringer:** U-PHONO UFO202
* **ART:** USB Phono Plus *(Early iterations)*
* **TC-772:** USB Phono Preamp

---

## 💻 System Requirements

* **Operating System:** Windows 11 (64-bit) or Windows 10 (64-bit)
* **DAW / Host Application:** Any ASIO 2.2 compliant host (e.g., Reaper, Ableton Live, FL Studio, Steinberg Cubase, PreSonus Studio One, Avid Pro Tools, Bitwig Studio)
* **Hardware Interface:** Compatible USB Audio Device with PCM2902 or class-compliant CODEC

---

## 📦 Installation & Setup

1. **Connect Hardware:** Plug your USB audio interface into a direct USB port on your PC.
2. **Run Installer:** Execute `UM2LinkSetup.exe` (run as Administrator if prompted).
3. **Configure Your DAW:**
   * Open your Digital Audio Workstation (DAW).
   * Navigate to **Audio Device Settings / Preferences**.
   * Set **Audio System / Driver Type** to `ASIO`.
   * Set **ASIO Driver** to `UM2 Link ASIO Driver`.
4. **Set Buffer Size:** Select your desired buffer size (recommended: 64 to 256 samples depending on CPU capacity and project size).

---

## 📁 File Structure & Binary Components

```text
UM2-Link/
├── UM2LinkSetup.exe           # Code-signed setup executable
├── um2_link.dll               # 64-bit ASIO COM server DLL
├── um2_link_32.dll            # 32-bit legacy DAW compatibility DLL
├── EULA.txt                   # End User License Agreement
└── README.md                  # Documentation and architecture overview
