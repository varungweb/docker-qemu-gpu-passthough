## Nvidia Driver Ubuntu

https://www.youtube.com/watch?v=2aHQbg9j_gI&t=286s

https://www.youtube.com/watch?v=g--fe8_kEcw

```bash
ubuntu-drivers devices
sudo apt purge 'nvidia-*' -y
sudo apt autoremove --purge -y
sudo apt purge 'nvidia-*' -y
sudo apt update
sudo apt install nvidia-driver-575 -y
```

## AMD/Intel

```bash
lscpu | grep "Virt"
lspci -nn | grep -E "NVIDIA"
# Output :-
# 10:00.0 VGA compatible controller [0300]: NVIDIA Corporation GA106 [GeForce RTX 3060 Lite Hash Rate] [10de:2504] (rev a1)
# 10:00.1 Audio device [0403]: NVIDIA Corporation GA106 High Definition Audio Controller [10de:228e] (rev a1)

```


```bash
sudo nano /etc/default/grub
```
grub
```bash
GRUB_CMDLINE_LINUX_DEFAULT="amd_iommu=on iommu=pt vfio-pci.ids=10de:2504,10de:228e"
```

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
sudo reboot
```
```bash
lspci -k | grep -E "vfio-pci|NVIDIA"
sudo touch /etc/modprobe.d/vfio.conf
nano /etc/modprobe.d/vfio.conf
```

vfio.conf
```bash
options vfio-pci ids=10de:2504,10de:228e
softdep nvidia pre: vfio-pci
```

```bash
sudo update-initramfs -c -k $(uname -r)
sudo reboot
```
```bash
lspci -k | grep -E "vfio-pci|NVIDIA"
```

## Dokur QEMU
https://github.com/dockur/windows/issues/22#issuecomment-2036369646

```bash
modprobe vfio vfio_pci
```

compose.yml
```bash
version: "3"
services:
  windows:
    image: dockurr/windows
    build: .
    container_name: windows
    privileged: true
    environment:
      VERSION: "win11"
      DEBUG: Y
      RAM_SIZE: "16G"
      CPU_CORES: "14"
      ARGUMENTS: "-device vfio-pci,host=03:00.0,multifunction=on -device vfio-pci,host=04:00.0,multifunction=on"
    devices:
      - /dev/kvm
      - /dev/vfio/1
    group_add:
      - "105"
    volumes:
      - ./storage:/storage
    cap_add:
      - NET_ADMIN
    ports:
      - 8006:8006
      - 3389:3389/tcp
      - 3389:3389/udp
    stop_grace_period: 2m
    restart: on-failure
```
