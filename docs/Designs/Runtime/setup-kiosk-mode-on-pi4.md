# GoGauges Pi4 Fyne Kiosk Setup

Version: 0.1  
Target: Raspberry Pi 4 running a minimal Linux install that boots directly into the GoGauges Fyne display.

---

## 1. Goal

Set up a Raspberry Pi 4 as a single-purpose GoGauges in-vehicle display.

The target stack is:

```text
Raspberry Pi OS Lite 64-bit
  -> minimal X11 environment
  -> Go + Fyne build dependencies
  -> GoGauges daemon as a systemd service
  -> GoGauges Fyne display launched automatically on tty1
```

This avoids a full desktop environment while still supporting Fyne now.

---

## 2. Recommended OS

Use:

```text
Raspberry Pi OS Lite 64-bit
```

Reason:

- Official Raspberry Pi support.
- Minimal base image.
- Good Pi4 display, input, Bluetooth, Wi-Fi, USB and Mesa support.
- Debian-based, so package installation and systemd setup are predictable.
- Less yak-shaving than Alpine, Armbian, or custom kiosk distributions.

Avoid installing the full Raspberry Pi desktop unless you specifically want panels, menus, background services, and other tiny gremlins in waistcoats.

---

## 3. Update the Pi

Run:

```bash
sudo apt update
sudo apt full-upgrade -y
sudo reboot
```

After reboot:

```bash
sudo apt update
```

---

## 4. Install Go, Fyne and minimal X11 dependencies

Install the build tools, Fyne dependencies, Mesa/OpenGL support, and a minimal X11 kiosk environment:

```bash
sudo apt install -y \
  golang gcc git make pkg-config \
  libgl1-mesa-dev libgl1-mesa-dri mesa-utils \
  xorg-dev libxkbcommon-dev \
  xserver-xorg xinit openbox dbus-x11 \
  x11-xserver-utils unclutter
```

What this gives you:

- `golang`, `gcc`, `pkg-config`: Go/Fyne build chain.
- `libgl1-mesa-dev`, `libgl1-mesa-dri`, `mesa-utils`: graphics/OpenGL support.
- `xorg-dev`, `libxkbcommon-dev`: Fyne/Linux input and display dependencies.
- `xserver-xorg`, `xinit`: enough X11 to run the GUI.
- `openbox`: tiny window manager.
- `unclutter`: hides the mouse cursor.

No full desktop. No taskbar. No “helpful” notification daemon waving from the dashboard.

---

## 5. Create the GoGauges runtime user

Create a dedicated user:

```bash
sudo useradd -r -m -s /bin/bash gogauges || true
```

Add it to the groups needed for serial, display, rendering and input access:

```bash
sudo usermod -aG dialout,video,input,render,tty gogauges
```

Notes:

- `dialout` is usually needed for USB serial OBD adapters.
- `video` and `render` are needed for graphics access.
- `input` may be needed for touchscreen or input devices.
- `tty` is useful for console display/session behaviour.

Reboot after group changes:

```bash
sudo reboot
```

---

## 6. Prepare GoGauges directories

Create config and data directories:

```bash
sudo mkdir -p /etc/gogauges
sudo mkdir -p /var/lib/gogauges
sudo chown -R gogauges:gogauges /var/lib/gogauges
```

Create or copy your config file:

```bash
sudo nano /etc/gogauges/config.yaml
```

Adjust the config path later if your app expects a different file.

---

## 7. Build GoGauges with Fyne support

From your GoGauges repository on the Pi:

```bash
go mod tidy
go build -tags fyne -o bin/gogauges ./cmd/gogauges
```

Install the binary:

```bash
sudo install -m 0755 bin/gogauges /usr/local/bin/gogauges
```

Check it exists:

```bash
/usr/local/bin/gogauges --help
```

Planned display command once the Fyne renderer exists:

```bash
/usr/local/bin/gogauges display fyne --config /etc/gogauges/config.yaml
```

Important: this Pi setup makes the system Fyne-ready now. The `display fyne` command still needs to exist in the GoGauges codebase.

---

## 8. Create the daemon systemd service

Create the service file:

