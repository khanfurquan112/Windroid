
# WinDroid Lite — Source Build Kit

WinDroid Lite is an experimental Debian 13 (Trixie) + XFCE-based desktop operating system with WinDroid branding, a Windows-style desktop layout, Google Chrome, Wine compatibility tools, NetworkManager, and optional Waydroid support.

**Status: Experimental — ISO build and physical hardware testing are still required.**

## 1. Features

- **Desktop:** XFCE desktop with Windows-style themes, icons, and taskbar layout.
- **Wi-Fi:** NetworkManager, WPA supplicant, rfkill, and common firmware packages.
- **Touchpad:** Tap-to-click helper for supported libinput devices.
- **Language:** English locale defaults with `en_US.UTF-8` and `en_IN.UTF-8`.
- **Branding:** WinDroid OS identity and custom branding assets.
- **Browser:** Google Chrome Stable downloaded from Google's official package source during the build.
- **Windows applications:** Wine, Wine64, and Winetricks for compatible `.exe` applications.
- **Boot animation:** Square 1080×1080 SVG preview and custom Plymouth theme source.
- **Android:** Optional Waydroid setup helper.
- **Build system:** GitHub Actions workflow for building and publishing ISO artifacts.

## 2. Requirements

For a local build, you need:

- A Debian 13 build environment, or Ubuntu/Linux Mint with Docker.
- A working internet connection.
- At least 20 GB of free disk space recommended.
- Administrator privileges for installing build dependencies.
- A GitHub account for cloud builds.

The generated ISO is not included in this source kit.

## 3. Build Using GitHub Actions

1. Create a new GitHub repository.
2. Extract the source ZIP.
3. Upload the contents of the `windroid/` directory to the repository root, or upload the complete `windroid/` directory.
4. Confirm that `.github/workflows/build-iso.yml` exists in the repository.
5. Open the **Actions** tab.
6. Select **Build WinDroid Lite ISO**.
7. Click **Run workflow**.
8. Wait for the build to finish.
9. Download the `windroid-lite-iso` artifact from the completed workflow run.

The artifact may contain:

- The generated ISO image.
- The SHA-256 checksum file.
- `build.log`, if generated.
- `workflow-container.log`, if generated.

A push of a Git tag matching `v*` also triggers the workflow.

### If the Build Fails

1. Open the failed GitHub Actions run.
2. Inspect the failed step.
3. Download the available log artifacts.
4. Find the first actual error in the logs.
5. Fix the underlying issue and rerun the workflow.

The workflow reports build failures; it does not automatically repair every package or source-code error.

## 4. Build Locally

### Debian 13

Open a terminal in the project directory and run:

```bash
chmod +x build.sh
sudo ./build.sh
```

### Ubuntu or Linux Mint with Docker

Install and configure Docker first, then run:

```bash
chmod +x build-docker.sh
./build-docker.sh
```

Check the build script for supported options before using additional flags.

### Expected Output

The expected output filenames are:

```text
windroid-lite-1.0-amd64.iso
windroid-lite-1.0-amd64.iso.sha256
```

The actual filenames depend on the build configuration.

## 5. Verify the ISO

After the build completes, verify the checksum.

```bash
sha256sum -c windroid-lite-1.0-amd64.iso.sha256
```

Run the command from the directory containing both files. The checksum file must reference the correct ISO filename.

Do not install an image if checksum verification fails.

## 6. Create a Bootable USB

You can use one of these tools:

- **Ventoy:** Install Ventoy on the USB drive, then copy the ISO onto it.
- **Rufus:** Select the ISO and follow the recommended writing mode.
- **balenaEtcher:** Select the ISO and target USB drive, then flash it.

Back up the USB drive first because creating bootable media can erase existing data.

Boot the laptop using its firmware boot menu. Test the live system before installing it.

## 7. Hardware Testing Checklist

Before calling this a final release, test the following:

