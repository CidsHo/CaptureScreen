# 捕捉Quest3中的摄像头截图画面并回传

*仅测试在Unity上开发Meta Quest应用的情况*

*Last Available in 7.18.2024*

在Unity中使用ScreenShotLoader插件来捕获摄像头画面，将设备拍摄到的截屏图像传出。

因为Quest3目前没有开放摄像头权限，无法使用实时视频流来实现识别操作，故出此下策来实现一些功能。

## 0. 准备

1. 将[ScreenShotLoader](https://github.com/CidsHo/CaptureScreen/tree/main/ScreenShotLoader)安装至项目Assets下，同时载入Meta XR All-in-one SDK。
2. 在Edit>Project Settings>Player>Android>Publishing Settings>Build中，将**Custom Main Manifest, Custom Main Gradle Template, Custom Gradle Properties Template**勾选。此举将启用Assets/Plugins/Android下的AndroidManifest.xml, mainTemplate.gradle, gradleTemplate.properties。

3. 将插件ScreenShotLoader/Plugins/Android中的mainTemplate.gradle, gradleTemplate.properties替换Assets/Plugins/Android下的对应文件。

4. 编辑Assets/Plugins/Android下的AndroidManifest.xml, 在<application>段前添加以下配置：
```
	<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"
				   android:maxSdkVersion="32" />
	<uses-permission android:name="com.oculus.permission.SET_VR_DEVICE_PARAMS" />
	<uses-permission android:name="com.oculus.permission.READ_VR_DEVICE_PARAMS" />
```
此举将允许应用读取Quest3头显中的文件存储位置。

## 1. 配置插件
