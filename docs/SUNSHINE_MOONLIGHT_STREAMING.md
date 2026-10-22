# Headless TV Streaming: Sunshine Host + Moonlight Client

Why run a long, tripping HDMI cable across your living room rug when you can turn your revived laptop into a silent, hidden home console?

This guide covers installing and configuring **Sunshine** on Batocera and streaming games in 1080p/60fps to **Moonlight** on a living room TV client (Amazon Fire TV Stick, Google TV Chromecast, Apple TV, or Smart TV).

---

## 🏗️ The Hardware Topology

```
+------------------------------------+           Tailscale Mesh / LAN
| Central Storage (Homelab NAS)      | <=================================>
| • Samba CIFS v3.0 ROMs & Saves     |
+------------------------------------+
                  |
                  v
+------------------------------------+           5GHz Wi-Fi / LAN
| ThinkPad T440p (Batocera + Sunshine)| <=================================>
| • Clamshell Closed Lid             |      (Ultra-Low Latency H.264 Stream)
| • Native Linux Bluetooth Stack     |
+------------------------------------+
                  ^                                     |
                  | Bluetooth BLE                       v
+------------------------------------+ +----------------------------------+
| Wireless Gamepads                  | | Living Room TV (Display Receiver)|
| (8BitDo Ultimate / Xbox Wireless)  | | • Amazon Fire TV Stick / Apple TV|
| • Native Hotkeys & Rumble Feedback | | • Running Moonlight Client       |
+------------------------------------+ +----------------------------------+
```

---

## 📦 Step 1: Install Sunshine on Batocera

Batocera v38+ includes built-in Flatpak support.

1. SSH into Batocera:
   ```bash
   ssh root@<BATOCERA_IP>
   # Default password is: linux
   ```
2. Install the official Sunshine Flatpak:
   ```bash
   flatpak install --system dev.lizardbyte.app.Sunshine
   ```

---

## ⚙️ Step 2: Configure Sunshine for Headless Operation

Sunshine enforces strict Cross-Origin Resource Sharing (CORS) and Cross-Site Request Forgery (CSRF) protections that block web UI access across local subnets by default.

Edit `/userdata/system/configs/sunshine/sunshine.conf`:

```ini
port = 47989
origin_pin_allowed = 0.0.0.0/0
origin_web_ui_allowed = 0.0.0.0/0
csrf_allowed_origins = https://<BATOCERA_IP>:47990,http://<BATOCERA_IP>:47990

# Video and audio settings for Intel VAAPI hardware acceleration:
encoder = vaapi
capture = kms
fps = [60]
min_log_level = info
audio_sink = auto
```

---

## 🚀 Step 3: Autostart Sunshine at Boot

Add the startup command to `/userdata/system/custom.sh`:

```bash
mkdir -p /userdata/system/logs
flatpak run dev.lizardbyte.app.Sunshine > /userdata/system/logs/sunshine.log 2>&1 &
```

Ensure the script is executable:
```bash
chmod +x /userdata/system/custom.sh
```

---

## 📺 Step 4: Pair with Living Room Moonlight Client

1. On your streaming device (Amazon Fire TV Stick, Android TV, Apple TV), open the App Store and install **Moonlight Game Streaming**.
2. Launch Moonlight. It will automatically discover the Batocera Sunshine host on your local network.
3. Select the host. Moonlight will display a **4-digit PIN**.
4. On your phone or laptop browser, navigate to:
   ```
   https://<BATOCERA_IP>:47990
   ```
5. Log in (default: choose your admin username/password on initial visit), click the **PIN** tab, and enter the 4-digit PIN.
6. The host is paired!

---

## 🎮 Step 5: Direct Controller Pairing to Batocera & Clamshell Mode

Rather than pairing your controllers to the Fire TV Stick or Android TV (which often introduces noticeable Bluetooth latency, lacks rumble, and swallows custom emulator hotkeys), **pair your wireless gamepads directly to Batocera on the ThinkPad**:

1. In Batocera's EmulationStation menu, navigate to **Controller & Bluetooth Settings** -> **Pair a Bluetooth Device**.
2. Put your controller (8BitDo, Xbox Wireless, DualShock 4, or DualSense) into pairing mode.
3. Once paired, Batocera natively handles:
   * **Sub-millisecond input responsiveness** with direct Linux evdev kernel polling.
   * **Full Dual-Motor Rumble / Haptic Feedback**.
   * **Global Hotkey Shortcuts:** `Hotkey + Start` (Exit game), `Hotkey + Y` (Quick Save), `Hotkey + X` (Quick Load), and `Hotkey + R2` (Fast-Forward).
4. Launch **Moonlight** on your TV stick. The TV acts strictly as a high-fps video and audio receiver.
5. **Close the laptop lid!** The machine will continue encoding and streaming silently in clamshell mode without overheating.