```bash
sudo tee /etc/systemd/system/gogauges.service >/dev/null <<'EOF'
[Unit]
Description=GoGauges OBD logger daemon
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=/usr/local/bin/gogauges daemon --config /etc/gogauges/config.yaml
Restart=always
RestartSec=5
User=gogauges
Group=gogauges
WorkingDirectory=/var/lib/gogauges

[Install]
WantedBy=multi-user.target
EOF
```

Enable and start it:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now gogauges.service
```

Check status:

```bash
systemctl status gogauges.service
```

View logs:

```bash
journalctl -u gogauges.service -f
```

Design rule: keep the daemon separate from the display. The daemon owns OBD and data capture. The display shows state. Do not let the GUI become the octopus holding all the spanners.

---

## 9. Create the Fyne kiosk startup script

Create `.xinitrc` for the `gogauges` user:

```bash
sudo -u gogauges tee /home/gogauges/.xinitrc >/dev/null <<'EOF'
#!/bin/sh

xset s off
xset -dpms
xset s noblank

unclutter -idle 0.2 -root &

exec openbox-session &
sleep 1

exec /usr/local/bin/gogauges display fyne --config /etc/gogauges/config.yaml
EOF

sudo chmod +x /home/gogauges/.xinitrc
```

This disables screen blanking and starts the GoGauges Fyne display inside X11.

---

## 10. Auto-start X on tty1

Add this to the `gogauges` user profile:

```bash
sudo -u gogauges tee -a /home/gogauges/.profile >/dev/null <<'EOF'

if [ -z "$DISPLAY" ] && [ "$(tty)" = "/dev/tty1" ]; then
    startx -- -nocursor
fi
EOF
```

---

## 11. Enable auto-login on tty1

Edit the systemd override:

```bash
sudo systemctl edit getty@tty1
```

Paste:

```ini
[Service]
ExecStart=
ExecStart=-/sbin/agetty --autologin gogauges --noclear %I $TERM
```

Apply:

```bash
sudo systemctl daemon-reload
sudo systemctl restart getty@tty1
```

On next boot, the Pi should:

1. Boot to tty1.
2. Auto-login as `gogauges`.
3. Run `startx`.
4. Launch the GoGauges Fyne display.

---

## 12. Test manually first

Before relying on auto-start, test as the `gogauges` user:

```bash
sudo -iu gogauges
startx
```

If X starts but the app fails, inspect:

```bash
cat /home/gogauges/.xsession-errors
journalctl -xe
```

Also test graphics:

```bash
glxinfo | grep -i renderer
```

If `glxinfo` is missing:

```bash
sudo apt install -y mesa-utils
```

---

## 13. Useful troubleshooting commands

Check daemon:

```bash
systemctl status gogauges.service
journalctl -u gogauges.service -f
```

Check boot/login:

```bash
systemctl status getty@tty1
```

Check groups:

```bash
id gogauges
```

Check OBD adapter:

```bash
ls -l /dev/ttyUSB* /dev/ttyACM* 2>/dev/null
dmesg | grep -i tty
```

Check display environment:

```bash
echo $DISPLAY
ps aux | grep -E 'Xorg|openbox|gogauges'
```

---

## 14. Optional hardening later

Do not do these until the app is working reliably. Premature hardening is how a five-minute job becomes a small religion.

Later improvements:

- Read-only root filesystem.
- Separate writable `/var/lib/gogauges`.
- Watchdog reboot.
- Automatic log rotation.
- Power-loss-safe shutdown strategy.
- Kiosk healthcheck service.
- Dedicated installer script.
- Systemd unit for display instead of `.profile` auto-start.
- Bluetooth OBD pairing automation if using Bluetooth.
- Hardware RTC if the vehicle is parked without network access.

---

## 15. Suggested repo branch for Fyne work

For the GoGauges code changes, branch from latest `main`:

```bash
git checkout main
git pull
git checkout -b add-fyne-display
```

Scope for that branch:

- Add the Fyne renderer.
- Keep it behind the `fyne` build tag.
- Add `gogauges display fyne`.
- Do not mix in daemon rewrites, OBD changes, or packaging work.

Keep the branch small. Small branches are like sharp chisels: useful, controllable, and less likely to remove a thumb.

---

## 16. References

- Raspberry Pi OS downloads: https://www.raspberrypi.com/software/operating-systems/
- Fyne Linux setup: https://docs.fyne.io/started/quick/
