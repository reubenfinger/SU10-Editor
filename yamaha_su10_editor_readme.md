# Yamaha SU10 Sampler Web MIDI & SDS Editor

An interactive, browser-based Web MIDI front-end editor, waveform trimmer, and Sample Dump Standard (SDS) converter built for the **Yamaha SU10 Portable Stereo Sampler**. 

Designed with a retro '90s hardware aesthetic, this tool provides visual control over all 48 pad locations, dynamic loop point editing with zero-crossing snapping, live ADPCM sample conversion, and real-time Web MIDI hardware interaction.

---

## 🌟 Key Features

* **Complete 48-Pad Architecture**:
  * Visual access to Banks A, B, C, and D (12 tactile pads per bank).
  * Direct parameter control for Cutoff, Resonance, Sample Pitch, and Pad Volume.
* **5 Native SU10 Loop Modes**:
  * Full real-time support and auditioning for `1Shot`, `WLoop` (Waveform Loop), `PLoop` (Pitch Loop), `RShot` (Reverse One-Shot), and `RLoop` (Reverse Loop).
* **Sample Import & Downsampler**:
  * Drag-and-drop support for `WAV`, `MP3`, `FLAC`, and `OGG` formats.
  * Real-time 4-bit ADPCM compression engine simulating Yamaha's 4:1 hardware compression ratio.
  * Quality selection matching exact hardware rates:
    * **High**: 44.1 kHz (~17.8s max stereo)
    * **Standard**: 22.05 kHz (~35.6s max)
    * **Long**: 14.7 kHz (~53.4s max)
    * **Extra Long**: 11.64 kHz (~67.4s max)
* **Waveform Visualizer & Loop Snapping**:
  * Visual waveform trimming with start and end sliders.
  * **Zero-Crossing Snap**: Automatic alignment of loop boundaries to zero-voltage points to eliminate pops and clicks.
  * Isolated 8-bit Lo-Fi audition engine with real-time looping preview.
* **Memory & Flash Guard**:
  * Real-time visual tracking of hardware Flash RAM usage strictly enforcing the **384 KB (393,216 Bytes)** non-volatile memory limit.
* **MIDI Sample Dump Standard (SDS) & SysEx Traffic**:
  * SDS sample request capability (`F0 7E ...`).
  * Real-time Yamaha parameter SysEx transmission (`F0 44 14 ...`).
  * Incoming MIDI Note-On routing for live pad selection and previewing.
  * Real-time SysEx traffic monitor.
  * Safe Bulk Backup & Restore procedures with hardware reset warning dialogs.

---

## 🚀 Quick Start Guide

### Prerequisites
* A Web MIDI compatible web browser (**Google Chrome**, **Microsoft Edge**, **Opera**, or **Brave**).
* A USB-to-MIDI interface connected to your computer and the Yamaha SU10's MIDI IN/OUT ports.

### Running the Application
1. Download or clone `index.html`.
2. Double-click or open `index.html` in any supported web browser (no local web server or build process required).
3. Connect your MIDI Interface ports in the top header section under **MIDI IN** and **MIDI OUT**.

---

## 🎹 Keyboard Shortcuts

When not typing in numeric inputs, you can trigger pads 1 through 12 using your computer keyboard:

| Pad # | Key | Pad # | Key |
| :--- | :--- | :--- | :--- |
| **Pad 1** | `1` | **Pad 7** | `7` |
| **Pad 2** | `2` | **Pad 8** | `8` |
| **Pad 3** | `3` | **Pad 9** | `9` |
| **Pad 4** | `4` | **Pad 10**| `0` |
| **Pad 5** | `5` | **Pad 11**| `-` |
| **Pad 6** | `6` | **Pad 12**| `=` |

---

## 📡 MIDI & SysEx Specification

### Parameter Control Messages
* **SysEx Header**: `F0 44 14 00 [Address] [Value] F7`
  * **Cutoff**: Address `0x10` (`0–127`)
  * **Resonance**: Address `0x11` (`0–127`)
  * **Pitch**: Address `0x12` (`-12 to +12`)
  * **Volume**: Address `0x13` (`0–127`)
  * **Loop Mode**: Address `0x15` (`0=1Shot, 1=WLoop, 2=PLoop, 3=RShot, 4=RLoop`)

### Audio Transfer & Bulk Dumps
* **Sample Transfer**: MIDI Sample Dump Standard (SDS) (`F0 7E ...`)
* **Bulk Backup Request**: `F0 44 14 00 05 F7`

---

## 📄 License

Distributed under the MIT License. Open-source and free for hardware enthusiasts and music producers alike.