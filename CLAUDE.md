# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Dotfiles for an Arch Linux + Hyprland desktop environment. All files under `.config/` mirror `~/.config/` on the live system. Applying a change means copying the file and reloading the relevant daemon.

## Applying changes to the live system

```bash
# Hyprland
cp .config/hypr/hyprland.conf ~/.config/hypr/hyprland.conf
hyprctl reload

# Waybar
cp .config/waybar/config.jsonc ~/.config/waybar/config.jsonc
cp .config/waybar/style.css ~/.config/waybar/style.css
killall waybar && waybar &

# Kitty (takes effect on next window)
cp .config/kitty/kitty.conf ~/.config/kitty/kitty.conf

# Starship (takes effect on next shell)
cp .config/starship.toml ~/.config/starship.toml

# Fastfetch
cp .config/fastfetch/config.jsonc ~/.config/fastfetch/config.jsonc
```

## Config file map

| File | Purpose |
|------|---------|
| `.config/hypr/hyprland.conf` | Main Hyprland config: monitor, autostart, env vars, keybinds, window rules, look & feel |
| `.config/waybar/config.jsonc` | Waybar module layout and per-module settings |
| `.config/waybar/style.css` | Waybar CSS (island-style floating modules) |
| `.config/kitty/kitty.conf` | Terminal colors, opacity, keymaps |
| `.config/starship.toml` | Two-line bash prompt with git branch/status |
| `.config/wofi/config` | App launcher settings |
| `.config/wofi/style.css` | Wofi styling |
| `.config/nwg-bar/bar.json` | Power menu buttons (logout / reboot / shutdown) |
| `.config/nwg-bar/style.css` | Power menu styling |
| `.config/nwg-look/config` | GTK appearance export settings |
| `.config/fastfetch/config.jsonc` | System info layout |
| `.config/fastfetch/logo/a.txt` | Custom ASCII logo |

---

## System

**Hardware:** NVIDIA GeForce RTX 5050 Laptop GPU · NVMe SSD (ext4 `/` + ext4 `/home` + swapfile)  
**Kernel:** Running dual: `linux` (latest) and `linux-lts` as fallback  
**Display:** `eDP-1` — 2560×1600 @ 60 Hz, scale 1  
**Keyboard:** Brazilian ABNT2 (`br` / `abnt2`)

---

## Design / Aesthetic

**Colorscheme:** Catppuccin Mocha  
**Accent color:** `#cba6f7` (Mauve) — used in borders, kitty cursor, starship prompt, cava, waybar highlights  
**Background colors:** `#1e1e2e` (base) · `#11111b` (crust) · `#313244` (surface0)  
**GTK theme:** Arc-Dark  
**Cursor:** default, size 24  
**Window rounding:** 10px · border 2px · inactive opacity 0.8 · blur enabled

---

## Fonts

| Package | Use |
|---------|-----|
| `ttf-jetbrains-mono-nerd` | Terminal (Kitty) + Waybar icons |
| `noto-fonts` + `noto-fonts-emoji` | UI fallback + emoji |
| `ttf-apple-emoji` | Apple-style emoji |
| `ttf-dejavu` | General fallback |
| `ttf-liberation` | Office/web compatibility |

Install:
```bash
sudo pacman -S ttf-jetbrains-mono-nerd noto-fonts noto-fonts-emoji ttf-dejavu ttf-liberation
yay -S ttf-apple-emoji
```

---

## Package list

### Core Wayland / Hyprland stack
```bash
sudo pacman -S hyprland waybar kitty wofi dunst grim slurp wl-clipboard \
  cliphist xdg-desktop-portal xdg-desktop-portal-hyprland \
  polkit-kde-agent network-manager-applet networkmanager \
  brightnessctl playerctl pipewire pipewire-alsa pipewire-pulse \
  pipewire-jack wireplumber pavucontrol
```

### Wallpaper daemon (awww — AUR, supports transitions)
```bash
yay -S awww
# Used in autostart with grow transition:
# awww-daemon
# sleep 1 && awww img ~/Imagens/Wallpapers/Meptl.png \
#   --transition-type grow --transition-duration 1.5 --transition-fps 60 \
#   --transition-pos center --transition-bezier 0.25,1,0.25,1
```

### GTK / Qt theming
```bash
sudo pacman -S arc-gtk-theme nwg-look kvantum kvantum-qt5 qt5ct qt6ct
```

