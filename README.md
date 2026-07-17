Credit for early ideas to https://www.jdwallace.com/post/printrbot-simple-metal-marlin2

Step 1 - Save Existing Settings
Run command M503 (report settings) to output your current Printrboard EEPROM settings. Save these.

Step 2 - Install Build Tools

Install Visual Studio Code.


Open VS Code and install the Auto Build Marlin extension. This will also automatically install the PlatformIO extension.



Step 3 - Download Marlin  Source Code
clone directly from GitHub.

https://github.com/MarlinFirmware/Marlin/


Step 4 - Customize Settings for Printrbot
Replace ./Marlin/Marlin/Configuration.h and ./Marlin/Marlin/Configuration_adv.h in the downloaded Marlin source code with versions customized for Printrboard.

In particular consider if you have the standard print bed or the XL print bed. 

// The size of the print bed
#define X_BED_SIZE 254   // Printrbot XL print bed
#define Y_BED_SIZE 152

// Travel limits (mm) after homing, corresponding to endstop positions.
#define X_MIN_POS 0
#define Y_MIN_POS 0
#define Z_MIN_POS 0
#define X_MAX_POS X_BED_SIZE
#define Y_MAX_POS Y_BED_SIZE
#define Z_MAX_POS 135    // Printrbot default is 152. This is too high for my Printrbot.

Step 5 - Build Firmware
From VS Code, select File > Open... and select the Marlin source code folder. Open the main folder which contains the platformio.ini file, NOT the Marlin sub-folder where configuration.h is.

This will open the PlatformIO Home window. Ignore this and instead open the Auto Build Marlin extension by clicking its icon in the sidebar.


Select the Build icon.


If everything goes well you'll eventually see a "SUCCESS" status.


The new firmware file is located at ./Marlin/.pio/build/at90usb1286_dfu/firmware.hex.


Step 6 - Program Printrboard with New Firmware

Download and install FLIP 3.4.7.112 for Windows (Java Runtime Environment Included).

Remove power from the Printrboard and connect a USB cable to the Windows PC.

Install a jumper on the two pins next to the Printrboards microcontroller labeled BOOT and then reapply power. Press and release the small black button.

Open Device Manager and you'll either find AT90USB128 DFU under Other devices or AT90USB128 under libusb-win32 devices. 

If you see AT90USB128 DFU, you'll need to install a driver. Right click and select Update driver. Select Browse my computer for drivers and browse to C:\Program Files (x86)\Atmel\Flip 3.4.7\usb.


Click Next. You'll see a Windows Security popup asking you to verify you would like to install the driver. Click Install.

Once the driver is successfully installed you'll see AT90USB128 with no warning triangle.

Open Atmel Flip. Select Device > Select and select ATU90USB1287 from the list. Then select Settings > Communication > USB. Select Open on the small popup. (Yes, both my Printrboard Rev. D and Rev. F5 have the ATU90USB1286 microcontroller; however this only worked for me if I selected AT90USB1287. ¯\_(ツ)_/¯ )

Now load the firmware.hex file you built earlier by selecting File > Load HEX File and browsing for firmware.hex.


Select Run to program the Printrboard.

Flip will issue a few popups and then a few seconds later you should see Verify PASS. 

Close Atmel Flip. Remove power from the Printrbot, remove the jumper, then reapply power.

Step 7 - Verify Settings
Reconnect to the Printrbot with your software of choice. Note that the COM port may have changed.

Issue command M115 (firmware info) to verify the new firmware version. Run M502 (factory reset) followed by M500 (save settings) to make sure there are no lingering settings in EEPROM from the previous firmware.

Issue command M503 to get your settings and compare those with your backed up settings from Step1.

