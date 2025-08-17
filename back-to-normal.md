### 🔄 Steps to revert GPU/USB passthrough

#### 1. Edit GRUB back to normal

Open:

```bash
sudo nano /etc/default/grub
```

Remove the passthrough options, so this:

```text
GRUB_CMDLINE_LINUX_DEFAULT="amd_iommu=on iommu=pt vfio-pci.ids=10de:2504,10de:228e,1a2c:2124,046d:c534"
```

Becomes (default, or whatever you had before):

```text
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
```

Update grub:

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

---

#### 2. Remove vfio overrides

Delete the custom VFIO configs:

```bash
sudo rm /etc/modprobe.d/vfio.conf
sudo rm /etc/modprobe.d/vfio-usb.conf
```

---

#### 3. Rebuild initramfs

```bash
sudo update-initramfs -c -k $(uname -r)
```

---

#### 4. Reboot

```bash
sudo reboot
```

After reboot:

* Your **GPU driver (NVIDIA/AMD)** will load normally again.
* Your **mouse and keyboard** will work on the host.

Check with:

```bash
lspci -k | grep -EA3 'VGA|NVIDIA'
```

You should now see `Kernel driver in use: nvidia` (or `amdgpu`) instead of `vfio-pci`.

---

#### 5. Clean up Docker Compose (optional)

Remove passthrough arguments from your `docker-compose.yml` so it doesn’t try to claim GPU/USB anymore:

```yaml
ARGUMENTS: ""
devices:
  - /dev/kvm
```

Do you want me to also give you a **toggle script** so you can easily switch between “Passthrough mode” and “Normal mode” without manually editing configs each time?
