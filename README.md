Boffin FX UM2-Link (Version 2.0.1)
![Platform](https://img.shields.io/badge/Platform-Windows%2010%20%2F%2011%20(x64%20%7C%20x86)-blue.svg)
![C++ Standard](https://img.shields.io/badge/C%2B%2B-20-blue.svg)
![Audio Standard](https://img.shields.io/badge/ASIO-2.3%20Compliant-orange.svg)
![Backend](https://img.shields.io/badge/WASAPI-Exclusive%20Event--Driven-green.svg)
![License](https://img.shields.io/badge/License-MIT-purple.svg)

Boffin FX UM2-Link V2 is a dedicated, user-mode ASIO 2.3 driver wrapper engineered from scratch in modern C++20 for Texas Instruments PCM2902-family USB class-compliant audio interfaces (including the popular Behringer U-Phoria UM2, UCA202, and UFO202).

By bridging host DAW ASIO calls directly into WASAPI in Exclusive Event-Driven Mode, UM2-Link bypasses the sluggish Windows consumer audio stack (audiodg.exe), eliminates OS-level communication filters, and unlocks studio-grade, ultra-low-latency tracking (down to ~6.0 ms round-trip) with zero third-party dependencies.

⚡ Key Highlights in Version 2
100% Standalone (No ASIO4ALL Required): Direct, native communication with the Windows USB audio kernel pipeline (usbaudio.sys).
Near 50% Latency Reduction: Drops round-trip execution from standard ~42 ms down to 23.3 ms at 512 samples, and 6.0 ms at 128 samples for imperceptible real-time monitoring.
Dual 32-bit & 64-bit Architecture: Includes both UM2LinkASIO64.dll (for Ableton Live, FL Studio, Reaper, Studio One) and UM2LinkASIO32.dll (for 32-bit hosts like VB-Audio VoiceMeeter).
True RAW Mode APO Severing: Enforces AUDCLNT_STREAMOPTIONS_RAW via IAudioClient2, completely shutting down Windows background microphone noise gates, echo cancelers, and "musical noise" whistling.
1 LSB TPDF Dithering: Linearizes 16-bit integer quantization using a persistent 32-bit Xorshift PRNG, eliminating metallic ringing and quantization fizz during note decay.
64,000-Frame Non-Dropping SPSC FIFO: Strict 4-byte packed StereoFrame circular buffers eliminate buffer underruns, waveform slicing, and Left/Right channel slippage.
Dynamic In-Memory Control Panel: Accessible directly from your DAW's "Hardware Setup" button to switch buffer quantum sizes (64 to 1024 samples) on the fly with user-level (HKCU) registry persistence.
Crash-Proof Memory Pinning: Integrated GET_MODULE_HANDLE_EX_FLAG_PIN prevents _unloaded.dll access violations (0xC0000005) during host audio engine reconfigurations.
📊 Latency Benchmarks
Tested on Windows 11 x64 (Build 26200), Ableton Live 12 Suite, sample rate locked at 48,000 Hz:

Driver Solution	Buffer Size	Input Latency	Output Latency	Overall Round-Trip
ASIO4ALL v2 (WDM-KS)	512 samples	21.0 ms	21.0 ms	41.9 ms
Boffin FX UM2-Link V2	512 samples	11.3 ms	12.0 ms	23.3 ms (~45% lower)
Boffin FX UM2-Link V2	256 samples	5.3 ms	6.0 ms	11.3 ms
Boffin FX UM2-Link V2	128 samples	2.7 ms	3.3 ms	6.0 ms (Live Tracking)
🎛️ Supported Hardware
UM2-Link V2 is specifically optimized for class-compliant USB 1.1 Full-Speed chipsets containing the Texas Instruments PCM2902 / PCM2902C or compatible Coolaudio V2902 architecture (VID: 0x08BB, PID: 0x2902):

Audio Interfaces & Mixers:
Behringer U-Phoria UM2
Behringer U-Control UCA202 / UCA222
Alesis MultiMix 4 USB (Original Non-FX)
ADS Tech Instant Music RDX-150
Phono Preamps & Vinyl Tools:
Behringer U-PHONO UFO202
ART USB Phono Plus (Early Iterations)
TC-772 USB Phono Preamp

### 🏗️ Architecture Overview

```mermaid
flowchart TD
    DAW["<b>Digital Audio Workstation</b><br/>(Ableton Live 11/12, VoiceMeeter, Waveform 13)"]
    
    DAW -->|"ASIO 2.3 ABI (Planar Float32)"| PUMP

    subgraph COM ["Boffin FX UM2-Link V2 (In-Process COM Server)"]
        direction TB
        PUMP["<b>AudioPumpThread</b> (Priority: Time-Critical)<br/>• Monotonic Hardware Timestamps (QPC)<br/>• Strict Block Pacing (m_bufferSize: 64–1024 frames)<br/>• SIMD Float32 ↔ Int16 Conversion<br/>• 1 LSB TPDF Dithering<br/>• SIMD FTZ & DAZ Enabled"]
        
        CAP_RING["<b>Capture Ring Buffer</b> (64k Frames)<br/>• Packed StereoFrame Type<br/>• Non-Dropping SPSC FIFO"]
        REND_RING["<b>Render Ring Buffer</b> (64k Frames)<br/>• Packed StereoFrame Type<br/>• 2048-Frame Headroom Cushion"]
        
        CAP_RING -->|"(Read Real Frames)"| PUMP
        PUMP -->|"(Write Full Block)"| REND_RING
    end

    WASAPI_IN["<b>WASAPI Capture Endpoint</b><br/>• Exclusive Event-Driven Mode<br/>• RAW Mode (APO Bypass Active)"]
    WASAPI_OUT["<b>WASAPI Render Endpoint</b><br/>• Exclusive Event-Driven Mode<br/>• MMCSS Pro Audio Active"]

    WASAPI_IN --> CAP_RING
    REND_RING --> WASAPI_OUT

    HW["<b>Texas Instruments PCM2902 USB Audio CODEC</b><br/>(VID: 08BB, PID: 2902)"]

    HW -->|"(1 ms Isochronous Packets)"| WASAPI_IN
    WASAPI_OUT -->|"(actualBufferFrames)"| HW

    classDef default fill:#111827,stroke:#374151,stroke-width:1px,color:#f9fafb;
    classDef com fill:#0b1320,stroke:#06b6d4,stroke-width:2px,color:#f9fafb;
    classDef hw fill:#1f2937,stroke:#10b981,stroke-width:2px,color:#f9fafb;
    class COM com;
    class HW hw;
```

🛠️ Building from Source
Prerequisites
Operating System: Windows 10 or 11 (64-bit)
Compiler: Microsoft Visual Studio 2022 (v143 toolset) with Desktop development with C++ workload
SDK: Windows 10/11 SDK (10.0.19041.0 or newer)
Build Generator: CMake 3.20 or newer
Build Commands
Open Developer PowerShell for VS 2022 or x64 Native Tools Command Prompt:

# 1. Clone repository
git clone https://github.com/YourUsername/UM2-Link.git
cd UM2-Link

# 2. Build 64-bit Target (for Ableton Live, Reaper, Studio One, FL Studio 64-bit)
cmake -B build -G "Visual Studio 17 2022" -A x64
cmake --build build --config Release

# 3. Build 32-bit Target (for VB-Audio VoiceMeeter Standard/Pro)
cmake -B build32 -G "Visual Studio 17 2022" -A Win32
cmake --build build32 --config Release
📦 Registration & Installation
Because UM2-Link is a COM-based in-process server, the DLLs must be registered with Windows.

Open PowerShell as Administrator:

# Register 64-bit Driver:
regsvr32 "C:\path\to\UM2-Link\build\Release\UM2LinkASIO64.dll"

# Register 32-bit Driver (via SysWOW64):
C:\Windows\SysWOW64\regsvr32.exe "C:\path\to\UM2-Link\build32\Release\UM2LinkASIO32.dll"
(To unregister at any time, run the same commands with the /u flag).

🚀 DAW Setup & Optimization Guide
1. Ableton Live (10 / 11 / 12)
Navigate to Options → Preferences → Audio.
Set Driver Type to ASIO.
Set Audio Device to Boffin FX UM2-Link.
Click Hardware Setup to open the custom buffer selector dialog:
Tracking Guitar / Vocals: Choose 128 samples (~6.0 ms) or 256 samples (~11.3 ms).
Heavy Mixing / High Plugin Count: Choose 512 samples.
Click Apply. The engine will re-negotiate the quantum without requiring a DAW restart.
2. VB-Audio VoiceMeeter
Set Output A1 to ASIO: Boffin FX UM2-Link.
Go to Menu → System settings / Options...:
Verify Preferred Main SampleRate is set to 48000 Hz (matching your interface).
Ensure "Auto restart audio engine (all devices)" is DISABLED (prevents exclusive-mode watchdog cascades).
3. Windows Audio Levels (Crucial)
Press Win + R, type mmsys.cpl, and press Enter (Classic Sound Control Panel).
Go to the Recording tab → Double-click Microphone (USB Audio Device) → Levels.
Set the level slider to 0.0 dB (unity gain, typically ~75–80%) to prevent Windows from applying digital pre-gain clipping.
4. Physical Hardware Checklist (Behringer UM2)
Direct Monitor Button: Ensure the DIRECT MONITOR button on the front of the interface is OFF (out) when monitoring through DAW amp simulators. Leaving it pressed creates comb-filtered phase cancellations.
Tube Amps (Ground Loops): If connecting the UM2's unbalanced RCA outputs to a high-gain tube guitar amplifier (e.g., Fender Super Champ XD), insert an inexpensive line-level audio ground loop isolator to eliminate 1 kHz USB power rail ripple.
🔧 Solved Technical Post-Mortem
Symptom / Defect	Root Cause	Resolution
Heap Corruption (0xC0000374)	AudioPumpThread was accessing vectors while disposeBuffers() cleared them on the host thread.	Pre-allocated fixed 1024-frame memory pools at startup; added atomic re-entrancy guards.
Crash in _unloaded.dll (0xC0000005)	Host called FreeLibrary() while background worker threads were still active in memory.	Pinned the DLL into process memory via GET_MODULE_HANDLE_EX_FLAG_PIN.
VoiceMeeter Disconnect Loop	OnDeviceRemoved fired on unrelated system audio changes (e.g., virtual cable polls).	Filtered removal notifications strictly against the physical UM2 device endpoint ID string.
Tailing Whistle Buzz on Low E (82.4 Hz)	Mid-stream zero-padding sliced the continuous 582-sample waveform cycle.	Enforced a strict full-quantum read invariant; eliminated all mid-stream zero-filling.
Metallic Chime on Guitar Decay	Correlated 16-bit truncation distortion on decaying signals.	Implemented 1 LSB Triangular Probability Density Function (TPDF) dithering via 32-bit XorShift.
Transient Ringing / Echo Tail	1st-order IIR DC blocker filter oscillated; SPSC queue race condition rewound read indices.	Removed redundant IIR filter; strictly isolated Write() and Read() index manipulation.

## 📄 License & Distribution Terms

The Boffin FX UM2-Link driver is distributed as free-of-charge software (**Freeware**) licensed under the terms of the **Boffin FX End User License Agreement (EULA)**.

* **Usage:** Licensed solely for personal, internal, or commercial audio production hardware interfacing.
* **Redistribution:** Standalone redistribution, reselling, renting, leasing, sublicensing, or hosting on third-party download portals without express written permission is strictly prohibited.
* **Source & Reverse Engineering:** Reverse engineering, decompilation, derivation of source code, and creation of unauthorized derivative works are strictly restricted under Section 3 of the Agreement.
* **Warranty:** Provided strictly "AS IS" and without warranty of any kind. Use is at your sole risk.

Read the full, binding agreement in [LICENSE.txt](LICENSE.txt).

---

### Third-Party Trademarks & Legal Disclaimers
* **ASIO** is a registered trademark and software of **Steinberg Media Technologies GmbH**.
* **Behringer**, **U-Phoria**, and **U-Control** are registered trademarks of **Music Tribe Global Brands Ltd.**
* **Texas Instruments** is a registered trademark of **Texas Instruments Inc.**
* Reference to third-party hardware, protocol names, or digital audio workstations is solely for technical compatibility identification and does not imply endorsement, affiliation, or sponsorship by respective trademark holders.

* Technical Specification & Architecture Overview
Engineering reference for the Boffin FX UM2-Link user-mode ASIO driver wrapper targeting Texas Instruments PCM2902 class-compliant USB audio interfaces.

6.0 ms
Min Roundtrip Latency
~45%
Latency Cut vs. ASIO4ALL
65,536
Frame SPSC Capacity
1 LSB
TPDF Linearized Dither
1. Executive Summary & Design Philosophy
The Boffin FX UM2-Link V2 is a dedicated, self-contained, 64-bit and 32-bit user-mode ASIO 2.3 driver wrapper written from scratch in modern C++20. It bridges incoming Digital Audio Workstation (DAW) audio requests directly to the Windows Audio Session API (WASAPI in Exclusive Event-Driven Mode).

Standard Windows audio drivers route budget USB 1.0 class-compliant audio interfaces through consumer-grade middleware: the Windows Audio Service (audiosrv), the Multimedia Audio Engine (audiodg.exe), and automatic Audio Processing Objects (APOs). This introduces 30–60ms buffer delays, destructive high-frequency noise suppression, and crashes in modern 64-bit production environments.

Design Objective: Provide bit-perfect, uncolored, ultra-low-latency studio tracking (5ms – 12ms round-trip) without requiring third-party wrappers like ASIO4ALL or commercial proprietary driver stacks.
2. Hardware Architecture & Constraints
The UM2-Link driver is optimized specifically for the Texas Instruments PCM2902 / PCM2902C and compatible Coolaudio V2902 CODEC chipsets (Vendor ID: 0x08BB, Product ID: 0x2902).

Hardware Metric	Hardware Constraint	Driver Implementation Resolution
Native Word Depth	Fixed 16-bit Linear PCM (LPCM)	Floating-point conversion with 1 LSB TPDF dithering and symmetrical saturation clamping.
Sample Rates	Fixed at 44.1 kHz or 48.0 kHz	Rejects non-hardware rates (88.2k, 96k, 192k) with ASE_NoClock to avoid hardware PLL unlock.
Endpoint Architecture	USB 1.1 Full-Speed Isochronous (1 ms)	Transfers arrive in 48-sample frames (at 48 kHz). Decoupled via elastic ring buffers.
Clock Synchronization	Independent ADC & DAC USB endpoints	Elastic lock-free circular FIFO absorbs USB isochronous clock drift without waveform step-chopping.
3. System Architecture & Deep Dive
3.1 Subsystem A: In-Process COM Server & Registry Topology
The driver implements Steinberg’s IASIO interface as a native In-Process COM Server. To support both modern 64-bit DAWs and 32-bit hosts (like standard VoiceMeeter), dual compilation targets are maintained:

UM2LinkASIO64.dll: 64-bit native binary for Ableton Live 12, Waveform, Reaper x64.
UM2LinkASIO32.dll: 32-bit binary targeting WoW64 subsystems and VoiceMeeter 32-bit.
To eliminate _unloaded.dll access violation crashes (0xC0000005) during driver teardown, the binary locks itself into the host address space on attachment:

Copy// In DllMain.cpp during DLL_PROCESS_ATTACH:
HMODULE hPinned = NULL;
GetModuleHandleExW(
    GET_MODULE_HANDLE_EX_FLAG_FROM_ADDRESS | GET_MODULE_HANDLE_EX_FLAG_PIN,
    reinterpret_cast<LPCWSTR>(hModule),
    &hPinned
);
3.2 Subsystem B: WASAPI Exclusive Mode & True RAW Processing
Windows 10/11 classifies USB audio inputs as communications microphones and silently applies spectral-subtraction noise gates. This causes high-frequency fluttering (“musical noise”) on decaying notes.

UM2-Link V2 queries IAudioClient2 and sets AUDCLNT_STREAMOPTIONS_RAW to permanently sever Windows APOs:

CopyIAudioClient2* pClient2 = nullptr;
if (SUCCEEDED(pClient->QueryInterface(__uuidof(IAudioClient2), reinterpret_cast<void**>(&pClient2))) && pClient2) {
    AudioClientProperties props = {};
    props.cbSize = sizeof(AudioClientProperties);
    props.bIsOffload = FALSE;
    props.eCategory = AudioCategory_Media;
    props.Options = AUDCLNT_STREAMOPTIONS_RAW; // Completely severs Windows APOs
    pClient2->SetClientProperties(&props);
    pClient2->Release();
}
3.3 Subsystem C: Lock-Free Elastic Dual-Ring Buffer Architecture
To guarantee that audio transfers never shift Left and Right channel alignment, the circular buffer operates on packed 4-byte StereoFrame types:

Copy#pragma pack(push, 2)
struct StereoFrame {
    int16_t left;
    int16_t right;
};
#pragma pack(pop)
static_assert(sizeof(StereoFrame) == 4, "StereoFrame must be exactly 4 bytes");
Strict SPSC Contract: The producer method (Write) modifies exclusively m_writeIndex, and the consumer method (Read) modifies exclusively m_readIndex. SPSC race conditions that cause audio repeat/echo tails are mathematically impossible.
3.4 Subsystem D: Real-Time Host Pump & Zero-Discontinuity Processing
The AudioPumpThread runs with real-time MMCSS priority and configures CPU floating-point control registers to prevent denormal underflow stalls:

Copy_MM_SET_FLUSH_ZERO_MODE(_MM_FLUSH_ZERO_ON);
_MM_SET_DENORMALS_ZERO_MODE(_MM_DENORMALS_ZERO_ON);

DWORD taskIndex = 0;
HANDLE hMmcss = AvSetMmThreadCharacteristicsW(L"Pro Audio", &taskIndex);
SetThreadPriority(GetCurrentThread(), THREAD_PRIORITY_TIME_CRITICAL);
3.5 Subsystem E: 1 LSB TPDF Dithering
Quantizing floating-point audio to 16-bit generates correlated distortion that sounds like a metallic ringing chime on decaying guitar notes. UM2-Link V2 linearizes the DAC stage with uncorrelated 1 LSB Triangular Probability Density Function (TPDF) dither:

Copy// Fast 32-bit Xorshift PRNG for 1 LSB TPDF Dither
inline float GenerateTpdfDither(uint32_t& state) noexcept {
    if (state == 0) [[unlikely]] state = 0x12345678;
    state ^= state << 13;
    state ^= state >> 17;
    state ^= state << 5;
    float r1 = (static_cast<float>(state) * (1.0f / 4294967296.0f)) - 0.5f;

    state ^= state << 13;
    state ^= state >> 17;
    state ^= state << 5;
    float r2 = (static_cast<float>(state) * (1.0f / 4294967296.0f)) - 0.5f;

    return r1 + r2; // TPDF range: [-1.0f LSB, +1.0f LSB]
}
3.6 Subsystem F: Dynamic Control Panel & Registry Configuration
DAWs expect ASIO hardware settings to be configured through a driver-provided dialog. UM2-Link V2 provides an in-memory Win32 dialog created on the fly with zero resource dependencies:

Supports buffer sizes: 64, 128, 256, 512, and 1024 samples.
Persists settings to HKEY_CURRENT_USER\Software\BoffinFX\UM2-Link\BufferSize, avoiding administrative permission blocks.
Dispatches kAsioResetRequest (3) asynchronously after dialog exit, preventing DAW UI deadlocks.
4. Solved Engineering Defects & Post-Mortem
Defect	Root Cause	Engineering Fix
Heap Corruption (0xC0000374)	AudioPumpThread accessed heap vectors while disposeBuffers() cleared them on the DAW thread.	Pre-allocated fixed 1024-frame buffer pools at startup; added atomic re-entrancy guards.
Crash in _unloaded.dll	DAW called FreeLibrary() while worker threads were still finishing in memory.	Pinned the DLL into memory using GET_MODULE_HANDLE_EX_FLAG_PIN.
VoiceMeeter Disconnect Loop	OnDeviceRemoved caught every audio event across the entire OS (virtual cables, Bluetooth).	Filtered removal notifications strictly against the physical UM2 device endpoint ID string.
Tailing Whistle Buzz on Low E (82.4 Hz)	Mid-stream zero-padding sliced the continuous 582-sample wave cycle.	Enforced a strict full-quantum read invariant; eliminated all mid-stream zero-filling.
Transient Ringing / Echo Tail	1st-order IIR DC blocker filter oscillated; SPSC queue race condition rewound read indices.	Removed the redundant IIR filter; strictly isolated Write() and Read() index manipulation.
5. Latency Benchmarks & Empirical Performance
Benchmarked in Ableton Live 12 on Windows 11 x64 (Build 26200) at 48,000 Hz:

Driver Solution	Buffer Size	Input Latency	Output Latency	Overall Roundtrip
ASIO4ALL v2 (WDM-KS)	512 samples	21.0 ms	21.0 ms	41.9 ms
Boffin FX UM2-Link V2	512 samples	11.3 ms	12.0 ms	23.3 ms (~45% lower)
Boffin FX UM2-Link V2	256 samples	5.3 ms	6.0 ms	11.3 ms
Boffin FX UM2-Link V2	128 samples	2.7 ms	3.3 ms	6.0 ms (Live Tracking)
6. Repository Layout
UM2-Link Ver2/
├── CMakeLists.txt              # Build configuration for x64 and Win32 /MT static linkage
├── build/                      # 64-bit build artifacts (UM2LinkASIO64.dll)
├── build32/                    # 32-bit build artifacts (UM2LinkASIO32.dll)
└── src/
    ├── DllMain.cpp             # COM server registration, DLL pinning, lifetime locks
    ├── Driver.h                # IASIO interface definition, types, driver state
    ├── Driver.cpp              # ASIO logic, control panel dialog, AudioPumpThread
    ├── RingBuffer.h            # 64k-frame lock-free SPSC FIFO, packed StereoFrame
    ├── SampleConverter.h       # Int16 <-> Float32, 1 LSB TPDF dither, lrintf clamp
    ├── UM2Link.def             # COM function export definitions
    ├── WasapiEngine.h          # WASAPI Exclusive endpoint and thread declarations
    └── WasapiEngine.cpp        # Endpoint enumeration, RAW mode, hardware sync events
7. Build & Registration Guide
Run within an Administrator PowerShell console:

Build 32-Bit Target (for VoiceMeeter Standard / Pro)
Copycmake -B build32 -G "Visual Studio 17 2022" -A Win32
cmake --build build32 --config Release
C:\Windows\SysWOW64\regsvr32.exe "build32\Release\UM2LinkASIO32.dll"
Build 64-Bit Target (for Ableton Live, Reaper, Waveform)
Copycmake -B build -G "Visual Studio 17 2022" -A x64
cmake --build build --config Release
regsvr32 "build\Release\UM2LinkASIO64.dll"
8. Verification & Operational Guidelines
Ableton Live Configuration: Set Driver Type to ASIO and Audio Device to Boffin FX UM2-Link. Click Hardware Setup to select 128 or 256 samples for tracking.
VoiceMeeter Configuration: Set Output A1 to ASIO: Boffin FX UM2-Link. Under System Settings, verify Preferred Main SampleRate is 48000 Hz and ensure “Auto restart audio engine (all devices)” is DISABLED.
Windows Mixer Gain: Run mmsys.cpl, open Recording properties for the USB Audio Device, and set the Levels slider to 0.0 dB (unity gain, ~75–80%) to avoid digital pre-gain clipping.
Physical Hardware Checks: Keep the DIRECT MONITOR button on the Behringer UM2 OFF (out) to prevent comb-filtering phase cancellation. If tracking into high-gain tube amps, use an audio isolation transformer on the unbalanced RCA lines to eliminate physical USB 5V ground loop hum.
