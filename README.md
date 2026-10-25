# WinDroid Lite — Source Build Kit

WinDroid Lite is an experimental Debian 13 (Trixie) + XFCE-based desktop image with WinDroid branding, Windows-style layout, Google Chrome, Wine compatibility tools, NetworkManager, hardware firmware, and an optional Waydroid setup helper.

**Status: Experimental — not yet a verified release.** Build and test it on the target laptop before daily use.

## What This Revision Changes

* **Wi-Fi:** Adds NetworkManager, WPA supplicant, rfkill, common Wi-Fi firmware, and libinput utilities. Actual Wi-Fi support depends on the laptop's chipset and firmware.
* **Touchpad:** Adds a session helper to enable libinput tap-to-click when supported by the device.
* **Start Menu:** Adds GTK3 contrast rules intended to improve visibility of logout and session controls.
* **English Language:** Sets the default session locale to `en_US.UTF-8` and generates `en_IN.UTF-8`.
* **WinDroid Branding:** Changes visible OS branding to WinDroid and sets `ID=windroid` and `ID_LIKE=debian` for compatibility.
* **Google Chrome:** Replaces the Firefox menu favorite with Google Chrome Stable from Google's official Debian package. An internet connection is required during the build.
* **Windows EXE Support:** Adds Wine, Wine64, and Winetricks for compatible Windows applications. Not every `.exe` will work.
* **Boot Animation:** Adds a square 1080×1080 animated SVG preview and a custom Plymouth theme source featuring Windows and Android logos, convergence, a white flash, and a breathing loop.
* **GitHub Actions:** Supports repositories with source files at the root or inside the `windroid/` directory, verifies the checksum, and uploads build logs.

## Build with GitHub Actions

1. Create a new GitHub repository.
2. Upload the contents of the `windroid/` directory to the repository root, or upload the directory itself.
3. Open **Actions → Build WinDroid Lite ISO → Run workflow**.
4. Wait for the build to complete.
5. Download the `windroid-lite-iso` artifact from the completed workflow run.
6. The artifact should contain the ISO, SHA-256 checksum, and `build.log` when available.

A `v*` tag push also triggers a build.

The first build requires a working internet connection and substantial temporary disk space. Allow approximately 1–4 hours, depending on network speed and runner availability.

If the build fails, download `build.log` and inspect the first actual error.

## Build Locally

### On Debian 13

Requirements:

* Root or sudo access
* Working internet connection
* At least 20 GB of free disk space

Run:

```bash
./build.sh
```

### On Ubuntu or Linux Mint

Install Docker first, then run:

```bash
./build-docker.sh
```

### Clean the Build Cache

```bash
./build.sh --clean
```

The expected output is:

* `windroid-lite-1.0-amd64.iso`
* `windroid-lite-1.0-amd64.iso.sha256`

This source kit does not include a prebuilt ISO.

## Hardware Validation Checklist

Before considering WinDroid Lite ready for release, verify the following:

* [ ] Wi-Fi appears in the network menu.
* [ ] Wi-Fi scans, connects, reconnects after reboot, and works after installation.
* [ ] Touchpad pointer movement works.
* [ ] Tap-to-click works without pressing the physical button.
* [ ] Two-finger scrolling and physical clicking work.
* [ ] English locale is active.
* [ ] Check the login screen, installer, boot menu, and system utilities for remaining Debian references.
* [ ] Google Chrome launches and updates correctly.
* [ ] Wine runs a known compatible Windows application.
* [ ] Start-menu logout and session controls remain visible without hovering.
* [ ] Boot animation works on actual hardware and in a virtual machine.
* [ ] Suspend/resume, audio, brightness keys, Bluetooth, and USB storage work.
* [ ] Installer and boot process work on the required UEFI or legacy BIOS configuration.

## Known Limitations and Status

* **The ISO has not yet been built or hardware-tested in this environment.**
* Changes to source files do not prove that the ISO will build successfully.
* No separate list of previously reported bugs was included in the supplied source files; those issues cannot be claimed fixed.
* Google Chrome requires network access to `dl.google.com` during the image build.
* Wi-Fi compatibility depends on the chipset, kernel driver, and firmware.
* Wine supports compatible Windows applications only. Windows drivers and some modern applications or anti-cheat systems are not supported.
* A Debian-derived system can be rebranded, but it cannot be guaranteed to contain absolutely no Debian references.
* Segoe UI is proprietary. This kit uses Selawik where available as an open-source substitute, with Open Sans as a fallback.
* Plymouth animation behavior varies with firmware, display resolution, and graphics drivers. The 1080×1080 SVG is a preview; the boot theme scales to the actual boot display.
* Waydroid support depends on kernel binder support and available RAM. It is not enabled as a background service by default.

## Main Files

* `.github/workflows/build-iso.yml` — GitHub Actions ISO build workflow.
* `config/package-lists/windroid.list.chroot` — Base package list.
* `config/hooks/live/0030-theme.hook.chroot` — Theme and font configuration.
* `config/hooks/live/0040-branding.hook.chroot` — OS and installer branding.
* `config/hooks/live/0060-final-fixes.hook.chroot` — Chrome, locale, and final integration.
* `boot-animation-preview.svg` — Square 1080×1080 animated SVG preview.
* `config/includes.chroot/usr/share/plymouth/themes/windroid/` — Boot animation theme source.
* `config/includes.chroot/usr/local/bin/windroid-touchpad` — Tap-to-click helper.

## Licensing

Review the licenses of Debian packages, downloaded themes, icons, fonts, Wine, Google Chrome, Waydroid, and logos before redistribution.

Google Chrome is proprietary software downloaded from Google's official endpoint during the build. Verify all applicable license terms before distributing a finished ISO.

---

**WinDroid Lite — Experimental Build Kit**

Build first. Test on real hardware. Release only after verification.