- [ ] Wi-Fi adapter is detected.
- [ ] Wi-Fi networks appear and connections work.
- [ ] Wi-Fi reconnects after reboot.
- [ ] Touchpad pointer movement works.
- [ ] Tap-to-click works.
- [ ] Physical clicking and two-finger scrolling work.
- [ ] English locale is active.
- [ ] Login screen and installer branding are correct.
- [ ] Google Chrome launches and can browse websites.
- [ ] Wine launches a known compatible Windows application.
- [ ] Start-menu logout and session controls are visible.
- [ ] Boot animation works on the target laptop.
- [ ] Audio and Bluetooth work.
- [ ] Brightness and volume keys work.
- [ ] USB storage is detected.
- [ ] Suspend and resume work.
- [ ] Installer completes successfully.
- [ ] The installed OS boots successfully.
- [ ] The ISO checksum passes verification.

## 8. Windows Application Compatibility

Wine provides a compatibility layer for some Windows applications.

It does not guarantee that every `.exe` will work.

Compatibility depends on the application, CPU architecture, graphics support, libraries, and other requirements. Windows device drivers and some applications using unsupported kernel-level components will not work.

Install and test applications from trusted sources only.

## 9. Android Support

Waydroid is an optional Android container integration.

Compatibility depends on kernel binder support, graphics configuration, and available memory. Android apps may require additional downloads and configuration.

A laptop with limited RAM may experience slow performance when running Android applications alongside the desktop.

Google Play availability and certification are not guaranteed.

## 10. Branding and Fonts

WinDroid uses its own branding assets and Windows-inspired desktop styling.

Segoe UI is a proprietary font and is not bundled with this source kit. An available open-source alternative such as Selawik may be used instead.

Rebranding does not remove every Debian reference from a Debian-derived system. Package metadata, licenses, installer components, and low-level system files may retain upstream references.

## 11. Known Limitations

- The ISO has not yet been confirmed to build successfully in this environment.
- Hardware compatibility has not been verified on the target laptop.
- Wi-Fi support depends on the exact wireless chipset and available firmware.
- Tap-to-click depends on the device exposing the expected libinput configuration.
- Google Chrome requires network access during the build.
- Wine supports compatible Windows applications only.
- Plymouth animations may behave differently across firmware and graphics drivers.
- Waydroid requires additional configuration and suitable hardware support.
- Previously reported bugs from earlier conversations cannot be individually confirmed without their issue list and test results.

A successful source-code check is not proof that the complete ISO works.

## 12. Important Source Files

| File or directory | Purpose |
|---|---|
| `.github/workflows/build-iso.yml` | GitHub Actions ISO build workflow |
| `auto/config` | Live-build configuration |
| `build.sh` | Local ISO build script |
| `build-docker.sh` | Docker build helper |
| `config/package-lists/windroid.list.chroot` | Base package list |
| `config/hooks/live/0030-theme.hook.chroot` | Theme and font setup |
| `config/hooks/live/0040-branding.hook.chroot` | OS branding |
| `config/includes.chroot/usr/local/bin/` | WinDroid helper scripts |
| `config/includes.chroot/usr/share/` | Branding assets and themes |

Check the actual project tree before relying on any optional file path.

## 13. Troubleshooting

### Wi-Fi is missing

Check the adapter, kernel driver, firmware packages, rfkill status, and NetworkManager service.

```bash
rfkill list
nmcli device status
systemctl status NetworkManager
```

### Touchpad tap-to-click does not work

Check whether the device uses libinput and whether the session helper ran. Test the touchpad after logging out and back in.

### Google Chrome is missing

Inspect the build logs for download errors, repository errors, or package installation failures.

### A Windows `.exe` does not launch

Run it from a terminal to inspect the Wine error output. Some applications require additional dependencies or are not supported.

### GitHub Actions fails

Download the available build logs. Correct the first underlying error before rerunning the workflow.

## 14. Licensing

Review the licenses and redistribution terms for Debian packages, desktop themes, icons, fonts, Wine, Google Chrome, Waydroid, and all included branding assets.

Google Chrome is proprietary software and is downloaded from Google's official package source during the build.

Verify trademark availability and applicable licensing terms before publicly distributing the operating system.

---

**WinDroid Lite — Experimental Source Build Kit**

Build the ISO, verify its checksum, test it on real hardware, and publish a release only after successful validation.
