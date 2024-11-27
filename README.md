---
# **Setting Up Android Device on Ubuntu for React Native Development**

This guide outlines the steps to configure your Ubuntu system to recognize and connect your Android device via `adb` for React Native development.
---

## **Prerequisites**

1. Android device with **USB Debugging** enabled:

   - Open **Settings** → **About Phone** → Tap **Build Number** 7 times to enable **Developer Options**.
   - Go to **Settings** → **Developer Options** → Enable **USB Debugging**.

2. Install `adb` on your Ubuntu system:

   ```bash
   sudo apt update
   sudo apt install adb
   ```

3. Ensure React Native CLI is installed:
   ```bash
   npm install -g react-native-cli
   ```

---

## **Steps to Fix `adb no permissions` Issue**

### **1. Check Device Detection**

Connect your Android device via USB and run:

```bash
lsusb
```

Look for your device in the output (e.g., `Google Inc. Nexus/Pixel Device`).

### **2. Verify `adb` Installation**

Run:

```bash
adb devices
```

If you see an error like `no permissions (missing udev rules?)`, proceed to the next steps.

---

### **3. Add Udev Rules for Your Device**

1. Create or edit the udev rules file:

   ```bash
   sudo nano /etc/udev/rules.d/51-android.rules
   ```

2. Add the following line (replace `18d1` with your device's vendor ID, found using `lsusb`):

   ```bash
   SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", MODE="0666", GROUP="plugdev"
   ```

3. Save the file and set the appropriate permissions:

   ```bash
   sudo chmod a+r /etc/udev/rules.d/51-android.rules
   ```

4. Reload udev rules:
   ```bash
   sudo udevadm control --reload-rules
   sudo service udev restart
   ```

---

### **4. Add User to `plugdev` Group**

Check if your user is in the `plugdev` group:

```bash
groups $USER
```

If not, add your user to the group:

```bash
sudo usermod -aG plugdev $USER
```

Log out and log back in for the group change to take effect.

---

### **5. Restart `adb`**

1. Restart the `adb` server:

   ```bash
   adb kill-server
   adb start-server
   ```

2. Check connected devices:
   ```bash
   adb devices
   ```

You should now see your device listed without any errors.

---

### **6. Confirm Debugging Access**

On your Android device:

- Unlock the screen.
- Confirm the prompt to trust the computer and select **Always Allow**.

---

## **Running React Native App**

1. Start the React Native Metro bundler:

   ```bash
   npm start
   ```

2. Run the app on the connected device:
   ```bash
   npx react-native run-android
   ```

---

## **Troubleshooting**

- **Device Not Listed in `adb devices`:**

  - Ensure USB Debugging is enabled on your device.
  - Reconnect the device and confirm any prompts.
  - Check udev rules and restart the `adb` server.

- **Build Issues:**
  - Ensure Android Studio and required SDK tools are installed.
  - Verify `ANDROID_HOME` and `PATH` environment variables are correctly set in `~/.zshrc`:
    ```bash
    export ANDROID_HOME=$HOME/Android/Sdk
    export PATH=$ANDROID_HOME/emulator:$ANDROID_HOME/tools:$ANDROID_HOME/tools/bin:$ANDROID_HOME/platform-tools:$PATH
    ```
    Apply changes:
    ```bash
    source ~/.zshrc
    ```

---

Now your Android device should be ready for React Native development on Ubuntu!
