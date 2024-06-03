> [!NOTE]
> Should also work for P-WAL-220-ELC-02; same device, smaller screen

## Enable debugging
- Plug USB keyboard in BEFORE BOOT
- Hold RESET button (pinhole on rear) and power on
- Scroll to Factory Reset
- Reboot once complete
- Tap cog icon on Zygote MDM
- Enable dev mode through settings

## Replace launcher & wipe app data
- `adb install <path-to-launcher-apk>` to load Nova Launcher
- `adb shell` then `su` for root shell
- `mount -o rw,remount /system` to allow changes to write-protected system parition
- `mv /data/app/com.teslacoilsw.launcher-1 /system/priv-app/` to move Nova Launcher to persistent storage
- Remove Zygote MDM
  - ```
    rm -r /data/data/com.contextmediainc.system.zygote /data/dalvik-cache/arm/system@priv-app@zygote1.apk /data/dalvik-cache/arm/system@priv-app@zygote_standalone.apk@classes.dex /system/priv-app/zygote_standalone.apk;
    ```
One-liner:
```
adb root; adb remount; adb shell mv /data/app/com.teslacoilsw.launcher-1 /system/priv-app/
```



## Clear user data
- ```
  cd /mnt/sdcard; rm -r ./multifunctionclock ./RVPlayer ./cmh ./Android/data/com.contextmediainc.system.zygote
  ```
> [!NOTE]
> This device is symlinked to various other locations, `/data/media/0` should be the main device. Some links, like `/mnt/sdcard`, are not protected. Not very relevant here, but good to know in general.

## Wallpaper
1. Push file to Pictures directory
2. `.\adb push <path-to-img> /data/media/0/Pictures/default_wallpaper.png`
   - On my system:
   - ```
     adb push .\pictures\default_wallpaper.png /mnt/sdcard/Pictures/default_wallpaper.png
     ```
4. Launch wallpaper changer
   - ```
     am start -a android.intent.action.ATTACH_DATA -c android.intent.category.DEFAULT -d file:///data/media/0/Pictures/default_wallpaper.png -t 'image/*' -e mimeType 'image/*'
     ```

## Clear boot splashes

### Bootanimation.zip
- ```
  rm /system/media/bootanimation.zip
  ```

Can be replaced with a custom animation, or deleted to fall back to the stock android animation.

### Boot splash baked into kernel
This is the most intesive step, as the static boot splash image is baked into the boot ROM and must be extracted and re-packed.

1) Download imgRePackerRK, Rockchip Driver Assistant, and RKDevTool.
2) Change RKDevTool to English (Rockchip is a Chinese company, and this is their in-house software)
![change_language](https://github.com/JohnHeinlein/testing_notes/assets/29853148/e08cdfcf-b7cc-4905-a60f-86baf778318d)
3) Dump boot img
   1) `.\adb shell`
   2) `su`
   3) `ls -l /dev/block/platform/*/by-name/recovery` to get the eMMC boot partition as a block device.
      - e.g. `/dev/block/mmcblk1p6`
      ![partitions](https://github.com/JohnHeinlein/testing_notes/assets/29853148/5590091c-d806-4a05-913f-e825b94ebf8c)

   5) `exit` shell
   6) `.\adb pull <mmc partition path> <relative destination on host>` to dump partition to host. May need `adb root` first.
      - e.g.  `.\adb pull /dev/block/mmcblk1p6 .\images\boot.img`
4) Unpack & modify boot image
   1) `.\imgRePackerRK.exe boot.img` to unpack
      - Creates a `boot.img.cfg` file and a `boot.img.dump` directory
   2) Replace `boot.img.dump\second.dump\logo.bmp` with a modified file
   3) `.\imgRePackerRK.exe boot.img.cfg` to repack
      - Might be a smaller file size, this is fine.
      - Orginal is renamed boot.img.bak
5) Upload modified image to device
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
