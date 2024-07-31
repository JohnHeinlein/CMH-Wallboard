> [!WARNING]
> This repo contains copyrighted material in the form of my employer's logos. This logo is present in the repackaged image and wallpaper - **follow the instructions for full customization**.
> 
> Addtionally, this repo is not licensed as I include tools that I may not be strictly allowed to redistribute.
> Namely, Rockchip's github repos are unlicensed and presumably not redistributable. If this is the case, I will remove them upon request. Original source for the RK tools can be found in [their github repo](https://github.com/rockchip-android/RKTools)
>
> This also applies for redistributing the repackaged ROM, as I'm not 100% sure that it does not contain copyrighted components. I will remove them upon request, in which case you should follow the instructions below to dump the ROM yourself.

> [!NOTE]
> Should also work for P-WAL-220-ELC-02; same device, smaller screen.
> 
> Written assuming windows powershell syntax. Adjust as appropriate for linux, etc.

## Command Summary.
*Broken up & explained better below*
1. Launcher
   ```
   adb root; adb remount; adb install '.\packages\com.teslacoilsw.launcher_6.2.19-62019_minAPI21(nodpi)_apkmirror.com.apk'; adb shell 'mv /data/app/com.teslacoilsw.launcher-1 /system/priv-app/; rm -r /data/data/com.contextmediainc.system.zygote /data/app/com.patientpoint.dmm-2 /data/app/com.contextmediainc.system.zygote-1 /data/dalvik-cache/arm/system@priv-app@zygote1.apk /data/dalvik-cache/arm/system@priv-app@zygote_standalone.apk@classes.dex /system/priv-app/zygote_standalone.apk /system/priv-app/zygote1/ /init.zygote32.rc /init.zygote64_32.rc';
   ```
2. Set boot splash with RKDevTool (Saves a reboot doing it here)
3. Clear data, set wallpaper, re-enable setup wizard, and scrub strings from build.prop
   ```
   adb root; adb remount; adb shell 'cd /mnt/sdcard; rm -r ./multifunctionclock ./RVPlayer ./cmh ./Android/data/com.contextmediainc.system.zygote /system/media/bootanimation.zip'; adb push .\pictures\default_wallpaper.png /mnt/sdcard/Pictures/default_wallpaper.png; adb shell am start -a android.intent.action.ATTACH_DATA -c android.intent.category.DEFAULT -d file:///mnt/sdcard/Pictures/default_wallpaper.png -t 'image/*' -e mimeType 'image/*'; adb push .\resources\build.prop /system/build.prop; adb shell pm enable com.google.android.setupwizard/com.google.android.setupwizard.SetupWizardActivity; adb shell settings put secure user_setup_complete 0;
   ```
---

## Enable debugging

- Hold RESET button (pinhole on rear) and power on
  - Unit will not boot if USB OTG port is plugged in (presumably switching to some kind of factory mode when it sees 5v?)
- Scroll to Factory Reset
- Reboot once complete
- Tap cog icon on Zygote MDM
- Enable dev mode through settings

## Replace launcher & wipe app data

**One-liner**: Installs Nova Launcher, removes Zygote, and reboots to ensure Zygote is properly dead before continuing.
```
adb root; adb remount; adb install '.\packages\com.teslacoilsw.launcher_6.2.19-62019_minAPI21(nodpi)_apkmirror.com.apk'; adb shell 'mv /data/app/com.teslacoilsw.launcher-1 /system/priv-app/; rm -r /data/data/com.contextmediainc.system.zygote /data/dalvik-cache/arm/system@priv-app@zygote1.apk /data/dalvik-cache/arm/system@priv-app@zygote_standalone.apk@classes.dex /system/priv-app/zygote_standalone.apk /system/priv-app/zygote1/ /init.zygote32.rc /init.zygote64_32.rc'; adb reboot
```
Otherwise, 
- Install Nova Launcher:
  - ```
    adb install '.\packages\com.teslacoilsw.launcher_6.2.19-62019_minAPI21(nodpi)_apkmirror.com.apk'
    ```
- Move launcher to private apps (persistent installation directory):
  - ```
    adb root; adb remount; adb shell mv /data/app/com.teslacoilsw.launcher-1 /system/priv-app/
    ```
- Remove Zygote MDM
  - ```
    adb shell rm -r /data/data/com.contextmediainc.system.zygote /data/dalvik-cache/arm/system@priv-app@zygote1.apk /data/dalvik-cache/arm/system@priv-app@zygote_standalone.apk@classes.dex /system/priv-app/zygote_standalone.apk /system/priv-app/zygote1/ /init.zygote32.rc /init.zygote64_32.rc; 
    ```

## Clear user data

- ```
  adb shell 'cd /mnt/sdcard; rm -r ./multifunctionclock ./RVPlayer ./cmh ./Android/data/com.contextmediainc.system.zygote'
  ```
> [!NOTE]
> This device is symlinked to various other locations, `/data/media/0` should be the main device. Some links, like `/mnt/sdcard`, are not protected. Not very relevant here, but good to know in general.

## Wallpaper & Boot Animation

