# Flatpak-symlinker

## What it does:
This is a service that upon a flatpak being installed, will check what command it runs under the hood, and allows the user to call it directly.

This effectively means:
flatpak run org.mozilla.firefox -> firefox

It will also remove broken symlinks when an app is uninstalled.

Note this is currently only setup to check for USER installed flatpak's, not system-wide ones.

## Dependencies:

- inotify-tools
- flatpak (duhh)
- systemd to run the service, or any other init system with a custom service.

## Installation:
1. Ensure ~/.local/bin/ is in PATH. Most distros already have this setup, but if not, then: echo PATH="$HOME/.local/bin:$HOME/bin:$PATH" >> ~/.bash_profile
2. Copy flatpak-symlinker to ~/.local/bin and ensure it's executable (chmod +x flatpak-symlinker && cp flatpak-symlinker ~/.local/bin/)
3. If it does not already exist, create ~/.config/systemd/user, then copy the systemd service file over (mkdir -p ~/.config/systemd/user && cp flatpak-symlinker.service ~/.config/systemd/user/)
4. Daemon-reload systemd and enable the service. (systemctl --user daemon-reload && systemctl --user enable --now flatpak-symlinker)

## How it works:

It uses inotifywait to listen to the flatpak exports bin directory, and once it detects a file is MOVED_TO there, it fires. 

For this reason, make sure you already have inotify-tools installed on your machine, as per above.

When a flatpak app is installed, it creates a temporary file there called .exports.symlink.something, it then renames it to the flatpak name. This is the functionality that by default allows users to run a flatpak by its name rather than call 'flatpak run something.something.something'. Then, we check for the command that the flatpak runs inside it's container (you can view this from flatpak metadata), and create a new symlink in .local/bin/ following the command. This makes user based flatpak installs a bit more sane.

Upon file deletion from this directory, it simply removes broken symlinks in the directory, nothing fancy.

## Run options:

Run with `-h` to show help info:

```shell
flatpak-symlinker [-d] [-h] [-g]
-d: Disable overwrite functionality entirely
-h: Display this help message
-g: Generate symlinks for all installed flatpaks at once
```