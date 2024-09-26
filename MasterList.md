# Android 4.4
Compatible models:
- P-WAL-107-ELC-03

1. Install Nova & remove "Zygote Launcher," then clear the cache and remove the boot animation
   ```
   adb root; adb remount; adb install .\'com.teslacoilsw.launcher_5.5.4-59400_minAPI16(nodpi)_apkmirror.com.apk'; adb shell 'cp /data/app/com.teslacoilsw.launcher-1.apk /system/priv-app; rm -r /system/priv-app/zygote_standalone.apk /data/data/com.contextmediainc* /data/dalvik-cache/system@priv-app@zygote_standalone.apk@classes.dex /system/media/bootanimation.zip'
   ```