**One-liner**: Push wallpaper to Pictures director, call Nova's wallpaper changer, and remove proprietary startup animation.
```
adb push .\pictures\default_wallpaper.png /mnt/sdcard/Pictures/default_wallpaper.png; adb shell am start -a android.intent.action.ATTACH_DATA -c android.intent.category.DEFAULT -d file:///mnt/sdcard/Pictures/default_wallpaper.png -t 'image/*' -e mimeType 'image/*'; adb root; adb remount; adb shell 'rm /system/media/bootanimation.zip'
```
Otherwise:
1. Push file to Pictures directory
   - ```
     adb push .\pictures\default_wallpaper.png /mnt/sdcard/Pictures/default_wallpaper.png
     ```
2. Launch wallpaper changer
   - ```
     adb shell am start -a android.intent.action.ATTACH_DATA -c android.intent.category.DEFAULT -d file:///mnt/sdcard/Pictures/default_wallpaper.png -t 'image/*' -e mimeType 'image/*'
     ```
3. Delete startup animation
   - ```
     adb root; adb remount; adb shell 'rm /system/media/bootanimation.zip'
     ```

## Boot splash (baked into ROM)
This is the most intesive step, as the static boot splash image is baked into the boot ROM and must be extracted and re-packed. 

> [!NOTE]
> **Not every step is mandatory, as I've included the pre-packed .img for the 230 & 220.** It is only strictly necessary to switch to Loader mode and push the image to the correct address, unless you're working on a different model and want to be safe.

### 1) Change RKDevTool to English (If not already)

![change_language](https://github.com/JohnHeinlein/testing_notes/assets/29853148/e08cdfcf-b7cc-4905-a60f-86baf778318d) 

### 2) Dump boot img
   - ```
      adb shell ls -l /dev/block/platform/*/by-name/boot
      ```
      ![partitions](https://github.com/JohnHeinlein/testing_notes/assets/29853148/5590091c-d806-4a05-913f-e825b94ebf8c)
   - Pull block device to host as a .img file
	- ```
        adb pull /dev/block/mmcblk1p6 .\images\boot.img
        ```
> [!WARNING]
> Make sure the partition `/dev/block/mmcblk1p6` corresponds to the boot partition!
> `ls -l /dev/block/platform/*/by-name` to see all partitions

### 3) Unpack & Modify Image
   1) Unpack
      ```
      .\utilities\imgRePackerRK_1077_test\imgRePackerRK.exe .\images\boot.img
      ```
      - Creates a `boot.img.cfg` file and a `boot.img.dump` directory
   3) Replace `boot.img.dump\second.dump\logo.bmp` with a modified file from `.\pictures`
   4) Repack
      ```
      .\utilities\imgRePackerRK_1077_test\imgRePackerRK.exe .\images\boot.img.cfg
      ```
> [!NOTE]
> - Might be a smaller file size, this is fine.
> - Orginal is renamed boot.img.bak

### 4) Upload modified image to device
   1) launch RKDevTool.exe
      -  Bottom should read "Found One ADB Device"
   3) Click empty space to far right of "Boot" entry & select repacked img file
   ![load_boot_img](https://github.com/JohnHeinlein/testing_notes/assets/29853148/e9cdb447-d3e0-4cad-b442-37961d0bf739)
   5) Click `Switch` to swap device into LOADER mode. Device will reboot to a black screen, but will still be detected.
   ![switch_to_loader](https://github.com/JohnHeinlein/testing_notes/assets/29853148/78b501e4-12d5-42ff-9982-e18d74b4e42c)
   7) Once in LOADER, click `Dev Partition` to populate memory addresses.
   ![dev partition](https://github.com/JohnHeinlein/testing_notes/assets/29853148/daa822bd-a870-4b94-9cba-c9a24b74b837)
   9) Check box next to `Boot` entry **ONLY**
      - This ensures that the firmware is only partially flashed
   ![partial_flash](https://github.com/JohnHeinlein/testing_notes/assets/29853148/cfff1032-48a8-4eef-acb8-9211136767b6)
   10) Click `Run` to flash. Device should reboot with the modified splash image.

## Re-enable first time setup & customize build.prop

One-liner:
- ```
  adb root; adb remount; adb push .\resources\build.prop /system/build.prop; adb shell pm enable com.google.android.setupwizard/com.google.android.setupwizard.SetupWizardActivity; adb root; adb shell settings pu secure user_setup_complete 0;
  ```

### Modify build.prop
Several configuration options can be changed, including the first-time setup wizard and the build string, by just modifying `build.prop` and rebooting

1) Pull `build.prop` from device
    ```
    adb root; adb remount; adb pull /system/build.prop .\build.prop
    ```
2) Modify locally
   - `ro.build.display.id` is the build string shown in  "About"
   - `ro.setupwizard.mode` controls whether the Android first-time setup wizard can run. Separate commands below will actually trigger it
3) Push modified file to device
   ```
   adb root; adb remount; adb push .\build.prop /system/build.prop
   ```

### Re-enable setup wizard
1) Enable in build.prop as shown in previous step
2) Enable package component
   ```
   adb root; adb shell pm enable com.google.android.setupwizard/com.google.android.setupwizard.SetupWizardActivity
   ```
4) Enable system setting
   ```
   adb root; adb shell settings put secure user_setup_complete 0
   ```

A factory reset would also trigger the setup wizard once it's enabled in build.prop, however this will also clear the preloaded wallpaper. Setting it to run manually like this allows for userspace customization to be done beforehand