### Terminal extras (launched by Hyprland at startup)
```bash
sudo pacman -S tty-clock cmatrix cava pipes.sh
yay -S lavat   # lava-lamp terminal animation
```

### System tools
```bash
sudo pacman -S btop htop fastfetch starship fzf tree jq \
  earlyoom ufw gst-libav gst-plugins-bad gst-plugins-good \
  gst-plugins-ugly gst-plugin-pipewire ffmpeg
```

### Bluetooth
```bash
sudo pacman -S blueman bluez bluez-utils
sudo systemctl enable --now bluetooth
```

### Compression / file utils
```bash
sudo pacman -S zip unzip unrar p7zip
```

### AUR helper
```bash
# Install yay first, then use it for AUR packages
yay -S awww hyprshot spicetify-cli lavat
```

---

## System optimizations

### earlyoom — OOM killer before the kernel freezes
```bash
sudo pacman -S earlyoom
sudo systemctl enable --now earlyoom
```
Config at `/etc/default/earlyoom`:
```
EARLYOOM_ARGS="-r 3600 -n --avoid '(^|/)(init|systemd|Xorg|sshd)$'"
```
Reports every hour, sends desktop notification, avoids killing init/sshd.

### zswap disabled — kernel param
`/etc/default/grub` has `GRUB_CMDLINE_LINUX="zswap.enabled=0 rootfstype=ext4"`.  
zswap is disabled in favor of a plain swapfile (no compressed swap cache).

### fstrim — SSD health
```bash
sudo systemctl enable fstrim.timer
```
Runs weekly TRIM on the NVMe.

### pacman — parallel downloads + eye candy
`/etc/pacman.conf` has:
```
ParallelDownloads = 15
ILoveCandy
```

### GRUB — quiet boot
```
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet"
```

### pacman cache bind mount
`/etc/fstab` binds `/home/.system/cache/pacman/pkg` → `/var/cache/pacman/pkg` so the package cache survives root partition changes.

---

## Enabled systemd services

| Service | Purpose |
|---------|---------|
| `NetworkManager` | Network |
| `bluetooth` | Bluetooth |
| `docker` | Containers |
| `earlyoom` | OOM protection |
| `mariadb` | Local DB |
| `fstrim.timer` | Weekly SSD TRIM |
| `systemd-timesyncd` | NTP time sync |

---

## Shell — bash + starship

`~/.bashrc` key lines:
```bash
alias hypr='dbus-run-session Hyprland'   # launch Hyprland from TTY
fastfetch                                 # show sysinfo on shell open
eval "$(starship init bash)"

export JAVA_HOME=/usr/lib/jvm/default
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/emulator:$ANDROID_HOME/platform-tools
export PATH=$PATH:~/.spicetify
```

Starship config is at `.config/starship.toml` — two-line prompt with Catppuccin purple/pink colors, Arch icon, git branch + status.

---

## NVIDIA setup

GPU: RTX 5050 Laptop (Blackwell GB207M)  
Driver: `nvidia-open-dkms` (open kernel module variant)  
```bash
sudo pacman -S nvidia-open-dkms linux-headers linux-lts-headers dkms libva-nvidia-driver
```
`yt6801-dkms` is also installed (network card DKMS driver).

---

## Key design choices

- **Wallpaper daemon:** `awww` (AUR) — not `swww`. Used for animated grow-transition on startup.
- **Screenshot:** `Print` → `grimbllast copy screen` (full screen); `SUPER+Print` → `grim -g "$(slurp)"` saves to `~/Imagens/screenshot.png` and copies to clipboard. `hyprshot` is also installed as alternative.
- **Power menu:** `⏻` button in Waybar calls `nwg-bar`. `SUPER+Escape` kills/restarts Waybar.
- **Clipboard manager:** `cliphist` stores both text and images via `wl-paste` watchers.
- **Touchpad:** `tap-to-click = yes`, `natural_scroll = no`.
- **Terminal decorations at startup:** `tty-clock`, `cmatrix -C magenta`, `cava`, and `lavat -c magenta` each open in a separate Kitty window on login (cava delayed 3 s, lavat delayed 5 s to avoid race).
- **Spicetify:** Spotify theming via `spicetify-cli` — config lives in `~/.config/spicetify`.
