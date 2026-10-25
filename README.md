# Sample Lab v3

**Independent and unofficial.** Sample Lab v3 is an independent, unofficial tool. It is not affiliated with, endorsed by, or sponsored by Making Sound Machines or the Plinky project. Plinky is mentioned only to identify device compatibility.

Open `sample-lab-v3.html` in a modern browser by double-clicking it. It is a self-contained offline application: no installation, server, account, external dependency, or upload is needed. WAV is the most portable audio input; support for compressed audio depends on the browser.

## Prepare a sample

1. Back up your current Plinky files. Load a fresh `PRESETS.UF2` into the presets input.
2. Load a WAV or an existing `SAMPLE0.UF2` through `SAMPLE7.UF2`. A sample backup automatically selects its embedded slot. Matching presets restore its recorded length, slice starts, notes and playback mode.
3. Choose the destination. UF2 slot 0 corresponds to the first of eight sample slots.
4. For new audio, optionally trim, normalize, reverse or fade, then choose **Apply & reset slices**. This always processes the imported source; it does not repeatedly process the last result. Edits are not applied until you press Apply. Applying edits spaces all eight slices evenly.
5. Adjust slice start times and notes, or select a slice and click the waveform. Use the individual play buttons to audition slices. Playback previews the audio without the synth's granular engine or tuning.
6. Export `SAMPLEn.UF2` and updated `PRESETS.UF2` using the two separate buttons. The new presets file is based on the loaded backup and updates only the selected slot's latest metadata page.
7. Enter Plinky USB drive mode (hold the encoder while connecting USB), copy both exported files to its drive, wait for writes to finish, then press the encoder to restart.

Use a current presets backup: copying PRESETS.UF2 also restores every other preset, pattern and system setting from that backup. Keep both original backups until you have verified playback. Renaming a sample UF2 does not change its slot; select the slot inside the application before export.

The tool requires a valid existing metadata page for the destination to export presets. If the slot has none, record and save a sample in that slot on Plinky, then take a fresh presets backup. Without presets, a sample UF2 has no recorded-length information; the application displays its entire slot and cannot recover original slices.

## Mounted USB drive integration

Reload the updated HTML after downloading it. The new **Mounted Plinky drive** panel supports reading and writing an explicitly selected mounted volume using the browser File System Access API. The page shows a compatibility message when the folder picker is unavailable. Desktop Chrome and Edge support this API; availability can depend on browser context and policy. See [Chrome's File System Access documentation](https://developer.chrome.com/docs/capabilities/web-apis/file-system-access).

1. Put Plinky in UF2 USB drive mode and wait for its volume to mount.
2. Click **Connect Plinky drive** and choose the volume's root folder. The tool checks `INFO_UF2.TXT` for `STM32L476-Plinky-v3`, verifies presets and lists available sample slots. Connecting does not change your current editing workspace.
3. Choose a slot and click **Load selected slot** to read its audio and matching presets, or import your own audio for that destination.
4. **Back up selected slot** downloads one ZIP with fresh original sample and preset UF2 files, without modifying the drive.
5. **Save both UF2 files to Plinky…** shows the destination and replacement duration. When you proceed, allow the browser's write-access request. The tool reads fresh presets, updates only that slot's metadata, requests an original-backup ZIP download, then writes the sample followed by presets. Keep the backup until physical playback is verified. Check your browser's Downloads to ensure it was saved.
6. Wait for Plinky's write activity to finish before restarting it with the encoder.

Direct access cannot silently discover or authorize a drive. You must select it. **Disconnect** forgets the page's handle; it does not eject the volume or revoke browser permissions.

Some virtual UF2 drives may reject the browser's temporary-file/write-stream implementation. A partial failure is reported explicitly. Leave USB mode active and use the two manual download buttons, then copy both files to the drive before restarting. After a fresh snapshot is obtained during Save, the manual presets download uses that snapshot as its baseline. A successful stream close means the browser finished its write operation; it is not a physical flash verification. The tool does not automatically reset the synth.

Filesystem integration checks used disposable disk fixtures with the supplied backups. They verified drive identification, fresh reads, a complete two-file save, preservation of the firmware file, ZIP backup integrity, missing slots, wrong boards, and both first-write and partial-write failures. No connected physical synth was written during development. Browser visual QA remains blocked by the browser tool's local-file URL policy.

## Decoded example

`SAMPLE3-decoded.wav` is the supplied SAMPLE3 audio decoded to a standard WAV file, with no gain, trimming or resampling changes. It contains 2,097,152 mono frames at 32,000 Hz, or 65.536 seconds. The latest valid metadata describes eight pitched slices with notes 12 through 19 and one-shot playback. Exact offsets and validation results are in `format-analysis.json`.

## Format

The codec was checked against the public Plinky firmware at commit `ee4a340fe66fd2705299b4c15381acec44008f42`:

- https://github.com/plinkysynth/plinky_public/blob/ee4a340fe66fd2705299b4c15381acec44008f42/bootloader/Core/Src/ghostfat.c
- https://github.com/plinkysynth/plinky_public/blob/ee4a340fe66fd2705299b4c15381acec44008f42/sw/Core/Src/params.h
- https://github.com/plinkysynth/plinky_public/blob/ee4a340fe66fd2705299b4c15381acec44008f42/sw/Core/Src/ui.h

Each UF2 block is 512 bytes with 256 payload bytes. The magic words are 0x0A324655, 0x9E5D5157, and the trailing 0x0AB16F30. Flags and family ID are zero in these Plinky backups. The parser validates block numbering, count, alignment, address ranges, coverage and non-overlap; it accepts shuffled blocks.

A sample slot has 4,194,304 payload bytes of signed 16-bit little-endian mono PCM. The sample address is `0x40000000 + slot * 0x400000`, for slots 0–7. Unused frames in newly generated samples are zero-filled. The rate is 32,000 Hz; capacity is 65.536 seconds. Exported UF2 size is 8,388,608 bytes.

PRESETS has 522,240 payload bytes starting at 0x08080000: 255 flash journal pages of 2,048 bytes. Its UF2 is 1,044,480 bytes. Latest valid version-2 page is selected by greatest sequence per object index. Sample object indices are 128–135. Within a page:

| Byte offset | Field |
| --- | --- |
| 0 | 1,024 bytes of waveform data: 2,048 four-bit peaks, one per 1,024 frames; low nibble first |
| 1024 | Eight signed little-endian 32-bit slice starts, in frames |
| 1056 | Signed little-endian 32-bit recorded frame count |
| 1060 | Eight signed 8-bit note values |
| 1068 | Pitched flag |
| 1069 | Loop flags: bit 0 loops; bit 1 selects whole-sample bounds |
| 1070 | Two padding bytes |
| 2024 | Sixteen bytes of system parameters |
| 2040 | Object index, one byte |
| 2041 | Page version, one byte |
| 2042 | Unsigned 16-bit checksum |
| 2044 | Unsigned 32-bit sequence |

Checksum starts at 123; for the first 2,040 bytes, set `hash = (hash * 23 + byte) & 0xFFFF`. The editor preserves page sequence, padding, system settings and every other page. It regenerates sample waveform peaks, metadata and checksum. Plinky's note display adds one octave: internal 0 is shown as C1.

