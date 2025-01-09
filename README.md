<a href="https://github.com/Insta360Develop/CameraSDK-Android/releases">
    <img src="https://img.shields.io/badge/version-1.5.3-green">
</a> 
<a href="https://developer.android.com/studio/publish/versioning#minsdkversion">
    <img src="https://img.shields.io/badge/minSdkVersion-21-green">
</a> 
<a href="https://developer.android.com/studio/publish/versioning#minsdkversion">
    <img src="https://img.shields.io/badge/targetSdkVersion-28-green">
</a> 
<a href="https://developer.android.com/studio/publish/versioning#minsdkversion">
    <img src="https://img.shields.io/badge/abi-arm64--v8a-orange">
</a> 


## Note: 
32-bit library (armeabi-v7a) is no longer maintained, please use 64-bit library (arm64-v8a) to build!

## How to get?
Please visit [https://www.insta360.com/sdk/apply](https://www.insta360.com/sdk/apply) to apply for the latest SDK.


# Table of Contents
- [Camera SDK Function](#CameraSDK功能)
  - [Initialization](#CameraSDK初始化)
  - [Connect / Disconnect / Observe Camera](#CameraSDK连接/断开/监听相机)
  - [Preview & Live](#CameraSDK预览)
  - [Capture](#CameraSDK拍摄)
  - [Settings](#CameraSDK属性设置)
  - [Firmware Upgrade](#CameraSDK固件升级)
  - [Other Functions](#CameraSDK其他功能)
  - [Get other camera information](#CameraSDK获取相机其他信息)
  - [One X2 Switch Camera Lens](#CameraSDK切换相机镜头)
- [Media SDK Function](#MediaSDK功能)
  - [Initialization](#MediaSDK初始化)
  - [Preview & Live](#MediaSDK预览)
  - [Player](#MediaSDK播放)
  - [Export](#MediaSDK导出)
  - [Generate HDR Image](#MediaSDK生成HDR图像)
  - [Generate PureShot Image](#MediaSDK生成HDR图像)
- [Error Code Comparison Table](https://github.com/Insta360Develop/CameraSDK-Android/blob/master/ErrorCode.md)
- [OSC](#CameraSDKOSC)
- [Proguard](#Proguard)
  
# <a name="CameraSDK功能" />Camera SDK Function

## <a name="CameraSDK初始化" />Initialization

First, add the maven address to your build file (the `build.gradle` file in the project root directory)

```Groovy
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

Second import the dependent library in your `build.gradle` file of app directory

```Groovy
dependencies {
    implementation 'com.arashivision.sdk:sdkcamera:1.5.3'
}
```

Then initialize SDK in Application

```Java
public class MyApp extends Application {

    @Override
    public void onCreate() {
        super.onCreate();      
        // Init SDK
        InstaCameraSDK.init(this);
    }

}
```



## <a name="CameraSDK连接/断开/监听相机" />Connect / Disconnect / Observe Camera

### Connect

You can connect the camera by bluetooth, Wi-Fi or USB.

> Note: You must do this on the main thread.

By BLE

```java
InstaCameraManager.getInstance().connectBle(bleDevice);
```

By Wi-Fi

```Java
InstaCameraManager.getInstance().openCamera(InstaCameraManager.CONNECT_TYPE_WIFI);
```

By USB

```Java
InstaCameraManager.getInstance().openCamera(InstaCameraManager.CONNECT_TYPE_USB);
```

You can also get the current camera connection type

```Java
int type = InstaCameraManager.getInstance().getCameraConnectedType();
```

And the result is one of `CONNECT_TYPE_NONE`, `CONNECT_TYPE_USB`, `CONNECT_TYPE_WIFI`.

For example, to check if the camera is connected, you can do the following:

```Java
private boolean isCameraConnected() {
    return InstaCameraManager.getInstance().getCameraConnectedType() != InstaCameraManager.CONNECT_TYPE_NONE;
}
```

### Close

When you want to disconnect the camera, you can call

```Java
InstaCameraManager.getInstance().closeCamera();
```

### Observe

You can `register / unregister` `ICameraChangedCallback` on multiple pages to to monitor changes in camera status.

```Java
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
     * Camera status changed
     *
     * @param enabled: Whether the camera is available
     */
    @Override
    public void onCameraStatusChanged(boolean enabled) {
    }

    /**
     * Camera connection failed
     * <p>
     * A common situation is that other phones or other applications of this phone have already
     * established a connection with this camera, resulting in this establishment failure,
     * and other phones need to disconnect from this camera first.
     */
    @Override
    public void onCameraConnectError(int errorCode) {
    }

    /**
     * SD card insertion notification
     *
     * @param enabled: Whether the current SD card is available
     */
    @Override
    public void onCameraSDCardStateChanged(boolean enabled) {
    }

    /**
     * SD card storage status changed
     *
     * @param freeSpace:  Currently available size
     * @param totalSpace: Total size
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
     * @param batteryLevel: Current power (0-100, always returns 100 when charging)
     * @param isCharging:   Whether the camera is charging
     */
    @Override
    public void onCameraBatteryUpdate(int batteryLevel, boolean isCharging) {
    }

}
```



## <a name="CameraSDK预览" />Preview & Live

### Preview

After the camera is successfully connected, you can manipulate the camera preview stream like this

```Java
public class PreviewActivity extends BaseObserveCameraActivity implements IPreviewStatusListener {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_preview);
        // Auto open preview after page gets focus
        InstaCameraManager.getInstance().setPreviewStatusChangedListener(this);
        InstaCameraManager.getInstance().startPreviewStream(PreviewStreamResolution.STREAM_1440_720_30FPS, InstaCameraManager.PREVIEW_TYPE_NORMAL);
    }

    @Override
    protected void onStop() {
        super.onStop();
        if (isFinishing()) {
            // Auto close preview after page loses focus
            InstaCameraManager.getInstance().setPreviewStatusChangedListener(null);
            InstaCameraManager.getInstance().closePreviewStream();
        }
    }

    @Override
    public void onOpening() {
        // Preview Opening       
    }

    @Override
    public void onOpened() {
        // Preview stream is on and can be played      
    }

    @Override
    public void onIdle() {
        // Preview Stopped
    }

    @Override
    public void onError() {
        // Preview Failed
    }
	
	@Override
    public void onExposureData(ExposureData exposureData) {
	// Callback frequency 500Hz
        // exposureData.timestamp: The time since the camera was turned on
	// exposureData.exposureTime: Rolling shutter exposure time
    }

    @Override
    public void onGyroData(List<GyroData> gyroList) {
	// Callback frequency 10Hz, 50 data per group
	// gyroData.timestamp: The time since the camera was turned on
	// gyroData.ax, gyroData.ay, gyroData.az: Three-axis acceleration
	// gyroData.gx, gyroData.gy, gyroData.gz: Three-axis gyroscope
    }

    @Override
    public void onVideoData(VideoData videoData) {
	// Callback frequency 500Hz
	// videoData.timestamp: The time since the camera was turned on
	// videoData.data: Preview raw stream data every frame
	// videoData.size: videoData.data.length
    }

}
```

> If the camera is passively disconnected or if you call `closeCamera` directly during the preview process, the SDK will automatically close the preview stream processing related state without the need to call `closePreviewStream`.

If you want to display the preview content, please see [Media SDK Function - Preview & Live](#MediaSDK预览)

### Preview Stream Resolution

You can choose one of the resolutions supported by the camera to set arguments. Get the supported resolutions by

```Java
List<PreviewStreamResolution> supportedList = InstaCameraManager.getInstance().getSupportedPreviewStreamResolution(int previewType);
```

### Preview Type

There are 3 kinds of `PreviewType` in `InstaCameraManager`. You must restart preview stream (close first and then start again) to switch between different operations.

* PREVIEW_TYPE_NORMAL - for normal preview or capture
* PREVIEW_TYPE_RECORD - for record
* PREVIEW_TYPE_LIVE - for live

### Live

You need to get supported resolution of the camera for live by

```Java
List<PreviewStreamResolution> supportedList = InstaCameraManager.getInstance().getSupportedPreviewStreamResolution(InstaCameraManager.PREVIEW_TYPE_LIVE);
```

For the X4, the preview stream resolution cannot be adjusted and is set to 832x1664 by default. For other models, the preview stream resolution must be set before starting live preview.

```Java
InstaCameraManager.getInstance().startPreviewStream(PreviewStreamResolution, InstaCameraManager.PREVIEW_TYPE_LIVE);
```

> Note: The preview stream is different between `Preview`, `Record` and `Live`, so you must restart preview stream if you want to switch between them.

Display the live preview content, please see [Media SDK Function - Preview & Live](#MediaSDK预览)

When the preview is ready, you can start live like this

```Java
LiveParamsBuilder builder = new LiveParamsBuilder()

        // (Must) Set the rtmp adress to push stream
        .setRtmp(String rtmp)
        
        // (Must) Set width to push, such as 1440
        .setWidth(int width)
        
        // (Must) Set height to push, such as 720
        .setHeight(int height)
        
        // (Must) Set fps to push, such as 30
        .setFps(int fps)
        
        // (Must) Set bitrate to push, such as 2*1024*1024
        .setBitrate(int bitrate)
        
        // (Optional) Whether the live is panorama or not, the default value is true
        .setPanorama(true);
     
InstaCameraManager.getInstance().startLive(builder, new ILiveStatusListener() {
        @Override
        public void onLivePushStarted() {
        }

        @Override
        public void onLivePushFinished() {
        }

        @Override
        public void onLivePushError() {
        }

        @Override
        public void onLiveFpsUpdate(int fps) {
        }
    });
```

You can stop live by this

```Java
InstaCameraManager.getInstance().stopLive();
```



## <a name="CameraSDK拍摄" />Capture

//For X4 you should switch the capture mode first before capture, InstaCameraManager.setFunctionModeToCamera(@FunctionMode int funcMode).
After the camera is successfully connected, you can control its capture. We provide you with several capture interfaces.

### Normal Capture

> set `true` if you want to capture the original image in RAW format

```Java
InstaCameraManager.getInstance().startNormalCapture(false);
```

### HDR Capture

> set `true` if you want to capture the original image in RAW format

```Java
InstaCameraManager.getInstance().startHDRCapture(false);
```

### Interval Capture

You can set interval time first (in ms)

```Java
InstaCameraManager.getInstance().setIntervalTime(int intervalMs);
```

Then capture
```Java
// Start
InstaCameraManager.getInstance().startIntervalShooting();

// Stop
InstaCameraManager.getInstance().stopIntervalShooting();
```

### Normal Record

```Java
// Start
InstaCameraManager.getInstance().startNormalRecord();

// Stop
InstaCameraManager.getInstance().stopNormalRecord();
```

### HDR Record

```Java
// Start
InstaCameraManager.getInstance().startHDRRecord();

// Stop
InstaCameraManager.getInstance().stopHDRRecord();
```

### TimeLapse

You can set interval time first (in ms)

```Java
InstaCameraManager.getInstance().setTimeLapseInterval(int intervalMs);
```

Then record
```Java
// Start
InstaCameraManager.getInstance().startTimeLapse();

// Stop
InstaCameraManager.getInstance().stopTimeLapse();
```

You can also get the current camera capture type

```Java
int type = InstaCameraManager.getInstance().getCurrentCaptureType();
```

And the result is one of `CAPTURE_TYPE_NORMAL_CAPTURE`, `CAPTURE_TYPE_HDR_CAPTURE`, `CAPTURE_TYPE_INTERVAL_SHOOTING`, `CAPTURE_TYPE_NIGHT_SCENE_CAPTURE`,
`CAPTURE_TYPE_BURST_CAPTURE`, `CAPTURE_TYPE_NORMAL_RECORD`, `CAPTURE_TYPE_HDR_RECORD`, `CAPTURE_TYPE_TIMELAPSE`, `CAPTURE_TYPE_STATIC_TIMELAPSE`, `CAPTURE_TYPE_BULLET_TIME_RECORD`, `CAPTURE_TYPE_TIME_SHIFT_RECORD`, `CAPTURE_TYPE_INTERVAL_RECORD`, `CAPTURE_TYPE_UNKNOWN`, `CAPTURE_TYPE_IDLE`

### Observe Status Changed

You can set `ICaptureStatusListener` to observe capture status changed.

> Note: You need to call this method every time the camera is successfully connected, because the listener is bound to the camera instance.

```Java
InstaCameraManager.getInstance().setCaptureStatusListener(new ICaptureStatusListener() {
            @Override
            public void onCaptureStarting() {
            }

            @Override
            public void onCaptureWorking() {
            }

            @Override
            public void onCaptureStopping() {
            }

            @Override
            public void onCaptureFinish(String[] filePaths) {
                // If you use sdk api to capture, the filePaths could be callback
                // Otherwise, filePaths will be null
            }

            @Override
            public void onCaptureCountChanged(int captureCount) {
                // Interval shots
                // Only Interval Capture type will callback this
            }

            @Override
            public void onCaptureTimeChanged(long captureTime) {            
                // Record Duration, in ms
                // Only Record type will callback this 
            }
        });
```


## <a name="CameraSDK属性设置" />Settings

### AAA Parameters

You can `set / get` the AAA parameters of the camera. 

> You need to specify a camera mode to set the parameters, see InstaCameraManager.FUNCTION_MODE_XXX.

Eg. Set/Get WB value

```Java
InstaCameraManager.getInstance().setWhiteBalance(InstaCameraManager.FUNCTION_MODE_CAPTURE_NORMAL, InstaCameraManager.WHITE_BALANCE_2700K);
int wb = InstaCameraManager.getInstance().getWhiteBalance(InstaCameraManager.FUNCTION_MODE_CAPTURE_NORMAL);
```

> Please check the AAA parameters and range table of each camera.
> [[ONE X](https://www.insta360.com/product/insta360-onex/)]
> [[ONE R](https://onlinemanual.insta360.com/oner/camera/parameters)]
> [[ONE X2](https://onlinemanual.insta360.com/onex2/camera/parameters)]
> [[X3](https://onlinemanual.insta360.com/x3/camera/basicuse/parameter#:~:text=Swipe%20left%20from%20the%20right,white%20balance%2C%20and%20other%20parameters.)]
> [[X4](https://onlinemanual.insta360.com/x4/faq/feature/shootingmode)]

### CaptureSetting

- **Set/GetResolutionToCamera**

   Set/Get the resolution of the specified camera mode

  > @params FunctionMode 
  >
  > ​	see InstaCameraManager.FUNCTION_MODE_XXX
  >
  > @params PreviewStreamResolution 
  >
  >  	You can choose one of the resolutions supported by the camera to set arguments. Get the supported resolutions by CaptureSettingSupportConfig#getSupportedResolution

  ```java
   InstaCameraManager#setResolutionToCamera(FunctionMode, PreviewStreamResolution)
  ```

  ```java
  InstaCameraManager#getResolutionFromCamera(FunctionMode)
  ```


### Camera Beep

You can `set / get` whether there is a beep sound when the camera shoots.

Set Beep Enabled

```Java
InstaCameraManager.getInstance().setCameraBeepSwitch(boolean beep);
```

Determine Whether beep is enabled

```Java
boolean isBeep = InstaCameraManager.getInstance().isCameraBeep();
```

## <a name="CameraSDK固件升级" />Firmware Upgrade

> Obtain the firmware upgrade file from our person in charge and put it on your server or directly use the firmware link on our official website.
>
> You can use the `Firmware Upgrade` interface after downloading the file to the local.

> Note: Please do not upload the wrong file to the camera to avoid bricking. Example: Do not upload OneX firmware upgrade package to OneX2 camera.

Start Upgrading

```Java
String fwFilePath = "/xxx/InstaOneXFW.bin";
FwUpgradeManager.getInstance().startUpgrade(fwFilePath, new FwUpgradeListener() {
            @Override
            public void onUpgradeSuccess() {

            }

            @Override
            public void onUpgradeFail(int errorCode, String message) {
                // errorCode see FwUpgradeErrorCode.java
                // -1000: Already in upgrading
                // -1001: Please connect the camera first
                // -1002: The upgrade will fail when the battery level of the camera is lower than 12%, 
                //        please make sure that the battery level of the camera is sufficient
                // -14004/400/500: Http Server Error
                // -14005: Socket IO exception
                // -14046: be cancelled, it will not be called back to this interface, but will be called back to onUpgradeCancel()
            }

            @Override
            public void onUpgradeCancel() {

            }

            @Override
            public void onUpgradeProgress(double progress) {
                // progress: 0-1
            }
        });
```

Cancel Upgrade

```Java
FwUpgradeManager.getInstance().cancelUpgrade();
```

Determine whether it is being upgraded

```Java
FwUpgradeManager.getInstance().isUpgrading();
```


## <a name="CameraSDK其他功能" />Other Function

### Calibrate Gyro

> This function must be used when the camera is connected to WIFI
>
> Before calibrate, please stand the camera upright on a stable and level surface.

```Java
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
### Set GPS Data
> Add GPS data to recording videos

```Java
    InstaCameraManager.getInstance().setCaptureStatusListener(new ICaptureStatusListener() {

        @Override
        public void onCaptureWorking() {
            startSendGpsData();
        }

        @Override
        public void onCaptureFinish(String[] filePaths) {
            stopCameraWork();
        }
    });

    private void startSendGpsData() {
        mCollectGpsDataList.clear();
        mMainHandler.post(mCollectGpsDataRunnable);
    }

    private void stopSendGpsData() {
        if (!mCollectGpsDataList.isEmpty()) {
            InstaCameraManager.getInstance().setGpsData(GpsData.GpsData2ByteArray(mCollectGpsDataList));
            mCollectGpsDataList.clear();
        }
        mMainHandler.removeCallbacks(mCollectGpsDataRunnable);
    }

    private List<GpsData> mCollectGpsDataList = new ArrayList<GpsData>();
    private Runnable mCollectGpsDataRunnable = new Runnable(){
        @Override
        public void run() {
            Location location = LocationManager.getInstance().getCurrentLocation();
            if (location != null) {
                GpsData gpsData = new GpsData();
                gpsData.setLatitude(location.getLatitude());
                gpsData.setLongitude(location.getLongitude());
                gpsData.setGroundSpeed(location.getSpeed());
                gpsData.setGroundCrouse(location.getBearing());
                gpsData.setGeoidUndulation(location.getAltitude());
                gpsData.setUTCTimeMs(location.getTime());
                gpsData.setVaild(true);
                mCollectGpsDataList.add(gpsData);
                if (mCollectGpsDataList.size() >= 20) {
                    InstaCameraManager.getInstance().setGpsData(GpsData.GpsData2ByteArray(mCollectGpsDataList));
                    mCollectGpsDataList.clear();
                }
            }
            mMainHandler.postDelayed(this, 100);
        }
    };
```

### Format SD card

```Java
InstaCameraManager.getInstance().formatStorage(new ICameraOperateCallback() {
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

### Delete Files from Camera

> Note: Please do not pass in wrong path parameters, so we recommend that you get the valid urls from the WorkWrapper.

```Java
List<String> fileUrls = Arrays.asList(workWrapper.getUrlsForDelete());
InstaCameraManager.getInstance().deleteFileList(fileUrls, new ICameraOperateCallback() {
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



## <a name="CameraSDK获取相机其他信息" />Get other camera information

The following methods are only used as parameters for other interfaces to call. Developers don't need to pay much attention to them, and they will be explained later at the places where they need to be called.

> InstaCameraManager.getInstance().xxx();

|Method|Type|Explanation|
|---|---|---|
|getCameraType()|String|Camera Type|
|getCameraVersion()|String|Camera Version|
|getCameraSerial()|String|Camera Serial|
|getMediaOffset()|String|Camera Media Offset|
|getMediaOffsetV2()|String|Camera Media Offset V2|
|getMediaOffsetV3()|String|Camera Media Offset V3|
|isCameraSelfie()|boolean|Camera Selfie|
|getGyroTimeStamp()|double|Only use for preview|
|getBatteryType()|int|Only use for preview|
|getCameraCurrentBatteryLevel()|int|Camera Battery Level, form 0 to 100|
|isCameraCharging()|boolean|Camera Charge State|
|getCameraStorageTotalSpace()|long|Camera Storage Total Space, bytes|
|getCameraStorageFreeSpace()|long|Camera Storage Free Space, bytes|
|isSdCardEnabled()|boolean|Camera SD card state|
|getCameraHttpPrefix()|String|Camera Host|
|getAllUrlList()<br>getRawUrlList()<br>getCameraInfoMap()|List\<String\>|Camera File List (Exclued Recording File)|
|getAllUrlListIncludeRecording()|List\<String\>|Camera File List (Include Recording File)|
|getWifiInfo()|WifiInfo|WifiInfo(ssid: String, pwd: String, macAddress: String, channel: int, mode: int, state: int, pwdVersion: int)|
|setInternalSplicingEnable(boolean enable)|void|Enable device stitching only support on x4.|


## <a name="CameraSDK切换相机镜头" />OneX2 Switch Camera Lens

```java
/**
*	CameraMode: lens mode
*	CAMERA_MODE_SINGLE_FRONT -> Single Lens Mode - Front Lens
*	CAMERA_MODE_SINGLE_REAR -> Single Lens Mode - Rear Lens
*	CAMERA_MODE_PANORAMA -> Panoramic Lens Mode
*/

/**
*	FocusSensor：focus mode
*	FOCUS_SENSOR_FRONT -> The front lens is the main focus
*	FOCUS_SENSOR_REAR -> The rear lens is the main focus
*	FOCUS_SENSOR_ALL -> Both lenses are the main focus
*/
InstaCameraManager.getInstance().switchCameraMode(InstaCameraManager.CAMERA_MODE_xxx, InstaCameraManager.FOCUS_SENSOR_xxx, callback);

// Get current camera lens state
InstaCameraManager.getInstance().getCurrentCameraMode();
InstaCameraManager.getInstance().getCurrentFocusSensor();
```

> Note: When CameraMode is `Single Lens Mode`, FocusSensor should correspond to CameraMode `Front/Rear`.
>        When CameraMode is `Panorama Lens Mode`, if FocusSensor is `All`, it is a normal panorama, and FocusSensor is `Front/Rear`, which should be used in Insta Pano photo mode.



------



# <a name="MediaSDK功能" />Media SDK Function

## <a name="MediaSDK初始化" />Initialization


First add the maven address to your build file (`build.gradle` file in the project root directory)

```Groovy
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

Second import the dependent library in your `build.gradle` file of app directory

```Groovy
dependencies {
    implementation 'com.arashivision.sdk:sdkmedia:1.5.3'
}
```

Then initialize SDK in Application

```Java
public class MyApp extends Application {

    @Override
    public void onCreate() {
        super.onCreate();      
        // Init SDK
        InstaMediaSDK.init(this);
    }

}
```



## <a name="MediaSDK预览" />Preview & Live

If you already import `CameraSDK`, you can open and display the preview content.

Put `InstaCapturePlayerView` in your xml file

```xml
<com.arashivision.sdkmedia.player.capture.InstaCapturePlayerView
    android:id="@+id/player_capture"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@android:color/darker_gray" />
```

Bind lifecycle when your activity created

```Java
@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_preview);
    mCapturePlayerView.setLifecycle(getLifecycle());
}
```

Display preview in `IPreviewStatusListener.onOpened ()` callback of `CameraSdk`

```Java
@Override
public void onOpened() {
    InstaCameraManager.getInstance().setStreamEncode();
    mCapturePlayerView.setPlayerViewListener(new PlayerViewListener() {
        @Override
        public void onLoadingFinish() {
            // Must do this
            Object pipeline = mCapturePlayerView.getPipeline();
            InstaCameraManager.getInstance().setPipeline(pipeline);
        }
        
        @Override
        public void onLoadingStatusChanged(boolean isLoading) {
        }

        @Override
        public void onFail(String desc) {
        }
    });
    mCapturePlayerView.prepare(createParams());
    mCapturePlayerView.play();
}

private CaptureParamsBuilder createParams() {
    CaptureParamsBuilder builder = new CaptureParamsBuilder()
            .setCameraType(InstaCameraManager.getInstance().getCameraType())
			.setMediaOffset(InstaCameraManager.getInstance().getMediaOffset())
			.setMediaOffsetV2(InstaCameraManager.getInstance().getMediaOffsetV2())
			.setMediaOffsetV3(InstaCameraManager.getInstance().getMediaOffsetV3())
			.setCameraSelfie(InstaCameraManager.getInstance().isCameraSelfie())
			.setGyroTimeStamp(InstaCameraManager.getInstance().getGyroTimeStamp())
			.setBatteryType(InstaCameraManager.getInstance().getBatteryType());
    return builder;
}
```

Release `InstaCapturePlayerView` when preview is closed

```Java
@Override
protected void onStop() {
    super.onStop();
    if (isFinishing()) {
        // Auto close preview after page loses focus
        InstaCameraManager.getInstance().setPreviewStatusChangedListener(null);
        InstaCameraManager.getInstance().closePreviewStream();
        mCapturePlayerView.destroy();
    }
}
    
@Override
public void onIdle() {
    // Preview Stopped
    mCapturePlayerView.destroy();
}
```

You can configure more options by `CaptureParamsBuilder`

```Java
private CaptureParamsBuilder createParams() {
    CaptureParamsBuilder builder = new CaptureParamsBuilder()
    
            // (Must) The parameters are fixed
            .setCameraType(InstaCameraManager.getInstance().getCameraType())
            
            // (Must) The parameters are fixed
            .setMediaOffset(InstaCameraManager.getInstance().getMediaOffset())
			
			// (Must) The parameters are fixed
			.setMediaOffsetV2(InstaCameraManager.getInstance().getMediaOffsetV2())
			
			// (Must) The parameters are fixed
			.setMediaOffsetV3(InstaCameraManager.getInstance().getMediaOffsetV3())

            // (Must) The parameters are fixed
            .setCameraSelfie(InstaCameraManager.getInstance().isCameraSelfie())
			
			// (Must) The parameters are fixed
			.setGyroTimeStamp(InstaCameraManager.getInstance().getGyroTimeStamp())
			
			// (Must) The parameters are fixed
			.setBatteryType(InstaCameraManager.getInstance().getBatteryType())
            
            // (Depends on) If you start preview for live, this is required.
            .setLive(true)
            
            // (Depends on) If you use custom resloution to start preview, this is required.
            .setResolutionParams(mCurrentResolution.width, mCurrentResolution.height, mCurrentResolution.fps);
            
            // (Optional) Whether to enable stabilization, the default is true
            .setStabEnabled(true)
            
            // (Optional) if setStabEnabled(true), you can choose the type of stabilization type, the default is InstaStabType.STAB_TYPE_AUTO
            // InstaStabType.STAB_TYPE_AUTO: Use the official default stabilization type for different cameras.
            // InstaStabType.STAB_TYPE_PANORAMA: Panoramic stabilization, keep the screen still.
            // InstaStabType.STAB_TYPE_CALIBRATE_HORIZON: Align the horizon only. (The roll axis does not move, the yaw pitch changes)
            // InstaStabType.STAB_TYPE_FOOTAGE_MOTION_SMOOTH: Smooth footage motion, no horizon alignment. (yaw pitch roll changes with the camera)
            .setStabType(InstaStabType.STAB_TYPE_AUTO)
            
            // (Optional) Whether to allow gesture operations, the default is true
            .setGestureEnabled(true)
            
            // (Optional) If you want to show preview stream on your custom surface
            // (Note) Every time you open a new preview, you must create a new Surface and pass in parameters. You cannot reuse Surface
            // (Note) Destroy your surface when priview is idle
            .setCameraRenderSurfaceInfo(surface, surfaceWidth, surfaceHeight);
            
    if (You want to preview as plane mode) {
    
        // Plane Mode
        // (Optional) Set Render Model Type, the default is `RENDER_MODE_AUTO`
        builder.setRenderModelType(CaptureParamsBuilder.RENDER_MODE_PLANE_STITCH)
        
                // (Optional) Set the aspect ratio of the screen, the default is full screen display (ie full canvas)
                // If the rendering mode type is `RENDER_MODE_PLANE_STITCH`, the recommended setting ratio is 2:1
                .setScreenRatio(2, 1);
                
    } else {
    
        // Normal Mode
        builder.setRenderModelType(CaptureParamsBuilder.RENDER_MODE_AUTO);
        
    }
    return builder;
}
```

if you use `RENDER_MODE_AUTO`, you can switch between the following modes.

Switch to Normal Mode

```Java
mCapturePlayerView.switchNormalMode();
```

Switch to Fisheye Mode

```Java
mCapturePlayerView.switchFisheyeMode();
```

Switch to Perspective Mode

```Java
mCapturePlayerView.switchPerspectiveMode();
```

You can customize the player angle according to your preferences.

> Fov: Viewing width. Range 0(inclusive) ~ 180(exclusive)
>
> Distance: Distance from observation point to observed point. Range 0(inclusive) ~ ∞

```Java
mCapturePlayerView.setConstraint(float minFov, float maxFov, float defaultFov, float minDistance, float maxDistance, float defaultDistance);
```

if you want to switch between `Plane Mode` and others, you must restart preview first.

```Java
if (current mode is Normal && new mode is Plane){
    InstaCameraManager.getInstance().closePreviewStream();
    InstaCameraManager.getInstance().startPreviewStream(resolution, previewType);
}
```

You can set `PlayerGestureListener` to observe gesture operation.

```Java
mCapturePlayerView.setGestureListener(new PlayerGestureListener() {
    @Override
    public boolean onDown(MotionEvent e) {
        return false;
    }

    @Override
    public boolean onTap(MotionEvent e) {
        return false;
    }

    @Override
    public void onUp() {
    }

    @Override
    public void onLongPress(MotionEvent e) {
    }

    @Override
    public void onZoom() {
    }

    @Override
    public void onZoomAnimation() {
    }

    @Override
    public void onZoomAnimationEnd() {
    }

    @Override
    public void onScroll() {
    }

    @Override
    public void onFlingAnimation() {
    }

    @Override
    public void onFlingAnimationEnd() {
    }
});
```



## <a name="MediaSDK播放" />Player

If you want to play an image or a video, you must create a `WorkWrapper` object first.

You can scan media files from camera or a local directory to get `List<WorkWrapper>`.

Scan from camera 

> Note: This is a time-consuming operation and needs to be processed in a child thread. Not recommended for non-essential situations, you can get the file path and store it after shooting.

```Java
List<WorkWrapper> list = WorkUtils.getAllCameraWorks(
    InstaCameraManager.getInstance().getCameraHttpPrefix(),
    InstaCameraManager.getInstance().getCameraInfoMap(),
    InstaCameraManager.getInstance().getAllUrlList(),
    InstaCameraManager.getInstance().getRawUrlList());
```

Scan from local directory

> Note: This is a time-consuming operation and needs to be processed in a child thread.

```Java
List<WorkWrapper> list = WorkUtils.getAllLocalWorks(String dirPath);
```

or if you have urls of the media file, you can also create `WorkWrapper` by yourself.

```Java
String[] urls = {img1.insv, img2.insv, img3.insv};
WorkWrapper workWrapper = new WorkWrapper(urls);
```

You can get the media info based from the `workWrapper`

|Method|Type|Explanation|
|---|---|---|
|getIdenticalKey()|String|Uniquely Identify, used for `DiskCacheKey of Glide` or others|
|getUrls()|String[]|Get media file urls|
|getUrls(boolean)|String[]|Get media file urls. Set true/false to decide whether to include the dng file path|
|getWidth()|int|Get media width|
|getHeight()|int|Get media height|
|getBitrate()|int|Get video bitrate, return 0 if is photo|
|getFps()|double|Get video fps, return 0 if is photo|
|isPhoto()|boolean|Whether the media is photo|
|isHDRPhoto()|boolean|Whether the media is hdr photo|
|supportPureShot()|boolean|Whether the media support PureShot algo|
|isVideo()|boolean|Whether the media is video|
|isHDRVideo()|boolean|Whether the media is hdr video|
|isCameraFile()|boolean|Whether the media file is from camera|
|isLocalFile()|boolean|Whether the media is is from device|
|getCreationTime()|long|Get media capture timestamp, in ms|
|getFirstFrameTimeOffset()|long|Get the offset of the timestamp relative to the camera startup when the media file was taken, used to match Gyro, Exposure and other data, in ms|
|getGyroInfo()|GyroInfo[]|Get media gyro data|
|getExposureData()|ExposureData[]|Get media exposure data|


### Image Player

You can play an image by `InstaImagePlayerView`.

Put `InstaImagePlayerView` in your xml file

```xml
<com.arashivision.sdkmedia.player.image.InstaImagePlayerView
    android:id="@+id/player_image"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

Bind the lifecycle when your activity is created.

```Java
@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_preview);
    mImagePlayerView.setLifecycle(getLifecycle());
}
```

Build parameters and play

```Java
mImagePlayerView.prepare(workWrapper, new ImageParamsBuilder());
mImagePlayerView.play();
```

Release the `InstaImagePlayerView` when the activity is destroyed.

```Java
@Override
protected void onDestroy() {
    super.onDestroy();
    mImagePlayerView.destroy();
}
```

You can configure more options by `ImageParamsBuilder`

```Java
ImageParamsBuilder builder = new ImageParamsBuilder()

      // (Optional) Whether to enable dynamic stitching, the default is true.
      .setDynamicStitch(boolean dynamicStitch)
      
      // (Optional) Whether to enable stabilization, the default is true
      .setStabEnabled(true)
      
      // (Optional) Set playback proxy file, such as HDR.jpg generated by stitching, the default is null
      .setUrlForPlay(String url)
      
      // (Optional) Set Render Model Type, the default is `RENDER_MODE_AUTO`
      .setRenderModelType(int renderModeType)
      
      // (Optional) Set the aspect ratio of the screen, the default is full screen display (ie full canvas)
      // If the rendering mode type is `RENDER_MODE_PLANE_STITCH`, the recommended setting ratio is 2:1
      .setScreenRatio(int ratioX, int ratioY)
      
      // (Optional) Eliminate the chromatic aberration of image stitching, the default is false
      .setImageFusion(boolean imageFusion)
      
      // (Optional) Whether to allow gesture operations, the default is true
      .setGestureEnabled(boolean enabled);
      
      // (Optional) Cache Folder, the default is getCacheDir() + "/work_thumbnail"
      .setCacheWorkThumbnailRootPath(String path)
      
      // (Optional) Cache Folder, the default is getCacheDir() + "/stabilizer"
      .setStabilizerCacheRootPath(String path)
      
      // (Optional) Cache Folder, the default is getCacheDir() + "/cut_scene"
      .setCacheCutSceneRootPath(String path)
```

if you use `RENDER_MODE_AUTO`, you can switch between the following modes.

> if you want to switch between `Plane Mode` and others, you must restart player first.

Switch to Normal Mode

```Java
mImagePlayerView.switchNormalMode();
```

Switch to Fisheye Mode

```Java
mImagePlayerView.switchFisheyeMode();
```

Switch to Perspective Mode

```Java
mImagePlayerView.switchPerspectiveMode();
```

You can customize the player angle according to your preferences.

> Fov: Viewing width. Range 0(inclusive) ~ 180(exclusive)
>
> Distance: Distance from observation point to observed point. Range 0(inclusive) ~ ∞

```Java
mImagePlayerView.setConstraint(float minFov, float maxFov, float defaultFov, float minDistance, float maxDistance, float defaultDistance);
```

You can set `PlayerViewListener` to observe player status changed.

```Java
mImagePlayerView.setPlayerViewListener(new PlayerViewListener() {
    @Override
    public void onLoadingStatusChanged(boolean isLoading) {
    }

    @Override
    public void onLoadingFinish() {
    }

    @Override
    public void onFail(String desc) {
    }
});
```

You can set `PlayerGestureListener` to observe gesture operation.

```Java
mImagePlayerView.setGestureListener(new PlayerGestureListener() {
    @Override
    public boolean onDown(MotionEvent e) {
        return false;
    }

    @Override
    public boolean onTap(MotionEvent e) {
        return false;
    }

    @Override
    public void onUp() {
    }

    @Override
    public void onLongPress(MotionEvent e) {
    }

    @Override
    public void onZoom() {
    }

    @Override
    public void onZoomAnimation() {
    }

    @Override
    public void onZoomAnimationEnd() {
    }

    @Override
    public void onScroll() {
    }

    @Override
    public void onFlingAnimation() {
    }

    @Override
    public void onFlingAnimationEnd() {
    }
});
```


### Video Player

You can play a video by `InstaVideoPlayerView`.

Put `InstaVideoPlayerView` in your xml file

```xml
<com.arashivision.sdkmedia.player.video.InstaVideoPlayerView
    android:id="@+id/player_video"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

Bind lifecycle when your activity created

```Java
@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_preview);
    mVideoPlayerView.setLifecycle(getLifecycle());
}
```

Build parameters and play

```Java
mVideoPlayerView.prepare(workWrapper, new VideoParamsBuilder());
mVideoPlayerView.play();
```

Release `InstaVideoPlayerView` when activity is destory

```Java
@Override
protected void onDestroy() {
    super.onDestroy();
    mVideoPlayerView.destroy();
}
```

You can configure more options by `VideoParamsBuilder`

```Java
VideoParamsBuilder builder = new VideoParamsBuilder()

      // (Optional) Loading icon, default is none
      .setLoadingImageResId(int resId)
      
      // (Optional) Background color when loading, default is black
      .setLoadingBackgroundColor(int color)
      
      // (Optional) Whether to play automatically after preparation, the default is true
      .setAutoPlayAfterPrepared(boolean autoPlayAfterPrepared)
      
      // (Optional) Whether to enable stabilization, the default is true
      .setStabEnabled(true)
      
      // (Optional) Whether to loop playback, the default is true
      .setIsLooping(boolean isLooping)
      
      // (Optional) Set Render Model Type, the default is `RENDER_MODE_AUTO`
      .setRenderModelType(int renderModeType)
      
      // (Optional) Set the aspect ratio of the screen, the default is full screen display (ie full canvas)
      // If the rendering mode type is `RENDER_MODE_PLANE_STITCH`, the recommended setting ratio is 2:1
      .setScreenRatio(int ratioX, int ratioY)
      
      // (Optional) Whether to allow gesture operations, the default is true
      .setGestureEnabled(boolean enabled)
      
      // (Optional) Cache Folder, the default is getCacheDir() + "/work_thumbnail"
      .setCacheWorkThumbnailRootPath(String path)
      
      // (Optional) Cache Folder, the default is getCacheDir() + "/cut_scene"
      .setCacheCutSceneRootPath(String path)
```

if you use `RENDER_MODE_AUTO`, you can switch between the following modes.

> if you want to switch between `Plane Mode` and others, you must restart player first.

Switch to Normal Mode

```Java
mVideoPlayerView.switchNormalMode();
```

Switch to Fisheye Mode

```Java
mVideoPlayerView.switchFisheyeMode();
```

Switch to Perspective Mode

```Java
mVideoPlayerView.switchPerspectiveMode();
```

You can customize the player angle according to your preferences.

> Fov: Viewing width. Range 0(inclusive) ~ 180(exclusive)
>
> Distance: Distance from observation point to observed point. Range 0(inclusive) ~ ∞

```Java
mVideoPlayerView.setConstraint(float minFov, float maxFov, float defaultFov, float minDistance, float maxDistance, float defaultDistance);
```

You can set `PlayerViewListener` to observe player status changed.

```Java
mVideoPlayerView.setPlayerViewListener(new PlayerViewListener() {
    @Override
    public void onLoadingStatusChanged(boolean isLoading) {
    }

    @Override
    public void onLoadingFinish() {
    }

    @Override
    public void onFail(String desc) {
    }
});
```

You can set `VideoStatusListener` to observe video status changed.

```Java
mVideoPlayerView.setVideoStatusListener(new VideoStatusListener() {
    @Override
    public void onProgressChanged(long position, long length) {
    }

    @Override
    public void onPlayStateChanged(boolean isPlaying) {
    }

    @Override
    public void onSeekComplete() {
    }

    @Override
    public void onCompletion() {
    }
});
```

You can set `PlayerGestureListener` to observe gesture operation.

```Java
mVideoPlayerView.setGestureListener(new PlayerGestureListener() {
    @Override
    public boolean onDown(MotionEvent e) {
        return false;
    }

    @Override
    public boolean onTap(MotionEvent e) {
        return false;
    }

    @Override
    public void onUp() {
    }

    @Override
    public void onLongPress(MotionEvent e) {
    }

    @Override
    public void onZoom() {
    }

    @Override
    public void onZoomAnimation() {
    }

    @Override
    public void onZoomAnimationEnd() {
    }

    @Override
    public void onScroll() {
    }

    @Override
    public void onFlingAnimation() {
    }

    @Override
    public void onFlingAnimationEnd() {
    }
});
```

Other `VideoPlayerView` operates

```Java
// Whether the video is playing
isPlaying();

// Whether the video is looping
setLooping(boolean isLooping);
isLooping();

// Set Volume (Range 0-1)
setVolume(float volume);

// Pause Video
pause();

// Resume Video
resume();

// Seek to Play
seekTo(long position);
isSeeking();

// Get the current playback position
getVideoCurrentPosition();

// Get video total duration
getVideoTotalDuration();
```


## <a name="MediaSDK导出" />Export

You can export `WorkWrapper` to an image or a video file.

if you want to export an image, you need to know `ExportImageParamsBuilder` first.

```Java
ExportImageParamsBuilder builder = new ExportImageParamsBuilder()

    // (Must) Set the export file path
    .setTargetPath(String path)

    // (Optional) Set the width of the exported image, the default is get from WorkWapper.
    .setWidth(int width)

    // (Optional) Set the height of the exported image, the default is get from WorkWapper.
    .setHeight(int height)

    // (Optional) Set export mode, default is `PANORAMA`
    // ExportMode.PANORAMA: Use when exporting panorama media
    // ExportMode.SPHERE: Use when exporting flat thumbnails
    .setExportMode(ExportUtils.ExportMode mode)

    // (Optional) Whether to enable dynamic stitching, the default is true.
    .setDynamicStitch(boolean dynamicStitch)
    
    // (Optional) Eliminate the chromatic aberration of image stitching, the default is false
    .setImageFusion(boolean imageFusion)

    // (Optional) Whether to enable stabilization, the default is true
    .setStabEnabled(true)
    
    // (Optional) Set such as HDR.jpg generated by stitching, the default is null
    .setUrlForExport(String url)
      
    // (Optional) Set the camera angle. It is recommended to use when exporting thumbnails. 
    // The currently displayed angle parameters can be obtained from `PlayerView.getXXX()`.
    .setFov(float fov)
    .setDistance(float distance)
    .setYaw(float yaw)
    .setPitch(float pitch)

    // (Optional) Cache Folder, the default is getCacheDir() + "/work_thumbnail"
    .setCacheWorkThumbnailRootPath(String path)

    // (Optional) Cache Folder, the default is getCacheDir() + "/stabilizer"
    .setStabilizerCacheRootPath(String path)

    // (Optional) Cache Folder, the default is getCacheDir() + "/cut_scene"
    .setCacheCutSceneRootPath(String path)
```

if you want to export a video, you need to know `ExportVideoParamsBuilder` first. 

> Note: Exporting videos demands high performance from your mobile device. If you encounter OOM or if the app is forcibly closed by the system during export, please try setting a smaller width and height.

```Java
ExportVideoParamsBuilder builder = new ExportVideoParamsBuilder()

    // (Must) Set the export file path
    .setTargetPath(String path)

    // (Optional) Set the width of the exported video. It must be a power of 2, the default is get from WorkWapper.
    .setWidth(int width)

    // (Optional) Set the height of the exported video. It must be a power of 2, the default is get from WorkWapper.
    .setHeight(int height)

    // (Optional) Set the bitrate of the exported video, the default is get from WorkWapper.
    .setBitrate(int bitrate)
    
    // (Optional) Set the fps of the exported video, the default is get from WorkWapper.
    .setFps(int fps)

    // (Optional) Set export mode, default is `PANORAMA`
    // ExportMode.PANORAMA: Use when exporting panorama media
    // ExportMode.SPHERE: Used when exporting flat thumbnails
    .setExportMode(ExportUtils.ExportMode mode)

    // (Optional) Whether to enable dynamic stitching, the default is true.
    .setDynamicStitch(boolean dynamicStitch)
    
    // (Optional) Whether to enable stabilization, the default is true
    .setStabEnabled(true)
    
    // (Optional) Cache Folder，the default is getCacheDir() + "/work_thumbnail"
    .setCacheWorkThumbnailRootPath(String path)

    // (Optional) Cache Folder，the default is getCacheDir() + "/cut_scene"
    .setCacheCutSceneRootPath(String path)
```

Next let's see how to export

Export Panorama Image (Image to Image)

```Java
ExportImageParamsBuilder builder = new ExportImageParamsBuilder()
        .setExportMode(ExportUtils.ExportMode.PANORAMA)
        .setTargetPath(path)
        .setWidth(2048)
        .setHeight(1024);
int exportId = ExportUtils.exportImage(WorkWrapper, builder, new IExportCallback() {
            @Override
            public void onSuccess() {               
            }

            @Override
            public void onFail() {
            }

            @Override
            public void onCancel() {
            }

            @Override
            public void onProgress(float progress) {
                // callback only when exporting video
            }
        });
```

Export Image Thumbnail (Image to Image)

```Java
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
            }

            @Override
            public void onFail() {
            }

            @Override
            public void onCancel() {
            }

            @Override
            public void onProgress(float progress) {
                // callback only when exporting video
            }
        });
```

Export Panorama Video (Video to Video)

> Note: Exporting videos demands high performance from your mobile device. If you encounter OOM or if the app is forcibly closed by the system when exporting 5.7k videos, please try setting a smaller width and height or increase app priority.

```Java
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
            }

            @Override
            public void onFail() {
            }

            @Override
            public void onCancel() {
            }

            @Override
            public void onProgress(float progress) {
                // callback only when exporting video
            }
        });
```

Export Video Thumbnail (Video to Image)

```Java
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
            }

            @Override
            public void onFail() {
            }

            @Override
            public void onCancel() {
            }

            @Override
            public void onProgress(float progress) {
                // callback only when exporting video
            }
        });
```

You can stop export by the `exportId` returned by `ExportUtils.exportXXX()`

```Java
ExportUtils.stopExport(int exportId);
```



## <a name="MediaSDK生成HDR图像" />Generate HDR Image

If you have a `WorkWrapper` of HDR Image, you can generate it to one image file by `HDR Stiching`.

> Note: This is a time-consuming operation and needs to be processed in a child thread.

```Java
boolean isSuccessful = StitchUtils.generateHDR(WorkWrapper workWrapper, String hdrOutputPath);
```

After the generateHDR call is successful, the outputPath can be played as a proxy or set as export url.

```Java
ImageParamsBuilder builder = new ImageParamsBuilder()
        // If HDR stitching is successful then set it as the playback proxy
        .setUrlForPlay(isSuccessful ? hdrOutputPath : null);
mImagePlayerView.prepare(workWrapper, builder);
mImagePlayerView.play();
```

```Java
ExportImageParamsBuilder builder = new ExportImageParamsBuilder()
        .setExportMode(ExportUtils.ExportMode.PANORAMA)
        .setTargetPath(exportPath)
        .setUrlForExport(hdrOutputPath);
```



## <a name="MediaSDK生成PureShot图像" />Generate PureShot Image

If you have a `WorkWrapper` of PureShot Image, you can generate it to one image file by `PureShot Stiching`.

> Note: This is a time-consuming operation and needs to be processed in a child thread.

```Java
boolean isSuccessful = StitchUtils.generatePureShot(WorkWrapper workWrapper, String pureshotOutputPath, String algoFolderPath);
```

> Note: `algoFolderPath` can be either `LocalStoragePath` or `AssetsRelativePath`. Please ask our technical support staff for the algorithm model file corresponding to the sdk version.

After the generatePureShot call is successful, the outputPath can be played as a proxy or set as export url.

```Java
ImageParamsBuilder builder = new ImageParamsBuilder()
        // If PureShot stitching is successful then set it as the playback proxy
        .setUrlForPlay(isSuccessful ? pureshotOutputPath : null);
mImagePlayerView.prepare(workWrapper, builder);
mImagePlayerView.play();
```

```Java
ExportImageParamsBuilder builder = new ExportImageParamsBuilder()
        .setExportMode(ExportUtils.ExportMode.PANORAMA)
        .setTargetPath(exportPath)
        .setUrlForExport(pureshotOutputPath);
```



# <a name="CameraSDKOSC" />OSC

OSC Official Document: [Open Spherical Camera API](https://developers.google.cn/streetview/open-spherical-camera/guides/osc)

Get the camera network address by `InstaCameraManager.getInstance().getCameraHttpPrefix()`. Execute commands via Open Sepherial Camera API[`/osc/commands/execute`](https://developers.google.cn/streetview/open-spherical-camera/guides/osc/commands/execute)

You need to poll yourself to call [`/osc/commands/status`](https://developers.google.cn/streetview/open-spherical-camera/guides/osc/commands/status) to get the execution status of the camera on the current command. The polling cycle can be adjusted according to specific conditions.

Camera support [Open Spherical Camera API level 2](https://developers.google.com/streetview/open-spherical-camera/reference), except preview stream.

#### Take picture & Record

* You can use [`camera.takePicture`](https://developers.google.com/streetview/open-spherical-camera/reference/camera/takepicture) to take picture.
* You can use [`camera.startCapture`](https://developers.google.com/streetview/open-spherical-camera/reference/camera/startcapture) to start record.
* You can use [`camera.stopCapture `](https://developers.google.com/streetview/open-spherical-camera/reference/camera/stopcapture) to stop record.

#### Options

* You can use [`camera.setOptions`](https://developers.google.com/streetview/open-spherical-camera/reference/camera/setoptions) to set options.
* You can use [`camera.getOptions`](https://developers.google.com/streetview/open-spherical-camera/reference/camera/getoptions) to get options.
* You can know the relevant parameters supported by the camera from [Open Spherical Camera API options](https://developers.google.com/streetview/open-spherical-camera/reference/options).

#### List files

You can use [`camera.listFiles`](https://developers.google.com/streetview/open-spherical-camera/reference/camera/listfiles) to get the files list.

#### Open Spherical Camera API

Familiarity with [`Open Spherical Camera API`](https://developers.google.cn/streetview/open-spherical-camera/guides/osc) official documentation is a prerequisite for OSC development.

* You can use [`/osc/info`](https://developers.google.cn/streetview/open-spherical-camera/guides/osc/info) to get the basic information about the camera and functionality it supports.
* You can use [`/osc/state`](https://developers.google.cn/streetview/open-spherical-camera/guides/osc/state) to get the attributes of the camera.


# <a name="Proguard" />Proguard

```
-keep class java.**{*;}
-keep class com.arashivision.**{*;}
```
