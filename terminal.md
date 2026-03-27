# Configuration Terminal — Alacritty + Zellij + Zsh + Starship
---

## Prérequis

- Un système Fedora ou Debian/Ubuntu à jour
- Un thème Plasma cohérent avec le terminal (ex. Utterly Nord, Utterly Sweet, Catppuccin)
- Un accès sudo

---

## 1. Installation des paquets

### Fedora

```bash
# Terminal + multiplexer + shell
sudo dnf install alacritty cargo zsh

cargo install zellij

# Outils CLI modernes
sudo dnf install fzf ripgrep fd-find


```

> Sur Fedora, le binaire s'appelle directement `fd`. Pas besoin de symlink.

### Debian / Ubuntu

```bash
sudo apt update
sudo apt install alacritty zellij zsh fonts-jetbrains-mono
sudo apt install fzf ripgrep fd-find

# Sur Debian/Ubuntu, le binaire s'appelle "fdfind" — on crée un alias
mkdir -p ~/.local/bin
ln -sf $(which fdfind) ~/.local/bin/fd
```
## PATH

```bash
chsh -s /bin/zsh
mkdir -p ~/.local/bin
echo 'export PATH="$HOME/.cargo/bin:$HOME/.local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

---

## 2. Nerd Font (obligatoire pour les icônes)

La font `jetbrains-mono-fonts` du dépôt ne contient **pas** les glyphes Nerd Font. Sans ça, Starship et la barre Zellij afficheront des carrés `□` à la place des icônes.

```bash
# Telecharger la Nerd Font JetBrains Mono
NERD_FONT_VERSION="v3.3.0"
FONT_DIR="$HOME/.local/share/fonts/JetBrainsMonoNerd"

mkdir -p "$FONT_DIR"
curl -fLo /tmp/JetBrainsMono.zip \
  "https://github.com/ryanoasis/nerd-fonts/releases/download/${NERD_FONT_VERSION}/JetBrainsMono.zip"
unzip -o /tmp/JetBrainsMono.zip -d "$FONT_DIR"
rm /tmp/JetBrainsMono.zip

# Rafraichir le cache des fonts
fc-cache -fv
```

Pour vérifier l'installation :

```bash
fc-list | grep -i "JetBrains.*Nerd"
```

---

## 3. Passer Zsh en shell par défaut

```bash
chsh -s $(which zsh)
```

Déconnecte-toi et reconnecte-toi pour que le changement prenne effet.

---

## 4. Installer Starship (prompt cross-shell)

```bash
curl -sS https://starship.rs/install.sh | sh
```

---

## 5. Configuration Zsh

Fichier : `~/.zshrc`

```zsh
# -------------------------------------------
# Starship prompt
# -------------------------------------------
eval "$(starship init zsh)"

# -------------------------------------------
# Lancer Zellij automatiquement
# "exec" remplace le processus shell courant :
# quand on quitte Zellij, le terminal se ferme proprement
# au lieu de retomber dans un shell "nu" inutile.
# -------------------------------------------
if [[ -z "$ZELLIJ" ]]; then
  exec zellij
fi

# -------------------------------------------
# Aliases utiles
# -------------------------------------------
alias ll="ls -lah --color=auto"
alias grep="grep --color=auto"

# -------------------------------------------
# Historique
# -------------------------------------------
HISTFILE=~/.zsh_history
HISTSIZE=10000
SAVEHIST=10000
setopt SHARE_HISTORY
setopt HIST_IGNORE_DUPS
```

> **Pourquoi `exec` ?** Sans `exec`, quand tu quittes Zellij tu retombes dans le Zsh parent qui l'a lancé. Avec `exec`, le shell est remplacé par Zellij : quitter Zellij = fermer le terminal. C'est le comportement attendu.

---

## 6. Configuration Alacritty

Fichier : `~/.config/alacritty/alacritty.toml`

```toml
# -------------------------------------------
# Font — utiliser la version Nerd Font
# -------------------------------------------
[font]
size = 12.0

[font.normal]
family = "JetBrainsMono Nerd Font"
style = "Regular"

[font.bold]
family = "JetBrainsMono Nerd Font"
style = "Bold"

[font.italic]
family = "JetBrainsMono Nerd Font"
style = "Italic"

# -------------------------------------------
# Fenetre
# -------------------------------------------
[window]
padding = { x = 8, y = 8 }
opacity = 0.95
decorations = "None"

# -------------------------------------------
# Desactiver les keybinds qui entrent en conflit avec Zellij
# Alacritty capture Ctrl+Shift+<key> par defaut,
# ce qui empeche Zellij de recevoir ces raccourcis.
# -------------------------------------------
[keyboard]
bindings = [
  # Liberer Ctrl+Shift+D/E/X/H/L/K/J/T/W/Z pour Zellij
  { key = "D",     mods = "Control|Shift", action = "ReceiveChar" },
  { key = "E",     mods = "Control|Shift", action = "ReceiveChar" },
  { key = "X",     mods = "Control|Shift", action = "ReceiveChar" },
  { key = "H",     mods = "Control|Shift", action = "ReceiveChar" },
  { key = "L",     mods = "Control|Shift", action = "ReceiveChar" },
  { key = "K",     mods = "Control|Shift", action = "ReceiveChar" },
  { key = "J",     mods = "Control|Shift", action = "ReceiveChar" },
  { key = "T",     mods = "Control|Shift", action = "ReceiveChar" },
  { key = "W",     mods = "Control|Shift", action = "ReceiveChar" },
  { key = "Z",     mods = "Control|Shift", action = "ReceiveChar" },
]

