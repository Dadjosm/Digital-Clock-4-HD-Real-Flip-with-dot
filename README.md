# Digital Clock 4 – Flipclock HD with Real Flip

![Clock Preview](J%20Clock%20Preview.gif)

A customised build of [Digital Clock 4](https://github.com/Kolcha/DigitalClock4) (v4.7.8, Windows portable)
with an improved **Flipclock HD** skin and a real **card-flip animation**.

Everything here is built from the official source at tag `stable_4.7.8`, plus one patch
(`flip-animation.patch`) and a modified skin folder. Only `digital_clock.exe` and `plugins\countdown_timer.dll` are replaced;
all other DLLs and plugins in your `Digital Clock 4` folder stay as they are.

---

## What has changed

### 1. Skin changes (`skins/flipclock-hd/`, no rebuild needed)

| Change | Details |
|---|---|
| **Separator dots** | The original skin's separator image was fully transparent, so there was no colon. `s.png` now holds two round dots (digit-grey, soft shadow, placed at 36% and 64% of the card height). Used for both the hours/minutes and the seconds separator. |
| **Blinking separator fixed** | In a skin, `s1` = separator *on*, `s2` = separator *off*. Both pointed at the same image, so "separator flashes" had nothing to blink. `s2` now points to a blank image `s_off.png` (same 89×360 size, fully transparent). |
| **Darker lower half** | Lower half of every digit and of the AM/PM images darkened from grey level **183 → 160** (≈25 % darker than the top half's 214). Card background (10 → 9) and the black seam are effectively unchanged. |
| Skin version | `1.2.0` in `skin.ini`. |

Reference values measured from the skin images (RGB 0–255):

| | Top half | Bottom half |
|---|---|---|
| Digit | 214 | 160 (was 183) |
| Card body | 18 | 9–10 |
| Seam | 0 (about 2–3 px thick near row 178–181 of 360) | |

### 2. Program changes (`flip-animation.patch`, rebuild required)

| Change | Files | Details |
|---|---|---|
| **Real flip animation** | `gui/flip_animator.{h,cpp}` (new), `gui/clock_widget.{h,cpp}`, `digital_clock.pro` | When the time changes, each digit card that changed flips like a mechanical flip clock: old top half folds down, new bottom half unfolds underneath, with ease-in/out and shading on the moving flap. Separators and unchanged digits don't animate. Duration **420 ms**, ~66 fps. |
| **Flip only for flip skins** | `gui/clock_widget.cpp`, `core/clock_application.cpp` | Animation and the colorize behaviour below are used only when the selected skin's name contains "flip" (case-insensitive). All other skins behave exactly as before. |
| **Flip works while settings is open** | `gui/clock_widget.cpp` | The settings dialog puts the clock in "preview mode"; the flip no longer pauses there, so countdown-timer ticks flip while you configure. If a setting change alters the clock size (skin/zoom), the animation is skipped and the new look just appears. |
| **Colorize tints only the card** | `gui/colorize_effect.{h,cpp}`, `gui/clock_widget.{h,cpp}` | With *Colorize* enabled on a flip skin, only the dark card body takes the chosen colour. The white top digit, the grey lower digit and the separator dots are untouched; the lower half of the card stays proportionally darker and the seam stays black. The existing colour picker and strength slider are used. The flip is drawn from already-coloured images, so colour and flip work together. |
| **Flips on the exact second** | `gui/clock_widget.{h,cpp}` | Flip skins are refreshed exactly on every half second (digits change on the full second, the separator blinks on the half) instead of whenever the periodic refresh timer fires. Removes up to 0.5 s of random delay and the uneven gaps between flips. |
| **Blink follows the wall clock** | `gui/clock_display.cpp` | The separator is shown during the first half of each second, based on the current time rather than the previous refresh. |
| **Separator blink no longer cuts a flip** | `gui/flip_animator.{h,cpp}`, `gui/clock_widget.cpp` | If only the separator changes while a flip is running, the flip continues (`retarget`) instead of snapping to the end. |
| **Smoother start of each flip** | `gui/flip_animator.{h,cpp}`, `gui/colorize_effect.cpp`, `gui/clock_widget.cpp` | Static parts and flap images are prepared once per flip, the previous coloured image is reused instead of recomputed, and card-colorize uses lookup tables. Start-up cost dropped to about 1.5 ms in the test program. |
| **High priority (Windows)** | `main.cpp` | Process priority set to *High* and the GUI thread to *Above normal*, so the animation isn't starved by other programs. |
| **Countdown timer flips too** | `plugins/countdown_timer/` (`countdown_timer_plugin.{h,cpp}`, `countdown_timer.pro`, new `gui/flip_animator.*`, `gui/colorize_effect.*`) | The countdown timer plugin draws its own display, so it gets its own copy of the flip code. With a flip skin its digits flip every second. The card stays black (the clock's *Colorize* setting does **not** affect it) and turns **red** when the countdown reaches zero, then back to black when it restarts. Other skins are unchanged. This needs the rebuilt `countdown_timer.dll`. |

### 3. Settings you can use (already in the program, nothing changed)

- **Transparent for mouse** (first settings tab) – clicks and drags pass through the clock.
  For full click-through, also tick **become almost transparent on mouse hover** and choose
  **hide completely** (the option under it). If you don't choose *hide completely*, the mouse
  can still move the clock, and it only becomes transparent while hovered.
  While click-through is on you can't right-click the clock; use the **tray icon** to open settings.
- **Move clock on click** – this does not control dragging. It only nudges the clock's position by a
  small step, relative to where on the clock you click. Dragging keeps working whether it is on or off;
  to stop dragging use click-through (above).
- **Separator flashes** – blinks the dots (works now with the `s2` fix).
- **Colorize** – colour + strength; on flip skins it tints the card only (see above).

---

## Repository contents

| Path | Purpose |
|---|---|
| `.github/workflows/build.yml` | GitHub Actions workflow that builds `digital_clock.exe` |
| `flip-animation.patch` | All source changes (applies cleanly on tag `stable_4.7.8`) |
| `README.md` | This file |
| `skins/flipclock-hd/` | The modified skin, versioned with the code |

---

## Building the exe (GitHub Actions)

1. Push the repo contents above to GitHub.
2. **Actions** tab → *Build Digital Clock (flip animation)* → **Run workflow**.
3. When it finishes (about 10 minutes), open the run, scroll to **Artifacts**, download
   **digital_clock-flip-build** and unzip it. It contains `digital_clock.exe` and
   `plugins\countdown_timer.dll`.

What the workflow does: installs Qt **5.12.9** (MinGW 7.3 32-bit, the same toolchain as the
official 4.7.8 build, so it works with your existing DLLs), clones the `stable_4.7.8` source and the
QHotkey / SingleApplication dependencies, applies the patch, skips the plugins, runs
`lupdate`, `qmake -config release -r`, `mingw32-make`, and uploads `digital_clock.exe`.

Whenever you change `flip-animation.patch`, run the workflow again to get a new exe.

## Installing

1. Close Digital Clock (tray icon → Exit).
2. Copy the new `digital_clock.exe` over the old one in `Digital Clock 4\`, and the new
   `countdown_timer.dll` over the old one in `Digital Clock 4\plugins\`.
3. Copy the `flipclock-hd` folder into `Digital Clock 4\skins\` (overwrite).
4. Start the clock, choose **Flipclock HD** in settings. Optional: tick **separator flashes**,
   **transparent for mouse**, or **Colorize**.

---

## Tweaking

| What | Where | Value now |
|---|---|---|
| Flip speed | `flip_animator.cpp`, constructor: `duration_(420)` | 420 ms |
| Flip frame interval | `flip_animator.cpp`: `timer_.setInterval(15)` | 15 ms |
| Time-up colour of the countdown | `countdown_timer_plugin.cpp`, `Tinted()`: `QColor(215, 25, 25)` | red |
| Process priority | `main.cpp`: `HIGH_PRIORITY_CLASS` (use `ABOVE_NORMAL_PRIORITY_CLASS` for gentler) | High |
| Flap shading while flipping | `flip_animator.cpp`, `drawFlap(...)` calls: `0.55` | up to 55 % black |
| Which skins flip / get card-colorize | `clock_widget.cpp`, `OPT_SKIN_NAME`: name contains `"flip"` | any skin with "flip" in its name |
| Narrowest "card" that flips | `flip_animator.cpp`: `r.width() * 10 < widest * 6` | cells under 60 % of the widest are treated as separators |
| Lower-half digit darkness | skin images (`0–9.png`, `am.png`, `pm.png`, rows below the seam) | 160 |

## Technical notes

- Skins are static images. The program swaps one image for the next, so the flip is generated in
  code from the "before" and "after" images: cards are found by splitting the picture at fully
  transparent columns, and every changed card is rendered frame by frame.
- Colorize on flip skins estimates, per card and per half, the most frequent dark level (card
  body) and the most frequent bright level (digit face), then recolours only the body and the
  anti-aliased edge share that belongs to it.
- Tested: the flip, flip-in-settings, and colour behaviour were checked in a Linux test program and
  confirmed working in the built Windows exe.

## Credits and licence

- Digital Clock 4 © Nick Korotysh, GPL-3.0-or-later: <https://github.com/Kolcha/DigitalClock4>
- Original *Flipclock HD* skin by guy hoffman.
- Modifications in this repository (patch and skin edits) are released under the same
  **GPL-3.0-or-later** licence.

If you like my patches, now you can "say thanks" by donating any crypto to my Ethereum (EVM-multichain wallet) address 0xBdbA6B84Ca1A2DA181cE1BBA9b8F2B1dDdae355D 

<img width="4032" height="1083" alt="image" src="https://github.com/user-attachments/assets/ae5e7c33-9606-40be-b5a5-c21e65088f15" />



donate

Thanks
