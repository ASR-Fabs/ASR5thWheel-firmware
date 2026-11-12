# ASR 5th Wheel firmware

Firmware for the **ASR 5th Wheel** (ESP32). Each 5th Wheel on a home Wi-Fi checks this repository when it starts and installs a newer version from it. The source code is not published here.

Copyright (c) 2026 ASR-LLC. All rights reserved.

## Files

|File|What it is|Read by|
|-|-|-|
|`signed\_version.txt`|The current release, one line, for example `7.8.0`.|7.7.5 and later|
|`signed\_latest.bin`|That release, signed with the ASR signing key (target **ASR 5th Wheel**).|7.7.5 and later|
|`version.txt`|`7.7.5`. **Frozen: never change it.**|7.7.4 only|
|`latest.bin`|The plain (unsigned) 7.7.5. **Frozen: never change it.**|7.7.4 only|

`signed\_version.txt` is a single line holding only the version (1 to 23 characters: letters, digits, `.`, `-`, `\_`). A trailing line break is fine. Anything else on the line, or a second line, means "no update".

### Why there are two pairs

Up to 7.7.4, the 5th Wheel read `version.txt` and `latest.bin` and installed any version that was different from its own, unsigned. From 7.7.5 on it reads only the signed pair and installs only a file signed with the ASR key.

The old pair stays on 7.7.5 for good. That way a unit still on 7.7.4 always updates in two hops, whenever it is next turned on:

1. 7.7.4 reads `version.txt` (7.7.5), installs the plain `latest.bin` and restarts.
2. 7.7.5 reads `signed\_version.txt`, installs the newest signed release and restarts.

Once 7.8.0 or later is out, a 7.7.4 unit does both hops back to back. If the old pair ever pointed at a signed file or a newer version, 7.7.4 units would fail or install something they can't check, so leave it alone.

## How a 5th Wheel updates (7.7.5 and later)

1. **When.** At every power-on, once it has joined the home Wi-Fi, and before it starts handling filament. If Wi-Fi isn't available at power-on, there is no check until the next power-on.
2. **Only a newer version counts.** Numbers compare as numbers: `7.7.9` < `7.7.10` < `7.8.0`. The same or an older version in `signed\_version.txt` means "up to date". A release is newer than any `-dev.N` build of the same number, so never publish a dev build.
3. **The file is checked while it downloads.** It is written to the spare program slot. The check covers the marker, the target (`5wheel`), the length and the signature. The version inside the signature must also match `signed\_version.txt`. Only if everything passes is the new program selected, and the unit restarts into it.
4. **LEDs.** Purple while updating. Red for 2 seconds if the update fails, then the unit carries on with the version it had. The serial log (115200) says why.

Any signed 5th Wheel `.bin` can also be installed by hand on the unit's **/update** page (`http://asr5thwheel-xxxx.local/update`), including an older one (a deliberate roll-back). Unsigned files, and files signed for the Helper or the bridge, are refused, and the page says why. The Wi-Fi setup network takes no firmware at all: the unit restarts if an upload is tried there.

## The 7.7.5 changeover (once)

7.7.5 is built once, and that one export is published twice. The old pair always holds this plain 7.7.5, even when the signed pair starts at a later release (for example 7.8.0): a 7.7.4 unit takes the plain 7.7.5 first, then the newest signed release.

1. Export `filament\_handler\_v7\_7\_5.ino.bin` as in step 2 above.
2. **Signed pair:**

   1. Sign it (target **ASR 5th Wheel (ESP32)**, version `7.7.5`) and upload it as `signed\_latest.bin`.
   2. Upload `signed\_version.txt` containing `7.7.5`.
3. **Old pair:**

   1. Upload the same, unsigned export as `latest.bin`, replacing 7.7.4.
   2. Upload `version.txt` containing `7.7.5`, last.
4. From then on, never change `version.txt` or `latest.bin` again.

If the signed pair starts at 7.8.0 instead, do step 3 with the 7.7.5 build (the sketch tagged `v7.7.5`) and publish 7.8.0 in the signed pair as in "Publishing a release".

## Releases

|Version|What changed|
|-|-|
|7.8.0|Aligned with the ASR BMCU Helper. **Feed mode** Standard or TPU (App and Settings page): Standard is the 5th Wheel's proven feed, unchanged; TPU runs the Helper's TPU load, timed (contact, 2nd push wait, 2nd push, Engage Delay, relief, tug, Engaged), with its own values. **Autoload** counts its distance from the exit switch, and the exit switch must be reached within a time window (5 s). **App**: 60-second tension graph with the motor strip and the current feed step.|
|7.7.5|Signed updates: `signed\_version.txt` and `signed\_latest.bin`, newer versions only; `/update` accepts only signed 5th Wheel files; ArduinoOTA removed; the Wi-Fi setup page shows only Configure WiFi and takes no firmware. No change to filament handling.|
|7.7.4|The last unsigned release (reads `version.txt` and `latest.bin`).|

## First install

A new ESP32 is flashed once over USB (Arduino IDE, the board setting above). After that, updates come from here.

## Third-party software in the 5th Wheel firmware

* ESP32 Arduino core (Espressif): LGPL-2.1.
* WiFiManager (tzapu): MIT.
* FastLED: MIT.

\---

Document history: 1.0 (2026-11-12), written with ASR 5th Wheel 7.7.5 (the signed update channel). 1.1 (2026-11-12): ASR 5th Wheel 7.8.0; the Releases table; the old pair stays on the plain 7.7.5 when the signed pair starts at a later release.

