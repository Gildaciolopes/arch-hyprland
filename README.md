# arch-hyprland

Rice do Arch Linux + Hyprland. Este guia reproduz o ambiente do zero, do ISO até o desktop pronto.

**Preview rápido do que você vai ter:**

- Hyprland com bordas Catppuccin Mauve, blur e animações suaves
- Waybar com módulos flutuantes (ilha style)
- Terminal Kitty com fundo transparente e cor roxa
- Prompt de duas linhas com Starship (Arch icon + git status)
- awww como daemon de wallpaper com transição animada
- tty-clock, cmatrix e cava abrindo automaticamente no login

---

## Requisitos

- ISO do Arch Linux (boot via USB)
- Placa NVIDIA? Leia a seção de drivers antes de reiniciar
- Conexão com internet

---

## 1. Instalar o Arch Linux

No boot pelo ISO, conecte à internet e rode:

```bash
archinstall
```

Configurações recomendadas no `archinstall`:

- **Disk:** particionamento guiado, ext4, separar `/home`
- **Bootloader:** GRUB
- **Profile:** minimal (sem desktop — vamos instalar o Hyprland manualmente)
- **Audio:** pipewire
- **Network:** NetworkManager
- **Locale:** pt_BR.UTF-8 (ou o seu)
- **Timezone:** America/Sao_Paulo (ou o seu)
- Crie um usuário com permissão sudo

---

## 2. Primeira inicialização — pacotes base

Após o reboot, logue no TTY e instale tudo de uma vez:

```bash
sudo pacman -Syu

# Base do Wayland + Hyprland
sudo pacman -S hyprland waybar kitty wofi dunst grim slurp wl-clipboard \
  cliphist xdg-desktop-portal xdg-desktop-portal-hyprland \
  polkit-kde-agent network-manager-applet networkmanager \
  brightnessctl playerctl pipewire pipewire-alsa pipewire-pulse \
  pipewire-jack wireplumber pavucontrol

# GTK / Qt theming
sudo pacman -S arc-gtk-theme nwg-look nwg-bar kvantum kvantum-qt5 qt5ct qt6ct

# Fontes
sudo pacman -S ttf-jetbrains-mono-nerd noto-fonts noto-fonts-emoji ttf-dejavu ttf-liberation

# Terminal extras (abrem automaticamente no login)
sudo pacman -S tty-clock cmatrix cava pipes.sh fastfetch starship fzf tree jq

# Ferramentas de sistema
sudo pacman -S btop htop earlyoom ufw zip unzip unrar p7zip ffmpeg \
  gst-libav gst-plugins-bad gst-plugins-good gst-plugins-ugly gst-plugin-pipewire

# Bluetooth
sudo pacman -S blueman bluez bluez-utils

# Multilib (Steam, Wine etc.) — ativar em /etc/pacman.conf se necessário
sudo pacman -S lib32-nvidia-utils  # só se tiver NVIDIA
```

---

## 3. Instalar o yay (AUR helper)

```bash
sudo pacman -S --needed base-devel git
git clone https://aur.archlinux.org/yay.git
cd yay && makepkg -si
cd .. && rm -rf yay
```

### Pacotes AUR

```bash
yay -S awww hyprshot ttf-apple-emoji
```

> `awww` é o daemon de wallpaper usado aqui (tem suporte a transições animadas).

---

## 3.1 Emojis da Apple

Depois de instalar `ttf-apple-emoji`, configure o fontconfig para que **todos** os apps usem os emojis da Apple por padrão (sobrescrevendo Noto Emoji):

```bash
mkdir -p ~/.config/fontconfig
```

Crie `~/.config/fontconfig/fonts.conf`:

