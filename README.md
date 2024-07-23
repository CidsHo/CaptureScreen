# Snap! Capture the Camera Screenshots in Quest3 and Review Them Immediately

*Tested on developing Meta Quest applications in Unity only*

*Last Available in 7.18.2024*

Use the ScreenShotLoader plugin in Unity to capture camera images and export the screenshots taken by the device.

Since Quest3 currently does not have camera access permissions, it is not possible to use real-time video streams for recognition operations. Hence, this workaround is used to achieve some functionalities.

Unity project source file: Snap!(https://github.com/CidsHo/CaptureScreen/blob/main/Snap!.rar)

## 0. Configuration

1. Install [ScreenShotLoader](https://github.com/CidsHo/CaptureScreen/tree/main/ScreenShotLoader) in the project **Assets** and also load the Meta XR All-in-one SDK.
2. In Unity, go to Edit > Project Settings > Player > Android > Publishing Settings > Build, check ***Custom Main Manifest, Custom Main Gradle Template, Custom Gradle Properties Template***. This will enable the use of AndroidManifest.xml, mainTemplate.gradle, and gradleTemplate.properties in Assets/Plugins/Android.

3. Replace the mainTemplate.gradle and gradleTemplate.properties in Assets/Plugins/Android with those in ScreenShotLoader/Plugins/Android.

4. Edit AndroidManifest.xml in Assets/Plugins/Android and add the following configuration before the **<application>** segment:
```
	<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"
				   android:maxSdkVersion="32" />
	<uses-permission android:name="com.oculus.permission.SET_VR_DEVICE_PARAMS" />
	<uses-permission android:name="com.oculus.permission.READ_VR_DEVICE_PARAMS" />
```
This will allow the application to read the screenshot storaged in the Quest3 headset. (Further authorization is required in Horizon OS)

## 1. Demonstration

## 2. Explaination

*In progress*
