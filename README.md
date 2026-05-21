# diy-oscilloscope
500kHz bandwidth oscilloscope on STM32F103 Blue Pill with 12-bit ADC, DMA capture, trigger logic, FFT mode, and ST7789 TFT display. Written in embedded C.
# 📡 DIY Digital Oscilloscope

> A 500kHz bandwidth oscilloscope built on STM32F103 Blue Pill — 12-bit ADC with DMA capture, hardware trigger logic, three display modes (time domain / FFT / XY), and a custom analog protection front end. Written entirely in embedded C.

## 🚧 Status
`🔧 In Progress` — Target: July 2026

---

## Demo
> *Oscilloscope screenshots and waveform captures coming on completion*

---

## Why build an oscilloscope?

Because using one tells you nothing. Building one teaches you everything — ADC theory, DMA transfers, trigger algorithms, Fourier analysis, and display rendering. This one can actually measure signals you'd encounter in a real lab.

**Comparison to Arduino-based alternatives:**

| | Arduino Uno | This build |
|--|-------------|------------|
| ADC resolution | 10-bit | 12-bit |
| Max sample rate | 9.6 kHz | 1 MHz |
| Max signal frequency | ~4.8 kHz | ~500 kHz |
| Trigger logic | Software polling | Hardware-assisted |
| FFT | No (too slow) | Yes (ARM CMSIS-DSP) |

---

## System Architecture

```
BNC Probe Input
      │
      ▼
┌─────────────────────────────┐
│     Analog Front End        │
│                             │
│  1MΩ input impedance        │
│  1N4148 clamp diodes        │  ← Protects STM32 from overvoltage
│  10kΩ + 1kΩ voltage divider │  ← ÷10 attenuation (1× / 10× switch)
│  1.65V bias (mid-rail)      │  ← Centres AC signal in ADC range
│  MCP6002 op-amp buffer      │  ← High-Z in, Low-Z out
└─────────────────────────────┘
      │
      ▼  0 – 3.3V conditioned signal
      │
STM32F103C8T3 "Blue Pill"
  ├── 12-bit ADC on PA0
  ├── DMA fills 1024-sample ring buffer
  ├── Trigger engine scans for edge at threshold
  ├── CMSIS-DSP arm_cfft_f32() for FFT mode
  └── SPI → ST7789 2.0" TFT display (240×320)

EC11 Rotary Encoders ×3
  ├── Encoder 1: Timebase (µs/div)
  ├── Encoder 2: Volt/div scale
  └── Encoder 3: Trigger level

Tactile buttons ×2
  ├── Mode: Time domain / FFT / XY
  └── Hold / Run toggle
```

---

## Components

| Component | Spec | Purpose |
|-----------|------|---------|
| STM32F103C8T3 Blue Pill | 72MHz, 12-bit ADC, 1MSPS | Main processor + ADC |
| ST-Link V2 programmer | USB dongle | Flashing STM32 firmware |
| ST7789 TFT display | 2.0", 240×320, SPI | Waveform display |
| MCP6002 op-amp | Dual rail-to-rail, 3.3V | Analog front end buffer |
| BNC chassis connector | Through-hole PCB mount | Probe input |
| 1N4148 diode ×6 | Fast switching | Input overvoltage clamp |
| SPDT toggle switch | Panel mount | 1× / 10× attenuation |
| EC11 rotary encoder ×3 | With push button | Timebase / Volt-div / Trigger |
| AMS1117-3.3 LDO | TO-220 package | 5V → 3.3V regulation |
| BNC oscilloscope probe | 100MHz rated | Measurement probe |
| ABS project box | ~12×8×4cm | Enclosure |

**Approximate build cost: ₹1,500 – ₹2,200**

---

## Display Modes

### 1. Time Domain
Classic voltage vs. time plot. Trigger engine locks waveform stable on screen by finding a rising/falling edge at the threshold and slicing exactly 240 samples from that point.

### 2. FFT / Frequency Domain
1024-sample buffer passed to `arm_cfft_f32()` from ARM's CMSIS-DSP library. Renders as bar chart. Shows harmonic content, noise floor, distortion.

### 3. XY / Lissajous
Two capture buffers plotted against each other. Shows phase relationships between signals.

---

## The 1.65V Bias Trick

The STM32 ADC only reads 0–3.3V. Real AC signals swing positive and negative. Solution: bias the entire signal up by 1.65V (half of 3.3V) before the ADC, so both halves of the waveform fit in range. Software subtracts the offset back when plotting. Standard oscilloscope technique.

---

## Build Timeline

| Week | Milestone |
|------|-----------|
| 1 | STM32 + TFT display working — draw test sine wave from lookup table |
| 2 | Analog front end on perfboard — capture real ADC samples — display raw voltage |
| 3 | Trigger algorithm — waveform locks stable on screen |
| 4 | FFT mode, measurement overlays (Vpp, freq, period), enclosure, calibration |

---

## Firmware

**IDE:** STM32CubeIDE (free from STMicroelectronics)
**Language:** Embedded C using STM32 HAL library
**DSP:** ARM CMSIS-DSP (bundled with CubeIDE)

Key firmware modules:
- `adc_capture.c` — DMA-driven ring buffer, configurable sample rate
- `trigger.c` — Edge detection on ring buffer with configurable threshold
- `display.c` — ST7789 SPI driver, polyline renderer, text overlays
- `fft.c` — CMSIS-DSP wrapper, bar chart renderer
- `encoder.c` — Interrupt-driven EC11 handler

---

## Skills Demonstrated

- STM32 bare-metal embedded C programming
- DMA-driven ADC capture (no CPU overhead during sampling)
- Analog front end design (buffer, clamp, bias, attenuation)
- Digital trigger algorithm implementation
- FFT signal analysis using ARM CMSIS-DSP
- SPI display protocol and pixel-level rendering
- Rotary encoder interrupt handling

---

## License
MIT
