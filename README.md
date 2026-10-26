# WinDroid Lite 1.1

A lightweight Linux desktop with a **Windows 11-style look**, **Android apps** (Waydroid + Google Play)
and **Windows `.exe` support** (Wine), tuned for older laptops with a hard disk (built and tested around a
Dell Inspiron 15 3000 with 4 GB RAM).

It is built on Debian 13 "trixie" + XFCE with `live-build`. This repository is a **build kit**: it does not
contain the ISO, it contains everything needed to produce one.

> **Status:** version 1.0 of this kit built an ISO that booted on the target laptop (WiFi, webcam, DVD drive,
> audio, touchpad and the AHCI disk were detected in the live session). Everything added in 1.1 (Chrome, Wine,
> Plymouth boot animation, branding clean-up, touchpad, fonts, menu clean-up) has been written carefully and
> checked for syntax, but **still needs a test on real hardware**.

---

## What you get

| Area | What WinDroid Lite does |
|---|---|
| Look | Windows 11-style GTK theme + icons, centred taskbar (Start button + pinned/running apps), tray and clock on the right, Selawik font (open Segoe UI-compatible font; falls back to Open Sans) |
| Start button | Your custom WinDroid logo (`config/includes.chroot/usr/share/icons/hicolor/*/apps/windroid-logo.png`) |
| Boot animation | Plymouth theme: Windows logo slides in from the left, Android logo from the right, white flash when they meet, then a dim â†’ light â†’ dim breathing loop. 1:1 canvas (1080Ã—1080) |
| Browser | Google Chrome (downloaded from Google during the build; Chromium is used automatically if that fails) |
| Windows programs | Wine (64-bit + 32-bit). Double-click an `.exe` / `.msi` / `.bat`. `windroid-setup wine-test` checks it |
| Android apps | Waydroid with Google Play (set up from Start menu â†’ **Android (WinDroid)**). Runs in a window via a nested compositor because XFCE is X11 |
| Language | English everywhere (locale `en_US.UTF-8`, timezone `Asia/Kolkata`) |
| Touchpad | Tap-to-click, two-finger scroll and tap-and-drag are enabled by default |
| Start menu | Only Windows-style apps are shown (File Explorer, Notepad, Task Manager, Media Player, Settings ...); the Debian default entries are hidden |
| Branding | OS name, boot menu, login screen, installer and GRUB entry all say **WinDroid**; no Debian name or logo in the boot menu |
| Speed | `--apt-recommends false`, no docs/man pages/extra languages, zstd squashfs, BFQ I/O scheduler + larger read-ahead for hard disks, zram swap, fewer disk writes, slow boot services disabled, compositor off |

### Known limits (please read)
- **Segoe UI cannot be shipped** (Microsoft licence). Selawik is the closest open font and Fontconfig maps "Segoe UI" to it.
- **Android on 4 GB RAM + hard disk is slow.** It needs about 2 GB extra RAM and the first boot can take 3-5 minutes. It works only on an *installed* system, not in the live session, and appears in one window.
- Windows 11 features such as snap layouts and animations are not available (the compositor is off for speed).
- Not every Windows program runs under Wine (anti-cheat games, some Adobe/Office versions).
- **Trademarks:** the supplied logo combines the Windows and Android logos. That is fine for personal use, but check the Microsoft and Google trademark rules before distributing the ISO. Google Chrome may not be redistributable inside a public ISO; Chrome is downloaded from Google at build time on *your* build.

---

## 1. Get the ISO

### Option A - GitHub Actions (no Linux needed, works from a phone)
1. Create a GitHub repository (private is fine).
2. Upload **`windroid-lite-build-kit.zip`** to the repository root (*Add file -> Upload files*; do **not** unzip it). If a file with that name already exists, uploading it again replaces it.
3. Add the workflow: *Actions -> set up a workflow yourself*, delete the sample text, paste the contents of **`build-iso.yml`**, commit.
   (If you already have `.github/workflows/main.yml`, open it with the pencil icon and replace its contents.)
4. The build starts automatically on every push. You can also start it by hand: *Actions -> Build WinDroid Lite ISO -> Run workflow*.
5. After roughly 25-60 minutes open the finished run, scroll to **Artifacts** and download **windroid-lite-iso**. Unzip it on a PC to get the `.iso`. The artifact is kept for 14 days.

If the run shows a red X, open it, expand the failed step and read the last lines (the workflow prints the end of `build.log`). `build.log` is also inside the artifact.

### Option B - on Debian 13 / Ubuntu
```bash
unzip windroid-lite-build-kit.zip && cd windroid
./build.sh            # on Debian 13 (needs ~12 GB free disk and internet)
./build-docker.sh     # on Ubuntu/Mint/Fedora (needs docker)
```
Output: `windroid-lite-1.1-amd64.iso` and a `.sha256` file.

## 2. Put it on a USB stick

