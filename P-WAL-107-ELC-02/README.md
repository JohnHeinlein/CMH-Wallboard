(Purple Wallboards)

Based heavily on [this XDA forum post](https://xdaforums.com/t/unlocking-32-inch-wallboard-outcome-health.3936524/), with the TWRP image being sourced from a user and uploaded here for posterity.

Also requires the ARM/4.4 version of [OpenGApps](https://opengapps.org/) to get Google Play Services functional.

## Initial factory reset
1. Plug in USB keyboard to port NEAR DC BARREL JACK
2. Power on / Enter Recovery Mode
   - Pinhole button on rear (Recovery/Volume Up) during boot. Press slightly after barrel jack inserted
   - At "No command" screen, hold power button and single press recovery button
3. Select Wipe/Reset & reboot with keyboard
4. Boots to "Zygote" MDM launcher
5. Enable developer mode & USB debugging from options cog (YIT Model)

## Nice Copy-and-Paste commands
1) Install Nova Launcher
   - ```
     adb root; adb remount; adb install .\com.teslacoilsw.launcher_5.5.4-59400_minAPI16(nodpi)_apkmirror.com.apk; adb shell 'cp /data/app/com.teslacoilsw.launcher-1.apk /system/priv-app; rm /system/priv-app/zygote-standalone.apk /data/data/com.contextmediainc.system.zygote /data/dalvik-cache/system@priv-app@zygote_standalone.apk@classes.dex /system/media/bootanimation.zip'
     ```
## Install Launcher
1. Sideload Install Nova Launcher APK
   - `adb install <path/to/launcher.apk>`
   
Generally we'd install through ADB and be done, but we want a launcher to remain in-tact er a factory reset.

2. Re-mount system partition as RW
   1. `adb shell`
   2. `shell@rk3288:/ # su`
   3. `root@rk3288:/ # mount -o rw,remount /system`  
3. Copy launcher app to system apps directory
   1. `root@rk3288:/ # cp /data/app/<launcher.apk> /system/priv-app/`
   2. `root@rk3288:/ # chmod 664 /system/priv-app/<launcher.apk>`
      - Allows app to be installed at launch (`rw-r--r--` flags)
4. Remove custom Zygote MDM launcher
   1. `rm /system/priv-app/zygote_standalone.apk`

MAY be necessary to disable Zygote first; `pm list packages | grep zygote` and `pm disable <zygote APK name>`

## Remove CMH & Zygote apps / configs
Cache: `/data/dalvik-cache/system@priv-app@zygote_standalone.apk@classes.dex`

Data: 
- `rm -r /data/data/com.contextmediainc.system.synop`
- `rm -r /data/data/com.contextmediainc.system.zygote`
- etc.

WiFi APs:
1. `adb root`
2. `adb remount`
3. `push <local-path-to-wpa-supplicant> /system/etc/wifi/wpa-supplicant.conf`
- No `nano` or `pico` on android to do it manually. Faster for mutiple units to just push a file anyway.

Kill any other apps as necessary.

## TWRP & Google Play Services
Since these devices were originally digital signage, they don't include any google play services. These devices also don't have a sideload mode, so we must `dd` TWRP (`recovery.img`) directly to the appropriate block device.
This will require that `recovery.img` and the `open_gapps-arm-4.4[...].zip` both be placed onto an SD card.
1. `adb shell`
2. `shell@rk3288:/ # su`
3. `root@rk3288:/ # DEV=$(ls /dev/block/platform/*/by-name/recovery); ls $DEV` to save recovery block path as a variable
4. `root@rk3288:/ # dd of=$DEV if=/mnt/external_sd/recovery.img` to flash the recovery partition
5. `adb reboot recovery` to boot into TWRP
6. Install OpenGApps from the SD card via TWRP menu

## Final reset
Finally, run a normal factory reset through the android settings. With everything wiped, the unit should reboot to first-time setup and fall back to Nova Launcher, with google apps accessible. If this is confirmed, do one more factory reset and leave the device in OOB mode.

Units may still be a little buggy; android 4 is VERY old and these weren't necessarily designed for this. This does bring it as close as we can to a stock Android tablet, though

## Copy/Paste Shell Commands
Flash recovery:
```
DEV=$(ls /dev/block/platform/*/by-name/recovery) && echo $DEV; dd of=$DEV if=/mnt/external_sd/recovery.img
```

Clear zygote data/cache:
```
rm -r /data/data/com.contextmediainc.system.zygote/ /data/dalvik-cache/system@priv-app@zygote_standalone.apk@classes.dex
```

Replace zygote as default launcher:
```
mount -o rw,remount /system && cp /data/app/com.teslacoilsw.launcher-1.apk /system/priv-app/ && chmod 664 /system/priv-app/com.teslacoilsw.launcher-1.apk && rm -r /system/priv-app/zygote_standalone.apk
```
