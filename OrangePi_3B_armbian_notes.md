# Armbian Notes for Orange Pi 3B

## Installation

### OS

Armbian download page for Orange Pi 3B:

[https://www.armbian.com/orangepi3b/](https://www.armbian.com/orangepi3b/)

### NVMe Install

Official Orange Pi installation guide / documentation:

[https://drive.google.com/drive/folders/18YyPnq_f0gbdNlbsNmzouFPxin08C_gf](https://drive.google.com/drive/folders/18YyPnq_f0gbdNlbsNmzouFPxin08C_gf)

Recommended device-tree settings in `/boot/armbianEnv.txt`:

```text
overlay_prefix=rk3566
fdtfile=rockchip/rk3566-orangepi-3b-v{your_board_version}.dtb
```

### SATA Install

Boot from SPI, system on SATA SSD:

[https://github.com/armbian/build/pull/9388](https://github.com/armbian/build/pull/9388)

---

## Configuration

### Reduce NVMe SSD Idle Temperature

Measured improvement: **~10–15°C lower idle temperature**.

## Runtime Commands

```bash
echo powersave | sudo tee /sys/module/pcie_aspm/parameters/policy
echo auto | sudo tee /sys/class/nvme/nvme0/device/power/control
```

## Persist Settings

### 1. Edit boot config

```bash
sudo nano /boot/armbianEnv.txt
```

Set:

```text
extraargs=cma=256M pcie_aspm.policy=powersave
```

### 2. Create systemd service

```bash
sudo nano /etc/systemd/system/nvme-powersave.service
```

Contents:

```ini
[Unit]
Description=Enable NVMe runtime PM
After=multi-user.target

[Service]
Type=oneshot
ExecStart=/bin/sh -c 'echo auto > /sys/class/nvme/nvme0/device/power/control'
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

Enable:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now nvme-powersave.service
```

### 3. Reboot

```bash
sudo reboot
```

### 4. Verify

```bash
cat /sys/module/pcie_aspm/parameters/policy
cat /sys/class/nvme/nvme0/device/power/control
```

Expected:

```text
default performance [powersave] powersupersave
auto
```
