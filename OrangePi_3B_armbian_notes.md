# Armbian Notes for Orange Pi 3B

## Device-Tree Fix

The default Armbian image for the OrangePi 3B ships with a **non-working** `overlay_prefix` and no `fdtfile` specified (likely to avoid board version conflicts). Without fixing this, boot from **SPI+NVMe** may fail.

Steps:

1. Burn your Armbian image onto an SD card.
2. Boot the board and edit `/boot/armbianEnv.txt`:

   ```bash
   sudo nano /boot/armbianEnv.txt
   ```
3. Modify `overlay_prefix` to `rk3566` and add:

   ```
   fdtfile=rockchip/rk3566-orangepi-3b-v{your_board_version}.dtb
   ```
4. Save and reboot.

This ensures the board functions properly and enables NVMe boot.

---

## Full Workflow for NVMe Boot

1. Follow **User Manual: 2.6. How to write Linux image to SPIFlash+NVMe SSD**.
2. Burn the same OS image to an SD card.
3. Boot from SD card and apply the **device-tree fix** (above).
4. Reboot → your NVMe drive should appear. Mount it and apply the same device-tree fix on it.
5. Shutdown, remove the SD card, and boot → the system should start from the NVMe SSD.

💡 **Alternative method**: If you can edit `armbianEnv.txt` inside the `.img` before burning, you can skip using an SD card and follow the manual directly.

---

## SATA SSD Boot (SD card + SATA system)

This method boots Orange Pi 3B from a **SATA SSD** while keeping the bootloader on the SD card.

> ⚠ Limitation: SPI + SATA boot is not supported because the default `rkspi_loader.img` lacks SATA support.

### ✨ Quick Summary

* Follow official OrangePi documentation
* (Optional) Clear SPI flash
* Write SPI flash loader
* Flash Armbian OS to SD card
* Modify `armbianEnv.txt`
* Move system to SATA SSD
* **Do not overwrite bootloader**

### 🔍 Step-by-Step

1. **(Optional) Clear SPI Flash**
   See documentation: *2.12. Using RKDevTool to clear SPIFlash*.

2. **Write Linux Loader to SPI Flash + NVMe**

   * Use *2.6. How to write Linux image to SPIFlash+NVMe SSD* as reference.
   * Select `rk356x_linux_spiflash.cfg` (not `rk356x_linux_pcie.cfg`).
   * Flash only:

     * `MiniLoaderAll.bin`
     * `rkspi_loader.img`

   → This sets a minimal SPI loader without PCIe configs.

3. **Flash Armbian Image to SD Card**

   * Download from \[Armbian OrangePi 3B page].
   * Flash with Balena Etcher.

4. **Modify `armbianEnv.txt`**

   ```bash
   sudo nano /boot/armbianEnv.txt
   ```

   Add:

   ```
   overlays=rk3566-roc-pc-sata2
   ```

5. **Move System to SATA SSD**

   ```bash
   sudo armbian-config
   ```

   Navigate:
   `System -> Storage -> Install to internal storage -> Boot from SD card - system on SATA, USB or NVMe`
   → When asked to flash the bootloader, **do not flash it**.

6. **Confirm Boot**
   After reboot, `/` should reside on SATA SSD while bootloader stays on SD.

### 🚶 Alternative (Untested) Method

* Correct `overlay_prefix` in `armbianEnv.txt`.
* Use `armbian-config -> System -> Kernel -> Manage device tree overlays` to enable `roc-pc-sata2`.
* Reboot.

---

## 📅 Useful Links

* [OrangePi 3B Documentation (Google Drive)](https://drive.google.com/drive/folders/18YyPnq_f0gbdNlbsNmzouFPxin08C_gf)
* [Armbian for OrangePi 3B](https://www.armbian.com/orangepi3b/)
* [Balena Etcher](https://www.balena.io/etcher/)
