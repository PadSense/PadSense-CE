# PadSense Community Edition

**Full DualSense support on Windows — input, audio, mic, and haptics over Bluetooth — with no custom kernel driver.**

PadSense Community Edition bridges your Bluetooth-connected PlayStation DualSense controller to a virtual **USB** DualSense that Windows and games treat as the real thing. Games with native DualSense support just work — including the features that normally die the moment you go wireless.

## Why

Connect a DualSense over Bluetooth and you lose most of what makes it special: controller speaker, headset jack, microphone, HD haptics, and games that expect a USB pad stop recognizing it. The usual fixes are input-only remappers or custom kernel drivers.

PadSense takes a different route: a **user-space USBIP server** that synthesizes a complete composite USB device (HID + audio) and hands it to Windows through the signed, WHQL'd [usbip-win2](https://github.com/vadimgrn/usbip-win2) client driver. No unsigned drivers, no test mode, no kernel code of our own — the "USB device" is a program.

## Features

- **Full input at real-time rate** — sticks, buttons, d-pad, gyro/accelerometer, touchpad, and factory calibration forwarded per-packet (~700 Hz measured; a 250 Hz stock-compatible mode is one click away)
- **Adaptive triggers and rumble** — trigger effects and HD haptics pass through to the real pad
- **Controller speaker** — game audio to the pad's built-in speaker, clock-drift-corrected for dropout-free playback
- **Headset jack** — plug headphones/headset into the pad's 3.5 mm jack, get game audio and use the mic
- **Microphone** — the pad's built-in mic works as a Windows recording device
- **Native game compatibility** — games with built-in DualSense support (tested: Helldivers 2 with Steam Input off) see a genuine USB DualSense: correct firmware/version info, calibration, lightbar, player LEDs
- **Invisible plumbing** — the physical Bluetooth pad is hidden from games via [HidHide](https://github.com/nefarius/HidHide) so nothing double-inputs
- **Set and forget** — runs as a tray app, optional start-at-logon, auto-reconnects when the pad powers on/off

## Requirements

- Windows 10 (1809+) or Windows 11, x64
- Bluetooth adapter, DualSense paired over Bluetooth
- Administrator rights (driver attach + device hiding)

The installer takes care of the rest: it installs [usbip-win2](https://github.com/vadimgrn/usbip-win2) and [HidHide](https://github.com/nefarius/HidHide) (only if missing — and the uninstaller removes only what it installed), plus the VC++ runtime.

## Install

1. Grab the latest `PadSense-Setup-x.y.z.exe` from [Releases](../../releases).
2. Run it. Windows SmartScreen will warn because the installer is not yet code-signed — click **More info → Run anyway**.
3. If drivers were installed, restart when prompted.
4. Pair your DualSense over Bluetooth and launch PadSense — the tray icon turns green ("DualSense Bridged") when the virtual pad is live.

Right-click the tray icon for status, live polling rate, polling-rate mode, autostart, and logs.

## How it works

```
DualSense ──Bluetooth──▶ PadSense (user space)              Windows
              HID 0x31 input       │  USBIP protocol           │
              0x36 audio/haptics   ├──── TCP (loopback) ──────▶ usbip-win2 vhci driver
              Opus mic frames      │                            │
                                   │                            ▼
                            re-synthesized as a         virtual USB DualSense
                            composite USB device        (HID + speaker + mic)
```

PadSense reads the pad's Bluetooth HID reports, translates the wire formats (input reports, Opus-coded audio in both directions, trigger/haptic feedback), and serves a byte-faithful USB DualSense — descriptors captured from real hardware — over the USBIP protocol to the local vhci driver. Clock-drift servos keep speaker and mic audio rate-matched to the pad's own crystal.

## PadSense Pro

CE is the free, single-controller pass-through edition. **PadSense Pro** (in development, planned for Steam) adds cross-controller mapping (e.g. use any pad as a virtual DualSense), remapping, macros, shift layers, turbo, and multi-controller setups.

If CE is useful to you, you can [support development on Ko-fi](https://ko-fi.com/up2urheadlights). ☕

## Notes and disclaimers

- **Anti-cheat**: PadSense presents an emulated controller. Some games and anti-cheat systems restrict input emulation — use at your own risk and check the rules of the games you play.
- PadSense is an independent project, not affiliated with or endorsed by Sony Interactive Entertainment. "PlayStation" and "DualSense" are trademarks of Sony Interactive Entertainment Inc., referenced only to describe compatibility.

## Third-party software

| Component | Role | License |
|---|---|---|
| [usbip-win2](https://github.com/vadimgrn/usbip-win2) | signed USBIP vhci driver | BSD-2-Clause |
| [HidHide](https://github.com/nefarius/HidHide) | hides the physical pad | MIT |
| [Opus](https://opus-codec.org) | speaker/mic codec | BSD-3-Clause |
| [WDL](https://www.cockos.com/wdl/) | resampling | zlib-style |

## License

PadSense CE is **free to use, but not open source**. It is provided under a custom license — see [LICENSE.md](LICENSE.md). All rights reserved. The third-party components above remain under their own licenses.
