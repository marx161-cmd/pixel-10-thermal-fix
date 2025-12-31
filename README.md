# Pixel 10 Pro Thermal Polling Fix (Android 16)

A surgical Magisk/KernelSU module to correct a 5-minute thermal polling delay found in the initial Android 16 QPR builds for the Pixel 10 Pro.

## 🚨 The Issue
In the stock Android 16 firmware for the Pixel 10 Pro, the `PollingDelay` for critical thermal sensors is incorrectly set to **300,000ms** (5 minutes). 

This creates a massive "blind spot" in the system's thermal management. During heavy workloads, the CPU can hit 90°C+ before the Thermal HAL even attempts to sample the temperature, leading to heat soak and potential long-term battery degradation.

## ✅ The Fix
This module overlays a corrected `thermal_info_config.json` that:
1.  **Reduces Polling Delay:** Changes the interval from 300,000ms to **5,000ms** (5s) for real-time response.
2.  **Enforces Safety Clamp:** Inserts a hard `HotThreshold` at **65°C** (Virtual Skin) to prevent extreme temperatures.
3.  **Restores Hysteresis:** Optimizes `PassiveDelay` to 7,000ms to prevent erratic CPU frequency oscillations.

---

## 📋 Technical Evidence & Root Cause
**Affected Component:** `/vendor/etc/thermal_info_config.json`  
**Target Config:** `VIRTUAL-SKIN-CPU-LIGHT-ODPM`

The stock configuration effectively disables real-time thermal management for burst workloads. By the time the 300s timer expires, the device has already reached thermal saturation. This module restores the expected behavior for a flagship Tensor G5 device.

### Verification
Run the following command in Termux (with root) to verify the fix is active:
```bash
cat /vendor/etc/thermal_info_config.json | grep -A 15 "VIRTUAL-SKIN-CPU-LIGHT-ODPM"
# Ensure "PollingDelay" is now 5000.
```

🛠 Installation
 * Download the thermal_fix.zip from the Releases tab.
 * Install via Magisk Manager or KernelSU.
 * Reboot your device.

 ⚠️ Disclaimer
 This module modifies system thermal parameters. Use at your own risk. While it is designed to increase safety by enforcing earlier throttling, the author is not responsible for any damage to your device.
