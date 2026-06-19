# MSC Tally Light

A visual Go tally for QLab via MSC/MIDI, with adjustable colors and timing.

A lightweight, local tally indicator: sits at a rest color, jumps **instantly** to a trigger color the moment a matching MIDI message arrives, and fades back to rest after an adjustable hold time. If a new trigger arrives while it's still fading, it snaps straight back to full trigger color and starts the hold/fade cycle over.

Built for use alongside QLab via MIDI Show Control (MSC), but also works with plain Note On / Control Change messages, or "any MIDI message."

---

## Contents

- [Quick start](#quick-start)
- [Requirements](#requirements)
- [Opening the app](#opening-the-app)
- [Interface overview](#interface-overview)
- [Configuring trigger types](#configuring-trigger-types)
- [Colors, hold time, and fade time](#colors-hold-time-and-fade-time)
- [Collapsing panels and fullscreen mode](#collapsing-panels-and-fullscreen-mode)
- [QLab side: setting up a MOTU loopback](#qlab-side-setting-up-a-motu-loopback)
- [QLab side: MSC Show Control Broadcast](#qlab-side-msc-show-control-broadcast)
- [QLab side: lighting desk and tally at the same time](#qlab-side-lighting-desk-and-tally-at-the-same-time)
- [Troubleshooting](#troubleshooting)
- [Technical background](#technical-background)

---

## Quick start

1. Open `msc-tally-light.html` locally in **Chrome** or **Edge** (double-clicking the file is enough).
2. Allow MIDI access when the browser asks.
3. Choose your MIDI input from the dropdown at the top left.
4. Set your trigger type (default: MIDI Show Control, command **Go**).
5. Press Go in QLab — the panel on the right jumps to the trigger color and fades back.

Use the **"▸ test trigger"** button to verify behavior without QLab.

---

## Requirements

- **Browser:** Chrome or Edge (desktop). Safari does not fully support the Web MIDI API — avoid it for production use. Firefox does not support Web MIDI at all.
- The **Web MIDI API** only works over `https://`, `localhost`, or a locally opened file (`file://`). A downloaded HTML file opened by double-click works fine.
- No installation, no build step, no internet connection required — everything runs client-side in a single HTML file.

---

## Opening the app

**Recommended for production:** download the file and open it locally (double-click, or drag it into a Chrome window). This is the most reliable way to get Web MIDI access, and it doesn't depend on a browser sandbox or an internet connection.

Each new session (tab closed and reopened, or file reopened) requires the browser to ask for MIDI permission again. Check this via the MIDI icon in the address bar, or via `chrome://settings/content/midi`.

> **Note:** if you create a new virtual MIDI device (e.g. an IAC port) *after* the page is already open, the device list may be stale. Close the tab completely and reopen it.

---

## Interface overview

| Section | Function |
|---|---|
| **Configuration** (left) | All settings: MIDI input, trigger type, colors, timing |
| **Status** (top right) | The tally itself: color panel, status text, last-cue readout, test button |
| **MIDI log** (bottom right) | Live log of every incoming MIDI message, with match indication |

Connection status (top right of the header) shows how many MIDI devices were found, or an error message if MIDI access fails.

---

## Configuring trigger types

Choose via **Trigger type** in the configuration panel:

### MIDI Show Control (default)

For QLab's MSC Go messages (and other MSC commands). Configurable fields:

| Field | Meaning | Example |
|---|---|---|
| **Command** | The MSC command to match on | Go, Stop, Resume, Timed_Go, Fire, etc. — or "All commands" |
| **Device ID** | MSC Device ID of the sender | `1`, or `*` for any device |
| **Cmd Format** | MSC command format (category) | Lighting (Generic), Sound (Generic), All-types, or "All" |
| **Q Number** | Cue number to match | `89`, blank = any |
| **Q List** | Cue list to match | blank = any |
| **Accept all Q numbers** | Checkbox — ignores Q Number/Q List entirely | on = match any Q number within the chosen command/device |

Device ID `127` (all-call, `7F`) always matches, regardless of the configured Device ID.

### Any MIDI message

Triggers on literally anything that arrives — useful for quickly testing whether any MIDI data is getting through at all.

### Note On / Control Change

For simpler MIDI setups without MSC. Configurable by channel (1–16 or `*`) and note/CC number (0–127 or `*`).

---

## Colors, hold time, and fade time

| Setting | Behavior |
|---|---|
| **Rest color** | The color the panel sits at when nothing is happening |
| **Trigger color** | The color immediately after a match |
| **Hold time** | How long the trigger color stays at **full** intensity before the fade begins (default 0.1 s) |
| **Fade time** | How long it takes to fade from trigger color back to rest color (default 0.5 s) |

Colors can be changed via the color picker or directly as a hex code. Both fields stay in sync.

**Important behavior:**
- The transition from rest → trigger is **always instant** (0 seconds) — this isn't adjustable, and doesn't need to be, since that's the whole point of a tally light.
- Only the trigger → rest transition (after the hold time) is a fade, using the configured fade time.
- If a new match arrives while the fade is still in progress, the panel snaps straight back to full trigger color and the hold/fade cycle restarts.

---

## Collapsing panels and fullscreen mode

- **Configuration** and **MIDI log**: click the title bar (with the chevron arrow) to collapse or expand the panel. Settings stay active even while a panel is collapsed.
- **Fullscreen mode**: the **"⤢ fullscreen"** button at the top right of the Status panel. Hides the header, footer, configuration, and log; the tally fills the entire browser window with significantly enlarged text, suitable for reading at a distance (e.g. on a separate monitor or tablet next to the lighting desk).
  - Exit via the **"✕ exit fullscreen"** button at the bottom, or with **Esc**.
  - The test button remains accessible in fullscreen mode too.
  - Fullscreen is implemented as a fixed-position overlay pinned directly to the viewport, independent of the normal layout grid — so it stays correctly sized through window resizes, including when crossing the responsive breakpoint at 760px width.

> Settings (colors, timing, panel state) are **not** saved between sessions. Reopening the file resets the app to its default values.

---

## QLab side: setting up a MOTU loopback

Web MIDI in the browser cannot create a virtual MIDI port — the browser can only read from ports the operating system already knows about. A practical solution is a **hardware loopback** via a MIDI interface (e.g. MOTU):

1. Connect a MIDI **Out** port on the interface to a MIDI **In** port (a physical cable, or done in software via the interface's own control panel software if available).
2. In QLab: patch your MSC source to the **Out** port you've looped back.
3. In the tally app: select the **In** port the loopback returns on, from the MIDI input dropdown.

Alternative: use an **IAC bus** (Audio MIDI Setup → MIDI Studio → IAC Driver → check "Device is online", add a port). This is a purely virtual MIDI cable inside macOS and requires no physical hardware loopback, but only works for signals that stay in software (i.e. not for a physical lighting desk).

---

## QLab side: MSC Show Control Broadcast

In addition to manually created MSC cues, QLab has a built-in **Show Control Broadcast** feature that automatically sends an MSC GO message on **every** Go in the cue list — no need to create separate cues for it.

**Setup:**

1. `QLab → Workspace Settings → MIDI → MSC Broadcast` tab.
2. Check the box to enable broadcast for the workspace.
3. Click **New Destination** (⌘N).
4. Choose the desired **MIDI Patch** (e.g. an IAC bus pointing to the tally app), **Command Format**, and **Device ID**.

Note: the Q number sent here is the **QLab cue number** as shown in the cue list — it won't necessarily match a Q number you previously set manually in a separate MSC cue. In that case, use the **"Accept all Q numbers"** option in the tally app, or set the Q number field to match the actual incoming value (visible in the app's MIDI log).

---

## QLab side: lighting desk and tally at the same time

An MSC cue in QLab 5 can only point to **one** MIDI patch at a time — there's no built-in way to send one cue to two destinations. The recommended, cleanest solution that requires no extra software:

- **Manual MSC cues** → stay patched to the lighting desk, as before.
- **Show Control Broadcast** → set up separately on its own MIDI patch (e.g. an IAC bus), going to the tally app.

This keeps both streams fully separate, with no risk of duplicate messages reaching the lighting desk, and no need for extra routing software (such as MIDI Patchbay or MidiPipe).

---

## Troubleshooting

| Symptom | Likely cause |
|---|---|
| Dropdown shows "no devices found" | MIDI access denied, or a device was created *after* the page was opened — close the tab and reopen it |
| Tally doesn't respond, MIDI log stays empty | Wrong MIDI input selected, or QLab is sending to a different patch/port than expected |
| Tally responds, but never shows a "match" (gray dot in the log) | Trigger settings (Command/Device ID/Q number) don't match the incoming message — check the exact values shown in the MIDI log |
| Color transition feels slow/dull instead of a sharp flash | Check that hold time and fade time are set as intended; rest → trigger is always instant, only trigger → rest fades |
| Browser never asks for MIDI permission | Go to `chrome://settings/content/midi` and check the site isn't set to "Block" |
| Everything worked, then suddenly stopped — even with "Any MIDI message" selected | A MIDI device/port was removed or its driver toggled off while the page was already open. Toggle the driver (e.g. IAC) back online and reopen the tab/file so the browser picks up a fresh device list |
| Fullscreen mode only fills part of the screen after resizing | Fixed in the current version — fullscreen now uses a fixed-position overlay independent of the page layout. If you're on an older copy of the file, download the latest version |

Use the **MIDI log** as your first diagnostic tool: every incoming message is shown with a green dot (●, match → trigger) or gray dot (○, ignored), including the parsed MSC fields (device, command, Q number, Q list), so you can see exactly what's arriving versus what your filter expects.

If nothing arrives at all — not even as an ignored/gray entry — the issue is upstream of the app's filtering logic: check the MIDI input dropdown still shows the expected port, and confirm the port still exists in Audio MIDI Setup / your interface's driver.

---

## Technical background

- A single self-contained HTML file, no external dependencies, no build step.
- MIDI is received via the **Web MIDI API** (`navigator.requestMIDIAccess`), including SysEx access for MSC parsing.
- MSC messages are parsed according to the structure `F0 7F <device_id> 02 <cmd_format> <command> [<Q_number>] [<Q_list>] F7`.
- Color transitions are driven entirely by JavaScript via `requestAnimationFrame`, not CSS transitions — this guarantees the trigger transition is always instant within a single frame, regardless of re-triggers during an ongoing fade.
- No server side, no data storage, no network traffic — everything stays local in the browser.

### Why OSC/UDP isn't supported

Browsers don't allow web pages to open raw UDP sockets, for security reasons — there's no "Web UDP API" the way there's a Web MIDI API. Since OSC is conventionally sent over UDP, a browser-based app can't receive it directly. A workaround would require a small local bridge program (e.g. a Node.js script using `osc` and `ws` packages) that listens for OSC over UDP and forwards it to the browser over a WebSocket — out of scope for this version, but possible to add later if the tally ever needs to run on a device where MIDI isn't practical (e.g. a tablet elsewhere on the network).
