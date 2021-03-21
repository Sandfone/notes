# Apk 系统签名的问题

系统如果想要获得某些系统权限（比如读写根目录文件），需要给予系统签名。

作法是首先在 `AndroidManifest.xml` 文件的根标签 `manifest` 添加属性 `android:sharedUserId="android.uid.system"`，然后用系统证书签名后 push 到 `/system/priv-app/` 下即可。

## 问题提出

在做完上述操作后，将签名后的 apk push 到指定路径后 apk 无法在桌面显示启动图标，用命令 `pm list packages` 也查看已安装的应用也找不到目标 APP 的身影。

 ## 问题的解决

多种尝试无果后，意识到是自己签名文件的问题，原来我自己用的是测试签名文件，非系统签名文件。在 AOSP 路径的 build （或 tools）路径下找到系统签名文件 `signapk.jar`，重新签名后问题解决。