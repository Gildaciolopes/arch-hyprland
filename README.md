# arch-hyprland - meu rice do Arch + Hyprland

Este repositório guarda as minhas configurações (`dotfiles`) para Arch Linux + Hyprland. O objetivo é permitir que você replique o meu ambiente (terminais, atalhos, aparências, fontes e utilitários) rápida e seguramente.

**Aviso:** este README contém instruções avançadas que assumem familiaridade com o processo de instalação do Arch Linux e permissões de administrador. Use com cuidado e adapte caminhos/nomes conforme seu hardware.

**Sumário rápido**

- **Requisitos:** instalação do Arch Linux com acesso à internet
- **Instalação com:** `archinstall` (perfil automático + perfil customizável)
- **Principais pacotes:** `hyprland`, `kitty`, `waybar`, `wofi`, `swww`, `dunst`, `grim`, `slurp`, `wl-clipboard`, `xdg-desktop-portal`, `xdg-desktop-portal-hyprland` (AUR)
- **Configuração:** copie os arquivos em `~/.config/` e ajuste `hyprland.conf` conforme necessário

**O que tem neste repositório**

- `.config/hypr/hyprland.conf` — configuração principal do Hyprland (monitor, binds, autostart, aparência)
- `.config/kitty/kitty.conf` — tema/opacidade do terminal

## Como usar

1. Preparar a instalação do Arch
   - Inicialize pelo ISO do Arch e conecte à internet.
   - Instale o `archinstall` (se não estiver no ISO atual):

```bash
pacman -Sy archinstall
```

2. Instalação automatizada (exemplo com `archinstall`)
   - Crie um perfil JSON mínimo (exemplo `profile.json`) ou use a opção interativa.
   - Exemplo mínimo de comando interativo:

```bash
archinstall
# Siga as perguntas: escolha idioma, timezone, particionamento (manual ou guiado), bootloader,
# criar usuário, locale e selecione "desktop" (ou instale sem desktop e adicione pacotes depois).
```

    - Para um perfil automatizado, crie `profile.json` e rode:

```bash
archinstall --config profile.json
```

3. Pacotes recomendados (após chroot/primeira inicialização)
   - Pacotes essenciais Wayland + Hyprland:

```bash
sudo pacman -Syu hyprland waybar kitty wofi wl-clipboard grim slurp swaybg swayidle swaylock cliphist dunst network-manager-applet nm-connection-editor
```

    - Outros utilitários úteis:

```bash
sudo pacman -S xdg-desktop-portal xdg-desktop-portal-gtk polkit-kde-agent-1 brightnessctl playerctl cava cmatrix
```

    - AUR (alguns podem estar em AUR, use `paru`/`yay`):

```bash
# Exemplo: instalar xdg-desktop-portal-hyprland via AUR
yay -S xdg-desktop-portal-hyprland swww
```

Observação: confirme se `swww` e `xdg-desktop-portal-hyprland` estão disponíveis nos repositórios ou AUR na data em que for instalar.

4. Portais e captura de tela

- Instale `xdg-desktop-portal` e um backend compatível (`xdg-desktop-portal-hyprland`) para integração com screenshots e diálogos do Flatpak.
- Use `grim` + `slurp` para capturas no Hyprland. Exemplo de bind já presente no `hyprland.conf`.

5. Fontes e ícones recomendados

- JetBrainsMono Nerd Font (ou outra Nerd Font) — terminal e ícones de status
- Inter / Noto Sans — interface
- Material Icons / Font Awesome — ícones de aplicativos

Instalação rápida de fontes (exemplo):

```bash
sudo pacman -S ttf-jetbrains-mono ttf-dejavu noto-fonts
# Para Nerd Fonts via AUR
yay -S ttf-jetbrains-mono-nerd
```

6. Como aplicar estes dotfiles (passo a passo)
   - Faça backup das suas configs atuais.
   - Copie os arquivos do repositório para `~/.config`:

```bash
mkdir -p ~/.config/hypr ~/.config/kitty
cp -r .config/hypr/hyprland.conf ~/.config/hypr/hyprland.conf
cp -r .config/kitty/kitty.conf ~/.config/kitty/kitty.conf
```

    - Garanta permissões e reinicie o Hyprland (logout/login) ou reinicie a sessão Wayland.

## Trechos importantes das suas configurações

Vou incluir aqui os trechos principais detectados nos seus arquivos para facilitar a compreensão.

Monitor

```
monitor = eDP-1,2560x1600@60,0x0,1
```

Terminal e programas padrão

```
$terminal = kitty
$fileManager = dolphin
$menu = wofi --show drun --allow-images
```

Autostart (exemplos já configurados)

```
exec-once = dbus-update-activation-environment --systemd WAYLAND_DISPLAY XDC_CURRENT_DESKTOP
exec-once = systemctl --user import-enviroment WAYLAND_DISPLAY XDG_CURRENT_DESKTOP
exec-once = waybar
exec-once = swww-daemon & swww img Imagens/Wallpapers/Meptl.png
exec-once = dunst
exec-once = nm-applet --indicator
exec-once = wl-paste --type text --watch cliphist store
exec-once = /usr/lib/polkit-kde-authentication-agent-1
```

Aparência e comportamento (resumo)

- Gaps internos/externos: `gaps_in = 5`, `gaps_out = 8`
- Bordas: `border_size = 2` com cores usando `rgba( cba6f7ee )`
- Layout padrão: `dwindle` com `pseudotile = true`
- Decoração: `rounding = 10`, opacidades `active_opacity = 1.0`, `inactive_opacity = 0.8`
- Blur ativado (`enabled = true`) com `size = 3`

Keybinds relevantes

```
bind = $mainMod, Return, exec, $terminal
bind = $mainMod, B, exec, zen-browser
bind = $mainMod, G, exec, code
bind = $mainMod, Print, exec, grim -g "$(slurp)" - | tee ~/Imagens/screenshot.png | wl-copy
```

Kitty (tema mínimo detectado)

```
color5  #cba6f7
color13 #cba6f7
background_opacity 0.8
cursor #cba6f7
```

## Personalize antes de usar

- Ajuste `monitor` caso seu monitor tenha outro identificador ou resolução.
- Revise `exec-once` para remover aplicativos que você não quer iniciar.
- Verifique variáveis `env` (ex.: `GTK_THEME`, `QT_QPA_PLATFORMTHEME`) para combinar com seu tema.

## Problemas comuns e soluções rápidas

- Nada aparece na sessão Wayland: verifique `sddm`/`gdm` ou inicie `Hyprland` manualmente (`Hyprland` no tty).
- Portal não funciona para screenshots/flatpaks: instale `xdg-desktop-portal-hyprland` e reinicie o serviço `systemctl --user restart xdg-desktop-portal`.
- Atalhos não funcionam: confira se o `mainMod` está definido (`$mainMod = SUPER`).