# -------------------------------------------
# Theme de couleurs — Nord
# Adapter selon le theme Plasma choisi (Nord, Catppuccin, etc.)
# -------------------------------------------
[colors.primary]
background = "#2E3440"
foreground = "#D8DEE9"

[colors.normal]
black   = "#3B4252"
red     = "#BF616A"
green   = "#A3BE8C"
yellow  = "#EBCB8B"
blue    = "#81A1C1"
magenta = "#B48EAD"
cyan    = "#88C0D0"
white   = "#E5E9F0"

[colors.bright]
black   = "#4C566A"
red     = "#BF616A"
green   = "#A3BE8C"
yellow  = "#EBCB8B"
blue    = "#81A1C1"
magenta = "#B48EAD"
cyan    = "#8FBCBB"
white   = "#ECEFF4"
```

> **Important :** La section `[keyboard] bindings` avec `ReceiveChar` est ce qui permet à Zellij de recevoir les raccourcis `Ctrl+Shift+<key>`. Sans ça, Alacritty les intercepte et les raccourcis Zellij ne fonctionnent pas.

---

## 7. Configuration Zellij

Fichier : `~/.config/zellij/config.kdl`

```kdl
// -------------------------------------------
// Apparence
// -------------------------------------------
default_layout "compact"

// Choisir UN theme coherent avec Alacritty et Plasma
// Options courantes : "nord", "catppuccin-mocha", "tokyo-night", "dracula"
theme "nord"

// -------------------------------------------
// Keybinds personnalises
// -------------------------------------------
keybinds {
    normal {
        // -- Splits --
        bind "Ctrl Shift d" { NewPane "Right"; }
        bind "Ctrl Shift e" { NewPane "Down"; }
        bind "Ctrl Shift x" { CloseFocus; }

        // -- Navigation vim-style --
        bind "Ctrl Shift h" { MoveFocus "Left"; }
        bind "Ctrl Shift l" { MoveFocus "Right"; }
        bind "Ctrl Shift k" { MoveFocus "Up"; }
        bind "Ctrl Shift j" { MoveFocus "Down"; }

        // -- Resize --
        bind "Ctrl Shift Alt h" { Resize "Increase Left"; }
        bind "Ctrl Shift Alt l" { Resize "Increase Right"; }
        bind "Ctrl Shift Alt k" { Resize "Increase Up"; }
        bind "Ctrl Shift Alt j" { Resize "Increase Down"; }

        // -- Zoom (toggle fullscreen sur le pane actif) --
        bind "Ctrl Shift z" { ToggleFocusFullscreen; }

        // -- Tabs --
        bind "Ctrl Shift t" { NewTab; }
        bind "Ctrl Shift w" { CloseTab; }
        bind "Ctrl Tab" { GoToNextTab; }
        bind "Ctrl Shift Tab" { GoToPreviousTab; }
    }
}
```

---

## 8. Configuration Starship (optionnel)

Fichier : `~/.config/starship.toml`

```toml
# Prompt minimaliste, adapte selon tes gouts
# Doc complete : https://starship.rs/config/

format = """
$directory\
$git_branch\
$git_status\
$kubernetes\
$docker_context\
$python\
$nodejs\
$character
"""

[character]
success_symbol = "[➜](bold green)"
error_symbol = "[✗](bold red)"

[directory]
truncation_length = 3
truncate_to_repo = true

[git_branch]
format = "[$symbol$branch]($style) "

[kubernetes]
disabled = false
format = "[$symbol$context( \\($namespace\\))]($style) "

[docker_context]
disabled = false
```

---

## Cohérence des thèmes

Pour un rendu visuel homogène, aligner les thèmes sur toute la stack :

| Composant | Fichier de config | Thème |
|-----------|-------------------|-------|
| Plasma | Paramètres KDE | Utterly Nord / Catppuccin |
| Alacritty | `alacritty.toml` → `[colors]` | Nord / Catppuccin |
| Zellij | `config.kdl` → `theme` | `"nord"` / `"catppuccin-mocha"` |
| Starship | `starship.toml` | Suit les couleurs du terminal |

> Mélanger Nord côté Plasma avec Tokyo Night côté terminal donne un résultat visuellement incohérent. Choisir une palette et s'y tenir.

---

## Vérification post-installation

```bash
# Verifier que Zsh est le shell par defaut
echo $SHELL

# Verifier la Nerd Font
fc-list | grep -i "JetBrains.*Nerd"

# Verifier que Starship fonctionne
starship --version

# Lancer Alacritty et verifier que Zellij demarre automatiquement
alacritty
```

Si les icônes du prompt ou de la barre Zellij s'affichent en carrés `□`, c'est que la Nerd Font n'est pas correctement détectée : vérifier le nom exact de la famille dans `alacritty.toml` avec `fc-list`.
