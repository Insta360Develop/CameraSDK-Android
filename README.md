# Note:

32-bit library (armeabi-v7a) is no longer maintained, please use 64-bit library (arm64-v8a) to build!

# How to get?

Please visit [https://www.insta360.com/sdk/apply](https://www.insta360.com/sdk/apply) to apply for the latest SDK.

# Support

Developers' Page: https://www.insta360.com/developer/home  
Insta360 Enterprise: https://www.insta360.com/enterprise  
Issue Report: https://insta.jinshuju.com/f/hZ4aMW  

# [中文文档](README_zh.md)

# Overview

The Android SDK is mainly used for connecting, setting and obtaining camera parameters, controlling the camera for taking photos and recording, downloading files, firmware upgrades, and supporting video and image exports.

Supported models: X5, X4, X3, ONE X2, ONE X, ONE RS, ONE RS 1-Inch.

# Table of contents
* [CameraSDK Function Instructions](#camerasdk-function-instructions)
    * [Preparation Work](#preparation-work)
    * [Camera Connection and Disconnection](#camera-connection-and-disconnection)
    * [Shoot](#shoot)
    * [Shooting Properties](#shooting-properties)
    * [Support list](#support-list)
    * [Camera files](#camera-files)
    * [Preview](#preview)
    * [Firmware upgrade](#firmware-upgrade)
    * [Camera activation](#camera-activation)
    * [Other features](#other-features)
    * [Error code](#error-code)
* [MediaSDK Function Instructions](#mediasdk-function-instructions)
    * [Environmental preparation](#environmental-preparation)
    * [Preview](#preview-1)
    * [WorkWrapper](#workwrapper)
    * [Image player](#image-player)
    * [Video Player](#video-player)
    * [Export](#export)
    * [Generate HDR images](#generate-hdr-images)
    * [Generate PureShot images](#generate-pureshot-images)
    * [Error code](#error-code-1)

# CameraSDK Function Instructions

## **Preparation Work**

* Add the Maven repository to the build file (the `build.gradle` file in the project root):

```groovy
allprojects {
    repositories {
        ...
        maven {
            url 'http://nexus.arashivision.com:9999/repository/maven-public/'
            credentials {
                // see sdk demo
                username = '***'
                password = '***'
            }
        }
    }
}
```

* Import the dependency library in the `build.gradle` file located in the app directory:

```groovy
dependencies {
    implementation 'com.arashivision.sdk:sdkcamera:1.8.0'
}
```

* Initialize the SDK in the application:

```java
public class MyApp extends Application {

    @Overridepublic void onCreate() {
        super.onCreate();      
        InstaCameraSDK.init(this);
    }
}
```

## Camera Connection and Disconnection**

###  **Connecting the Camera**

You can connect to the camera via Bluetooth, Wi-Fi, or USB.

> **Note:** You must perform the connection operation on the main thread.

> **Note:** When the camera is connected, if the app switches to the background, it will disconnect due to the characteristics of the Android system. Therefore, it is necessary to enable a foreground service to prevent disconnection from the camera. This foreground server is implemented by the developer themselves. The relevant code can refer to the SDK Demo.

####  **Bluetooth Connection**

> **Note:** Bluetooth connection requires successful authorization of Bluetooth-related permissions.

> **Note:** In the case of a Bluetooth connection, large data transmission is not supported. Therefore, functions that transmit large amounts of data are not supported.
>
> For example: live streaming, preview, support list, firmware upgrade, playback, export, etc.

1. Initiate Bluetooth scanning:

```java
InstaCameraManager.getInstance().startBleScan(30_000);
```

2. Set the Bluetooth scanning listener:

```java
InstaCameraManager.getInstance().setScanBleListener(new IScanBleListener() {
    @Override
    public void onScanStartSuccess() {
        // Bluetooth is about to start scanning
    }

    @Override
    public void onScanStartFail() {
        // Bluetooth scanning failed
    }
    
    @Override
    public void onScanning(BleDevice bleDevice) {
        // A new Bluetooth device has been scanned; you can update the Bluetooth device list here
    }
    
     @Override
    public void onScanFinish(List<BleDevice> bleDeviceList) {
        // Bluetooth scanning finished
    }
});
```

3. Stopping Bluetooth Scanning

```java
InstaCameraManager.getInstance().stopBleScan();
```

4. Checking if Bluetooth is Scanning

```java
InstaCameraManager.getInstance().isBleScanning();
```

5. Establishing a Bluetooth Connection with the Camera

```plain&#x20;text
InstaCameraManager.getInstance().connectBle(bleDevice);
```

#### **Wi-Fi Connection**

> **Note:** Ensure that the phone is already connected to the camera’s Wi-Fi.The Wi-Fi password of the camera can be viewed in the settings menu of the camera, or by connecting via Bluetooth and calling to read the Wi-Fi information of the camera.

```java
InstaCameraManager.getInstance().openCamera(InstaCameraManager.CONNECT_TYPE_WIFI);
```



####  **USB Connection**

> **Note:** Ensure that the phone and the camera have established a USB connection.

```java
InstaCameraManager.getInstance().openCamera(InstaCameraManager.CONNECT_TYPE_USB);
```



###  **Disconnecting the Camera**

When you want to disconnect the camera, you can call the following code.

> **Note:** After using the firmware upgrade interface, you need to call the camera closing interface. After waiting for the upgrade to succeed, recreate the camera instance.

```java
InstaCameraManager.getInstance().closeCamera();
```

### Checking Whether the Camera is Connected**

You can determine if the camera is connected by obtaining the current camera connection type.

```java
private boolean isCameraConnected() {
    int type = InstaCameraManager.getInstance().getCameraConnectedType();
    return type != InstaCameraManager.CONNECT_TYPE_NONE;
}
```

The connection types have the following four outcomes:

1. Not connected: *`InstaCameraManager.CONNECT_TYPE_NONE`*
2. USB connection: *`InstaCameraManager.CONNECT_TYPE_USB`*
3. Wi-Fi connection: *`InstaCameraManager.CONNECT_TYPE_WIFI`*
4. Bluetooth connection: *`InstaCameraManager.CONNECT_TYPE_BLE`*

###  **Camera Event Listener**

You can register/unregister the `ICameraChangedCallback` on multiple pages to monitor changes in the camera status.

```java
public abstract class BaseObserveCameraActivity extends AppCompatActivity implements ICameraChangedCallback {
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        InstaCameraManager.getInstance().registerCameraChangedCallback(this);
    }
    @Override
    protected void onDestroy() {
        super.onDestroy();
        InstaCameraManager.getInstance().unregisterCameraChangedCallback(this);
    }
    /**
     * Camera status change
     *
     * @param enabled: whether the camera is available
     */
    @Override
    public void onCameraStatusChanged(boolean enabled) {
    }
    /**
     * Camera connection failure
     *
     * A common scenario is that another phone or another application on this phone has already established a connection with the camera, leading to a failure in establishing the connection.
     * @Param errorCode: Error code, refer to the camera connection error code
     */
    @Override
    public void onCameraConnectError(int errorCode) {
    }
    /**
     * SD card insertion notification
     *
     * @param enabled: whether the current SD card is available
     */
    @Override
    public void onCameraSDCardStateChanged(boolean enabled) {
    }
    /**
     * SD card storage status change
     *
     * @param freeSpace: the current available capacity
     * @param totalSpace: the total capacity
     */
    @Override
    public void onCameraStorageChanged(long freeSpace, long totalSpace) {
    }
    /**
     * Low battery notification
     */
    @Override
    public void onCameraBatteryLow() {
    }
    /**
     * Camera power change notification
     *
     * @param batteryLevel: current power level (0-100, always returns 100 when charging)
     * @param isCharging:   whether the camera is charging
     */
    @Override
    public void onCameraBatteryUpdate(int batteryLevel, boolean isCharging) {
    }
    /**
     * Camera temperature change notification
     *
     * @param tempLevel: temperature level
     */
    @Override
    public void onCameraTemperatureChanged(TemperatureLevel tempLevel) {
    }
}
```

###  **Synchronizing Camera Parameters**

Synchronizing camera parameters means reading all the internal camera parameters at once and caching them in the application. Call the following code to synchronize all camera parameters.
```java
InstaCameraManager.getInstance().fetchCameraOptions(new ICameraOperateCallback() {
    @Override
    public void onSuccessful() {
        // Camera parameters synchronized successfully
    }
    @Override
    public void onFailed() {
        // Camera parameter synchronization failed
    }
    @Override
    public void onCameraConnectError() {
        // Camera connection failed, which may indicate a camera disconnection event
    }
});
```
### **Reading Camera Parameters and Status**

####  Camera type

```java
// Get the camera type, such as: Insta360 X4
String cameraType = InstaCameraManager.getInstance().getCameraType();
// Get enumeration
CameraType type = CameraType.getForType(cameraType);
```
####  Firmware version

```java
// Get the camera firmware version, e.g. 0.9.24
String fwVersion = InstaCameraManager.getInstance().getCameraVersion();
```
#### Camera serial number

```java
InstaCameraManager.getInstance().getCameraSerial();
```
####  Camera activation time

```java
// Get the camera activation time
long activeTime = InstaCameraManager.getInstance().getActiveTime();
```
#### Battery

```java
// Get battery type
// THICK = 0;       //Thick battery
// THIN = 1;        //Thin battery (camera default)
// VERTICAL;        //Vertical shot battery
// NONE;            //Only USB power supply without battery
InstaCameraManager.getInstance().getBatteryType();

// Get current battery: 0-100
int level = InstaCameraManager.getInstance().getCameraCurrentBatteryLevel();

// Is it charging?
boolean isCharging = InstaCameraManager.getInstance().isCameraCharging();
```

####  Wi-Fi information

```java
// Get camera Wi-Fi information
WifiInfo wifiInfo = InstaCameraManager.getInstance().getWifiInfo();

// Wi-Fi name
String ssid = wifiInfo.getSsid();

// Wi-Fi password
String ssid = wifiInfo.getPwd();

// Password version
int pwdVersion= wifiInfo.getPwdVersion();

// Wi-Fi channel
int channel= wifiInfo.getChannel();

// Mac address
String address = wifiInfo.getMacAddress();
```

####  SD card

```java
// Get the total capacity size of the SD card
long totalSpace = InstaCameraManager.getInstance().getCameraStorageTotalSpace();

// Get the available space size of the SD card
long freeSpace = InstaCameraManager.getInstance().getCameraStorageFreeSpace();

// Is the SD card available?
boolean isEnable = InstaCameraManager.getInstance().isSdCardEnabled();
```

#### Other

```java
// Offset status
InstaCameraManager.getInstance().getOffsetState();

// Get protective mirror type
InstaCameraManager.getInstance().getOffsetDetectedType();

// Get Offset
InstaCameraManager.getInstance().getMediaOffset();
InstaCameraManager.getInstance().getMediaOffsetV2();
InstaCameraManager.getInstance().getMediaOffsetV3();

// Whether to take selfies
InstaCameraManager.getInstance().isCameraSelfie();

// Get gyroscope timestamp
InstaCameraManager.getInstance().getGyroTimeStamp();

// Get Window Crop
InstaCameraManager.getInstance().getWindowCropInfo();
```

## Shoot

### Shooting mode

The current SDK supports the following shooting modes:
1. Normal Capture: *`CaptureMode. CAPTURE_NORMAL`*
2. HDR Capture: *`CaptureMode. HDR_CAPTURE`*
3. Super Night View: *`CaptureMode. NIGHT_SCENE`*
4. Continuous Photography: *`CaptureMode. BURST`*
5. Interval photography: *`CaptureMode. INTERVAL_SHOOTING`*
6. Star Mode: *`CaptureMode. STARLAPSE_SHOOTING`*
7. Normal video recording: *`CaptureMode. RECORD_NORMAL`*
8. HDR Recording: *`CaptureMode. HDR_RECORD`*
9. Bullet time: *`CaptureMode. BULLETTIME`*
10. Time-lapse photography: *`CaptureMode. TIMELAPSE`*
11. Move delay: *`CaptureMode. TIME_SHIFT`*
12. Loop recording: *`CaptureMode. LOOPER_RECORDING`*
13. Super Video: *`CaptureMode. SUPER_RECORD`*
14. Slow Motion: *`CaptureMode. SLOW_MOTION`*
15. Follow mode: *`CaptureMode. SELFIE_RECORD`*
16. Night Video: *`CaptureMode. PURE_RECORD`*
17. LIVE: *`CaptureMode.LIVE`*

### Get the current shooting mode

```java
CaptureMode captureMode = InstaCameraManager.getInstance().getCurrentCaptureMode();
```

### Determine if the camera is working

```java
// Check if the camera is working
boolean isWorking = InstaCameraManager.getInstance().isCameraWorking();

// Check if the specified shooting mode is currently working
boolean isNormalCapturing = InstaCameraManager.getInstance().isCameraWorking(CaptureMode.CAPTURE_NORMAL);
```

### Set up camera shooting monitoring

```java
InstaCameraManager.getInstance().setCaptureStatusListener(new ICaptureStatusListener() {
    @Override
    public void onCaptureCountChanged(int captureCount) {
        // Number of shots captured for modes such as interval shooting and starry sky mode,
    }

    @Override
    public void onCaptureStarting() {
        // Capture is about to start
    }

    @Override
    public void onCaptureStopping() {
        // Capture is about to stop
    }
  
    @Override
    public void onFileSaving() {
        // File is being saved.
    }

    @Override
    public void onCaptureTimeChanged(long captureTime) {
       // Recording time callback, applicable only in video recording mode
    }

    @Override
    public void onCaptureWorking() {
        // Capturing in progress
    }

    @Override
    public void onCaptureFinish(String[] filePaths) {
        // Capture finished, returns the file paths of the final products
    }
});
```

###  Camera lens

Camera lenses are divided into the following three types:

1. Panoramic lens: *`SensorMode. PANORAMA`*
2. Front camera: *`SensorMode. FRONT`*
3. Rear lens: *`SensorMode. REAR`*

#### Get the current lens type

```java
SensorMode sensorMode = InstaCameraManager.getInstance().getCurrentSensorMode();
```

#### Switch camera lenses

> Note: The SDK currently only supports panoramic lenses. Unknown issues may occur with non-panoramic lenses. Please switch to using the SDK under panoramic lenses.

```java
// Switch to camera panorama lens
InstaCameraManager.getInstance().switchPanoramaSensorMode(new ICameraOperateCallback() {
    @Override
    public void onSuccessful() {
        // Lens switch successful
    }

    @Override
    public void onFailed() {
        // Lens switch failed
    }

    @Override
    public void onCameraConnectError() {
        // Camera connection error
    }
});
```

### Photo Capture

#### Determine whether the camera is in photo mode

```java
boolean isPhotoMode = CaptureMode.PURE_RECORD.isPhotoMode();
```

#### Normal Photo

```java
// Start Normal Photo
InstaCameraManager.getInstance().startNormalCapture();

// Start Normal Photo with GPS data
InstaCameraManager.getInstance().startNormalCapture(byte[]);
```

#### HDR Photo

```java
// Start HDR capture
InstaCameraManager.getInstance().startHDRCapture();

// Start HDR capture with GPS data
InstaCameraManager.getInstance().startHDRCapture(byte[]);
```

#### Super Night Scene

```java
// Start super night scene
InstaCameraManager.getInstance().startNightScene();

// Start super night scene with GPS data
InstaCameraManager.getInstance().startNightScene(byte[]);
```

#### Burst Photo

```java
// Start burst capture
InstaCameraManager.getInstance().startBurstCapture();

// Start burst capture with GPS data
InstaCameraManager.getInstance().startBurstCapture(byte[]);
```

####  Interval Shooting

```java
// Start Interval Shooting
InstaCameraManager.getInstance().startIntervalShooting();

// Start taking photos at intervals and pass in GPS data
InstaCameraManager.getInstance().stopIntervalShooting(byte[]);

// Stop interval shooting
InstaCameraManager.getInstance().stopIntervalShooting();
```

####  Star mode

```java
// Start interval shooting
InstaCameraManager.getInstance().startStarLapseShooting();

// Start interval shooting with GPS data
InstaCameraManager.getInstance().startStarLapseShooting(byte[]);

// Stop interval shooting
InstaCameraManager.getInstance().stopStarLapseShooting();
```

### Video recording

####  Determine if it is video mode

```java
boolean isVideoMode = CaptureMode.PURE_RECORD.isVideoMode();
```

####  Normal Video Recording

```java
// Start normal video recording
InstaCameraManager.getInstance().startNormalRecord();

// Stop normal video recording
InstaCameraManager.getInstance().stopNormalRecord();

// Stop normal video recording with GPS data
InstaCameraManager.getInstance().stopNormalRecord(byte[]);
```

#### HDR Video Recording

```java
// Start HDR video recording
InstaCameraManager.getInstance().startHDRRecord();

// Stop HDR video recording
InstaCameraManager.getInstance().stopHDRRecord();

// Stop HDR video recording with GPS data
InstaCameraManager.getInstance().stopHDRRecord(byte[]);
```

####  Bullet Time

```java
// Start Bullet Time
InstaCameraManager.getInstance().startBulletTime();

// Stop Bullet Time
InstaCameraManager.getInstance().stopBulletTime();

// Stop Bullet Time with GPS data
InstaCameraManager.getInstance().stopBulletTime(byte[]);
```

#### Timelapse

```java
// Start timelapse recording
InstaCameraManager.getInstance().startTimeLapse();

// Stop timelapse recording
InstaCameraManager.getInstance().stopTimeLapse();

// Stop timelapse recording with GPS data
InstaCameraManager.getInstance().stopTimeLapse(byte[]);
```

#### Time Shift

```java
// Start time shift recording
InstaCameraManager.getInstance().startTimeShift();

// Stop time shift recording
InstaCameraManager.getInstance().stopTimeShift();

// Stop time shift recording with GPS data
InstaCameraManager.getInstance().stopTimeShift(byte[]);
```

#### Loop Recording

```java
// Start loop recording
InstaCameraManager.getInstance().startLooperRecord();

// Stop loop recording
InstaCameraManager.getInstance().stopLooperRecord();

// Stop loop recording with GPS data
InstaCameraManager.getInstance().stopLooperRecord(byte[]);
```

#### Super Video Recording

```java
// Start super video recording
InstaCameraManager.getInstance().startSuperRecord();

// Stop super video recording
InstaCameraManager.getInstance().stopSuperRecord();

// Stop super video recording with GPS data
InstaCameraManager.getInstance().stopSuperRecord(byte[]);
```

####  Slow Motion

```java
// Start slow motion recording
InstaCameraManager.getInstance().startSlowMotionRecord();

// Stop slow motion recording
InstaCameraManager.getInstance().stopSlowMotionRecord();

// Stop slow motion recording with GPS data
InstaCameraManager.getInstance().stopSlowMotionRecord(byte[]);
```

#### Selfie Tracking Mode

```java
// Start selfie tracking mode
InstaCameraManager.getInstance().startSelfieRecord();

// Stop selfie tracking mode
InstaCameraManager.getInstance().stopSelfieRecord();

// Stop selfie tracking mode with GPS data
InstaCameraManager.getInstance().stopSelfieRecord(byte[]);
```

#### Night scene video

```java
// Start night scene recording
InstaCameraManager.getInstance().startPureRecord();

// Stop night scene recording
InstaCameraManager.getInstance().stopPureRecord();

// Stop night scene recording with GPS data
InstaCameraManager.getInstance().stopPureRecord(byte[]);
```

###  LIVE

#### Check if it is a live streaming mode

```java
boolean isLiveMode = CaptureMode.PURE_RECORD.isLiveMode();
```

####  Start Live Streaming

> Note: After the preview stream is turned off, the camera will automatically switch the shooting mode to normal recording. Before spending LIVE, you need to check whether the current shooting mode is LIVE.

```java
LiveParamsBuilder builder = new LiveParamsBuilder()
         // (Required) Set the RTMP address for stream pushing
        .setRtmp(String rtmp)
        // (Required) Set the stream width, e.g., 1440
        .setWidth(int width)
        // (Required) Set the stream height, e.g., 720
        .setHeight(int height)
        // (Required) Set the frame rate, e.g., 30
        .setFps(int fps)
        // (Required) Set the bitrate, e.g., 2*1024*1024
        .setBitrate(int bitrate)
        // (Optional) Whether it is panoramic streaming or not, default is true
        .setPanorama(true);
        
InstaCameraManager.getInstance().startLive(builder, new ILiveStatusListener() {

    @Override
    public void onLiveFpsUpdate(int fps) {
        // fps: current live streaming frame rate
    }

    @Override
    public void onLivePushError(int error, String desc) {
        // Live stream push failed
    }

    @Override
    public void onLivePushFinished() {
        // Live stream push finished
    }

    @Override
    public void onLivePushStarted() {
        // Live stream push started
    }
});
```

#### End LIVE

```java
InstaCameraManager.getInstance().stopLive();
```

###  GPS Data

####  Photography Settings GPS Data

```java
// android.location.Location class, developer obtains it via system API
Location location;

// Set GPS data when taking a photo
GpsData gpsData = new GpsData();
gpsData.setLatitude(location.getLatitude());
gpsData.setLongitude(location.getLongitude());
gpsData.setAltitude(location.getAltitude());
InstaCameraManager.getInstance().startNormalCapture(gpsData.simpleGpsDataToByteArray());
```

#### Video Recording Settings GPS Data

GPS data for recording settings can be divided into the following two situations:

#####  When the recording is stopped

Stopping recording is the same as taking photos. The example code is as follows:

```java
// android.location.Location class, developer obtains it via system API
Location location;

// Set GPS data when stopping the recording
GpsData gpsData = new GpsData();
gpsData.setLatitude(location.getLatitude());
gpsData.setLongitude(location.getLongitude());
gpsData.setAltitude(location.getAltitude());
InstaCameraManager.getInstance().stopNormalRecord(gpsData.simpleGpsDataToByteArray());
```

##### In the video

GPS data can be set at any time in the video recording. GPS data is usually used to create the camera's motion trajectory. Therefore, it is recommended to set the GPS interval as much as possible each time. The example code is as follows:

```java
// Collect GPS data every 100ms; once 20 GPS data points are accumulated, send them to the camera

private Handler mMainHandler = new Handler(Looper.getMainLooper());
// List of GPS data
private List<GpsData> mCollectGpsDataList = new ArrayList<GpsData>();

private Runnable mCollectGpsDataRunnable = new Runnable() {
    @Override
    public void run() {
        // android.location.Location class, developer obtains it via system API
        Location location；
        if (location != null) {
            GpsData gpsData = new GpsData();
            gpsData.setLatitude(location.getLatitude());
            gpsData.setLongitude(location.getLongitude());
            gpsData.setSpeed(location.getSpeed());
            gpsData.setBearing(location.getBearing());
            gpsData.setAltitude(location.getAltitude());
            gpsData.setUTCTimeMs(location.getTime());
            gpsData.setVaild(true);
            mCollectGpsDataList.add(gpsData);
            if (mCollectGpsDataList.size() >= 20) {
                InstaCameraManager.getInstance().setGpsData(GpsData.gpsDataToByteArray(mCollectGpsDataList));
                mCollectGpsDataList.clear();
            }
        }
        mMainHandler.postDelayed(this, 100);
    }
};

// Start collecting GPS data
mMainHandler.post(mCollectGpsDataRunnable);
```

## Shooting Properties

The current SDK supports the following shooting properties:
1. **Exposure Mode**: `CaptureSetting.EXPOSURE`
    - Full Auto Exposure: `Exposure.FULL_AUTO`
    - Auto Exposure: `Exposure.AUTO`
    - ISO Priority: `Exposure.ISO_FIRST`
    - Shutter Priority: `Exposure.SHUTTER_FIRST`
    - Manual Exposure: `Exposure.MANUAL`
    - Panorama Independent Exposure: `Exposure.ADAPTIVE`
2. **EV** **Compensation**: `CaptureSetting.EV`
3. **EV** **Interval**: `CaptureSetting.EV_INTERVAL`
4. **Shutter** **Speed**: `CaptureSetting.SHUTTER`
5. **Shutter** **Mode**: `CaptureSetting.SHUTTER_MODE`
    - Auto: `ShutterMode.AUTO`
    - Indoor Low-light Anti-shake: `ShutterMode.SPORT`    
    - Ultra Fast: `ShutterMode.FASTER`
6. **ISO**: `CaptureSetting.ISO`
7. **Maximum** **ISO**: `CaptureSetting.ISO_TOP_LIMIT`
8. **Video Resolution** **& Frame Rate**: `CaptureSetting.RECORD_RESOLUTION`
9. **Photo** **Resolution**: `CaptureSetting.PHOTO_RESOLUTION`
10. **White Balance**: `CaptureSetting.WB`
11. **AEB** **(****Auto Exposure Bracketing****)**: `CaptureSetting.AEB`
12. **Interval Duration**: `CaptureSetting.INTERVAL`
13. **Color Mode**: `CaptureSetting.GAMMA_MODE`
    - Standard: `GammaMode.STAND`
    - LOG: `GammaMode.VIVID`
    - Vivid: `GammaMode.LOG`
    - Flat (Grayscale): `GammaMode.FLAT`
14. **Photo Format**: `CaptureSetting.RAW_TYPE`
    - JPG: `RawType.OFF`
    - JPG & RAW: `RawType.DNG`
    - PURE & SHOT: `RawType.PURESHOT`
    - PURESHOT & RAW: `RawType.PURESHOT_RAW`
    - INSP: `RawType.INSP`
    - RAW & INSP: `RawType.INSP_RAW`
15. **Video Duration**: `CaptureSetting.RECORD_DURATION`
16. **Low Light** **EIS** **(***Electronic Image Stabilization****)**: `CaptureSetting.DARK_EIS_ENABLE`
17. **Panorama** **Independent Exposure**: `CaptureSetting.PANO_EXPOSURE_MODE`
    - Off: `PanoExposureMode.OFF`
    - On: `PanoExposureMode.ON`
    - Standard: `PanoExposureMode.LIGHT`
    - High: `PanoExposureMode.LSOLATED`

>Note: For cameras prior to Insta360 X4, only ON and OFF are supported. Starting from X4, ON is split into `LIGHT` and `LSOLATED`.
18. **Burst Shot Count**: `CaptureSetting.BURST_CAPTURE`
19. **In-camera Stitching**: `CaptureSetting.INTERNAL_SPLICING`
20. **Video HDR Mode**: `CaptureSetting.HDR_STATUS`
>Note: Starting with the Insta360 X5 camera, cancel the DHR recording `CaptureMode.HDR_RECORD` and add the `CaptureSetting.HDR_STATUS` property setting instead.
21. **Photo HDR Mode**: `CaptureSetting.PHOTO_HDR_TYPE`
>Note: Starting with the Insta360 X5 camera, cancel the HDR camera `CaptureMode.HDR_CAPTURE` and add the `CaptureSetting.PHOTO_HDR_TYPE` property setting instead.

### Get Shooting Properties

You can use the `InstaCameraManager.getInstance().getXxx(CaptureMode)` method to retrieve the corresponding shooting properties.

| Method                      | Parameter   | Return Type      | Description                               |
| --------------------------- | ----------- | ---------------- | ----------------------------------------- |
| getRecordDuration()         | CaptureMode | RecordDuration   | Get recording duration                    |
| getRecordResolution()       |             | RecordResolution | Get video resolution and frame rate       |
| getPhotoResolution()        |             | PhotoResolution  | Get photo resolution                      |
| getInterval()               |             | Interval         | Get shooting interval duration            |
| getHdrStatus()              |             | HdrStatus        | Get HDR status for normal video recording |
| getPhotoHdrType()           |             | PhotoHdrType     | Get HDR status for normal photo capture   |
| getGammaMode()              |             | GammaMode        | Get color mode                            |
| getRawType()                |             | RawType          | Get photo format                          |
| getPanoExposureMode()       |             | PanoExposureMode | Get panorama independent exposure status  |
| getExposure()               |             | Exposure         | Get exposure mode                         |
| getISO()                    |             | ISO              | Get ISO value                             |
| getISOTopLimit()            |             | ISOTopLimit      | Get ISO upper limit                       |
| getShutterMode()            |             | ShutterMode      | Get shutter mode                          |
| getShutter()                |             | Shutter          | Get shutter speed                         |
| getEv()                     |             | Ev               | Get EV value                              |
| getEVInterval()             |             | EVInterval       | Get EV interval                           |
| getAEB()                    |             | AEB              | Get AEB                                   |
| getWB()                     |             | WB               | Get white balance                         |
| getInternalSplicingEnable() |             | InternalSplicing | Get in-camera stitching status            |
| getDarkEisType()            |             | DarkEisType      | Get low-light stabilization status        |
| getBurstCapture()           |             | BurstCapture     | Get burst mode parameters                 |

###  Set Shooting Properties

You can use the `InstaCameraManager.getInstance().setXxx(CaptureMode)` method to set the corresponding shooting properties.

| Method                      | Parameter 1 | Parameter 2      | Return Type | Description                              |
| --------------------------- | ----------- | ---------------- | ----------- | ---------------------------------------- |
| setRecordDuration()         | CaptureMode | RecordDuration   | void        | Set recording duration                   |
| setRecordResolution()       |             | RecordResolution | void        | Set video resolution and frame rate      |
| setPhotoResolution()        |             | PhotoResolution  | void        | Set photo resolution                     |
| setInterval()               |             | Interval         | void        | Set shooting interval duration           |
| setHdrStatus()              |             | HdrStatus        | void        | Set HDR status for normal video          |
| setPhotoHdrType()           |             | PhotoHdrType     | void        | Set HDR status for normal photo          |
| setGammaMode()              |             | GammaMode        | void        | Set color mode                           |
| setRawType()                |             | RawType          | void        | Set photo format                         |
| setPanoExposureMode()       |             | PanoExposureMode | void        | Set panorama independent exposure status |
| setExposure()               |             | Exposure         | void        | Set exposure mode                        |
| setISO()                    |             | ISO              | void        | Set ISO value                            |
| setISOTopLimit()            |             | ISOTopLimit      | void        | Set ISO upper limit                      |
| setShutterMode()            |             | ShutterMode      | void        | Set shutter mode                         |
| setShutter()                |             | Shutter          | void        | Set shutter speed                        |
| setEv()                     |             | Ev               | void        | Set EV value                             |
| setEVInterval()             |             | EVInterval       | void        | Set EV interval                          |
| setAEB(CaptureMode          |             | AEB              | void        | Set AEB                                  |
| setWB()                     |             | WB               | void        | Set white balance                        |
| setInternalSplicingEnable() |             | InternalSplicing | void        | Set in-camera stitching                  |
| setDarkEisType()            |             | DarkEisType      | void        | Set low-light stabilization              |
| setBurstCapture()           |             | BurstCapture     | void        | Set burst mode parameters                |

### Dependency Between Shooting Properties

There are interdependencies among different shooting properties. Setting property A may affect the support list of property B. Therefore, after setting a property, it is necessary to check the other affected properties.

Example:
```java
// Example shows setting `PhotoHdrType`. Other properties follow the same approach.
InstaCameraManager.getInstance().setPhotoHdrType(captureMode, photoHdrType, new InstaCameraManager.IDependChecker() {
    @Override
    public void check(List<CaptureSetting> captureSettings) {
        // captureSettings: list of affected shooting properties
        // UI updates can be handled here
    }
});
```

## Support list

The shooting mode and shooting properties of the camera are based on the properties configured in the support list. Different cameras support different shooting modes, and there are also differences in the shooting properties supported under different shooting modes.

### Initialize support list

When entering the prep page or switching lenses, the support list needs to be initialized. This can be achieved by calling the following code:

```java
InstaCameraManager.getInstance().initCameraSupportConfig(success -> {
    // success: Whether initialization was successful
});
```

### Shooting mode support

Each lens of the camera supports different shooting modes. You need to use the following code to obtain its supported shooting modes.

```java
// Get all supported shooting modes
List<CaptureMode> support = InstaCameraManager.getInstance().getSupportCaptureMode();
```

### Shooting attribute support

Each shooting mode supports different shooting attributes. You need to use the following code to obtain the supported shooting attributes.

```java
// Pass in the specified shooting mode as parameters
List<CaptureSetting> support = InstaCameraManager.getInstance().getSupportCaptureSettingList(CaptureMode.CAPTURE_NORMAL);
```

### The range of values for shooting attributes

The value range of the shooting attribute is also configured in the support list. You need to use the following code to obtain the value range of the supported shooting attribute.

```java
// Gets the property value range of CaptureSetting.RECORD_DURATION, returns a List < RecordDuration >
InstaCameraManager.getInstance().getSupportRecordDurationList(CaptureMode.TIMELAPSE);

// Get the value range of the property value of CaptureSetting.RECORD_RESOLUTION and return a List < RecordResolution >
InstaCameraManager.getInstance().getSupportRecordResolutionList(CaptureMode.TIMELAPSE);

// Get the value range of the property value of CaptureSetting.PHOTO_RESOLUTION and return a List < PhotoResolution >
InstaCameraManager.getInstance().getSupportPhotoResolutionList(CaptureMode.CAPTURE_NORMAL);

// Get the range of property values of CaptureSetting. INTERVAL and return a List < Interval > list
InstaCameraManager.getInstance().getSupportIntervalList(CaptureMode.TIMELAPSE);

// Get the value range of the property value of CaptureSetting.HDR_STATUS and return a List < HdrStatus >
InstaCameraManager.getInstance().getSupportHdrStatusList(CaptureMode.TIMELAPSE);

// Get the value range of the property value of CaptureSetting.PHOTO_HDR_TYPE and return a List < PhotoHdrType >
InstaCameraManager.getInstance().getSupportPhotoHdrTypeList(CaptureMode.CAPTURE_NORMAL);

// Get the value range of the property value of CaptureSetting.GAMMA_MODE and return a List < GammaMode >
InstaCameraManager.getInstance().getSupportGammaModeList(CaptureMode.TIMELAPSE);

// Get the value range of the property value of CaptureSetting.RAW_TYPE and return a List < RawType > 
InstaCameraManager.getInstance().getSupportRawTypeList(CaptureMode.TIMELAPSE);

// Get the value range of the property value of CaptureSetting.PANO_EXPOSURE_MODE and return a List < PanoExposureMode >
InstaCameraManager.getInstance().getSupportPanoExposureList(CaptureMode.TIMELAPSE);

// Get the property value range of CaptureSetting. EXPOSURE and return a List < Exposure >
InstaCameraManager.getInstance().getSupportExposureList(CaptureMode.TIMELAPSE);

// Get the value range of the CaptureSetting. ISO property and return a List < ISO >
InstaCameraManager.getInstance().getSupportISOList(CaptureMode.TIMELAPSE);

// Gets the value range of the CaptureSetting.ISO_TOP_LIMIT property and returns a List < ISOTopLimit >
InstaCameraManager.getInstance().getSupportISOTopLimitList(CaptureMode.TIMELAPSE);

// Gets the value range of the CaptureSetting.SHUTTER_MODE property and returns a List < ShutterMode >
InstaCameraManager.getInstance().getSupportShutterModeList(CaptureMode.TIMELAPSE);

// Get the value range of the property value of CaptureSetting. SHUTTER and return a List < Shutter > 
InstaCameraManager.getInstance().getSupportShutterList(CaptureMode.TIMELAPSE);

// Get the value range of the attribute value of CaptureSetting. EV and return a List < Ev >
InstaCameraManager.getInstance().getSupportEVList(CaptureMode.CAPTURE_NORMAL);

// Get the value range of the property value of CaptureSetting.EV_INTERVAL and return a List < EVInterval >
InstaCameraManager.getInstance().getSupportEVIntervalList(CaptureMode.TIMELAPSE);

// Get the value range of the attribute value of CaptureSetting. AEB and return a List < AEB >
InstaCameraManager.getInstance().getSupportAEBList(CaptureMode.TIMELAPSE);

// Get the value range of the CaptureSetting. WB attribute and return a List < WB >
InstaCameraManager.getInstance().getSupportWBList(CaptureMode.TIMELAPSE);

// Gets the property value range of CaptureSetting.INTERNAL_SPLICING and returns a List < InternalSplicing >
InstaCameraManager.getInstance().getSupportInternalSplicingList(CaptureMode.TIMELAPSE);

// Gets the value range of the CaptureSetting.DARK_EIS_ENABLE property and returns a List < DarkEisType >
InstaCameraManager.getInstance().getSupportDarkEisList(CaptureMode.TIMELAPSE);

// Gets the property value range of CaptureSetting.BURST_CAPTURE, returns a List < BurstCapture >
InstaCameraManager.getInstance().getSupportBurstCaptureList(CaptureMode.TIMELAPSE);
```

## Camera files

### Get the camera socket address

```java
// Return value example: http://192.168.42.1:80
InstaCameraManager.getInstance().getCameraHttpPrefix();
```

### Get camera files

```java
// Return value example: [/DCIM/Camera01/VID_20250326_184618_00_001.insv]
List<String> usls = InstaCameraManager.getInstance().getAllUrlList();

// Includes files in the video
List<String> usls = InstaCameraManager.getInstance().getAllUrlListIncludeRecording();
```

### Delete camera files

```java
// Delete a single camera file
InstaCameraManager.getInstance().deleteFile(String filePath，new ICameraOperateCallback() {
    @Override
    public void onSuccessful() {
        // Delete successfully
    }

    @Override
    public void onFailed() {
       // Delete failed
    }

    @Override
    public void onCameraConnectError() {
      // Camera connection error
    }
});

// Batch delete camera files
InstaCameraManager.getInstance().deleteFileList(List<String> fileUrlList，new ICameraOperateCallback() {
    @Override
    public void onSuccessful() {
        // Delete successfully
    }

    @Override
    public void onFailed() {
       // Delete failed
    }

    @Override
    public void onCameraConnectError() {
      // Camera connection error
    }
});
```

## Preview

### Open preview stream

After successfully connecting the camera, you can manipulate the camera preview stream like this.

> If the camera is passively disconnected, or if `closeCamera ()`is called directly during the preview, the SDK will automatically turn off the preview stream processing related state without calling `closePreviewStream ()`.

> Note: The following code only enables the preview stream and cannot display the preview content. You need to use the InstaCapturePlayerView control of **MediaSDK** to display the preview content.

```java
public class PreviewActivity extends BaseObserveCameraActivity implements IPreviewStatusListener {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_preview);
        // Open preview
        InstaCameraManager.getInstance().setPreviewStatusChangedListener(this);
        InstaCameraManager.getInstance().startPreviewStream(PreviewStreamResolution.STREAM_1440_720_30FPS, InstaCameraManager.PREVIEW_TYPE_NORMAL);
    }

    @Override
    protected void onStop() {
        super.onStop();
        if (isFinishing()) {
            // Close preview
            InstaCameraManager.getInstance().setPreviewStatusChangedListener(null);
            InstaCameraManager.getInstance().closePreviewStream();
        }
    }

    @Override
    public void onOpening() {
        // Preview stream is opening      
    }

    @Override
    public void onOpened() {
        // The preview stream has been opened and can be played
    }

    @Override
    public void onIdle() {
        // Preview stream has stopped
    }

    @Override
    public void onError() {
        // Preview failed
    }
        
        @Override
    public void onExposureData(ExposureData exposureData) {
        // Callback frequency 500Hz
        // exposureData.timestamp: Time after the camera is turned on
        // exposureData.exposureTime: Rolling shutter exposure time
    }

    @Override
    public void onGyroData(List<GyroData> gyroList) {
        // Callback frequency 10Hz, 50 data per group
        // gyroData.timestamp: Time after the camera is turned on
        // gyroData.ax, gyroData.ay, gyroData.az: Triaxial acceleration
        // gyroData.gx, gyroData.gy, gyroData.gz: Three-axis gyroscope
    }

    @Override
    public void onVideoData(VideoData videoData) {
        // Callback frequency 500Hz
        // videoData.timestamp: Time after the camera is turned on
        // videoData.data: Preview raw stream data per frame
        // videoData.size: Length of each frame of data
    }
}
```

### Set preview parameters

#### Preview type

There are three types of preview streams:

1. `PREVIEW_TYPE_NORMAL`: For normal preview or shooting

2. `PREVIEW_TYPE_RECORD`: Only for recording

3. `PREVIEW_TYPE_LIVE`: LIVE

> Note: You must restart the preview stream (close first, then start) to switch between different types.

####  Preview resolution

You need to use the following code to get real-time support for camera resolution

```java
List<PreviewStreamResolution> supportedList = InstaCameraManager.getInstance().getSupportedPreviewStreamResolution(InstaCameraManager.PREVIEW_TYPE_LIVE);
```

> Note: For X4, the preview stream resolution cannot be adjusted and is set to 832x1664 by default. For other models, the preview stream resolution must be set before starting the live preview.

```java
//Set resolution and preview stream type
InstaCameraManager.getInstance().startPreviewStream(
    PreviewStreamResolution.STREAM_368_368_30FPS，
    InstaCameraManager.PREVIEW_TYPE_LIVE
);
```
#### Preview audio

You can set whether to turn on audio during preview.

```java
PreviewParamsBuilder builder = new PreviewParamsBuilder()
        .setStreamResolution(PreviewStreamResolution.STREAM_368_368_30FPS)
        .setPreviewType(InstaCameraManager.PREVIEW_TYPE_LIVE)
        // Whether to turn on audio
        .setAudioEnabled(true);
InstaCameraManager.getInstance().startPreviewStream(builder);
```

## Firmware upgrade

Use the `FwUpgradeManager` class to upgrade the firmware version. The example code is as follows:

> Note: Please make sure to upload the correct files to the camera to prevent any potential damage. For example, do not upload the ONE X firmware upgrade package to the ONE X2 camera.

```java
// Determine if an upgrade is in progress
FwUpgradeManager.getInstance().isUpgrading()；

// Cancel the upgrade
FwUpgradeManager.getInstance().cancelUpgrade();

String fwFilePath = "/xxx/InstaOneXFW.bin";
// Upgrade firmware
FwUpgradeManager.getInstance().startUpgrade(fwFilePath, new FwUpgradeListener() {
    @Override
    public void onUpgradeSuccess() {
        // Callback after successful upgrade
    }

    @Override
    public void onUpgradeFail(int errorCode, String message) {
         // Upgrade failure callback
        // Firmware error code
    }

    @Override
    public void onUpgradeCancel() {
         // Upgrade cancel callback
    }

    @Override
    public void onUpgradeProgress(double progress) {
        // Upgrade progress callback: 0-1
    }
});
```

## Camera activation

Activating the camera requires the use of two data, `appId` and `secretKey`. These two data need to be applied for by the official Insta360.

> Note: Activation requires network permissions

Example code is as follows:

```java
String appid = "xxxxxxxx";
String mSecretKey = "xxxxxxx";
InstaCameraManager.getInstance().activateCamera(new SecretInfo(appid , mSecretKey), new InstaCameraManager.IActivateCameraCallback() {
    @Override
    public void onStart() {
        // Began to activate
    }

    @Override
    public void onSuccess() {
        // Activation successful
    }

    @Override
    public void onFailed(String message) {
        // Activation failed
    }
});
```

## Other features

### Turn off the camera

Instructions to control camera shutdown, sample code is as follows:

> Note: Only Insta360X5 and later cameras support this feature.

```java
InstaCameraManager.getInstance().shutdownCamera();
```

### Camera beep

The switch for camera prompts sound. Example code is as follows:

```java
// Turn on the camera prompt tone
InstaCameraManager.getInstance().setCameraBeepSwitch(true);

// Determine whether the camera has a prompt tone turned on
boolean isBeep = InstaCameraManager.getInstance().isCameraBeep();
```

### Format the SD card

```java
InstaCameraManager.getInstance().formatStorage(new ICameraOperateCallback() {
    @Override
    public void onSuccessful() {
        // SD card formatted successfully
    }

    @Override
    public void onFailed() {
        // SD card formatting failed
    }

    @Override
    public void onCameraConnectError() {
        // Camera connection error
    }
});
```

### Calibration of gyroscope

When the camera is connected to Wi-Fi, this function should be used. Before calibration, make sure the camera is placed vertically on a stable and horizontal surface.

```java
InstaCameraManager.getInstance().calibrateGyro(new ICameraOperateCallback() {
    @Override
    public void onSuccessful() {
    }

    @Override
    public void onFailed() {
    }

    @Override
    public void onCameraConnectError() {
    }
});
```

## Error code

### Camera connection error code

| Error code | Explanation                                              |
| ---------- | -------------------------------------------------------- |
| 2201       | System error BleException.*ERROR_CODE_SYSTEM_STATUS_133* |
| 2002       | Open camera timed out                                    |
| 2206       | System error  BleException.*ERROR_CODE_NOTIFY_FAILURE*   |
| 2207       | System error  BleException.*ERROR_CODE_SHAKE_HAND*       |
| 3201       | Socket connection failed                                 |
| 3205       | Socket connection timed out                              |
| 3206       | Socket reconnect timed out                               |
| 4001       | System error  BleException.*ERROR_CODE_SYNC_PACK*        |
| 4101       | Camera type error                                        |
| 4103       | *Camera sync parameter timed out*                        |
| 4402       | *Camera* *operating system* *error*                      |
| 4403       | Camera is occupied                                       |
| 4405       | Parsing proto failed                                     |
| 4406       | *Camera sync parameter* *retry* *timeout*                |
| 4407       | Camera data exception                                    |
| 4414       | *Failed to wake up Bluetooth*                            |
| 4417       | Camera disconnected during operation                     |
| 4504       | Camera disconnected when connected                       |
| 4505       | Bluetooth binding rejected                               |
| 4507       | Bluetooth write failed                                   |
| 4508       | *Device Bluetooth busy*                                  |
| 4602       | Device Wi-Fi busy                                        |
| 4605       | *USB* *error*                                            |
| 4606       | *Socket Disconnected Due to* *Dirty Data*                |
|            |                                                          |

### Firmware upgrade error code

| Error code | Explanation                                                  |
| ---------- | ------------------------------------------------------------ |
| -1000      | Already upgrading                                            |
| -1001      | Camera not connected                                         |
| -1002      | When the camera battery is below 12%, the upgrade will fail, please make sure the camera battery is fully charged |
| -14004     | Http server error                                            |
| 400        | Http server error                                            |
| 500        | Http server error                                            |
| -14005     | Socket IO exception                                          |

# MediaSDK Function Instructions

## Environmental preparation

* Add the maven address to the build file (`build.gradle` file in the project root directory)

```groovy
allprojects {
    repositories {
        ...
        maven {
            url 'http://nexus.arashivision.com:9999/repository/maven-public/'
            credentials {
                // see sdk demo
                username = '***'
                password = '***'
            }
        }
    }
}
```

* Import the dependency library in the `build.gradle` file of the app directory

```groovy
dependencies {
    implementation 'com.arashivision.sdk:sdkmedia:1.8.0'
}
```

* Initialize SDK in application

```java
public class MyApp extends Application {

    @Overridepublic void onCreate() {
        super.onCreate();      
        InstaMediaSDK.init(this);
    }
}
```

## Preview

If you have integrated the CameraSDK, you can open and display the preview content.

### Control Initialization

You need to put `InstaCapturePlayerView` into your XML file and bind the lifecycle of the Activity:

```xml
<com.arashivision.sdkmedia.player.capture.InstaCapturePlayerView
    android:id="@+id/player_capture"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@android:color/darker_gray"
/>
```

```java
@Override 
protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_preview);
    mCapturePlayerView.setLifecycle(getLifecycle());
}
```

### Display preview screen

Show preview screen in *`IPreviewStatusListener.onOpen()`* callback of CameraSDK

```java
@Overridepublic void onOpened() {
    InstaCameraManager.getInstance().setStreamEncode();
    mCapturePlayerView.setPlayerViewListener(new PlayerViewListener() {
        @Override
        public void onLoadingFinish() {
            // Fixed writing
            InstaCameraManager.getInstance().setPipeline(mCapturePlayerView.getPipeline());
        }
        
        @Override
        public void onLoadingStatusChanged(boolean isLoading) {
            // Loading status change
        }

        @Override
        public void onFail(String desc) {
        }
    });
    // Set parameters, which will be described in detail later.
    mCapturePlayerView.prepare(createParams());
    // 播放(Play)
    mCapturePlayerView.play();
    // 保持屏幕常亮(Keep the screen always on)
    mCapturePlayerView.setKeepScreenOn(false);
}
```

### Release resources

When closing the preview stream, you need to release the `InstaCapturePlayerView`

```java
@Override
protected void onStop() {
    super.onStop();
    if (isFinishing()) {
        InstaCameraManager.getInstance().setPreviewStatusChangedListener(null);
        InstaCameraManager.getInstance().closePreviewStream();
        mCapturePlayerView.destroy();
    }
}

// yilanli
@Override
public void onIdle() {
    mCapturePlayerView.destroy();
    mCapturePlayerView.setKeepScreenOn(false);
}
```

### Set preview parameters

```typescript
private CaptureParamsBuilder createParams() {
    CaptureParamsBuilder builder = new CaptureParamsBuilder()
            // (Must) fixed call is enough
            .setCameraType(InstaCameraManager.getInstance().getCameraType())
            // (Must) fixed call is enough
            .setMediaOffset(InstaCameraManager.getInstance().getMediaOffset())
            // (Must) fixed call is enough
            .setMediaOffsetV2(InstaCameraManager.getInstance().getMediaOffsetV2())
            // (Must) fixed call is enough
            .setMediaOffsetV3(InstaCameraManager.getInstance().getMediaOffsetV3())
            // (Must) fixed call is enough
            .setCameraSelfie(InstaCameraManager.getInstance().isCameraSelfie())
            // (Must) fixed call is enough
            .setGyroTimeStamp(InstaCameraManager.getInstance().getGyroTimeStamp())
            // (Must) fixed call is enough
            .setBatteryType(InstaCameraManager.getInstance().getBatteryType())
            // (Must) fixed call is enough
            .setRenderModelType(InstaCameraManager.getInstance().getSupportRenderModelType())
            // (Optional) Whether it is LIVE, default false
            .setLive(true)
            // (Optional) If you preview with a custom resolution and frame rate, you can set
            .setResolutionParams(mCurrentResolution.width, mCurrentResolution.height, mCurrentResolution.fps);
            // (Optional) Whether to enable stabilization, default is true
            .setStabEnabled(true)
            // (Optional) If setStabEnabled (true), the stable type can be selected, default is InstaStabType. STAB_TYPE_OFF
            // InstaStabType.STAB_TYPE_OFF：Do not use any stabilization.
            // InstaStabType.STAB_TYPE_PANORAMA：Panoramic stability, keep the screen still
            // InstaStabType.STAB_TYPE_CALIBRATE_HORIZON：Only align with the horizon. (Roll axis does not move, yaw pitch angle changes)
            // InstaStabType.STAB_TYPE_FOOTAGE_MOTION_SMOOTH：Smooth camera movement without horizon alignment.
            .setStabType(InstaStabType.STAB_TYPE_AUTO)
            // (Optional) Whether to allow gesture operations, default is true
            .setGestureEnabled(true);
    return builder;
}
```

### Switch rendering mode

```java
// Ordinary
mCapturePlayerView.switchNormalMode();
// Fish eye
mCapturePlayerView.switchFisheyeMode();
// Perspective
mCapturePlayerView.switchPerspectiveMode();
```

### Field of view distance

You can customize the viewing angle and distance according to your preferences.

> Fov: Angle range 0\~ 180
> Distance: line of sight range 0\~ ∞

```java
mCapturePlayerView.setConstraint(float minFov, float maxFov, float defaultFov, float minDistance, float maxDistance, float defaultDistance);
```

### Gesture operation

You can set `PlayerGestureListener` to observe gesture actions:

```java
mCapturePlayerView.setGestureListener(new PlayerGestureListener() {
    @Override
    public boolean onDown(MotionEvent e) {
        // User finger press event
        return PlayerGestureListener.super.onDown(e);
    }

    @Override
    public boolean onTap(MotionEvent e) {
        // User finger click event, similar to View. OnClickListener. OnClick (v)
        return PlayerGestureListener.super.onTap(e);
    }

    @Override
    public void onUp() {
        // User finger lift event
        PlayerGestureListener.super.onUp();
    }

    @Override
    public void onLongPress(MotionEvent e) {
         // User long press event
        PlayerGestureListener.super.onLongPress(e);
    }

    @Override
    public void onZoom() {
        // User two-finger zoom event
        PlayerGestureListener.super.onZoom();
    }

    @Override
    public void onZoomAnimation() {
        // Rebound animation when scaling to maximum or minimum
        PlayerGestureListener.super.onZoomAnimation();
    }

    @Override
    public void onZoomAnimationEnd() {
        // Zoom rebound animation ends
        PlayerGestureListener.super.onZoomAnimationEnd();
    }

    @Override
    public void onScroll() {
        // User finger swipe event
        PlayerGestureListener.super.onScroll();
    }

    @Override
    public void onFlingAnimation() {
        // Quick swipe animation
        PlayerGestureListener.super.onFlingAnimation();
    }

    @Override
    public void onFlingAnimationEnd() {
         // Quick swipe animation ends
        PlayerGestureListener.super.onFlingAnimationEnd();
    }
});
```

## WorkWrapper

WorkWrapper is a collection of all data in a slice file.

###  How to get WorkWrapper

You can scan media files from your camera or local directory to obtain`List < WorkWrapper >`.

> Note: This is a time-consuming operation that needs to be handled in a child thread.

```java
// Get from the camera
List<WorkWrapper> list = WorkUtils.getAllCameraWorks(
    InstaCameraManager.getInstance().getCameraHttpPrefix(),
    InstaCameraManager.getInstance().getCameraInfoMap(),
    InstaCameraManager.getInstance().getAllUrlList(),
    InstaCameraManager.getInstance().getRawUrlList()
);

// Get from local file
List<WorkWrapper> list = WorkUtils.getAllLocalWorks(String dirPath);

// If you have the URL of the media file, you can also create WorkWrapper yourself
String[] urls = {img1.insv, img2.insv, img3.insv};
WorkWrapper workWrapper = new WorkWrapper(urls);
```

### Read the content of WorkWrapper

The information you can obtain from `WorkWrapper` is as follows:

| Method                    | Parameter | Return value type | 说明                                                                                                                                               |
| ------------------------- | --------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| getIdenticalKey()         | -         | String            | For Glide or other DiskCacheKeys                                                                                                                 |
| getUrls()                 | -         | String\[]         | Get media file URL                                                                                                                               |
| getUrls()                 | boolean   | String\[]         | Get the URL of the media file. Set true/false to determine whether to include the dng file path                                                  |
| getWidth()                | -         | int               | Get width                                                                                                                                        |
| getHeight()               | -         | int               | Get highly                                                                                                                                       |
| getBitrate()              | -         | int               | Get the video bitrate. If it is a photo, return 0                                                                                                |
| getFps()                  | -         | double            | Get the video frame rate. If it is a photo, return 0                                                                                             |
| isPhoto()                 | -         | boolean           | Whether it is a photo                                                                                                                            |
| isHDRPhoto()              | -         | boolean           | Is it an HDR photo                                                                                                                               |
| supportPureShot()         | -         | boolean           | Whether to support PureShot algorithm                                                                                                            |
| isVideo()                 | -         | boolean           | Whether it is a video                                                                                                                            |
| isHDRVideo()              | -         | boolean           | Is it HDR video                                                                                                                                  |
| isCameraFile()            | -         | boolean           | Is the media file sourced from the camera device                                                                                                 |
| isLocalFile()             | -         | boolean           | Is the media file sourced from a mobile device                                                                                                   |
| getCreationTime()         | -         | long              | Obtain media file shooting timestamp, unit: ms                                                                                                   |
| getFirstFrameTimeOffset() | -         | long              | Obtain the offset of timestamp relative to camera startup when shooting media files, used to match gyroscope, exposure, and other data, unit: ms |
| getGyroInfo()             | -         | GyroInfo\[]       | Obtain gyroscope data for media files                                                                                                            |
| getExposureData()         | -         | ExposureData\[]   | Obtain exposure data for media files                                                                                                             |

## Image player

If you want to play images or videos, you must first create a WorkWrapper.

### Player initialization

Put the `InstaImagePlayerView` into an XML file and bind the lifecycle of the Activity.

```xml
<com.arashivision.sdkmedia.player.image.InstaImagePlayerView
    android:id="@+id/player_image"
    android:layout_width="match_parent"
    android:layout_height="match_parent" 
/>
```

```java
@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_preview);
    mImagePlayerView.setLifecycle(getLifecycle());
}
```

### Set playback parameters

```java
ImageParamsBuilder builder = new ImageParamsBuilder()
      // (Optional) Whether to enable dynamic stitching, default is true
      .setDynamicStitch(boolean dynamicStitch)
      // (Optional) Set the anti-shake type, default is InstaStabType STAB_TYPE_OFF
      .setStabType(@InstaStabType.StabType int mStabType)
      // (Optional) Set the playback proxy file, such as HDR.jpg generated by splicing, which is empty by default
      .setUrlForPlay(String url)
      // (Optional) Sets the render model type, default is Render_MODE_AUTO
      .setRenderModelType(@ImageParamsBuilder.RenderModel int renderModeType)
      // (Optional) Set the aspect ratio of the screen, default is full screen display
      // If the render mode type is "RENDER_mode_PLANE_STITCH", the recommended setting ratio is 2:1
      .setScreenRatio(int ratioX, int ratioY)
      // (Optional) Eliminate color difference in image stitching, default is false
      .setImageFusion(boolean imageFusion)
      // (Optional) Set the protective mirror type, default is OffsetType. ORIGINAL
      .setOffsetType(OffsetType offsetType)
      // (Optional) Whether to allow gesture operations, default is true
      .setGestureEnabled(boolean enabled);
      // (Optional) thumbnail cache folder, default is getCacheDir () + "/ work_thumbnail "
      .setCacheWorkThumbnailRootPath(String path)
      // (Optional) Stabilizer data cache folder, default is get CacheD ir () + "/ stabilizer "
      .setStabilizerCacheRootPath(String path)
      // (Optional) Cache folder, default is getExternalCacheDir () + "/ template_blender "
      .setCacheTemplateBlenderRootPath(String path)
      // (Optional) Star mode image cache folder, default is getCacheDir () + "/ cut_scene "
      .setCacheCutSceneRootPath(String path)；
```

### Start playing

WorkWrapper and playback parameters need to be set before playback.

```java
// Set WorkWrapper and playback parameters
mImagePlayerView.prepare(workWrapper, new ImageParamsBuilder());
// Start playing
mImagePlayerView.play();
```

### Release resources

When the activity is destroyed, release `InstaImagePlayerView`.

```java
@Override
protected void onDestroy() {
    super.onDestroy();
    mImagePlayerView.destroy();
}
```

### Switch rendering mode

> Note: If you want to switch between tiling mode and other modes, you must first restart the player.

If the rendering mode you are using is `RENDER_MODE_AUTO`, you can switch between the following modes.

```java
// Switch to normal mode
mImagePlayerView.switchNormalMode();

// Switch to fisheye mode
mImagePlayerView.switchFisheyeMode();

// Switch to perspective mode
mImagePlayerView.switchPerspectiveMode();

// To switch to tiling mode, you need to restart the player
ImageParamsBuilder builder = new ImageParamsBuilder()；
builder.setRenderModelType(ImageParamsBuilder.RENDER_MODE_PLANE_STITCH);
builder.setScreenRatio(2, 1);
mImagePlayerView.prepare(mWorkWrapper, builder);
mImagePlayerView.play();
```

### Field of view distance

You can customize the viewing angle and distance according to your preferences.

> Fov: Angle range 0\~ 180
> Distance: line of sight range 0\~ ∞

```java
mImagePlayerView.setConstraint(float minFov, float maxFov, float defaultFov, float minDistance, float maxDistance, float defaultDistance);
```

### Set listening

You can set the `PlayerViewListener` to observe changes in the player state.

```java
mImagePlayerView.setPlayerViewListener(new PlayerViewListener() {
    @Overridepublic void onLoadingStatusChanged(boolean isLoading) {
        // Player loading state change
        // isLoading: Whether loading
    }

    @Overridepublic void onLoadingFinish() {
        // Player loading complete
    }

    @Overridepublic void onFail(String desc) {
        // Player error
    }
});
```

### Gesture operation

You can set `PlayerGestureListener` to observe gesture actions.

```typescript
mImagePlayerView.setGestureListener(new PlayerGestureListener() {
    @Override
    public boolean onDown(MotionEvent e) {
        // User finger press event
        return PlayerGestureListener.super.onDown(e);
    }

    @Override
    public boolean onTap(MotionEvent e) {
        // User finger click event, similar to View.OnClickListener.OnClick(v)
        return PlayerGestureListener.super.onTap(e);
    }

    @Override
    public void onUp() {
        // User finger lift event
        PlayerGestureListener.super.onUp();
    }

    @Override
    public void onLongPress(MotionEvent e) {
         // User long press event
        PlayerGestureListener.super.onLongPress(e);
    }

    @Override
    public void onZoom() {
        // User two-finger zoom event
        PlayerGestureListener.super.onZoom();
    }

    @Override
    public void onZoomAnimation() {
        // Rebound animation when scaling to maximum or minimum
        PlayerGestureListener.super.onZoomAnimation();
    }

    @Override
    public void onZoomAnimationEnd() {
        // Zoom rebound animation ends
        PlayerGestureListener.super.onZoomAnimationEnd();
    }

    @Override
    public void onScroll() {
        // User finger swipe event
        PlayerGestureListener.super.onScroll();
    }

    @Override
    public void onFlingAnimation() {
        // Quick slide animation
        PlayerGestureListener.super.onFlingAnimation();
    }

    @Override
    public void onFlingAnimationEnd() {
         // Quick swipe animation ends
        PlayerGestureListener.super.onFlingAnimationEnd();
    }
});
```

## Video Player

### Player initialization

Put the `InstaVideoPlayerView` into an XML file and bind the lifecycle of the Activity.

```xml
<com.arashivision.sdkmedia.player.video.InstaVideoPlayerView
   android:id="@+id/player_video"
   android:layout_width="match_parent"
   android:layout_height="match_parent"
/>
```

```java
@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_preview);
    mVideoPlayerView.setLifecycle(getLifecycle());
}
```

### Set playback parameters

```java
VideoParamsBuilder builder = new VideoParamsBuilder()
      // (Optional) Load icon, default is empty
      .setLoadingImageResId(int resId)
      // (Optional) Background color during loading, default is black
      .setLoadingBackgroundColor(int color)
      // (Optional) Whether to play automatically
      .setAutoPlayAfterPrepared(boolean autoPlayAfterPrepared)
      // (Optional) Set the anti-shake type, default is InstaStabType STAB_TYPE_OFF
      .setStabType(@InstaStabType.StabType int stabType)
      // (Optional) Whether to loop, default is true
      .setIsLooping(boolean isLooping)
      // (Optional) Whether to enable dynamic stitching, default is true
      .setDynamicStitch(boolean dynamicStitch)
      // (Optional) Eliminate color difference in image stitching, default is false
      .setImageFusion(boolean imageFusion)
      // (Optional) Set the protective mirror type, default is OffsetType. ORIGINAL
      .setOffsetType(OffsetType offsetType)
      // (Optional) Sets the render model type, default is Render_MODE_AUTO
      .setRenderModelType(int renderModeType)
      // (Optional) Set the aspect ratio of the screen, default is full screen display.
      // If the render mode type is "RENDER_MODE_PLANE_STITCH", it is recommended to set the ratio to 2:1
      .setScreenRatio(int ratioX, int ratioY)
      // (Optional) Whether to allow gesture operations, default is true
      .setGestureEnabled(boolean enabled)
      // (Optional) thumbnail cache folder, default is getCacheDir () + "/ work_thumbnail "
      .setCacheWorkThumbnailRootPath(String path)
      // (Optional) Star mode image cache folder, default is getCacheDir () + "/ cut_scene "
      .setCacheCutSceneRootPath(String path);
```

### Start playing

WorkWrapper and playback parameters need to be set before playback.

```java
mVideoPlayerView.prepare(workWrapper, new VideoParamsBuilder());
mVideoPlayerView.play();
```

### Release resources

*`InstaVideoPlayerView`* 
When the activity is destroyed, release `InstaVideoPlayerView`.

```java
@Override
protected void onDestroy() {
    super.onDestroy();
    mVideoPlayerView.destroy();
}
```

### Playback control

You can control the playback progress, pause/resume, playback volume, etc. of the video by calling the code in `VideoPlayerView`.

| Method                  | Parameter         | Return value | Explanation                                     |
| ----------------------- | ----------------- | ------------ | ----------------------------------------------- |
| isPlaying               | -                 | boolean      | Is the video playing?                           |
| isLooping               | -                 | boolean      | Whether to loop                                 |
| setLooping              | boolean           | void         | Set whether to loop                             |
| setVolume               | float (Range 0-1) | void         | Set playback volume                             |
| pause                   | -                 | void         | Pause playback                                  |
| resume                  | -                 | void         | Resume playback                                 |
| seekTo                  | long              | void         | Fast forward to the specified progress playback |
| isSeeking               | -                 | boolean      | Is it fast forwarding?                          |
| getVideoCurrentPosition | -                 | long         | Get the current playback progress of the video  |
| getVideoTotalDuration   | -                 | long         | Get the total length of the video.              |

### Switch rendering mode

> Note: If you want to switch between tiling mode and other modes, you must first restart the player.

If the rendering mode you are using is `RENDER_MODE_AUTO`, you can switch between the following modes.

```java
// Switch to normal mode
mVideoPlayerView.switchNormalMode();

// Switch to fisheye mode
mVideoPlayerView.switchFisheyeMode();

// Switch to perspective mode
mVideoPlayerView.switchPerspectiveMode();

// To switch to tiling mode, you need to restart the player
VideoParamsBuilder builder = new VideoParamsBuilder()；
builder.setRenderModelType(ImageParamsBuilder.RENDER_MODE_PLANE_STITCH);
builder.setScreenRatio(2, 1);
mImagePlayerView.prepare(mWorkWrapper, builder);
mImagePlayerView.play();
```

### Field of view distance

You can customize the viewing angle and distance according to your preferences.

> Fov: Angle range 0\~ 180
> Distance: line of sight range 0\~ ∞

```java
mVideoPlayerView.setConstraint(float minFov, float maxFov, float defaultFov, float minDistance, float maxDistance, float defaultDistance);
```

### Set listening

You can set the `PlayerViewListener` to observe changes in the player state.

```java
mVideoPlayerView.setPlayerViewListener(new PlayerViewListener() {
    @Override
    public void onLoadingStatusChanged(boolean isLoading) {
        // Player loading state change
        // isLoading: Whether loading
    }

    @Overrid
    epublic void onLoadingFinish() {
        // Player loading complete
    }

    @Override
    public void onFail(String desc) {
        // Player error
    }
});
```

You can set `VideoStatusListener` to observe changes in video status.

```java
mVideoPlayerView.setVideoStatusListener(new VideoStatusListener() {
    @Override
    public void onProgressChanged(long position, long length) {
        // Play progress callback
        // Position: played length
        // Length: Total length of the video
    }

    @Override
    public void onPlayStateChanged(boolean isPlaying) {
        // Video playback status callback
        // isPlaying: Whether it is playing
    }

    @Override
    public void onSeekComplete() {
        // Fast forward done
    }

    @Override
    public void onCompletion() {
        // Video playback finished
    }
});
```

### Gesture operation

You can set `PlayerGestureListener` to observe gesture actions.

```typescript
mVideoPlayerView.setGestureListener(new PlayerGestureListener() {
    @Override
    public boolean onDown(MotionEvent e) {
        // User finger press event
        return PlayerGestureListener.super.onDown(e);
    }

    @Override
    public boolean onTap(MotionEvent e) {
        // User finger click event, similar to View.OnClickListener.OnClick(v)
        return PlayerGestureListener.super.onTap(e);
    }

    @Override
    public void onUp() {
        // User finger lift event
        PlayerGestureListener.super.onUp();
    }

    @Override
    public void onLongPress(MotionEvent e) {
         // User long press event
        PlayerGestureListener.super.onLongPress(e);
    }

    @Override
    public void onZoom() {
        // User two-finger zoom event
        PlayerGestureListener.super.onZoom();
    }

    @Override
    public void onZoomAnimation() {
        // Rebound animation when scaling to maximum or minimum
        PlayerGestureListener.super.onZoomAnimation();
    }

    @Override
    public void onZoomAnimationEnd() {
        // Zoom rebound animation ends
        PlayerGestureListener.super.onZoomAnimationEnd();
    }

    @Override
    public void onScroll() {
        // User finger swipe event
        PlayerGestureListener.super.onScroll();
    }

    @Override
    public void onFlingAnimation() {
        // Quick slide animation
        PlayerGestureListener.super.onFlingAnimation();
    }

    @Override
    public void onFlingAnimationEnd() {
         // Quick swipe animation ends
        PlayerGestureListener.super.onFlingAnimationEnd();
    }
});
```

## Export

### Export parameters

#### Image export parameters

If you want to export images, you need to first understand the `ExportImageParamsBuilder` image export parameters.

```java
ExportImageParamsBuilder builder = new ExportImageParamsBuilder()
    // (Must) Export file path
    .setTargetPath(String path)
    // (Optional) Set the width of the exported image, the default value is obtained from WorkWapper.
    .setWidth(int width)
    // (Optional) Set the height of the exported image, the default value is obtained from WorkWapper.
    .setHeight(int height)
    // (Optional) Set export mode, default is'PANORAMA'
    // ExportMode.PANORAMA: Export panoramic images
    // ExportMode.SPHERE: Export flat thumbnail
    .setExportMode(ExportUtils.ExportMode mode)
    // (Optional) Whether to enable dynamic stitching, default is true.
    .setDynamicStitch(boolean dynamicStitch)
    // (Optional) Set the anti-shake type, default is InstaStabType.STAB_TYPE_OFF.
    .setStabType(@InstaStabType.StabType int stabType)
    // (Optional) Eliminate color difference in image stitching, default is false
    .setImageFusion(boolean imageFusion)
    // (Optional) Set the protective mirror type, default is OffsetType. ORIGINAL.
    .setOffsetType(OffsetType offsetType)
    // (Optional) Set as HDR.jpg generated by splicing, default is null
    .setUrlForExport(String url)
    // (Optional) Set to use software decoder, default is false
    .setUseSoftwareDecoder(boolean useSoftwareDecoder)
    // (Optional) Set the camera angle. It is recommended to use it when exporting thumbnails.
    // The currently displayed angle parameter can be obtained from'PlayerView.getXXX () '.
    .setFov(float fov)
    .setDistance(float distance)
    .setYaw(float yaw)
    .setPitch(float pitch)
    // (Optional) Cache folder, default is getCacheDir () + "/ work_thumbnail "
    .setCacheWorkThumbnailRootPath(String path)
    // (Optional) Cache folder, default is get CacheD ir () + "/ stabilizer "
    .setStabilizerCacheRootPath(String path)
    // (Optional) Cache folder, default is getCacheDir () + "/ cut_scene "
    .setCacheCutSceneRootPath(String path);
```

#### Video export parameters

If you want to export the video, you need to first understand the video export parameters `ExportVideoParamsBuilder`.

```java
ExportVideoParamsBuilder builder = new ExportVideoParamsBuilder()
    // (Must) Export file path
    .setTargetPath(String path)
    // (Optional)Set the width of the exported video. It must be a power of 2, and the default value is obtained from WorkWapper.
    .setWidth(int width)
    // (Optional)Set the height of the exported video. It must be a power of 2, and the default value is obtained from WorkWapper.
    .setHeight(int height)
    // (Optional)Set the bitrate for exporting videos, with the default value being obtained from WorkWapper.
    .setBitrate(int bitrate)
    // (Optional)Set the fps for exporting videos, with the default value being obtained from WorkWapper.
    .setFps(int fps)
    // (Optional) Set export mode, default is'PANORAMA '
    // ExportMode.PANORAMA: Export panoramic images
    // ExportMode.SPHERE: Export flat thumbnail
    .setExportMode(ExportUtils.ExportMode mode)
    // (Optional) Eliminate color difference in image stitching, default is false
    .setImageFusion(boolean imageFusion)
    // (Optional) Set the protective mirror type, default is OffsetType. ORIGINAL.
    .setOffsetType(OffsetType offsetType)
    // (Optional) Whether to enable dynamic stitching, default is true.
    .setDynamicStitch(boolean dynamicStitch)
    // (Optional) Set the anti-shake type, default is InstaStabType.STAB_TYPE_OFF.
    .setStabType(@InstaStabType.StabType int stabType)
    // (Optional) Set to use software decoder, default is false
    .setUseSoftwareDecoder(boolean useSoftwareDecoder)
    // (Optional) Cache folder, default is getCacheDir () + "/ work_thumbnail "
    .setCacheWorkThumbnailRootPath(String path)
    // (Optional) Cache folder, default is getCacheDir () + "/ cut_scene "
    .setCacheCutSceneRootPath(String path)；
```

### Export panoramic images

Export images. Since the exported result is an image, use image to export parameters.

```java
ExportImageParamsBuilder builder = new ExportImageParamsBuilder()
        .setExportMode(ExportUtils.ExportMode.PANORAMA)
        .setTargetPath(path)
        .setWidth(2048)
        .setHeight(1024);
int exportId = ExportUtils.exportImage(WorkWrapper, builder, new IExportCallback() {
            @Override
            public void onSuccess() {       
                // Export successful        
            }

            @Override
            public void onFail() {
                // Export failed
            }

            @Override
            public void onCancel() {
                // Cancel export
            }

            @Override
            public void onProgress(float progress) {
                // Export progress
            }
        });
```

### Export image thumbnail

Export images. Since the exported result is an image, use images to export parameters.

```java
ExportImageParamsBuilder builder = new ExportImageParamsBuilder()
        .setExportMode(ExportUtils.ExportMode.SPHERE)
        .setTargetPath(path)
        .setWidth(512)
        .setHeight(512)
        .setFov(mImagePlayerView.getFov())
        .setDistance(mImagePlayerView.getDistance())
        .setYaw(mImagePlayerView.getYaw())
        .setPitch(mImagePlayerView.getPitch());
int exportId = ExportUtils.exportImage(WorkWrapper, builder, new IExportCallback() {
           @Override
            public void onSuccess() {       
                // Export successful       
            }

            @Override
            public void onFail() {
                // Export failed
            }

            @Override
            public void onCancel() {
                // Cancel export
            }

            @Override
            public void onProgress(float progress) {
                // Export progress
            }
        });
```

### Export panoramic video

Export video from video. Since the exported result is a video, use video to export parameters.

```java
ExportVideoParamsBuilder builder = new ExportVideoParamsBuilder()
        .setExportMode(ExportUtils.ExportMode.PANORAMA)
        .setTargetPath(path)
        .setWidth(2048)
        .setHeight(1024)
        .setBitrate(20 * 1024 * 1024)
        .setFps(30);
int exportId = ExportUtils.exportVideo(WorkWrapper, builder, new IExportCallback() {
            @Override
            public void onSuccess() {       
                // Export successful         
            }

            @Override
            public void onFail() {
                // Export failed
            }

            @Override
            public void onCancel() {
                // Cancel export
            }

            @Override
            public void onProgress(float progress) {
                // Export progress
            }
        });
```

### Export video thumbnail

Export images from videos. Since the exported result is an image, use image to export parameters.

```java
ExportImageParamsBuilder builder = new ExportImageParamsBuilder()
        .setExportMode(ExportUtils.ExportMode.SPHERE)
        .setTargetPath(path)
        .setWidth(512)
        .setHeight(512)
        .setFov(mVideoPlayerView.getFov())
        .setDistance(mVideoPlayerView.getDistance())
        .setYaw(mVideoPlayerView.getYaw())
        .setPitch(mVideoPlayerView.getPitch());
int exportId = ExportUtils.exportVideoToImage(WorkWrapper, builder, new IExportCallback() {
           @Override
            public void onSuccess() {       
                // Export successful         
            }

            @Override
            public void onFail() {
                // Export failed
            }

            @Override
            public void onCancel() {
                // Cancel export
            }

            @Override
            public void onProgress(float progress) {
                // Export progress
            }
        });
```

### Stop exporting

```java
ExportUtils.stopExport(exportId);
```

## Generate HDR images

If you have a WorkWrapper with HDR images, you can generate them into an image file using HDR stitching.

> Note: This is a time-consuming operation that needs to be handled in a child thread.

```java
boolean isSuccessful = StitchUtils.generateHDR(WorkWrapper workWrapper, String hdrOutputPath);
```

After the successful call of `generateHDR`, outputPath can be played as a proxy or set as an export URL.

```java
// Set as playback agent
ImageParamsBuilder builder = new ImageParamsBuilder()
        // If HDR splicing is successful, set it as a playback proxy
        .setUrlForPlay(isSuccessful ? hdrOutputPath : null);
mImagePlayerView.prepare(workWrapper, builder);
mImagePlayerView.play();

// Set as export url
ExportImageParamsBuilder builder = new ExportImageParamsBuilder()
        .setExportMode(ExportUtils.ExportMode.PANORAMA)
        .setTargetPath(exportPath)
        // If HDR splicing is successful, set it as export url
        .setUrlForExport(hdrOutputPath);
```

## Generate PureShot images

If you have a WorkWrapper of a PureShot image, you can generate it into an image file by stitching it together with PureShot.

> Note: This is a time-consuming operation that needs to be handled in a child thread.

```java
boolean isSuccessful = StitchUtils.generatePureShot(WorkWrapper workWrapper, String pureshotOutputPath, String algoFolderPath);
```

> Note: algoFolderPath can be LocalStoragePath or AssetsRelativePath. Please request the algorithm model file corresponding to the sdk version from our technical support personnel.

After the successful call to `generatePureShot`, outputPath can be played as a proxy or set as an export URL.

```java
// Set as playback agent
ImageParamsBuilder builder = new ImageParamsBuilder()
        // If PureShot splicing is successful, set it as a playback proxy
        .setUrlForPlay(isSuccessful ? pureshotOutputPath : null);
mImagePlayerView.prepare(workWrapper, builder);
mImagePlayerView.play();

// Set as export url
ExportImageParamsBuilder builder = new ExportImageParamsBuilder()
        .setExportMode(ExportUtils.ExportMode.PANORAMA)
        .setTargetPath(exportPath)
        // If PureShot splicing is successful, set it as the export url
        .setUrlForExport(pureshotOutputPath);
```

## Error code

### Export error code

| Error code | Explanation                                      |
| ---------- | ------------------------------------------------ |
| -31000     | Exporting in progress.                           |
| -31010     | Stop exporting                                   |
| -31011     | Failed to load Stabilization data                |
| -31012     | *EXPORT_TERMINATED_BY_WORK_REFERENCE_RELEASABLE* |
| -31016     | *EXPORT_POST_EXECUTE_FAILED*                     |
| -31017     | Original thumbnail LRV not found                 |
| -31018     | Original thumbnail LRV failed to load            |
| -31019     | *EXPORT_VIDEO_TO_IMAGE_SEQUENCE_CANCEL*          |
| -13016     | Export Timeout                                   |
| -13017     | Cancel export                                    |
| -13018     | *SHADOW_EXPORT_INIT_SHADOW_CLONE_FAIL*           |
| -13019     | *SHADOW_EXPORT_PREPARE_EXPORT_INFO_FAIL*         |
| -13021     | Failed to open hardware encoding                 |
| -13022     | *SHADOW_EXPORT_FAIL_TARGET_TIME_NULL*            |
| -13023     | *SHADOW_EXPORT_PREPRARE_FRAME_EXPORTER_FAIL*     |
| -13024     | *SHADOW_EXPORT_INPUT_TARGET_FAIL*                |
| -13025     | *STOP_MOTION_SELECT_POSE_CLIP_NULL*              |
| -13026     | *STOP_MOTION_SELECT_CLIP_NOT_MATCH*              |
| -13027     | *STOP_MOTION_SELECT_LOAD_STABILIZER_FAIL*        |
| -13028     | *STOP_MOTION_SELECT_KEY_NULL*                    |
| -13029     | *SHADOW_EXPORT_TRACK_TARGET_EMPTY*               |
| -13031     | Frame Exporter preparation failed                |
| -13033     | Reverse parsing video failed                     |
| -13034     | *Dual stream frame format is different*          |
| -13035     | Thumbnail Stabilization error                    |
| -13036     | Video frame rate is equal to 0                   |
| -13038     | Video width is equal to 0                        |
| -13039     | Video height is equal to 0                       |
| -13040     | The noise reduction model is equal to null.      |
| -13041     | The video source frame rate is equal to 0.       |
| -13042     | Video source duration is equal to 0              |
| -13043     | Image noise reduction file move error            |
| -13034     | Dual stream frame format is different            |
| -13044     | Failed to load AAA parameters                    |
| -13045     | Failed to create noise reduction data            |
| -13046     | Image width is equal to 0                        |
| -13047     | Image height is equal to 0                       |
