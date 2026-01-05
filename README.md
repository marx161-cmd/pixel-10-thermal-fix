# Pixel 10 Pro Thermal Polling Fix (Android 16)

🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨
> [!WARNING]  
> **POTENTIAL BOOTLOOP ISSUE:** > A recent background Google Play System Update (January 2026) has introduced SELinux policy changes that may cause this thermal fix to trigger a bootloop. I am currently investigating the conflict and will update this repository with a fix as soon as possible. Use with extreme caution for now.
🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨🚨

## 📢 DISCLAIMER 
I am very much a hobby dev, and new to rooting android devices. I made this repository to document my findings as I am learning. Please only implement the module if you know what you are doing, and always keep backups in case of something not working as intended.
 
 Asurgical Magisk/KernelSU module to correct a 5-minute thermal polling delay found in the initial Android 16 QPR builds for the Pixel 10 Pro.

## 🚨 The Issue
In the stock Android 16 firmware for the Pixel 10 Pro, the `PollingDelay` for critical thermal sensors is incorrectly set to **300,000ms** (5 minutes). 

This creates a massive "blind spot" in the system's thermal management. During heavy workloads, the CPU can hit 90°C+ before the Thermal HAL even attempts to sample the temperature, leading to heat soak and potential long-term battery degradation.

## ✅ The Fix
This module overlays the stock thermal configs with focused edits:
1.  **Reduces Polling Delay:** Sets `PollingDelay` to **5,000ms** for the `VIRTUAL-SKIN*` sensors across `thermal_info_config.json`, `thermal_info_config_throttling.json`, and `thermal_info_config_charge.json`.
2.  **Keeps OEM thresholds:** Restores the original `HotThreshold`, `CdevCeilingFrequency`, and `Profile` data so only the sampling cadence changes.
3.  **Retains Hysteresis & Passive Delay:** Keeps the default `PassiveDelay` (7,000ms) and `HotHysteresis` values to avoid oscillating CPU-frequency decisions even as the sensor updates faster.

---

## 📋 Technical Evidence & Root Cause
**Affected Component:** `/vendor/etc/thermal_info_config.json`  
**Target Config:** `VIRTUAL-SKIN-CPU-LIGHT-ODPM`

The stock configuration effectively disables real-time thermal management for burst workloads. By the time the 300s timer expires, the device has already reached thermal saturation. This module restores the expected behavior for a flagship Tensor G5 device.

## ✅ Additional Overlays
* `/vendor/etc/thermal_info_config_throttling.json` and `thermal_info_config_charge.json` inherit the same 5s polling cadence so the HAL cannot revert to the 300s delay through the included files.

### Verification
Run the following command in Termux (with root) to verify the fix is active:
```bash
cat /vendor/etc/thermal_info_config.json | grep -A 15 "VIRTUAL-SKIN-CPU-LIGHT-ODPM"
# Ensure "PollingDelay" is now 5000.
```
Verify the included files as well:
```bash
cat /vendor/etc/thermal_info_config_throttling.json | rg -n "VIRTUAL-SKIN-CPU-LIGHT-ODPM" -C2
cat /vendor/etc/thermal_info_config_charge.json | rg -n "PollingDelay" -C1
```

🛠 Installation
 * Download the thermal_fix.zip from the Releases tab.
 * Install via Magisk Manager or KernelSU.
 * Reboot your device.

 ⚠️ Disclaimer
 This module modifies system thermal parameters. Use at your own risk. While it is designed to increase safety by enforcing earlier throttling, the author is not responsible for any damage to your device.
