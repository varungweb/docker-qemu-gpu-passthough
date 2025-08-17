### 1. Identify your USB devices (mouse + keyboard)

Run:

```bash
lsusb
```

Example output:

```
Bus 001 Device 004: ID 046d:c534 Logitech, Inc. Unifying Receiver
Bus 001 Device 005: ID 1a2c:2124 China Resource Semico Co., Ltd Keyboard
```

Take note of the **vendor\:product IDs** (like `046d:c534` for mouse, `1a2c:2124` for keyboard).

---

### 2. Bind USB devices to VFIO

Create a new VFIO override for USB devices:

```bash
sudo nano /etc/modprobe.d/vfio-usb.conf
```

Add:

```
options vfio-pci ids=10de:2504,10de:228e
options vfio-pci ids=1a2c:2124,046d:c534
```

⚠️ Careful: If you bind your **primary keyboard/mouse** to VFIO, you’ll lose host control once VFIO claims them. Best to use a **secondary USB keyboard/mouse** dedicated to the VM.

Update initramfs:

```bash
sudo update-initramfs -c -k $(uname -r)
```

Reboot.

---

### 3. Add USB passthrough to your Docker Compose

Modify your `windows` service in `docker-compose.yml`:

```yaml
    devices:
      - /dev/kvm
      - /dev/vfio/1
      - /dev/vfio/2
      - /dev/vfio/3
```

And add them to your QEMU arguments:

```yaml
    environment:
      VERSION: "win11"
      DEBUG: Y
      RAM_SIZE: "16G"
      CPU_CORES: "14"
      ARGUMENTS: >
        -device vfio-pci,host=03:00.0,multifunction=on 
        -device vfio-pci,host=04:00.0,multifunction=on
        -device vfio-pci,host=0000:01:00.0
        -device vfio-pci,host=0000:01:00.1
```

Replace `0000:01:00.x` with your actual mouse/keyboard PCI IDs (you can check via `lspci -nn` after binding).

---

### 4. Alternative: USB redirection (safer)

Instead of binding USB devices away from the host, you can use **QEMU USB passthrough**:

Add this in your `ARGUMENTS`:

```yaml
    ARGUMENTS: >
      -device vfio-pci,host=03:00.0,multifunction=on
      -device vfio-pci,host=04:00.0,multifunction=on
      -device usb-host,vendorid=0x046d,productid=0xc534
      -device usb-host,vendorid=0x1a2c,productid=0x2124
```

This way, you don’t need to detach them at kernel level — QEMU grabs them directly by `vendorid:productid`.

---

✅ **Recommended:** Use **method 4 (usb-host passthrough)** — it’s safer, avoids initramfs rebuilds, and lets you unplug/replug devices.
