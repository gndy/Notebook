### I can not set shortcuts for lunching terminal from system settings.
### So using gsettings can help me do that.
### Very easy!!

#### Attention: it will overwrite any custom shortcuts existed already!!

- gsettings set org.gnome.settings-daemon.plugins.media-keys custom-keybindings "['/org/gnome/-settings-daemon/plugins/media-keys/custom-keybindings/custom0/']"
- gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/ name "open-terminal"
- gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/ command gnome-terminal
- gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/ binding "<Alt><Ctrl>T"
