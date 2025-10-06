# Power Button Force Shutdown

## Overview
The reMarkable zero-sugar board now supports force shutdown via the power button during boot.

## How It Works

### Hardware Level
The i.MX7 SNVS (Secure Non-Volatile Storage) module continuously monitors the ONOFF/PWRON button input. When the button is held for approximately 5+ seconds, the SNVS hardware automatically sets the SPO (Set Power Off) bit in the LPSR (Low Power Status Register).

### Software Implementation
During U-Boot initialization (`board_late_init()`), the code checks the SNVS LPSR register for the SPO bit:

1. If the SPO bit is set, it means the user held the power button during boot
2. U-Boot immediately triggers a clean shutdown via `snvs_poweroff()`
3. Before shutdown, the glitch detector is cleared to ensure a proper power-off sequence

### Register Details

| Register | Offset | Purpose |
|----------|--------|---------|
| SNVS_LPCR | 0x38 | Low Power Control Register - controls power off |
| SNVS_LPSR | 0x4C | Low Power Status Register - SPO bit indicates power button press |
| SNVS_LPPGDR | 0x64 | Power Glitch Detector Register - cleared before shutdown |

### Usage
To force shutdown during boot:
1. Hold the power button for 5+ seconds while powering on the device
2. Keep holding until U-Boot detects the shutdown request
3. Device will automatically power off

### Code Location
Implementation in `board/reMarkable/zero-sugar/zero-sugar.c`:
- `check_power_button_shutdown()` - Detects power button force shutdown request
- `snvs_poweroff()` - Performs clean shutdown with glitch detector clearing
- Integration in `board_late_init()` - Called early in boot process

## Benefits
- Provides a hardware-backed force shutdown mechanism
- Useful for recovery when the system is unresponsive
- Clean shutdown sequence prevents potential power issues
- No additional GPIO configuration required (uses built-in SNVS functionality)
