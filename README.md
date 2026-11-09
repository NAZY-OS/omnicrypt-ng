# OmniCrypt

LUKS2 container images, ext4 filesystems and fscrypt directories — with a
GTK 4 interface and a system tray icon.

**License:** GPL-3.0-or-later
**Version:** 0.1.0

---

## 1. Overview

OmniCrypt creates encrypted container image files, formats them with LUKS2,
creates an ext4 filesystem inside and mounts them at a mount point. It also
manages fscrypt encrypted directories for per-file encryption.

```
                        +----------------------------+
                        |       omnicrypt-gui       |
                        |          (GTK 4)          |
                        |  sidebar | log | menubar  |
                        +-------------+--------------+
                                      |
                    stdin / stdout   |   "show" | "umount" | "quit"
                                      v
                        +----------------------------+
                        |       tray_helper.py      |
                        |   (GTK 3 + AyatanaApp-    |
                        |      Indicator3)          |
                        +----------------------------+

              omnicrypt-gui  -->  pkexec  -->  omnicrypt-helper.sh
                                                    |
                          +-------------+-----------+-------------+
                          |             |           |             |
                      losetup      cryptsetup   mkfs.ext4      fscrypt
```

### Why two processes?

`AyatanaAppIndicator3` (the modern, desktop-neutral tray library) is bound to
GTK 3. PyGObject cannot load GTK 3 and GTK 4 into the same process. So the
main window stays pure GTK 4 and the tray icon lives in a tiny GTK 3 helper
that talks to the GUI over stdin/stdout pipes.

---

## 2. Dependencies

All dependencies are OS packages — OmniCrypt uses PyGObject bindings, not pip
wheels. A generic list lives in `requirements.txt`; exact package names differ
per distribution.

| Purpose | Debian / Ubuntu | Fedora | Arch | openSUSE | Alpine |
|---|---|---|---|---|---|
| Build | `meson ninja-build pkg-config desktop-file-utils appstream` | `meson ninja-build pkgconf-pkg-config desktop-file-utils appstream` | `meson ninja pkgconf desktop-file-utils appstream` | `meson ninja pkg-config desktop-file-utils appstream` | `meson ninja-build pkgconf desktop-file-utils appstream` |
| GTK 4 | `gir1.2-gtk-4.0 libgtk-4-1 libadwaita-1-0` | `python3-gobject gtk4 libadwaita` | `python-gobject gtk4 libadwaita` | `python3-gobject gtk4 libadwaita-1` | `py3-gobject gtk4` |
| GTK 3 (tray) | `gir1.2-gtk-3.0 libgtk-3-0` | `python3-gobject gtk3` | `python-gobject gtk3` | `python3-gobject gtk3` | `py3-gobject gtk3` |
| Tray | `gir1.2-ayatanaappindicator3-0.1` | `libayatana-appindicator-gtk3` | `libayatana-appindicator` | `libayatana-appindicator-gtk3` | `libayatana-appindicator` |
| Notify | `libnotify-bin gir1.2-notify-0.7` | `libnotify` | `libnotify` | `libnotify-tools` | `libnotify` |
| Crypto | `cryptsetup-bin fscrypt udisks2 pkexec policykit-1` | `cryptsetup fscrypt udisks2 polkit` | `cryptsetup fscrypt udisks libpolkit` | `cryptsetup fscrypt udisks2 polkit` | `cryptsetup fscrypt udisks polkit` |
| Misc | `python3-gi e2fsprogs util-linux hicolor-icon-theme` | `python3-gobject e2fsprogs util-linux hicolor-icon-theme` | `python-gobject e2fsprogs util-linux hicolor-icon-theme` | `python3-gobject e2fsprogs util-linux hicolor-icon-theme` | `py3-gobject e2fsprogs util-linux hicolor-icon-theme` |

---

## 3. Install dependencies

### Debian / Ubuntu / Linux Mint / Pop!_OS

```bash
sudo apt update
sudo apt install $(tr '\n' ' ' < requirements.txt)
```

### Fedora / RHEL / Rocky / AlmaLinux

```bash
sudo dnf install -y meson ninja-build pkgconf-pkg-config desktop-file-utils \
    appstream python3-gobject gtk4 gtk3 libadwaita \
    libayatana-appindicator-gtk3 libnotify cryptsetup fscrypt udisks \
    polkit e2fsprogs util-linux hicolor-icon-theme
```

### Arch / Manjaro / EndeavourOS

```bash
sudo pacman -S --needed meson ninja pkgconf desktop-file-utils appstream \
    python-gobject gtk4 gtk3 libadwaita libayatana-appindicator \
    libnotify cryptsetup fscrypt udisks libpolkit e2fsprogs util-linux \
    hicolor-icon-theme
```

### openSUSE / SLES

```bash
sudo zypper install meson ninja pkg-config desktop-file-utils appstream \
    python3-gobject gtk4 gtk3 libadwaita libayatana-appindicator-gtk3 \
    libnotify-tools cryptsetup fscrypt udisks2 polkit e2fsprogs util-linux \
    hicolor-icon-theme
```

### Alpine Linux

