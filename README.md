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
   - [Workaround for ADB Permission Issues](#workaround-for-adb-permission-issues)
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

```
Apply the changes:
```bash
source ~/.bashrc
```
### For Both Ubuntu & Arch or Any Other OS

1. **Check Your Setup:**
   Before running your app, it's a good idea to verify that everything is set up correctly. You can do this by running:
   ```bash
   npx react-native doctor
   ```
   This command checks your development environment and provides recommendations for any issues that need to be addressed.

1. **Android Studio Location:**
   If `npx react-native doctor` reports that Android Studio is not found, ensure that your Android Studio installation is correctly located. On Linux, you may need to move the `android-studio` folder to `/opt` for proper recognition:
   ```bash
   sudo mv ~/path/to/android-studio /opt/android-studio
   ```

   After moving, make sure to update any environment variables pointing to the Android SDK or Android Studio paths accordingly.


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
# Android Studio
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin
export PATH=$PATH:$ANDROID_HOME/tools/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools

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
   SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", ATTR{idProduct}=="4ee7", MODE="0660", GROUP="wheel", TAG+="uaccess"
   ```
   This ensures that your user in the `wheel` group has the necessary permissions.

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

---

### Workaround for ADB Permission Issues

If you are still experiencing permission errors even after setting up udev rules, one workaround is to give the `adb` binary elevated privileges so it runs as root when executed by your user. 

**Warning:** This is not the ideal solution from a security standpoint but can resolve the “insufficient permissions” issue.

For example, if your `adb` is located at:
`/home/rayhan/Android/Sdk/platform-tools/adb`

You can run these commands:

```bash
sudo chown root:wheel /home/rayhan/Android/Sdk/platform-tools/adb
sudo chmod 4550 /home/rayhan/Android/Sdk/platform-tools/adb
```

**What this does:**
- `chown root:wheel`: Changes the owner to root and the group to wheel (since you are in the wheel group).
- `chmod 4550`: Sets the setuid bit (the leading 4) so that whenever `adb` is run, it runs with root privileges, while still only allowing members of the wheel group to execute it.

**Important:**  
After running these commands, you might encounter ADB “No Permissions” errors due to issues with the log file created by `adb`. 

1. **Check for ADB “No Permissions” Errors:**
   - If you see errors like `cannot open /tmp/adb.1000.log: Permission denied`, follow these steps:
     - Remove any stale log files from `/tmp`:
       ```bash
       sudo rm /tmp/adb.1000.log
       ```
     - Restart the ADB server:
       ```bash
       adb kill-server
       adb start-server
       adb devices
       ```
   - Ensure that the `/tmp` directory has the correct permissions:
     ```bash
     sudo chmod 1777 /tmp
     ```

2. **Avoid Running ADB as Root:**  
   The goal is to have correct permissions set via udev rules. Running `adb` as root can lead to further permission issues.

After these steps, try running:

```bash
adb devices
```

This should allow you to start the ADB server without permission issues.

> **Note:** While this workaround may resolve the issue, the ideal approach is to fix the udev rules so that `adb` can run as your normal user without needing SUID. Use this only as a temporary fix if you must.

---

## Running Your React Native App 0.78

1. **Start the Metro Bundler:**
   Open a terminal and run:
   ```bash
   npm start
   ```

2. **Run the App on Android Device:**
   In another terminal within your project folder, execute:
   ```bash
   npx react-native run-android
   ```

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