```xml
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "urn:fontconfig:fonts.dtd">
<fontconfig>
  <!-- Prefere Apple Color Emoji em qualquer pedido de "emoji" -->
  <alias>
    <family>emoji</family>
    <prefer>
      <family>Apple Color Emoji</family>
    </prefer>
  </alias>

  <!-- Fallback para sans-serif, serif e monospace -->
  <match target="pattern">
    <test qual="any" name="family"><string>sans-serif</string></test>
    <edit name="family" mode="append" binding="weak">
      <string>Apple Color Emoji</string>
    </edit>
  </match>

  <match target="pattern">
    <test qual="any" name="family"><string>serif</string></test>
    <edit name="family" mode="append" binding="weak">
      <string>Apple Color Emoji</string>
    </edit>
  </match>

  <match target="pattern">
    <test qual="any" name="family"><string>monospace</string></test>
    <edit name="family" mode="append" binding="weak">
      <string>Apple Color Emoji</string>
    </edit>
  </match>

  <!-- Substitui Noto Color Emoji pelo Apple quando os dois estão instalados -->
  <match target="pattern">
    <test qual="any" name="family"><string>Noto Color Emoji</string></test>
    <edit name="family" mode="prepend" binding="strong">
      <string>Apple Color Emoji</string>
    </edit>
  </match>
</fontconfig>
```

Atualize o cache de fontes:

```bash
fc-cache -fv
```

> O `noto-fonts-emoji` continua instalado como fallback, mas o Apple Color Emoji tem prioridade em tudo.

---

## 4. Driver NVIDIA

```bash
sudo pacman -S nvidia-open-dkms linux-headers linux-lts-headers dkms libva-nvidia-driver
```

Adicione `nvidia_drm.modeset=1` nos parâmetros do kernel (`/etc/default/grub`) e regenere:

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

> Se estiver usando kernel LTS como fallback, instale também `linux-lts` e `linux-lts-headers`.

---

## 5. Otimizações do sistema

### GRUB — boot silencioso

Edite `/etc/default/grub`:

```
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet"
GRUB_CMDLINE_LINUX="zswap.enabled=0 rootfstype=ext4"
```

Aplique:

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

> `zswap.enabled=0` desativa o cache de swap comprimido do kernel (desnecessário com swapfile simples).

### Pacman — downloads paralelos

Edite `/etc/pacman.conf` e descomente/adicione:

```
ParallelDownloads = 15
ILoveCandy
Color
```

### earlyoom — mata processos antes do sistema travar

```bash
sudo systemctl enable --now earlyoom
```

Edite `/etc/default/earlyoom`:

```
EARLYOOM_ARGS="-r 3600 -n --avoid '(^|/)(init|systemd|Xorg|sshd)$'"
```

### zram — swap comprimido na RAM

O sistema usa **zram** como swap primário (prioridade 100) e um swapfile de 8 GB como secundário (prioridade -2). O zram comprime dados com `zstd` e ocupa metade da RAM disponível — resultado: sistema responde melhor sob pressão de memória sem precisar tocar o disco.

Instale o gerador:

```bash
sudo pacman -S zram-generator
```

Crie `/etc/systemd/zram-generator.conf`:

```ini
[zram0]
zram-size = ram / 2
compression-algorithm = zstd
```

Ative:

```bash
sudo systemctl daemon-reload
sudo systemctl start systemd-zram-setup@zram0.service
```

Verifique:

```bash
zramctl
swapon --show
```

Deve aparecer `/dev/zram0` com prioridade 100 e seu swapfile com prioridade -2.

> Importante: o kernel param `zswap.enabled=0` (configurado no GRUB) desativa o zswap para não conflitar com o zram. Os dois fazem coisas parecidas — usar os dois ao mesmo tempo desperdiça RAM.

### fstrim — saúde do SSD

```bash
sudo systemctl enable fstrim.timer
```

### Serviços para habilitar

```bash
sudo systemctl enable NetworkManager bluetooth docker earlyoom fstrim.timer
```

---

## 6. Aplicar os dotfiles

```bash
# Clone o repo
git clone https://github.com/seu-usuario/arch-hyprland.git
cd arch-hyprland

# Copie tudo para ~/.config
cp -r .config/* ~/.config/
```