| Method | Steps |
|---|---|
| **Ventoy** | Install Ventoy on the stick once (ventoy.net), then copy the `.iso` onto it like a normal file. |
| **Rufus** (Windows) | Select the ISO -> Start -> choose **"Write in DD Image mode"**. |
| balenaEtcher | Select ISO and USB stick -> Flash. |
| Linux | `sudo ./scripts/write-usb.sh windroid-lite-1.1-amd64.iso /dev/sdX` |

Writing in DD mode makes the stick look "empty/unreadable" in Windows - that is normal, do not format it.
Boot with **F12** on a Dell. If it does not boot, try the other (UEFI / legacy) entry, or disable Secure Boot.

## 3. Live test checklist
1. The boot animation plays and the desktop appears with the taskbar at the bottom.
2. **Start menu -> Hardware Check (WinDroid)**: WiFi, webcam, DVD, audio, touchpad, disk (AHCI/HDD) and RAM.
3. A tap on the touchpad clicks. The WiFi icon in the tray connects to your network.
4. Open Chrome; open an `.exe`; open the Start menu and check that the Lock / Sleep / Logout buttons are visible.
5. In a terminal: `fc-list | grep -ci selawik` (should be above 0) and `windroid-setup wine-test`.

## 4. Install
Use the **Install WinDroid** icon on the live desktop. The SATA controller was already in AHCI mode on the test
laptop, so no BIOS change is needed. **Back up your data first**; the installer can erase disks. When dual-booting
with Windows, choose the "Install alongside" option; GRUB's OS detection is on.
After installing: *Start menu -> Android Setup (WinDroid)*, and optionally `windroid-setup cleanup` to remove the installer.

## 5. Tools (`windroid-setup`)
`android`, `android-arm`, `wine-test`, `winetricks`, `office`, `hw`, `status`, `update`, `cleanup`.

---

## Project layout
```
build.sh / build-docker.sh      build the ISO
auto/config                     live-build settings (name, boot options, compression)
config/package-lists/           core packages (a wrong name stops the build)
config/includes.chroot/         files copied into the system (scripts, theme config, icons, Plymouth theme ...)
config/hooks/live/              build steps: Waydroid, Chrome, Wine, optional packages, theme, branding,
                                performance, menu clean-up, Plymouth, cleanup, boot-menu branding (binary hook)
config/windroid-boot/           boot-menu logo and background
tools/render_boot_preview.py    renders the boot animation as a 1080x1080 video (same timeline as the Plymouth script)
.github/workflows/build-iso.yml GitHub Actions workflow
```

## Customising
| What | Where |
|---|---|
| Apps | `config/package-lists/windroid.list.chroot`, `config/includes.chroot/usr/share/windroid/optional-packages.list` |
| Start menu whitelist | `config/includes.chroot/usr/share/windroid/menu-whitelist.txt` |
| Taskbar | `config/includes.chroot/etc/skel/.config/xfce4/xfconf/xfce-perchannel-xml/xfce4-panel.xml` |
| Logo | the PNGs in `config/includes.chroot/usr/share/icons/hicolor/*/apps/windroid-logo.png` and `config/includes.chroot/usr/share/plymouth/themes/windroid/*.png` |
| Boot animation timing | `windroid.script` and `tools/render_boot_preview.py` (keep the constants identical) |
| Name / website | `auto/config`, `build.sh`, `config/hooks/live/0040-branding.hook.chroot` (`example.com` placeholders) |

## Troubleshooting
- **Installer says "dictionary check error" when creating the password:** fixed in this kit (the password dictionary packages `cracklib-runtime` and `wamerican` are installed, and the installer now allows simple passwords and ticks "Log in automatically" by default). On an older ISO: in the live session run
  `sudo apt-get update && sudo apt-get install -y cracklib-runtime wamerican`, or allow weak passwords with
  `sudo sed -i '/^allowWeakPasswords/d;/^doAutologin:/d' /etc/calamares/modules/users.conf` and
  `printf '\nallowWeakPasswords: true\nallowWeakPasswordsDefault: true\ndoAutologin: true\n' | sudo tee -a /etc/calamares/modules/users.conf`, then restart the installer.
  An empty password is not possible; use a short password plus the auto-login box (sudo still needs the password).
- **No WiFi icon / network list:** run `nmtui` in a terminal and connect there; the tray icon needs the panel's system tray (already configured).
- **A tool says "command not found" (rfkill, swapon ...):** open a new terminal; `/usr/sbin` is now in the PATH.
- **Chrome missing:** the build fell back to Chromium (see `/usr/share/windroid/browser.conf`).
- **`.exe` does nothing:** `windroid-setup wine-test` shows what is missing; `/usr/share/windroid/wine.conf` says whether Wine was installed at build time.
- **Android does not start:** `windroid-setup status`; the binder module is loaded by `windroid-setup android`.

## Licences
The Win11-style GTK theme and icons (GPLv3, `yeyushengfan258/*`) and Selawik (SIL OFL) are downloaded during the build.
Google Chrome, Android, Windows and Wine are trademarks of their respective owners.
