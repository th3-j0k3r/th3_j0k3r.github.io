---
title:  "Android"
mathjax: true
layout: post
categories: android
---

# Frida setup

```
unxz frida-server-15.1.4-android-arm.xz
mv frida-server-15.1.4-android-arm frida-server
adb connect 192.168.59.102:5555 #genymotion
adb push frida-server /data/local/tmp/
adb shell "chmod 755 /data/local/tmp/frida-server"
adb shell "su /data/local/tmp/frida-server &"
```

# Getting apk from device to pc

```
adb shell pm list packages | grep <package_name>
adb shell pm path com.dreamplug.androidapp
adb pull <apk_path>
```