Ou link simbólico por arquivo se preferir:

```bash
ln -sf $(pwd)/.config/hypr/hyprland.conf ~/.config/hypr/hyprland.conf
ln -sf $(pwd)/.config/waybar/config.jsonc ~/.config/waybar/config.jsonc
# etc...
```

---

## 7. Configurar o bash

Adicione ao final do `~/.bashrc`:

```bash
alias hypr='dbus-run-session Hyprland'

export JAVA_HOME=/usr/lib/jvm/default
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/emulator:$ANDROID_HOME/platform-tools

fastfetch
eval "$(starship init bash)"
```

---

## 8. GTK — tema Arc-Dark

Após copiar os dotfiles, rode o nwg-look para aplicar o tema:

```bash
nwg-look
```

Ou edite manualmente `~/.config/gtk-3.0/settings.ini` e `~/.config/gtk-4.0/settings.ini`:

```ini
[Settings]
gtk-theme-name=Arc-Dark
gtk-icon-theme-name=Adwaita
gtk-font-name=Adwaita Sans 11
gtk-cursor-theme-name=default
gtk-cursor-theme-size=24
gtk-application-prefer-dark-theme=0
```

---

## 9. Wallpaper

Coloque seu wallpaper em `~/Imagens/Wallpapers/`. O autostart no `hyprland.conf` espera:

```
~/Imagens/Wallpapers/Meptl.png
```

Para mudar, edite a linha no `hyprland.conf`:

```
exec-once = sleep 1 && awww img ~/Imagens/Wallpapers/SEU_WALLPAPER.png \
  --transition-type grow --transition-duration 1.5 --transition-fps 60 \
  --transition-pos center --transition-bezier 0.25,1,0.25,1
```

---

## 10. Iniciar o Hyprland

No TTY:

```bash
hypr
# (alias para: dbus-run-session Hyprland)
```

Se tiver display manager (sddm, gdm), selecione "Hyprland" na sessão.

---

## Atalhos principais

| Tecla                      | Ação                                                       |
| -------------------------- | ---------------------------------------------------------- |
| `SUPER + Return`           | Kitty                                                      |
| `SUPER + A`                | Wofi (launcher)                                            |
| `SUPER + Space`            | Dolphin (files)                                            |
| `SUPER + Q`                | Fechar janela                                              |
| `SUPER + F`                | Fullscreen                                                 |
| `SUPER + S`                | Toggle floating                                            |
| `SUPER + J`                | Toggle split                                               |
| `SUPER + Escape`           | Reiniciar Waybar                                           |
| `SUPER + B`                | Zen Browser                                                |
| `SUPER + G`                | VS Code                                                    |
| `Print`                    | Screenshot da tela toda                                    |
| `SUPER + Print`            | Screenshot de região → salva em `~/Imagens/screenshot.png` |
| `SUPER + 1..0`             | Trocar workspace                                           |
| `SUPER + SHIFT + 1..0`     | Mover janela para workspace                                |
| `SUPER + mouse drag`       | Mover janela                                               |
| `SUPER + ALT + mouse drag` | Redimensionar janela                                       |

---

## Problemas comuns

**Hyprland não inicia / tela preta com NVIDIA**  
Confirme que `nvidia_drm.modeset=1` está nos parâmetros do kernel e que o `nvidia-open-dkms` foi compilado para o seu kernel atual (`dkms status`).

**Portal não funciona (screenshots, Flatpak)**

```bash
systemctl --user restart xdg-desktop-portal xdg-desktop-portal-hyprland
```

**Waybar sumiu**  
`SUPER + Escape` reinicia. Se não voltar: `killall waybar; waybar &`

**awww: command not found**  
Instale pelo AUR: `yay -S awww`

**Sem som**

```bash
systemctl --user enable --now pipewire pipewire-pulse wireplumber
```
