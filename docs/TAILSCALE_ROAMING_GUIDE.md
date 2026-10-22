# Autonomous Tailscale Roaming & Host Drive Protection

This guide details how to build a **Portable Batocera Live USB** that boots cleanly on foreign laptops anywhere in the world, securely accesses your central home ROM library over Tailscale, and completely protects the host machine's internal drives.

---

## 🛑 The Critical Gotcha: Host NVMe LUKS Password Prompt

When booting vanilla Batocera from a USB drive on a modern laptop (such as an Arch Linux or Windows machine with BitLocker or LUKS encryption), Batocera executes an early boot probe:

```bash
# /etc/init.d/S11share
luks_enabled="$(/usr/bin/batocera-settings-get -f "$SHARECONFFILE" luks.enabled)"
if [ "${luks_enabled}" = "1" -o -z "${luks_enabled}" ]; then
    prepareLUKS
fi
```

Because `luks.enabled` is undefined by default, Batocera scans all block devices on the laptop, detects the host's encrypted NVMe partitions (e.g. `/dev/nvme1n1p2`), halts boot, switches to `tty3`, and demands a password!

### The Fix: Inhibit LUKS Probe

In `/boot/batocera-boot.conf` on the USB drive, explicitly set:

```ini
sharedevice=INTERNAL
luks.enabled=0
```

And in the bootloader kernel cmdline (`/EFI/BOOT/grub.cfg` and `syslinux.cfg`), append:

```text
rd.luks=0 luks=0
```

**Result:** Batocera boots directly to its internal USB share in under 15 seconds, never touching or probing the host laptop's internal operating system or files.

---

## 🌐 Autonomous Tailscale Mesh Integration

> [!WARNING]
> **Status: Experimental / Work-In-Progress (Community RFC)**  
> While the **Host Drive LUKS bypass** (`luks.enabled=0`) and **Local LAN CIFS mounting** work 100% reliably, roaming autonomously over public/hotel Wi-Fi via an embedded Tailscale daemon on a live USB is currently an open experimental track. See the [Community RFC](#-community-call-for-contributions-rfc) below to help refine this workflow!

When roaming on a hotel Wi-Fi, coffee shop, or friend's house, your laptop does not have local network access to your home server's LAN IP (`192.168.1.x`).

By running Tailscale in the background:
1. Batocera boots without blocking for network access.
2. The background watchdog (`/userdata/system/custom.sh`) detects when Internet connectivity is established.
3. It initializes an embedded `tailscaled` daemon with persistent keys in `/userdata/tailscale/state/`.
4. It connects to your private Tailnet mesh.
5. It mounts the central home server using its Tailscale 100.x.x.x IP:
   ```bash
   mount -t cifs "//100.64.0.1/roms" /userdata/roms -o username=batocera,password=retrogaming,vers=3.0,rw
   ```
6. It hot-reloads EmulationStation using Batocera's internal command:
   ```bash
   batocera-es-swissknife --restart
   ```

---

## 🤝 Community Call for Contributions (RFC)

If you've experimented with running mesh VPNs on Batocera, you know the friction points:
* **Read-Only Root Filesystem:** Batocera runs from a read-only SquashFS, so daemon persistence and socket paths must be carefully symlinked to `/userdata`.
* **Wi-Fi Association & Captive Portals:** Connecting to public Wi-Fi requiring web portal logins can delay or break automated WireGuard handshakes.
* **CIFS Mount Latency & EmulationStation Race Conditions:** If EmulationStation boots before the CIFS mount completes, it renders an empty catalog. Reloading via `batocera-es-swissknife --restart` works, but a cleaner autofs or overlayfs solution would be ideal.

**Have you built a more resilient roaming setup?**  
We welcome pull requests, alternative scripts, or feedback in our GitHub Issues!
