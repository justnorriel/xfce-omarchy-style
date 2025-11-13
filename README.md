# XFCE with OmArchY-Style Configuration - Complete Setup

This document contains all keybindings and configurations applied to transform XFCE into an OmArchY-style setup.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Required Packages](#required-packages)
- [Scripts Created](#scripts-created)
- [Keybindings Reference](#keybindings-reference)
- [Configuration Files](#configuration-files)
- [Testing Commands](#testing-commands)

---

## Prerequisites

```bash
# Update system
sudo pacman -Syu

# Install base utilities
sudo pacman -S xdotool wmctrl
```

---

## Required Packages

### Core Utilities
```bash
sudo pacman -S picom rofi nitrogen maim xclip xdotool wmctrl \
  playerctl brightnessctl pamixer btop signal-desktop
```

### Fonts
```bash
sudo pacman -S ttf-cascadia-code-nerd noto-fonts noto-fonts-emoji
```

### Themes
```bash
sudo pacman -S arc-gtk-theme papirus-icon-theme
```

### Browser
- Zen Browser (install separately via AUR or AppImage)

---

## Scripts Created

All scripts are located in `~/.local/bin/`

### 1. omarchy-launch-browser
**Location:** `~/.local/bin/omarchy-launch-browser`

```bash
#!/bin/bash
# Launch Zen Browser
if [ "$1" = "--private" ]; then
    zen-browser --private-window &
else
    zen-browser &
fi
```

### 2. omarchy-launch-webapp
**Location:** `~/.local/bin/omarchy-launch-webapp`

```bash
#!/bin/bash
# Launch URL in app mode with Zen Browser
URL="$1"
zen-browser --new-window --kiosk "$URL" &
```

### 3. omarchy-launch-or-focus-webapp
**Location:** `~/.local/bin/omarchy-launch-or-focus-webapp`

```bash
#!/bin/bash
# Launch or focus web app with Zen Browser
TITLE="$1"
URL="$2"

if hyprctl clients | grep -q "$TITLE"; then
    hyprctl dispatch focuswindow "title:$TITLE"
else
    zen-browser --new-window --kiosk "$URL" &
fi
```

### 4. omarchy-menu
**Location:** `~/.local/bin/omarchy-menu`

```bash
#!/bin/bash
# OmArchY menu (using rofi)
rofi -show drun
```

### 5. omarchy-launch-or-focus
**Location:** `~/.local/bin/omarchy-launch-or-focus`

```bash
#!/bin/bash
# Launch app or focus if already running
CLASS="$1"
CMD="$2"

if wmctrl -lx | grep -i "$CLASS"; then
    wmctrl -xa "$CLASS"
else
    eval "$CMD" &
fi
```

### 6. omarchy-screenshot-selection
**Location:** `~/.local/bin/omarchy-screenshot-selection`

```bash
#!/bin/bash
maim -s | xclip -selection clipboard -t image/png
```

### 7. omarchy-screenshot-file
**Location:** `~/.local/bin/omarchy-screenshot-file`

```bash
#!/bin/bash
mkdir -p ~/Pictures/Screenshots
maim -s ~/Pictures/Screenshots/$(date +'%Y%m%d_%H%M%S.png')
```

### 8. omarchy-lock-screen
**Location:** `~/.local/bin/omarchy-lock-screen`

```bash
#!/bin/bash
xflock4
```

### 9. omarchy-workspace-switch
**Location:** `~/.local/bin/omarchy-workspace-switch`

```bash
#!/bin/bash
# Switch to workspace by number (1-5)
WORKSPACE=$1
wmctrl -s $((WORKSPACE - 1))
```

### 10. omarchy-window-move-workspace
**Location:** `~/.local/bin/omarchy-window-move-workspace`

```bash
#!/bin/bash
# Move active window to workspace
WORKSPACE=$1
wmctrl -r :ACTIVE: -t $((WORKSPACE - 1))
```

### 11. omarchy-smart-copy
**Location:** `~/.local/bin/omarchy-smart-copy`

```bash
#!/bin/bash
# Smart copy - uses Ctrl+Shift+C for terminals, Ctrl+C for everything else

ACTIVE_WINDOW=$(xdotool getactivewindow getwindowclassname)

case "$ACTIVE_WINDOW" in
    Alacritty|kitty|*terminal*|*Terminal*)
        xdotool key --clearmodifiers ctrl+shift+c
        ;;
    *)
        xdotool key --clearmodifiers ctrl+c
        ;;
esac
```

### 12. omarchy-smart-paste
**Location:** `~/.local/bin/omarchy-smart-paste`

```bash
#!/bin/bash
# Smart paste - uses Ctrl+Shift+V for terminals, Ctrl+V for everything else

ACTIVE_WINDOW=$(xdotool getactivewindow getwindowclassname)

case "$ACTIVE_WINDOW" in
    Alacritty|kitty|*terminal*|*Terminal*)
        xdotool key --clearmodifiers ctrl+shift+v
        ;;
    *)
        xdotool key --clearmodifiers ctrl+v
        ;;
esac
```

### Make all scripts executable:
```bash
chmod +x ~/.local/bin/omarchy-*
```

---

## Keybindings Reference

### Core Applications
**Location:** Settings → Keyboard → Application Shortcuts

| Keybind | Command | Description |
|---------|---------|-------------|
| `Super + Return` | `alacritty` | Terminal |
| `Super + Alt + Space` | `~/.local/bin/omarchy-menu` | App Launcher (Rofi) |
| `Super + Shift + F` | `nautilus --new-window` | File Manager |
| `Super + Shift + B` | `~/.local/bin/omarchy-launch-browser` | Browser |
| `Super + Shift + Alt + B` | `~/.local/bin/omarchy-launch-browser --private` | Browser (Private) |
| `Super + Shift + N` | `code` | Editor |
| `Super + Shift + T` | `alacritty -e btop` | Activity Monitor |

### Window Management
**Location:** Settings → Keyboard → Application Shortcuts

| Keybind | Command | Description |
|---------|---------|-------------|
| `Super + W` | `xdotool getactivewindow windowkill` | Close Window |
| `Super + F` | `wmctrl -r :ACTIVE: -b toggle,fullscreen` | Fullscreen Toggle |
| `Super + C` | `~/.local/bin/omarchy-smart-copy` | Copy (Smart) |
| `Super + V` | `~/.local/bin/omarchy-smart-paste` | Paste (Smart) |
| `Super + L` | `xflock4` | Lock Screen |

### Screenshots
**Location:** Settings → Keyboard → Application Shortcuts

| Keybind | Command | Description |
|---------|---------|-------------|
| `Print` | `~/.local/bin/omarchy-screenshot-selection` | Screenshot to Clipboard |
| `Super + Print` | `~/.local/bin/omarchy-screenshot-file` | Screenshot to File |
| `Super + Shift + S` | `~/.local/bin/omarchy-screenshot-selection` | Screenshot Selection |

### Media Controls
**Location:** Settings → Keyboard → Application Shortcuts

| Keybind | Command | Description |
|---------|---------|-------------|
| `XF86AudioRaiseVolume` | `pamixer -i 5` | Volume Up |
| `XF86AudioLowerVolume` | `pamixer -d 5` | Volume Down |
| `XF86AudioMute` | `pamixer -t` | Mute Toggle |
| `XF86AudioPlay` | `playerctl play-pause` | Play/Pause |
| `XF86AudioNext` | `playerctl next` | Next Track |
| `XF86AudioPrev` | `playerctl previous` | Previous Track |
| `XF86MonBrightnessUp` | `brightnessctl set +5%` | Brightness Up |
| `XF86MonBrightnessDown` | `brightnessctl set 5%-` | Brightness Down |

### Workspace Switching
**Location:** Settings → Keyboard → Application Shortcuts

| Keybind | Command | Description |
|---------|---------|-------------|
| `Super + 1` | `xfce4-workspace-switch --ws 0` | Switch to Workspace 1 |
| `Super + 2` | `xfce4-workspace-switch --ws 1` | Switch to Workspace 2 |
| `Super + 3` | `xfce4-workspace-switch --ws 2` | Switch to Workspace 3 |
| `Super + 4` | `xfce4-workspace-switch --ws 3` | Switch to Workspace 4 |
| `Super + 5` | `xfce4-workspace-switch --ws 4` | Switch to Workspace 5 |

### Move Window to Workspace
**Location:** Settings → Keyboard → Application Shortcuts

| Keybind | Command | Description |
|---------|---------|-------------|
| `Super + Shift + 1` | `~/.local/bin/omarchy-window-move-workspace 1` | Move to Workspace 1 |
| `Super + Shift + 2` | `~/.local/bin/omarchy-window-move-workspace 2` | Move to Workspace 2 |
| `Super + Shift + 3` | `~/.local/bin/omarchy-window-move-workspace 3` | Move to Workspace 3 |
| `Super + Shift + 4` | `~/.local/bin/omarchy-window-move-workspace 4` | Move to Workspace 4 |
| `Super + Shift + 5` | `~/.local/bin/omarchy-window-move-workspace 5` | Move to Workspace 5 |

---

## Configuration Files

### 1. Picom Configuration
**Location:** `~/.config/picom/picom.conf`

Provides transparency, shadows, and blur effects. See full configuration in the detailed guide.

**Key Settings:**
- Backend: GLX
- Shadow radius: 16px
- Inactive opacity: 95%
- Blur method: dual_kawase
- Blur strength: 5

### 2. Rofi Configuration
**Location:** `~/.config/rofi/config.rasi`

OmArchY-style app launcher with dark theme.

**Key Settings:**
- Background: #1e1e2e
- Foreground: #cdd6f4
- Font: CaskaydiaMono Nerd Font 11
- Width: 600px
- No border radius

### 3. Alacritty Configuration
**Location:** `~/.config/alacritty/alacritty.toml`

Terminal with transparency and Catppuccin colors.

**Key Settings:**
- Opacity: 0.9
- Font: CaskaydiaMono Nerd Font 11
- Background: #1e1e2e
- Foreground: #cdd6f4

### 4. XFCE Panel Configuration
**Location:** Settings → Panel → Panel Preferences

**Panel 1 (Top Bar):**
- Height: 26 pixels
- Background: #1e1e2e
- Items: Workspace Switcher, Clock (center), System monitors, Audio, Network, Battery

### 5. XFCE Window Manager
**Location:** Settings → Window Manager

**Style:**
- Theme: Adwaita-dark or Arc-Dark
- Font: CaskaydiaMono Nerd Font 10

**Advanced:**
- XFCE Compositor: **DISABLED** (using picom instead)

### 6. Workspace Configuration
**Location:** Settings → Workspaces

- Number of workspaces: **5**
- Names: 1, 2, 3, 4, 5

### 7. Autostart Applications
**Location:** Settings → Session and Startup → Application Autostart

Add these applications:

1. **Picom**
   - Command: `picom --config ~/.config/picom/picom.conf`

2. **Nitrogen** (Wallpaper)
   - Command: `nitrogen --restore`

3. **Clipboard Manager**
   - Command: `xfce4-clipman`

---

## Testing Commands

### Test Scripts
```bash
# Test each script individually
~/.local/bin/omarchy-menu
~/.local/bin/omarchy-launch-browser
~/.local/bin/omarchy-screenshot-selection
```

### Test Media Controls
```bash
# Volume
pamixer -i 5      # Increase volume
pamixer -d 5      # Decrease volume
pamixer -t        # Toggle mute

# Playback
playerctl play-pause
playerctl next
playerctl previous

# Brightness
brightnessctl set +5%
brightnessctl set 5%-
```

### Test Window Management
```bash
# Close active window
xdotool getactivewindow windowkill

# Toggle fullscreen
wmctrl -r :ACTIVE: -b toggle,fullscreen

# List all windows
wmctrl -l
```

### Test Workspaces
```bash
# Switch to workspace 3 (index 2)
wmctrl -s 2

# List all workspaces
wmctrl -d

# Move active window to workspace 4 (index 3)
wmctrl -r :ACTIVE: -t 3
```

### Test Copy/Paste
```bash
# Simulate Ctrl+C
xdotool key ctrl+c

# Simulate Ctrl+V
xdotool key ctrl+v

# Check active window class
xdotool getactivewindow getwindowclassname
```

---

## Troubleshooting

### Scripts not working?
```bash
# Check if scripts exist
ls -l ~/.local/bin/omarchy-*

##Make sure they're executable
chmod +x ~/.local/bin/omarchy-*

# Check if ~/.local/bin is in PATH
echo $PATH | grep .local/bin
```

### Keybindings not working?
```bash
# List all shortcuts
xfconf-query -c xfce4-keyboard-shortcuts -lv | grep -i super

# Test if key is recognized
xev | grep -i super
```

### Picom not starting?
```bash
# Kill any existing picom
killall picom

# Start with config
picom --config ~/.config/picom/picom.conf

# Check for errors in output
```

### Media keys not recognized?
```bash
# Test with xev
xev | grep -i audio

# Check if playerctl works
playerctl status
```

---

## Quick Reference Card

```
╔══════════════════════════════════════════════════╗
║         XFCE OMARCHY KEYBINDINGS                 ║
╠══════════════════════════════════════════════════╣
║ APPLICATIONS                                     ║
║ Super + Return          = Terminal               ║
║ Super + Alt + Space     = App Menu               ║
║ Super + Shift + F       = File Manager           ║
║ Super + Shift + B       = Browser                ║
║                                                  ║
║ WINDOW MANAGEMENT                                ║
║ Super + W               = Close Window           ║
║ Super + F               = Fullscreen             ║
║ Super + C               = Copy (Smart)           ║
║ Super + V               = Paste (Smart)          ║
║ Super + L               = Lock Screen            ║
║                                                  ║
║ WORKSPACES                                       ║
║ Super + 1-5             = Switch Workspace       ║
║ Super + Shift + 1-5     = Move Window            ║
║                                                  ║
║ SCREENSHOTS                                      ║
║ Print                   = Screenshot Selection   ║
║ Super + Print           = Screenshot to File     ║
║                                                  ║
║ MEDIA (Function Keys)                            ║
║ Volume Up/Down/Mute     = Media Keys             ║
║ Play/Pause/Next/Prev    = Media Keys             ║
║ Brightness Up/Down      = Function Keys          ║
╚══════════════════════════════════════════════════╝
```

---

## Backup & Restore

### Backup Your Configuration
```bash
# Create backup directory
mkdir -p ~/xfce-omarchy-backup

# Backup all configs
cp -r ~/.config/picom ~/xfce-omarchy-backup/
cp -r ~/.config/rofi ~/xfce-omarchy-backup/
cp -r ~/.config/alacritty ~/xfce-omarchy-backup/
cp -r ~/.local/bin/omarchy-* ~/xfce-omarchy-backup/

# Export XFCE settings
xfconf-query -c xfce4-keyboard-shortcuts -lv > ~/xfce-omarchy-backup/keybindings.txt
```

### Restore Configuration
```bash
# Restore configs
cp -r ~/xfce-omarchy-backup/picom ~/.config/
cp -r ~/xfce-omarchy-backup/rofi ~/.config/
cp -r ~/xfce-omarchy-backup/alacritty ~/.config/
cp ~/xfce-omarchy-backup/omarchy-* ~/.local/bin/
chmod +x ~/.local/bin/omarchy-*
```

---

## Uninstall / Revert

To remove all OmArchY customizations:

```bash
# Remove scripts
rm ~/.local/bin/omarchy-*

# Remove configs
rm -rf ~/.config/picom
rm -rf ~/.config/rofi

# Uninstall packages (optional)
sudo pacman -Rns picom rofi nitrogen xdotool wmctrl

# Re-enable XFCE compositor
# Settings → Window Manager Tweaks → Compositor
# Check "Enable display compositing"

# Reset keyboard shortcuts
# Settings → Keyboard → Application Shortcuts
# Remove all custom omarchy shortcuts
```

---

## Additional Resources

- **Hyprland Wiki:** https://wiki.hyprland.org
- **XFCE Documentation:** https://docs.xfce.org
- **Arch Wiki - XFCE:** https://wiki.archlinux.org/title/Xfce
- **Picom:** https://github.com/yshui/picom
- **Rofi:** https://github.com/davatorium/rofi

---

## Credits

- **OmArchY OS:** Original inspiration for keybindings and aesthetics
- **CachyOS:** Base distribution
- **XFCE:** Desktop environment
- **Hyprland:** Original window manager design

---

**Last Updated:** 2024
**Configuration Version:** 1.0
