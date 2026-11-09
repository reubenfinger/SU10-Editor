# SU10 Editor

**▶ Use it online:** https://reubenfinger.github.io/SU10-Editor/su10_editor.html  
**⬇ Download:** [su10_editor.html](https://github.com/reubenfinger/SU10-Editor/releases/latest/download/su10_editor.html) (one file, runs offline in Chrome, Edge or Opera)

`su10_editor.html` is a single self-contained page: no server, no internet, no install. Open it in Chrome, Edge or Opera, click **Connect MIDI**, and allow System Exclusive access.

Everything it sends or expects comes from the *SU10 Owner's Manual* (MIDI implementation chart, pp. 45, 56–63) and from [vitaflo/susend](https://github.com/vitaflo/susend) for bulk-restore timing.

## What the SU10 actually supports over MIDI

| Direction | Message | Used for |
|---|---|---|
| Receives | Note On / Off (velocity 1–127) | Playing pads (note table, default A1 = C1/36 … D12 = B4/83) |
| Receives | CC 7 | Channel volume |
| Receives | Program Change 0–1 | Program 2 (data 01h) + velocity ≥ 110 → plays reversed |
| Receives | SU10 bulk dump (Yamaha SysEx) | Full restore of samples + songs |
| Sends | Note On (9n v=127, off = 9n v=0) | Pad presses |
| Sends | CC 0–120 or Pitch Bend | Ribbon (UTILITY › MIDI › RbnNum) |
| Sends | MIDI Sample Dump Standard | Single samples, sent from StPnt to EdPnt at the rate they were *recorded* at. IDs run A1 = 0 … D12 = 47 (the manual says 1–48, but a real unit sends 0–47). **Send only: the SU10 can't receive SDS.** |
| Sends | SU10 bulk dump | Backup, started from the unit (Dump Send?) |

The manual documents no messages for editing parameters (loop mode, volume, points, filter…) over MIDI. Those can only be set on the unit.

## Sections

- **Panel**: a replica of the front panel.
  - Pads send the notes in your note table. They use the velocity slider, and send a note off when you let go.
  - BANK cycles A–D. HOLD keeps notes on after you let go.
  - REV sends Program 2 and velocity 127, so the SU10 plays the sample reversed.
  - Notes coming in light the pads but are never echoed back, so there's no MIDI loop.
  - The ribbon window shows the SU10's ribbon CC or pitch bend.
  - Buttons that only work on the unit itself say so when clicked.
- **Pads & Note Table**: your 48-pad sheet.
  - **Learn**: click it, then press the pad on the SU10 and its note is picked up. **Learn sequence** steps through the pads one after another.
  - The table can be reset to the default, cleared, or exported and imported as JSON.
  - Name, LpMode, SdMode, PadVol, Mute group and Memo are notes for you. They decide how captured samples play when you audition them in the editor.
  - **Names belong to samples, not pads.** The SU10 doesn't store names, so the editor remembers each name against the sample's audio. Reopen a dump and the samples you named get their names back. Pads whose sample is new, deleted or empty start unnamed, so names from earlier dumps never carry over. A name follows its sample when you swap or move pads, and after an effect render once you save or send the dump.
- **Bulk Backup / Restore**
  - Backup records the SU10's Dump Send (all four segments) and saves it as a `.syx`. It starts by itself if bulk data arrives.
  - Restore uses susend's timing: header, 10 s wait, 128 ms between blocks, 1 s after each 259-byte section footer. The timing runs in a Web Worker, so a background tab can't stretch a gap past the SU10's 30 s time-out.
- **Sample Dump (MSDS)**: receives SDS dumps.
  - It replies with ACK, or NAK if a checksum fails.
  - It joins stereo halves and shows the exact sample rate.
  - Captured samples can be auditioned, saved as WAV, sent to Prep, or used to set a grade's rate (**Use as rate for…**).
- **Memory meter**: once a dump is open, a bar above the panel and another above the sample editor show used and free memory. A white box marks where the selected sample sits in memory, and "unsaved" appears when there are unsaved edits. Click the top bar to jump to SU10 Memory.
- **Selection follows everything**: clicking a sample in the SU10 Memory grid switches the panel to its bank, so computer keys play that bank. Hitting a computer key, clicking a panel pad, or pressing a learned pad on the SU10 itself opens that sample in the editor.
- **Always a memory open**: with no dump loaded, the editor starts with a **blank SU10 memory** built from the template (48 empty pads, plus the template's songs and system settings). Dropping on pads, Sample Prep's Add, chopping and saving all work straight away. Opening a real dump replaces it, and asks first if you've added anything.
- **Drag and drop** reads dropped files the same simple way Sample Prep does. The browser's folder API is used only when a folder is dropped, and if the browser blocks it on a local `file://` page, the editor falls back to the files and suggests **Add folder…**.
- **SU10 Memory**: opens a bulk dump and shows all 48 samples with their real settings: bank/pad, Note#, LpMode, SdMode, PadVol, Mute group, start/loop/end points, length, grade and pitch.
  - **Pads**: drag to swap, Alt-drag to copy. As on the unit, each Note# stays with its pad position.
  - **Points**: drag the markers or type values. Zoomed views show the loop seam. Tools: snap to zero crossings, find best loop end/start, and the TEMPO readout the SU10 itself shows.
  - **Effects** (baked in and re-encoded): normalise, gain, reverse, fades, loop crossfade, low/high-pass, pitch resample, bit-crush, crop (which frees memory).
  - **New from folder…**: with no dump open, choose a folder (or drop one onto the empty grid) to build a new SU10 memory. Sounds fill A1 onward in natural file-name order (1, 2 … 10), up to 48. The dialog offers strip silence, normalise, skip-or-trim if it doesn't all fit, and the grade table.
    - Songs and system settings come from a known-good template: the last real dump you opened, or a built-in copy of your 30 Oct backup. Delete unwanted songs on the unit after restoring.
    - Unused pads are written empty, the same way the SU10 writes them (see the dump format below).
  - **Airwindows effects**: ports of Chris Johnson's **TapeHack2** (tape saturation: Input / Output / Dry-Wet), **Pockey2** (µ-law bit and frequency crusher: DeFreq / DeRez / Dry-Wet) and **Tube2** (tube drive and fuzz: Input / Tube), from github.com/airwindows/airwindows under the MIT licence (notice kept in the source).
    - They run at the sample's SU10 rate, and each plugin's own sample-rate scaling is kept.
    - They were checked against the original C++, compiled and run on the same signals: identical to within 1–2 float LSB (about 1e-7). The 32-bit float dither (about −190 dB) is left out.
    - **Preview** plays the result without changing anything. **Render** bakes it in and re-encodes the sample, and **Undo** steps back through renders. Settings are remembered, and a **defaults** button restores each plugin's original defaults.
  - **Chop mode (✂)**: slices one sound across **2/4/8/12/16/32/48 sequential pads** (12 = one bank, 48 = the whole SU10). Open it with **✂ Chop…** on a pad, or with **Chop … into slices instead…** in the import dialog (single files, drops and folders, where each sound is chopped in turn with an option to reuse the same settings).
    - **Cuts**: at **transients** (energy rise against the previous 40 ms, with a sensitivity control and a minimum gap) or on an **even grid** for tempo loops. Cuts snap to zero crossings, just before the attack for transients. The dialog says how many hits it found.
    - **Interactive waveform**: click a slice to hear it, drag a cut line to move it (Shift snaps it to a zero crossing). Each slice shows the pad it will land on.
    - **Placement**: it finds the first run of N empty pads in a row (from the chosen pad, then from A1). Chopping a pad can free that pad so slice 1 takes its place. If there's no free run, it says which pads would be replaced. The memory needed and available is shown, and Chop is disabled if it won't fit.
    - **Play modes**: **Choke** (OShot, Trggr, one free mute group so each slice cuts off the last), **Choke + loop** (WLoop, Note, mute group), **Loop** (WLoop, Toggl), **One-shot** (no choke). SdMode can be overridden.
    - Chopping an existing pad copies its ADPCM bytes **exactly**, with the decoder state taken at each cut, so nothing is re-encoded. Ticking the 2 ms fades re-encodes instead. Slices of imported files are encoded at the unit's grade.
    - Pads without a Note# get the default (C1 + pad) if it's free. **Undo chop** in the summary puts every pad back.
  - **Strip silence**: trims audio below a threshold (default −48 dB) off the start and end, keeping a short tail (default 20 ms) and adding a 3 ms fade so the cut can't click. It frees the memory. Tick **crop to Start–End first** to crop and strip in one step. **Strip all one-shots** does every OShot/RShot/Trggr pad at once and skips looped pads, so loop timing isn't changed. Undo works per pad.
  - **Grade choice when importing**: dropping files or folders, or adding from Sample Prep, opens a dialog with all four grades. It also has options to strip silence from the new sounds (on by default), normalise, and skip or trim when they don't fit. It shows what the new sounds would take, the total memory, whether it fits, the audio bandwidth, and what the SU10 can't do at that grade (High: 2 voices and no effects; Standard: no filter or scale). It suggests the best grade that fits and warns about any pads it would replace.
    - Picking a grade other than the current one converts the whole dump, because the SU10 plays every sample at the one system grade.
    - If even the chosen grade doesn't fit, the last sound is trimmed.
  - **Convert dump grade…** re-encodes all samples at another grade so they keep their pitch. Use it to free memory (Long → Extra Long frees about 27 %) or to raise quality. It has a one-step **Undo grade conversion**. Each conversion adds a little more 4-bit grit, so convert from your original backup where you can.
  - **Adding sounds**: drop audio files or whole folders from Finder (or rows from **Open folder of sounds…**) onto any pad: the SU10 panel pads (current bank), the SU10 Memory grid, or a row of the Pads & Note Table. With no dump open, a drop starts a new memory from that pad.
    - When you drop several files, choose **empty pads only** (the default when the target pads are already used) or **consecutive pads**, which warns before replacing any.
    - Dropping on the grid background or the top memory bar, or using **Add folder to empty pads…**, fills only the empty pads and never replaces anything. You can also replace from a file or from Sample Prep. New audio is converted to the unit's grade (mono). The memory bar shows what's left.
  - **Output**: it rebuilds a complete, checksummed .syx, checks it by parsing it back, and sends it through the Bulk Restore routine. Unchanged samples keep their original bytes, and system settings and songs are kept as they are.

- **Sample Prep**: gets audio ready to record into the SU10 through LINE IN.
  - Trim (drag the handles; Shift for fine control), or click **Strip silence** to set the trim handles automatically. Choose mono mix / left only / stereo, fade, and normalise.
  - Preview at a grade and system pitch, with a proper anti-aliased resampler and optional approximate 4-bit ADPCM grit.
  - Shows a memory estimate against 384 KB.
  - "Play into SU10 LINE IN" adds a 0.5 s pre-roll for RecTrg.
  - **Add to SU10 Memory**: picks a pad in the open dump, shows the size at the unit's grade and the free memory, and warns before replacing an existing sample.

## SU10 bulk dump format (reverse-engineered from a real dump)

- **Messages**: `F0 43 00 7F 36 00 <count:2> <payload> <ck> F7`. The header's payload is `00 00 01 00 00`. After it come 1000 blocks: `<block# mod 128> <packed data>`, 480 bytes each, with 248 bytes in every 20th block.
- **Checksum** = `−(block# + sum of the block's unpacked bytes) & 7F`.
- **Packing**: every 20 blocks form one 8192-byte section of unit memory, packed 7→8 (an MSB byte, then 7 data bytes; bit 6 belongs to the first data byte).
- **Memory map**:

  | Location | Contents |
  |---|---|
  | `0000` | 48 sample records × 42 bytes |
  | `07E0` | System settings: grade code, pitch + 20, and more |
  | `07FC` | MIDI note → pad table (128 entries, FF = none) |
  | `2000` | Song memory |
  | `4000–63FFF` | Wave memory = unit addresses `20000–7FFFF` (384 KB). Samples are packed with no gaps. |

- **Record layout** (big-endian):

  | Offset | Field |
  |---|---|
  | 0 | Wave address (4 bytes) |
  | 4 | ADPCM predictor (2) + step index (2) |
  | 8 | Second copy of the predictor and index |
  | 12 | Length in bytes (4) |
  | 16 | Unknown (2) |
  | 18 | StPnt (4) |
  | 22 | LpPnt (4) |
  | 26 | EdPnt (4), in samples |
  | 30 | LpMode (0 OShot, 1 WLoop, 2 PLoop, 3 RShot, 4 RLoop) |
  | 31 | SdMode (0 Note, 1 Toggl, 2 Trggr) |
  | 32 | PadVol |
  | 33, 34 | Unknown (40h, 01h) |
  | 35 | AltGrp (shown as Mute group in the editor) |
  | 36 | Note# (FF = Off) |
  | 37 | Unknown |
  | 38 | Grade code |
  | 39 | Pitch + 20 |
  | 40 | Unknown (2) |

- **Audio**: 4-bit IMA-ADPCM, low nibble first, 2 samples per byte. The stored predictor and index are the decoder state at wave start. **Checked against the SU10's own MSDS output (C7 and D7): bit-exact**, to within 1 LSB of WAV rounding.
- **Empty pads**, confirmed from a dump made with D2 and D3 deleted on the unit: a **zero-length record**. The SU10 writes it either as all zeros (D2) or as address 0 and length 0 with the old parameters left in (D3). **An empty pad keeps its Note# in the note table.** The editor keeps the unit's own empty records exactly as they are, and writes all zeros for pads you delete.
- **What the SU10 does when it deletes a pad** (comparing Tonic 808 & TPhaze before and after deleting D2 and D3 on the unit):
  - The record is cleared (D2 to all zeros, D3 to length 0 with its old parameters kept).
  - The note table and songs are left as they are.
  - Wave memory is compacted: every later sample moves down, and its record address drops by exactly the 789 bytes freed.
  - Oddity: the moved **audio** didn't line up exactly with the new addresses. It sat 1 byte later than the records up to unit address 0x3E000 (an 8 KB flash boundary inside D7), with 24 scrambled bytes just before it, and 3 bytes earlier from there on. Because ADPCM resyncs, this may be barely audible, but it looks like a quirk of the unit's own compaction. The editor keeps such data exactly as dumped. Its own deletes and moves repack every sample in line with its record.
  - System bytes 7EDh/7EEh and 7FBh also changed during the delete (66 00 → 33 03, 3F → 6A). Their meaning is unknown, and they're preserved.
- **Free wave memory** is normally erased (FF). The unit can leave a stray byte after the last sample. The editor keeps those bytes whenever the samples end where the unit's did.
- **Aborted captures**: if a capture holds an aborted start followed by a full dump (for example, header + 40 blocks, then a full dump), the editor uses the last complete dump. That applies to both editing and restoring, so a restore never sends the broken part. A new header during capture also restarts the capture.
- **Round trip**: all four real dumps so far (the full 09:49 backup, Tonic 808 & TPhaze, Extra Long, and the one with empty pads) rebuild byte-identical.
- **Not yet confirmed**: stereo storage. High's rate is derived from Long rather than measured.
- **Can't be done**: a partial restore. Receiving a bulk dump clears the whole unit first, and the SU10 doesn't accept SDS. The editor works around this by always sending a complete image that already contains your changes.

## Notes

- Grade rates: the codes follow the manual's order (0 High, 1 Standard, 2 Long, 3 Extra Long), confirmed on hardware.
  - **Long = 20,050 Hz**, **Standard = 29,090 Hz**, **Extra Long = 14,550 Hz**. All three are exact, from the SU10's own MSDS output for samples recorded at each grade. By-ear checks against D7 came within 0.5–5.5 cents of these.
  - **High = 40,100 Hz** (2 × Long, not measured yet). Adding the manual's +10 % pitch gives 44.1 kHz, which checks out. A sample recorded at High and sent over MSDS would confirm it.
  - The grades come in two pairs: High ≈ 2 × Long and Standard ≈ 2 × Extra Long.
  - **MSDS only reports the recording rate.** It gives the grade × pitch the sample was recorded at, whatever grade the unit is on now. For example, C7 (recorded at Long +10 %) reports 22,055 Hz = 20,050 × 1.1. So MSDS can only measure Standard or Extra Long exactly from a sample recorded at that grade. The editor calibrates whichever grade a dumped sample was recorded at.
  - Changing the unit's grade just changes the playback pitch, and the editor's **System grade** selector does the same.
- Bulk dump contents are treated as opaque: the format isn't documented. The editor checks for the usual layout (1 header + 50 sections × 20 blocks).
- Before a restore, set **DevNum** to All (or to the dump's number) and unlock all banks and songs.
- Keyboard: `1`–`9` `0` `-` `=` pads · `B` bank · `H` hold · `R` rev.

## Licence

MIT, see [LICENSE](LICENSE). The Airwindows ports (TapeHack2, Pockey2, Tube2) are MIT too, © 2018 Chris Johnson, and their notice is kept in the source.

The SU10 Owner's Manual is Yamaha's and isn't included here. Page references above are to that manual.
