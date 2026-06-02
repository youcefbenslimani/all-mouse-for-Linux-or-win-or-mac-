# all-mouse-for-Linux-or-win-or-mac-
A driver program for controlling most mouse, such as the Chinese Attack Shark. x6
 Universal Mouse Configurator v9
  Universal Linux Setup Guide (Supports Any Mouse)
═══════════════════════════════════════════════════════════════════════

📋 DESCRIPTION
───────────────────────────────────────────────────────────────────────
This is a standalone GUI application for configuring custom gaming
mice on Linux (including Attack Shark, Ajazz, Darmoshark, and others).
It allows you to:
  • Adjust DPI settings
  • Configure RGB lighting effects
  • Change polling rate
  • Manage button mappings
  • Save/load mouse profiles

The executable is a self-contained ELF binary that does not require
Python or any libraries to be installed on the system.

🔍 HOW TO FIND YOUR MOUSE'S VENDOR ID (VID)
───────────────────────────────────────────────────────────────────────
Before configuring permissions, you must find your mouse's Vendor ID.

Open your terminal and run:
lsusb

Look for your mouse in the output. For example:
Bus 001 Device 004: ID 1d57:fa60 Attack Shark X6

In this example, "1d57" is the 4-digit hexadecimal VENDOR ID (VID),
and "fa60" is the PRODUCT ID (PID). Write down your 4-digit VID.

⚠️  IMPORTANT: UDEV RULES (REQUIRED FOR NON-ROOT ACCESS)
───────────────────────────────────────────────────────────────────────
By default, Linux restricts access to HID devices. To use this
application WITHOUT running as root/sudo, you MUST install the
following udev rules.

Step 1: Create the udev rules file
──────────────────────────────────
sudo nano /etc/udev/rules.d/99-mouse-configurator.rules

Step 2: Paste the following rules (Replace XXXX with your Vendor ID)
──────────────────────────────────

─── Custom Mouse — HID Access Rules ───

Allows non-root users to read/write to hidraw devices

Make sure to replace "XXXX" with your mouse's 4-digit Vendor ID (in lowercase)

Rule 1: Grant read/write access to ALL hidraw interfaces

KERNEL=="hidraw*", 

ATTRS{idVendor}=="XXXX", 

MODE="0666"

Rule 2: Specific match via SUBSYSTEM for extra compatibility

SUBSYSTEM=="hidraw", 

ATTRS{idVendor}=="XXXX", 

MODE="0666"

Rule 3: USB device-level rule (unlocks /dev/bus/usb access)

SUBSYSTEM=="usb", 

ATTR{idVendor}=="XXXX", 

MODE="0666"

Step 3: Reload udev rules and trigger
──────────────────────────────────
sudo udevadm control --reload-rules
sudo udevadm trigger

Step 4: Unplug and replug the mouse
──────────────────────────────────
Physically disconnect and reconnect the mouse for the rules
to take effect.

Step 5: Verify permissions
──────────────────────────────────
Run this command to check that hidraw devices are accessible:

ls -la /dev/hidraw*

You should see mode "crw-rw-rw-" (666) for your mouse's devices.

🚀 RUNNING THE APPLICATION
───────────────────────────────────────────────────────────────────────

Make the file executable:
chmod +x MouseConfigurator

Run the application:
./MouseConfigurator

If the GUI does not appear, try running:
QT_QPA_PLATFORM=xcb ./MouseConfigurator

🔧 TROUBLESHOOTING
───────────────────────────────────────────────────────────────────────
Problem: "libfuse.so.2: cannot open shared object file"
Solution: Install FUSE support:
sudo apt install libfuse2    # Debian/Ubuntu/Mint
sudo dnf install fuse-libs   # Fedora
sudo pacman -S fuse2         # Arch

Problem: "Permission denied" even after udev rules
Solution:
1. Make sure you replaced "XXXX" with your actual mouse Vendor ID
in lowercase (e.g., 1d57, NOT 1D57).
2. Check rules are loaded: sudo udevadm control --reload-rules
3. Replug the mouse.

📝 INTERACTIVE ONE-LINER INSTALLER (RECOMMENDED)
───────────────────────────────────────────────────────────────────────
Copy-paste this single command. It will automatically scan your USB ports
for connected mice, let you select yours, and install the rules instantly:

bash -c 'echo "🔍 Scanning USB devices..."; devices=$(lsusb | grep -iE "mouse|pointer|hid"); if [ -n "$devices" ]; then echo -e "\nFound the following potential devices:\n"; echo "$devices" | awk "{print \"  [\" NR \"] \" \$0}"; echo -e "\nSelect your mouse number or press Enter to type manually:"; read -r choice; if [[ "$choice" =~ ^[0-9]+$ ]]; then selected=$(echo "$devices" | sed -n "${choice}p"); vid=$(echo "$selected" | awk "{print \$6}" | cut -d: -f1); name=$(echo "$selected" | cut -d" " -f7-); echo -e "\nSelected: $name (VID: $vid)"; fi; fi; if [ -z "$vid" ]; then echo -e "\nEnter your 4-digit Mouse Vendor ID (e.g., 1d57):"; read -r vid; fi; vid=$(echo "$vid" | tr "A-Z" "a-z"); if [[ ! "$vid" =~ ^[0-9a-f]{4}$ ]]; then echo "❌ Invalid Vendor ID!"; exit 1; fi; echo "Installing rules for VID: $vid..."; rules="KERNEL=="hidraw*", ATTRS{idVendor}=="$vid", MODE="0666"\nSUBSYSTEM=="hidraw", ATTRS{idVendor}=="$vid", MODE="0666"\nSUBSYSTEM=="usb", ATTR{idVendor}=="$vid", MODE="0666""; echo -e "$rules" | sudo tee /etc/udev/rules.d/99-mouse-configurator.rules > /dev/null; sudo udevadm control --reload-rules && sudo udevadm trigger && echo -e "\n✅ Success! Rules installed for Vendor ID $vid.\n🔌 Please UNPLUG and REPLUG your mouse now."'

📄 LICENSE & CREDITS
───────────────────────────────────────────────────────────────────────
Universal Mouse Configurator v9
Platform: Linux (ELF x86_64)
GUI Framework: PyQt6 / PySide6
Build Tool: PyInstaller
═══════════════════════════════════════════════════════════════════════