```bash
sudo apk add meson ninja-build pkgconf desktop-file-utils appstream \
    py3-gobject gtk4 gtk3 libayatana-appindicator libnotify cryptsetup \
    fscrypt udisks polkit e2fsprogs util-linux hicolor-icon-theme
```

---

## 4. Build

```bash
meson setup builddir --prefix=/usr
ninja -C builddir
```

Run straight from the build tree, without installing:

```bash
python3 src/omnicrypt_gui.py
```

Build options:

```bash
meson setup builddir --prefix=/usr -Dtray=false   # build without the tray helper
```

| Option | Default | Meaning |
|---|---|---|
| `tray` | `true` | Install `tray_helper.py` (GTK 3 tray icon) |

---

## 5. Install

### From the build tree (Meson install)

```bash
sudo meson install -C builddir
sudo ldconfig
```

This installs:

```
/usr/bin/omnicrypt-gui
/usr/bin/tray_helper.py
/usr/lib/omnicrypt/omnicrypt-helper.sh
/usr/share/applications/org.omnicrypt.OmniCrypt.desktop
/usr/share/icons/hicolor/scalable/apps/org.omnicrypt.OmniCrypt.svg
/usr/share/metainfo/org.omnicrypt.OmniCrypt.metainfo.xml
```

### Uninstall

```bash
sudo meson uninstall -C builddir
```

### Package managers (optional)

The project is a plain Meson project and installs cleanly with most distro
packaging tools, e.g.:

```bash
# Fedora / openSUSE
sudo dnf install ./omnicrypt-0.1.0-1.noarch.rpm

# Arch
yay -Si omnicrypt

# Debian (after adding a debian/ directory)
dpkg-buildpackage -us -uc -b
sudo apt install ../omnicrypt_0.1.0-1_all.deb
```

---

## 6. Usage

1. Start OmniCrypt from your application menu, or run `omnicrypt-gui`.
2. **Create LUKS image** — choose a target file, enter a passphrase twice.
   OmniCrypt creates the image, attaches a loop device, formats LUKS2, writes
   an ext4 filesystem and mounts it.
3. **Open LUKS image** — select an existing image and unlock it.
4. **Create fscrypt dir** — pick a directory; it is encrypted with a random
   64-byte key stored in `~/.config/omnicrypt/fscrypt.key` (mode 0600).
5. **Unlock fscrypt dir** — decrypt the directory again with that key.
6. **Close volume** — unmount, close the LUKS mapper and detach the loop
   device.

The command entry at the bottom accepts shortcuts:

```
help | create-luks | open-luks | fscrypt-create | fscrypt-unlock | close
```

Closing the window hides it into the tray; use the tray menu to quit.

---

## 7. Privileged operations

Everything that needs root (loop devices, `cryptsetup`, `mkfs`, `mount`,
`fscrypt`) is delegated to `omnicrypt-helper.sh`, which the GUI launches via
`pkexec`. The user is asked for the admin password once per helper run.

Set `OMNICRYPT_HELPER` to point at a different helper path if the install
location differs:

```bash
OMNICRYPT_HELPER=/usr/local/lib/omnicrypt/omnicrypt-helper.sh omnicrypt-gui
```

---

## 8. Project layout

```
omnicrypt/
├── meson.build                     # top-level build definition
├── meson_options.txt               # build options (-Dtray=...)
├── requirements.txt                # dependencies for all distros
├── README.md                       # this file
├── data/
│   ├── meson.build
│   ├── org.omnicrypt.OmniCrypt.desktop
│   ├── org.omnicrypt.OmniCrypt.svg
│   └── org.omnicrypt.OmniCrypt.metainfo.xml
└── src/
    ├── meson.build
    ├── omnicrypt_gui.py            # GTK 4 main window
    ├── omnicrypt_core.py           # flat shared logic
    ├── omnicrypt_helper.sh         # privileged helper (pkexec)
    └── tray_helper.py              # GTK 3 tray icon
```

| File | Role |
|---|---|
| `omnicrypt_gui.py` | GTK 4 window, sidebar actions, log view, menus, notifications |
| `omnicrypt_core.py` | All logic as flat functions with explicit return values |
| `omnicrypt_helper.sh` | Every privileged command, one case per operation |
| `tray_helper.py` | GTK 3 tray icon, sends commands to the GUI over pipes |

---

## 9. Troubleshooting

**"Requiring namespace 'Gtk' version '3.0', but '4.0' is already loaded"**
The GUI and the tray must stay in separate processes. Do not import
`tray_helper` from `omnicrypt_gui.py`, and do not add a `gi.require_version`
for GTK 3 in the GUI.

**Tray icon does not appear**
Check that `gir1.2-ayatanaappindicator3-0.1` is installed and that the icon
name `org.omnicrypt.OmniCrypt` resolves. Verify with:
`gtk-launch org.omnicrypt.OmniCrypt`

**`pkexec` fails / helper not found**
The helper lives in `$prefix/lib/omnicrypt/`. Set `OMNICRYPT_HELPER` to the
correct path.

**Loop device errors**
`user.max_loop` may be too low. Raise it in `/etc/sysctl.d/` and reload, or
reboot.

---

## 10. License

GPL-3.0-or-later
