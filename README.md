# Snap! Capture the Camera Screenshots in Quest3 and Review Them Immediately

*Tested on developing Meta Quest applications in Unity only*

*Last Available in 7.18.2024*

Use the ScreenShotLoader plugin in Unity to capture camera images and export the screenshots taken by the device.

Since Quest3 currently does not have camera access permissions, it is not possible to use real-time video streams for recognition operations. Hence, this workaround is used to achieve some functionalities.

Unity project source file: Snap!(https://github.com/CidsHo/CaptureScreen/blob/main/Snap!.rar)

## Configuration

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

## Explaination

<details>
<summary>ScreenshotLoader.cs</summary>
<pre><code>
	
```
#nullable enable

using System.Collections;
using System.Linq;
using UnityEngine;
using UnityEngine.UI;

public class ScreenshotLoader : MonoBehaviour
{
    private const int MAX_CHECK_PERMISSION_COUNT = 100;

    [SerializeField] private Button requestPermissionButton = default!;  //Serialized fields to assign UI elements from the Unity Editor.
    [SerializeField] private Button loadScreenShotButton = default!; //Function as discription.
    [SerializeField] private RawImage rawImage = default!; //Where screenshot display.
    [SerializeField] private float checkPermissionInterval = 1.0f; //Interval time in seconds between each permission check.

    private Texture2D texture = default!;
    private bool permissionGranted = false;

    public void ReceiveHasReadStoragePermissionCallback(string message) // Call by the Android plugin to notify whether the read storage permission is granted.
    {
        if (message.Equals("true"))
        {
            permissionGranted = true;
            requestPermissionButton.gameObject.SetActive(false);
            loadScreenShotButton.gameObject.SetActive(true);
        }
    }

    private void Awake() //Add listener to buttons, initialize textures and check for permission.
    {
        requestPermissionButton.onClick.AddListener(RequestPermission);
        loadScreenShotButton.onClick.AddListener(loadScreenShot);

        loadScreenShotButton.gameObject.SetActive(false);

        texture = new Texture2D(1, 1);
        rawImage.texture = texture;
        
        RequestCheckPermission();
    }

    private void OnDestroy() //Removes listeners from the buttons.
    {
        if (requestPermissionButton != null)
        {
            requestPermissionButton.onClick.RemoveListener(RequestPermission);
        }
        if (loadScreenShotButton != null)
        {
            loadScreenShotButton.onClick.RemoveListener(loadScreenShot);
        }
    }

    private void RequestCheckPermission() //Requests the current storage permission status from the Android plugin.
    {
        if (permissionGranted)
        {
            return;
        }

        using (var unityPlayer = new AndroidJavaClass("com.unity3d.player.UnityPlayer"))
        {
            using (var currentActivity = unityPlayer.GetStatic<AndroidJavaObject>("currentActivity"))
            {
                using (var loaderClass = new AndroidJavaClass("com.example.screenshotloader.ScreenshotLoader"))
                {
                    loaderClass.CallStatic("requestHasReadStoragePermissionCallback", currentActivity, name, "ReceiveHasReadStoragePermissionCallback");
                }
            }
        }
    }

    public void RequestPermission() //requests the storage permission from the Android plugin.
    {
        using (var unityPlayer = new AndroidJavaClass("com.unity3d.player.UnityPlayer"))
        {
            using (var currentActivity = unityPlayer.GetStatic<AndroidJavaObject>("currentActivity"))
            {
                using (var loaderClass = new AndroidJavaClass("com.example.screenshotloader.ScreenshotLoader"))
                {
                    loaderClass.CallStatic("requestPermissionIfNeeded", currentActivity);
                    StartCoroutine(CheckPermissionCoroutine());
                }
            }
        }
    }

    public void loadScreenShot() //Retrieves the latest screenshot bytes and loads them into the texture, which is then displayed in the rawImage.
    {
        using (var unityPlayer = new AndroidJavaClass("com.unity3d.player.UnityPlayer"))
        {
            using (var currentActivity = unityPlayer.GetStatic<AndroidJavaObject>("currentActivity"))
            {
                using (var loaderClass = new AndroidJavaClass("com.example.screenshotloader.ScreenshotLoader"))
                {
                    using(var loader = loaderClass.CallStatic<AndroidJavaObject>("getInstance"))
                    {
                        var succeeded = loader.Call<bool>("getLatestScreenshot");
                        if (succeeded)
                        {
                            var screenshotBytes = loader.Call<byte[]>("getLatestScreenshotBytes");
                            Debug.Log($"Load latest screenshot: size={screenshotBytes.Length}");
                            texture.LoadImage(screenshotBytes);
                            texture.Apply();
                            rawImage.texture = texture;
                        }
                    }
                }
            }
        }
    }

    private IEnumerator CheckPermissionCoroutine() //Repeatedly check for permission status until granted or the maximum check count is reached.
    {
        foreach (var i in Enumerable.Range(0, MAX_CHECK_PERMISSION_COUNT))
        {
            if (permissionGranted)
            {
                yield break;
            }
            RequestCheckPermission();

            yield return new WaitForSeconds(checkPermissionInterval);
        }
    }
}

```

</code></pre>
</details>

<details>
<summary>ScreenshotLoader.java</summary>
<pre><code>

```

package com.example.screenshotloader;

import com.unity3d.player.UnityPlayer;

import android.Manifest;
import android.app.Activity;
import android.content.pm.PackageManager;
import android.util.Log;

import androidx.core.app.ActivityCompat;

import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.util.Arrays;
import java.util.Comparator;

public class ScreenshotLoader {
    private static final int REQUEST_PERMISSION_CODE = 222;
    private static final String SCREENSHOT_DIRECTORY_PATH = "/sdcard/Oculus/Screenshots/";

    private byte[] latestScreenshotBytes = new byte[0];

//Finds and loads the latest screenshot.

    public boolean getLatestScreenshot() {
        String latestScreenshotPath = getLatestImageFilePath(SCREENSHOT_DIRECTORY_PATH);
        Log.d("ScreenshotLoader", "Latest screenshot path: " + latestScreenshotPath);
        if (latestScreenshotPath != null) {
            try {
                latestScreenshotBytes = loadImageData(latestScreenshotPath);
                return true;
            } catch (IOException e) {
                return false;
            }
        }
        return false;
    }

    public byte[] getLatestScreenshotBytes() {
        return latestScreenshotBytes;
    }

    public static ScreenshotLoader getInstance() {
        return new ScreenshotLoader();
    }

//Checks if the app has read storage permission and sends a message back to Unity with the result.

    public static void requestHasReadStoragePermissionCallback(Activity activity, String gameObjectName, String callbackMethodName) {
        if (hasReadStoragePermission(activity)) {
            UnityPlayer.UnitySendMessage(gameObjectName, callbackMethodName, "true");
        }
        else {
            UnityPlayer.UnitySendMessage(gameObjectName, callbackMethodName, "false");
        }
    }
    public static void requestPermissionIfNeeded(Activity activity) {
        activity.runOnUiThread(() -> {
                    if (!hasReadStoragePermission(activity)) {
                        ActivityCompat.requestPermissions(activity,
                                new String[]{Manifest.permission.READ_EXTERNAL_STORAGE},
                                REQUEST_PERMISSION_CODE);
                    }
                }
        );
    }

    private static Boolean hasReadStoragePermission(Activity activity) {
        return activity.checkSelfPermission(Manifest.permission.READ_EXTERNAL_STORAGE) == PackageManager.PERMISSION_GRANTED;
    }

// Gets the path of the latest screenshot file in the specified directory.

    private static String getLatestImageFilePath(String directoryPath) {
        File directory = new File(directoryPath);
        File[] files = directory.listFiles((dir, name) -> name.toLowerCase().endsWith(".jpg"));

        if (files != null && files.length > 0) {
            Arrays.sort(files, Comparator.comparingLong(File::lastModified).reversed());
            return files[0].getAbsolutePath();
        } else {
            return null;
        }
    }

//Loads image data from the specified file path.

    private static byte[] loadImageData(String imagePath) throws IOException {
        File imageFile = new File(imagePath);
        byte[] imageData = new byte[(int) imageFile.length()];

        try (FileInputStream fileInputStream = new FileInputStream(imageFile)) {
            if (fileInputStream.read(imageData) == -1) {
                throw new IOException("Failed to read image data.");
            }
        }

        return imageData;
    }
}

```
 

</code></pre>
</details>
