# ErinOS ISO

Arch Linux installer for the ErinOS appliance. Builds a bootable ISO using archiso. GitHub Actions builds automatically on every push to main.


## What the Installer Does

The installer (`erinos-install`) is an interactive 7-step process powered by `gum`:

1. **Check internet** — Pings archlinux.org. Offers nmtui for WiFi if no connection.
2. **Select disk** — Shows available disks, user picks one.
3. **Disk encryption** — Prompts for a LUKS2 passphrase. Full-disk encryption is mandatory.
4. **Partition and format** — 512MB EFI + encrypted root (ext4).
5. **Install base system** — Pacstrap with all packages. Copies [erinos-core](https://github.com/erinos-ai/erinos-core) to `/opt/erinos`. Loads `.env` from USB if present.
6. **Configure system** — Locale, timezone, hostname, systemd-boot, LUKS, NetworkManager, mDNS. Creates `erinos` user. Installs Ruby 4.0.0 via rbenv. Creates systemd services.
7. **Set root password**


## First Boot

After installation, a one-shot service (`erinos-firstboot`) runs automatically:

- Waits for network
- Pulls the Ollama model (if configured)
- Downloads the Whisper speech recognition model
- Initializes the database
- Installs default skills
- Enables and starts all services

A console display (`erinos-console`) runs on tty1 showing the ErinOS logo, SSH info, and an activity log.


## Building the ISO

Push to `main` or create a tag for a release:

```bash
git tag v1.0.0
git push --tags
```

Download the ISO from the GitHub Actions artifacts tab, or from the GitHub Release.


## Flashing a USB Drive

The `dev/flash` script in [erinos-core](https://github.com/erinos-ai/erinos-core) automates this:

```bash
./dev/flash
```

It downloads the latest ISO, opens `.env` for editing, flashes to USB, and copies your `.env` to the EFI partition.


## USB Configuration

The installer loads a pre-configured `.env` from the USB drive's EFI partition. This lets you set up WiFi, model, Telegram token, and other settings before installation.


## Installing on Hardware

1. Flash the ISO to a USB drive
2. Boot from USB (F12 for boot menu)
3. Run `erinos-install`
4. Follow the 7 steps
5. Remove USB and reboot
6. Enter LUKS passphrase
7. Wait for first-boot setup to complete

Target hardware: Framework Desktop (Ryzen AI Max+ 395, 128GB, RDNA 3.5 iGPU).


## Profile Structure

```
airootfs/               Root filesystem overlay
  usr/local/bin/         erinos-install, erinos-firstboot, erinos-console
  usr/local/lib/erinos/  Shared shell library
efiboot/                Bootloader config
packages.x86_64         Package list
pacman.conf             Pacman config
profiledef.sh           archiso profile definition
```
