Below is a consolidated guide for setting up React Native development on Linux – with separate sections for Ubuntu and Arch Linux – including details on resolving adb permission issues.

---

# React Native Development on Linux: Ubuntu & Arch Linux

This guide explains how to configure your Linux system to work with an Android device (or emulator) for React Native development. It covers prerequisites, installing and configuring adb, resolving “no permissions” errors, and running your React Native app.

> **Note:**  
> Linux’s strict USB permissions sometimes prevent adb from communicating with your device. This is due to the fact that the USB device nodes (typically under `/dev/bus/usb/`) are owned by root. The recommended solution is to set up proper udev rules (or use provided packages like `android-udev` on Arch) so that your regular user account can access the device without running adb as root.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Setting Up on Ubuntu](#setting-up-on-ubuntu)
   - [Install Required Packages](#install-required-packages-ubuntu)
   - [Configure Environment Variables](#configure-environment-variables-ubuntu)
   - [Fixing ADB “No Permissions” Issue on Ubuntu](#fixing-adb-no-permissions-ubuntu)
3. [Setting Up on Arch Linux (and Hyprland)](#setting-up-on-arch-linux)
   - [Install Required Packages](#install-required-packages-arch)
   - [Configure Environment Variables](#configure-environment-variables-arch)
   - [Fixing ADB “No Permissions” Issue on Arch](#fixing-adb-no-permissions-arch)
4. [Running Your React Native App](#running-your-react-native-app)
5. [Troubleshooting Tips](#troubleshooting-tips)
6. [Additional Considerations](#additional-considerations)

---

## Prerequisites

- **Android Device Setup:**
  - Enable **Developer Options**:  
    *Go to **Settings** → **About Phone** and tap the **Build Number** 7 times.*
  - Enable **USB Debugging**:  
    *Go to **Developer Options** and toggle on USB Debugging.*
  - (For some devices) Also enable **Install via USB** and **USB Debugging (Security Settings)** if available.

- **Node.js & React Native CLI:**
  - Install Node.js (or use [nvm](https://github.com/nvm-sh/nvm)).
  - Install React Native CLI globally:
    ```bash
    npm install -g react-native-cli
    ```

---

## Setting Up on Ubuntu

### Install Required Packages

Update your package list and install adb:
```bash
sudo apt update
sudo apt install adb
```

### Configure Environment Variables

Add your Android SDK path to your shell profile (e.g., in `~/.bashrc` or `~/.profile`):
```bash
# Android Studio
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin
export PATH=$PATH:$ANDROID_HOME/tools/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools

export PATH=$HOME/flutter/bin:$PATH
# For arch only 
export CHROME_EXECUTABLE=/usr/bin/google-chrome-stable
```
Apply the changes:
```bash
source ~/.bashrc
```

### Fixing ADB “No Permissions” Issue on Ubuntu

1. **Check Device Detection:**
   Connect your device via USB and run:
   ```bash
   lsusb
   ```
   Look for your device (e.g., it might show as a “Google Inc. Nexus/Pixel Device”).

2. **Add udev Rules:**
   Create or edit the udev rules file:
   ```bash
   sudo nano /etc/udev/rules.d/51-android.rules
   ```
   Add a line with your device’s vendor ID (for example, if your device is listed as `18d1:4ee7`, use `18d1`):
   ```bash
   SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", MODE="0666", GROUP="plugdev"
   ```
   Save the file and set proper permissions:
   ```bash
   sudo chmod a+r /etc/udev/rules.d/51-android.rules
   ```
   
3. **Reload udev Rules:**
   ```bash
   sudo udevadm control --reload-rules
   sudo service udev restart
   ```
   Unplug and replug your device.

4. **Add User to the `plugdev` Group:**
   Check your groups:
   ```bash
   groups $USER
   ```
   If `plugdev` is missing, add your user:
   ```bash
   sudo usermod -aG plugdev $USER
   ```
   Then log out and log back in.

5. **Restart ADB Server:**
   ```bash
   adb kill-server
   adb start-server
   adb devices
   ```
   You should now see your device listed (and later confirm the debugging prompt on the device).

---

## Setting Up on Arch Linux (and Hyprland)

### Install Required Packages

On Arch, install adb from the official repositories. Optionally, install `android-udev` for pre-made rules:
```bash
sudo pacman -S android-tools android-udev
```

### Configure Environment Variables

In your shell configuration (for example, in `~/.zshrc` if using zsh with Hyprland), add:
```bash
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/platform-tools:$ANDROID_HOME/tools:$ANDROID_HOME/tools/bin
```
Apply changes:
```bash
source ~/.zshrc
```

### Fixing ADB “No Permissions” Issue on Arch

1. **Check Device Detection:**
   Connect your device and run:
   ```bash
   lsusb
   ```
   Confirm your device is listed (e.g., `18d1:4ee7` for Google devices).

2. **Add udev Rules:**
   Create or edit the file:
   ```bash
   sudo nano /etc/udev/rules.d/51-android.rules
   ```
   Add the rule (using your device’s vendor ID):
   ```bash
   SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", MODE="0666", GROUP="wheel"
   ```
   *Tip:* If you’re not in the `plugdev` group on Arch (often not created by default), you can use `wheel` since your user is already in that group. Alternatively, you can create and use a custom group.

3. **Set Rule File Permissions & Reload udev:**
   ```bash
   sudo chmod a+r /etc/udev/rules.d/51-android.rules
   sudo udevadm control --reload-rules
   sudo udevadm trigger
   ```
   Unplug and replug your device.

4. **Restart ADB Server:**
   Run (as your normal user):
   ```bash
   adb kill-server
   adb start-server
   adb devices
   ```
   Your device should now be detected without permission issues.

> **Important:**  
> If you still encounter permission issues and adb fails to start (for example, errors like “cannot open /tmp/adb.1000.log: Permission denied”), check if a stale log file exists:
> ```bash
> sudo rm /tmp/adb.1000.log
> ```
> Then restart the adb server.

---

## Running Your React Native App

1. **Start the Metro Bundler:**
   ```bash
   npm start
   ```

2. **Run the App on Android Device:**
   In your project folder:
   ```bash
   npx react-native run-android
   ```

---

## Troubleshooting Tips

- **Device Not Listed in `adb devices`:**
  - Verify that **USB Debugging** is enabled on your device.
  - Disconnect and reconnect the USB cable.
  - Recheck your udev rules and reload them.
  - Confirm your user is in the appropriate group (`plugdev` on Ubuntu or `wheel`/custom on Arch).

- **ADB “No Permissions” Errors:**
  - Check that your udev rules file has the correct vendor ID and proper file permissions.
  - Remove any stale log files from `/tmp` (e.g., `sudo rm /tmp/adb.1000.log`).
  - Avoid running adb as root. The goal is to have correct permissions set via udev rules.

- **Environment Variables Not Found:**
  - Ensure your `ANDROID_HOME` and PATH variables are correctly exported in your shell’s configuration file.
  - Reload your shell configuration (`source ~/.bashrc` or `source ~/.zshrc`).

---

## Additional Considerations

- **Security Implications:**  
  Running adb with elevated privileges (e.g., using sudo or SUID) is generally discouraged as it may expose your system to security risks. Properly configuring udev rules is the safest approach.

- **USB Cable and Port:**  
  Sometimes, issues may stem from a faulty or “charge-only” USB cable or connecting through a USB 3.0 port. Test with a known good USB data cable and consider switching ports if problems persist.

- **Emulator Setup:**  
  For running Android emulators, ensure that your hardware acceleration (e.g., KVM on Linux) is properly configured. Refer to [Android Studio’s documentation](https://developer.android.com/studio/run/emulator-acceleration) for details.

---

By following this guide, you should be able to set up a Linux development environment on both Ubuntu and Arch Linux (with Hyprland) for React Native. This documentation covers device recognition, permission fixes, and running your app, along with troubleshooting common issues. Happy coding!

---