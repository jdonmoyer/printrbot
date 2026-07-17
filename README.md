Credit for early ideas to https://www.jdwallace.com/post/printrbot-simple-metal-marlin2

Step 1 - Save Existing Settings
Run command M503 (report settings) to output your current Printrboard EEPROM settings. Save these.

Steps per unit:
  M92 X79.50 Y79.50 Z2020.00 E97.00
Maximum feedrates (mm/s):
  M203 X125.00 Y125.00 Z5.00 E14.00
Maximum Acceleration (mm/s2):
  M201 X2000 Y2000 Z30 E10000
Acceleration: S=acceleration, T=retract acceleration
  M204 S3000.00 T3000.00
Advanced variables: S=Min feedrate (mm/s), T=Min travel feedrate (mm/s), B=minimum segment time (ms), X=maximum XY jerk (mm/s),  Z=maximum Z jerk (mm/s),  E=maximum E jerk (mm/s)
  M205 S0.00 T0.00 B20000 X20.00 Z0.40 E5.00
Home offset (mm):
  M206 X0.00 Y0.00 Z0.00
PID settings:
   M301 P22.20 I1.08 D114.00
Min position (mm):
  M210 X0.00 Y0.00 Z0.00
Max position (mm):
  M211 X250.00 Y152.40 Z152.40
Bed probe offset (mm):
  M212 X27.00 Y0.00 Z-0.70 

Step 2 - Install Build Tools
I used MacOS, but I believe this should work on other platforms as well.


Install Visual Studio Code.


Open VS Code and install the Auto Build Marlin extension. This will also automatically install the PlatformIO extension.




Step 3 - Download Marlin bugfix-2.0.x Source Code
Download from MarlinFW.org or clone directly from GitHub.



git clone -b bugfix-2.0.x https://github.com/MarlinFirmware/Marlin.git
Note: I did try this on the latest stable release (2.0.7.2 as of this post) but was unsuccessful. A little internet research suggests it might be some USB related bug. If you try it and have better luck, let me know!


Step 4 - Customize Settings for Printrbot
Replace ./Marlin/Marlin/Configuration.h and ./Marlin/Marlin/Configuration_adv.h in the downloaded Marlin source code with versions customized for Printrboard.


Note: My version of these files is based on the the Marlin 2.0.1 configuration files published on the official Printrbot GitHub repo. You can also view the changes side by side with the default values.


G-Code commands M210 and M211 function differently in Marlin 2.0 than in the Printrbot Marlin version. In the Printrbot version M210 and M211 could be used to set the software Min and Max endstops. In the Marlin 2.0 version, M210 is missing and M211 can now be used to enable or disable the endstops, but not to change the value. Consider now if you need to adjust these values for your printer and remember you can check your M210 and M211 output from Step 1 for your current settings.


In particular consider if you have the standard print bed or the XL print bed. I've provided sample values for each.

// The size of the print bed
//#define X_BED_SIZE 152 // Printrbot standard print bed
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




If you receive an error the first time, try building again. For some reason I get a "No such file or directory" error the first time I try to build.




If everything goes well you'll eventually see a "SUCCESS" status.




The new firmware file is located at ./Marlin/.pio/build/at90usb1286_dfu/firmware.hex.


Step 6 - Program Printrboard with New Firmware
For this portion of the tutorial I'll be using a Windows 10 PC which is convenient I suppose because the "official" Atmel Flip programming tool is only distributed for Windows.


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


G-Code Changes from Printrbot Firmware to Marlin 2.0

M204 (Set Starting Acceleration) has replaced parameter S (move acceleration) with two new parameters P (printing acceleration) and T (travel acceleration).

Recall from Step 4 that M210 and M211 (min and max position) are different and should be modified in your Configuration.h file prior to compiling.

M212 (bed probe offset) is missing in Marlin 2.0. Instead use M851 (XYZ Probe Offset).

M851 Z-0.70
M500
M301 (set hotend PID) is still available, but instead we'll use M303 (PID autotune) to calibrate these values.

If you need to update a parameter, you can do so by issuing the command with the new values, saving to EEPROM with M500 (save settings), reloading setting from EEPROM with M501 (restore settings), and then confirming with M503 again.


Example:

M92 X80.00 Y80.00 Z2020.00 E94.50
M500
M501
M503
Step 8 - Recalibrate if Desired
Marlin 2 includes a PID Autotune feature. I had previously just reused what I found in the original firmware.

Hot end PID Autotune (turn on fan (M106 S255) for best results)

M106 S255
M303 C10 S200 E0 U1
M500
Heated bed PID Autotune

M303 C10 S60 E-1 U1
M500

Additional calibration calculations and results for my Printrbot on GitHub.
