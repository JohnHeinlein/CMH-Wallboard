1. Install Nova & remove "Zygote Launcher," then clear the cache and remove the boot animation
   ```
   adb root; adb remount; adb install .\'com.teslacoilsw.launcher_5.5.4-59400_minAPI16(nodpi)_apkmirror.com.apk'; adb shell 'cp /data/app/com.teslacoilsw.launcher-1.apk /system/priv-app; rm -r /system/priv-app/zygote_standalone.apk /data/data/com.contextmediainc* /data/dalvik-cache/system@priv-app@zygote_standalone.apk@classes.dex /system/media/bootanimation.zip'
   ```
2. Most, if not all, of the 4.4 tablets do not come loaded with google services, so we need to flash a custom recovery (TWRP) and install them manually.
   1. Launch RK's AndroidTool
   2. Load "Recovery" image into appropriate field
      ![AT_01](https://github.com/user-attachments/assets/6e0b6aa8-d399-4ca3-a739-f34e322df67b)
   3. Swap to LOADER mode
      ![AT_02](https://github.com/user-attachments/assets/76333e2b-f3fa-4c20-aee8-1992dd688f80)
   4. Run download
      ![AT_03](https://github.com/user-attachments/assets/dbc84410-c703-4c14-ac8f-ea2dfdd560ad)

3. Load Google Apps (First, download from [this link](https://sourceforge.net/projects/opengapps/files/arm/20220215/open_gapps-arm-4.4-nano-20220215.zip/download?use_mirror=pilotfiber&r=&use_mirror=autoselect))
   1. Reboot into the new recovery
      ```
      adb reboot recovery
      ```
   2. Plug in flash drive with `open_gapps[...].zip`
   3. `Mount` -> `USB Storage`
   4. Main Menu -> `Install` -> `(Up A Level)` on left pane -> scroll down to `usb-otg`
   5. Select OpenGApps zip file from right-hand pane
   6. Slide to accept; wait for install & reboot
