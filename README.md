# scrcpy Gamepad Launcher Documentation

This document explains how to use the `scrcpy Gamepad Launcher` script. The script allows you to launch scrcpy with gamepad support. **Note:** You must have scrcpy version 2.7 or later for gamepad support to work.

## Overview

This script automates several tasks when launching scrcpy:

- **Connection Type Selection:**  
  Choose between USB, Wireless, or both connection types to filter connected devices.

- **Device Selection:**  
  When multiple devices are connected, you can select the desired one.

- **Display Mode Selection:**  
  Choose whether to launch scrcpy in fullscreen or windowed mode.

- **Gamepad Support:**  
  The script launches scrcpy with the `--gamepad=uhid` option, which requires scrcpy version 2.7 or later.

## Prerequisites

Before using this script, ensure you have the following installed and configured:

1. **scrcpy (v2.7 or later):**
   - The script uses the gamepad option, which is available only in version 2.7 and above.
   - Download the latest version from the [scrcpy GitHub repository](https://github.com/Genymobile/scrcpy).

2. **adb (Android Debug Bridge):**
   - adb must be installed and included in your system’s PATH.
   - It is typically part of the Android SDK.

3. **bash:**
   - The script is written in bash. Use a bash-compatible shell to run it.

4. **Connected Android Device:**
   - Connect your Android device via USB or set up wireless debugging.
   - Ensure that your device has authorized your computer for adb connections.

## Installation

### 1. Save the Script

Copy the code below and save it into a file named `scrcpy_gamepad.sh`:

```bash
#!/bin/bash

# Prompt the user for the connection type.
echo "Select connection type:"
echo "1) USB"
echo "2) Wireless"
echo "3) Both"
read -p "Enter option (number or text): " option

# Normalize the input to lowercase.
option=$(echo "$option" | tr '[:upper:]' '[:lower:]')

case "$option" in
    1|usb)
       mode="usb"
       ;;
    2|wireless)
       mode="wireless"
       ;;
    3|both)
       mode="both"
       ;;
    *)
       echo "Invalid option."
       exit 1
       ;;
esac

# Retrieve list of connected devices in the 'device' state.
DEVICES=$(adb devices | tail -n +2 | awk '/device$/ {print $1}')

# Filter devices based on the selected mode.
case "$mode" in
  usb)
    FILTERED=$(echo "$DEVICES" | grep -v ':')
    ;;
  wireless)
    FILTERED=$(echo "$DEVICES" | grep ':')
    ;;
  both)
    FILTERED="$DEVICES"
    ;;
esac

# If no devices are found, exit.
if [ -z "$FILTERED" ]; then
    echo "No devices found for mode '$mode'."
    exit 1
fi

# If multiple devices are found, let the user choose one.
DEVICE_COUNT=$(echo "$FILTERED" | wc -l)
if [ "$DEVICE_COUNT" -gt 1 ]; then
    echo "Multiple devices found:"
    i=1
    for device in $FILTERED; do
        echo "  $i) $device"
        i=$((i+1))
    done

    read -p "Select device number: " choice
    DEVICE=$(echo "$FILTERED" | sed -n "${choice}p")
    if [ -z "$DEVICE" ]; then
        echo "Invalid selection."
        exit 1
    fi
else
    DEVICE="$FILTERED"
fi

echo "Using device: $DEVICE"

# Prompt the user to select the display mode.
echo "Select display mode:"
echo "1) Fullscreen"
echo "2) Windowed"
read -p "Enter option (number or text): " display_option

# Normalize display option.
display_option=$(echo "$display_option" | tr '[:upper:]' '[:lower:]')

# Determine scrcpy display flag based on the user's choice.
case "$display_option" in
    1|fullscreen)
        display_flag="--fullscreen"
        ;;
    2|windowed)
        display_flag=""
        ;;
    *)
        echo "Invalid display option. Defaulting to windowed mode."
        display_flag=""
        ;;
esac

# Run scrcpy with the selected device and options.
scrcpy -s "$DEVICE" --gamepad=uhid --max-size 0 --video-bit-rate 12M --max-fps 60 $display_flag
```

### 2. Make the Script Executable

Open a terminal and navigate to the directory containing `scrcpy_gamepad.sh`. Then run:

```bash
chmod +x scrcpy_gamepad.sh
```

## How to Use

1. **Run the Script:**

   Execute the script by entering:

   ```bash
   ./scrcpy_gamepad.sh
   ```

2. **Choose Connection Type:**

   You will be prompted to choose how your Android device is connected:
   - **USB:** Use if your device is connected via a USB cable.
   - **Wireless:** Use if your device is connected over Wi-Fi.
   - **Both:** Lists all devices regardless of connection type.

3. **Select a Device:**

   - If the script detects more than one device, it will list them.
   - Enter the number corresponding to the device you wish to use.
   - If only one device is detected, the script proceeds automatically.

4. **Choose Display Mode:**

   - You will be asked to choose between fullscreen and windowed mode.
   - Enter `1` for fullscreen or `2` for windowed mode.
   - If an invalid option is entered, the script defaults to windowed mode.

5. **Launch scrcpy:**

   The script then runs scrcpy with the following options:
   - `--gamepad=uhid`: Enables gamepad support.
   - `--max-size 0`: Uses the device’s native resolution.
   - `--video-bit-rate 12M`: Sets the video bitrate to 12 Mbps.
   - `--max-fps 60`: Limits the framerate to 60 fps.
   - Plus the flag for fullscreen mode if selected.

## Configuration Options

If you need to customize the script for your setup, you can adjust several parameters:

- **Video Bit Rate:**  
  The script sets the video bitrate to 12 Mbps. Modify the `--video-bit-rate` parameter if you require a different quality setting.

- **Maximum FPS:**  
  The default is set to 60 fps. Adjust the `--max-fps` parameter as necessary.

- **Display Mode:**  
  The fullscreen mode flag (`--fullscreen`) is added if chosen. You can change this behavior by editing the display mode section.

## Troubleshooting

If you encounter issues while using the script, consider the following steps:

- **No Devices Found:**
  - Verify that your Android device is properly connected.
  - For wireless connections, ensure that wireless debugging is enabled and configured.
  - Confirm that your device has granted adb access.

- **Invalid Option Errors:**
  - Double-check your input when prompted.
  - Use the provided numeric or text options exactly as shown.

- **Gamepad Support Issues:**
  - Ensure you are running scrcpy version 2.7 or later.
  - Check that your gamepad is correctly connected and recognized by your operating system.

- **adb Not Recognized:**
  - Verify that adb is installed and the directory containing adb is in your PATH.

## Additional Notes

- This script is intended for environments with a bash shell.
- It provides basic input validation; for production use, you might consider adding more robust error handling.
- The script is straightforward and can be modified to suit additional requirements or different setups.
