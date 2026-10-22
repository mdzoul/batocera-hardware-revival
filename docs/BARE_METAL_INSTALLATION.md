# Bare-Metal Installation: Network Block Streaming

This guide explains how to permanently flash Batocera.linux directly onto an old laptop's internal SATA hard drive or SSD without needing intermediate local USB drives or extraction tools.

---

## 🛠️ The Challenge: Flashing Without Double Transfers

Usually, installing Batocera to an internal drive requires:
1. Flashing a temporary Linux or rescue USB.
2. Downloading an 8GB+ uncompressed Batocera image onto that USB.
3. Running `dd` or Balena Etcher to write it to `/dev/sda`.

On older laptops with limited USB ports, slow USB 2.0 speeds, or small rescue thumb drives, storing that 8GB uncompressed image locally fails or takes forever.

---

## ⚡ Solution: Direct Network Block Streaming (`curl | gzip | dd`)

By streaming the compressed image over gigabit Wi-Fi or Ethernet directly into the raw block device, no local intermediate storage is needed.

### Prerequisites:
1. Boot the target laptop with any temporary live Linux environment (Arch Linux, Ubuntu Live, or a temporary Batocera USB).
2. Connect the laptop to your local Wi-Fi or plug in an Ethernet cable.
3. Identify the target internal disk using `lsblk`:
   ```bash
   lsblk
   ```
   *(Usually `/dev/sda` for 2.5" SATA HDDs or `/dev/nvme0n1` for M.2 drives)*.

---

## 🚀 Execution Command

Run the following one-line stream pipeline (replace `/dev/sda` with your target drive):

```bash
curl -s -L https://mirrors.o2switch.fr/batocera/x86_64/stable/last/batocera-x86_64-43.1-20260529.img.gz \
  | gzip -dc \
  | dd of=/dev/sda bs=4M status=progress conv=fsync && sync
```

### What this pipeline does:
1. `curl -s -L`: Fetches the official compressed Batocera image stream over HTTP.
2. `gzip -dc`: Decompresses the stream on the fly in memory (RAM).
3. `dd of=/dev/sda bs=4M`: Writes 4MB chunks directly to raw block storage at maximum bus speed.
4. `conv=fsync && sync`: Ensures all buffers are flushed to physical platters before finishing.

---

## 🏁 Post-Installation Boot

1. Unplug the temporary rescue USB.
2. Power on the laptop.
3. On the first boot, Batocera will automatically detect unallocated space and expand its secondary userdata partition (`/dev/sda2`) to consume the entire disk capacity.
4. Access the boot configuration in `/boot/batocera-boot.conf` to configure network share mounts.
