<a href="https://github.com/Insta360Develop/CameraSDK-Android/releases">
    <img src="https://img.shields.io/badge/version-1.1.8-green">
</a> 
<a href="https://developer.android.com/studio/publish/versioning#minsdkversion">
    <img src="https://img.shields.io/badge/minSdkVersion-21-green">
</a> 
<a href="https://developer.android.com/studio/publish/versioning#minsdkversion">
    <img src="https://img.shields.io/badge/targetSdkVersion-28-green">
</a> 



# Table of Contents
- [Camera SDK Function](#CameraSDK功能)
  - [Initialization](#CameraSDK初始化)
  - [Connect / Disconnect / Obersve Camera](#CameraSDK连接/断开/监听相机)
  - [Preview & Live](#CameraSDK预览)
  - [Capture](#CameraSDK拍摄)
  - [Settings](#CameraSDK属性设置)
  - [Other Function](#CameraSDK其他功能)
  - [Get other camera information](#CameraSDK获取相机其他信息)
- [Media SDK Function](#MediaSDK功能)
  - [Initialization](#MediaSDK初始化)
  - [Preview & Live](#MediaSDK预览)
  - [Player](#MediaSDK播放)
  - [Export](#MediaSDK导出)
  - [Generate HDR Image](#MediaSDK生成HDR图像)
- [OSC](#CameraSDKOSC)
- [Proguard](#Proguard)
  
# <a name="CameraSDK功能" />Camera SDK Function

## <a name="CameraSDK初始化" />Initialization

First add the maven address to your build file (`build.gradle` file in the project root directory)

```
allprojects {
    repositories {
        ...
        maven {
            url 'http://nexus.arashivision.com:9999/repository/maven-public/'
            credentials {
                username = 'deployment'
                password = 'test123'
            }
        }
    }
}
```

Second import the dependent library in your `build.gradle` file of app directory

```
dependencies {
    implementation 'com.arashivision.sdk:sdkcamera:1.1.8'
}
```

Then initialize SDK in Application

```
public class MyApp extends Application {

    @Override
    public void onCreate() {
        super.onCreate();      
        // Init SDK
        InstaCameraSDK.init(this);
    }

}
```



## <a name="CameraSDK连接/断开/监听相机" />Connect / Disconnect / Obersve Camera

### Connect

You can connect the camera by WIFI or USB.

> Note: You must do this on the main thread.

By WIFI

```
InstaCameraManager.getInstance().openCamera(InstaCameraManager.CONNECT_TYPE_WIFI);
```

By USB

```
InstaCameraManager.getInstance().openCamera(InstaCameraManager.CONNECT_TYPE_USB);
```

You can also get the current camera connection type

```
int type = InstaCameraManager.getInstance().getCameraConnectedType();
```

And the result is one of `CONNECT_TYPE_NONE`, `CONNECT_TYPE_USB`, `CONNECT_TYPE_WIFI`.

For example if you want to determine if the camera is connected, you can do like this

```
private boolean isCameraConnected() {
    return InstaCameraManager.getInstance().getCameraConnectedType() != InstaCameraManager.CONNECT_TYPE_NONE;
}
```

### Close

When you want to disconnect the camera, you can call

```
InstaCameraManager.getInstance().closeCamera();
```

### Obersve

You can `register / unregister` `ICameraChangedCallback` on multiple pages to observe camera status changed

```
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
    public void onCameraConnectError() {
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

```
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

}
```

> If the camera is passively disconnected or if you call `closeCamera` directly during the preview process, the SDK will automatically close the preview stream processing related state without the need to call `closePreviewStream`.

If you want to display the preview content, please see [Media SDK Function - Preview & Live](#MediaSDK预览)

### Preview Stream Resolution

You can choose one of the resolutions supported by the camera to set arguments. Get the supported resolutions by

```
List<PreviewStreamResolution> supportedList = InstaCameraManager.getInstance().getSupportedPreviewStreamResolution(int previewType);
```

### Preview Type

There are 3 kinds of `PreviewType` in `InstaCameraManager`. You must restart preview stream (close first and then start again) to switch between different operations.

* PREVIEW_TYPE_NORMAL - for normal preview or capture
* PREVIEW_TYPE_RECORD - for record
* PREVIEW_TYPE_LIVE - for live

### Live

You need to get supported resolution of the camrea for live by

```
List<PreviewStreamResolution> supportedList = InstaCameraManager.getInstance().getSupportedPreviewStreamResolution(InstaCameraManager.PREVIEW_TYPE_LIVE);
```

Then choose one resolution to start live preview

```
InstaCameraManager.getInstance().startPreviewStream(PreviewStreamResolution, InstaCameraManager.PREVIEW_TYPE_LIVE);
```

> Note: The preview stream is different between `Preview`, `Record` and `Live`, so you must restart preview stream if you want to switch between them.

If you want to display the live preview content, please see [Media SDK Function - Preview & Live](#MediaSDK预览)

When the preview is ready, you can start live like this

```
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

```
InstaCameraManager.getInstance().stopLive();
```



## <a name="CameraSDK拍摄" />Capture

After the camera is successfully connected, you can control its capture. We provide you with several capture interfaces.

### Normal Capture

> set `true` if you want to capture the original image in RAW format

```
InstaCameraManager.getInstance().startNormalCapture(false);
```

### HDR Capture

> set `true` if you want to capture the original image in RAW format

```
InstaCameraManager.getInstance().startHDRCapture(false);
```

### Interval Capture

You can set interval time first (in ms)

```
InstaCameraManager.getInstance().setIntervalTime(int intervalMs);
```

Then capture
```
// Start
InstaCameraManager.getInstance().startIntervalShooting();

// Stop
InstaCameraManager.getInstance().stopIntervalShooting();
```

### Normal Record

```
// Start
InstaCameraManager.getInstance().startNormalRecord();

// Stop
InstaCameraManager.getInstance().stopNormalRecord();
```

### HDR Record

```
// Start
InstaCameraManager.getInstance().startHDRRecord();

// Stop
InstaCameraManager.getInstance().stopHDRRecord();
```

### TimeLapse

You can set interval time first (in ms)

```
InstaCameraManager.getInstance().setTimeLapseInterval(int intervalMs);
```

Then record
```
// Start
InstaCameraManager.getInstance().startTimeLapse();

// Stop
InstaCameraManager.getInstance().stopTimeLapse();
```

You can also get the current camera capture type

```
int type = InstaCameraManager.getInstance().getCurrentCaptureType();
```

And the result is one of `CAPTURE_TYPE_NORMAL_CAPTURE`, `CAPTURE_TYPE_HDR_CAPTURE`, `CAPTURE_TYPE_INTERVAL_SHOOTING`, `CAPTURE_TYPE_NIGHT_SCENE_CAPTURE`,
`CAPTURE_TYPE_BURST_CAPTURE`, `CAPTURE_TYPE_NORMAL_RECORD`, `CAPTURE_TYPE_HDR_RECORD`, `CAPTURE_TYPE_TIMELAPSE`, `CAPTURE_TYPE_STATIC_TIMELAPSE`, `CAPTURE_TYPE_BULLET_TIME_RECORD`, `CAPTURE_TYPE_TIME_SHIFT_RECORD`, `CAPTURE_TYPE_INTERVAL_RECORD`, `CAPTURE_TYPE_UNKNOWN`, `CAPTURE_TYPE_IDLE`

### Observe Status Changed

You can set `ICaptureStatusListener` to observe capture status changed.

```
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

### EV Value

You can `set / get` the EV value of a certain capture mode. The value range is -4 ~ 4

> `funcMode` is one of `FUNCTION_MODE_CAPTURE_NORMAL`, `FUNCTION_MODE_HDR_CAPTURE`, `FUNCTION_MODE_INTERVAL_SHOOTING`, 
`FUNCTION_MODE_RECORD_NORMAL`, `FUNCTION_MODE_HDR_RECORD`, `FUNCTION_MODE_BULLETTIME`, `FUNCTION_MODE_TIMELAPSE`

Set value

```
InstaCameraManager.getInstance().setExposureEV(int funcMode, float ev);
```

Get value 

```
float ev = InstaCameraManager.getInstance().getExposureEV(int funcMode);
```


### Camera Beep

You can `set / get` whether there is a beep sound when the camera shoots.

Set Beep Enabled

```
InstaCameraManager.getInstance().setCameraBeepSwitch(boolean beep);
```

Determine Whether beep is enabled

```
boolean isBeep = InstaCameraManager.getInstance().isCameraBeep();
```


## <a name="CameraSDK其他功能" />Other Function

### Calibrate Gyro

> This function must be used when the camera is connected to WIFI
>
> Before calibrate, please stand the camera upright on a stable and level surface.

```
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

### Format SD card

```
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



## <a name="CameraSDK获取相机其他信息" />Get other camera information

The following methods are only used as parameters for other interfaces to call. Developers don't need to pay much attention to them, and they will be explained later at the places where they need to be called.

Camera Type

```
InstaCameraManager.getInstance().getCameraType();
```

Camera Version

```
InstaCameraManager.getInstance().getCameraVersion();
```

Camera Serial

```
InstaCameraManager.getInstance().getCameraSerial();
```

Camera Media Offset

```
InstaCameraManager.getInstance().getMediaOffset();
```

Camera Selfie

```
InstaCameraManager.getInstance().isCameraSelfie();
```

Camera Battery Level

```
InstaCameraManager.getInstance().getCameraCurrentBatteryLevel();
```

Camera Charge State

```
InstaCameraManager.getInstance().isCameraCharging();
```

Camera Storage Total Space

```
InstaCameraManager.getInstance().getCameraStorageTotalSpace();
```

Camera Storage Free Space

```
InstaCameraManager.getInstance().getCameraStorageFreeSpace();
```

Camera SD card state

```
InstaCameraManager.getInstance().isSdCardEnabled();
```

Camera Host

```
InstaCameraManager.getInstance().getCameraHttpPrefix();
```

Camera File List

```
InstaCameraManager.getInstance().getAllUrlList();
InstaCameraManager.getInstance().getRawUrlList();
InstaCameraManager.getInstance().getCameraInfoMap();
```


------



# <a name="MediaSDK功能" />Media SDK Function

## <a name="MediaSDK初始化" />Initialization


First add the maven address to your build file (`build.gradle` file in the project root directory)

```
allprojects {
    repositories {
        ...
        maven {
            url 'http://nexus.arashivision.com:9999/repository/maven-public/'
            credentials {
                username = 'deployment'
                password = 'test123'
            }
        }
    }
}
```

Second import the dependent library in your `build.gradle` file of app directory

```
dependencies {
    implementation 'com.arashivision.sdk:sdkmedia:1.1.8'
}
```

Then initialize SDK in Application

```
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

```
<com.arashivision.sdkmedia.player.capture.InstaCapturePlayerView
    android:id="@+id/player_capture"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@android:color/darker_gray" />
```

Bind lifecycle when your activity created

```
@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_preview);
    mCapturePlayerView.setLifecycle(getLifecycle());
}
```

Display preview in `IPreviewStatusListener.onOpened ()` callback of `CameraSdk`

```
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
            .setCameraSelfie(InstaCameraManager.getInstance().isCameraSelfie());
    return builder;
}
```

Release `InstaCapturePlayerView` when preview is closed

```
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

```
private CaptureParamsBuilder createParams() {
    CaptureParamsBuilder builder = new CaptureParamsBuilder()
    
            // (Must) The parameters are fixed
            .setCameraType(InstaCameraManager.getInstance().getCameraType())
            
            // (Must) The parameters are fixed
            .setMediaOffset(InstaCameraManager.getInstance().getMediaOffset())

            // (Must) The parameters are fixed
            .setCameraSelfie(InstaCameraManager.getInstance().isCameraSelfie())
            
            // (Depends on) If you start preview for live, this is required.
            .setLive(true)
            
            // (Depends on) If you use custom resloution to start preview, this is required.
            .setResolutionParams(mCurrentResolution.width, mCurrentResolution.height, mCurrentResolution.fps);
            
            // (Optional) Whether to enable stabilization, the default is true
            .setStabEnabled(true)
            
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

```
mCapturePlayerView.switchNormalMode();
```

Switch to Fisheye Mode

```
mCapturePlayerView.switchFisheyeMode();
```

Switch to Perspective Mode

```
mCapturePlayerView.switchPerspectiveMode();
```

or You can custom customize the player angle by yourself

> Fov: Viewing width. Range 0(inclusive) ~ 180(exclusive)
>
> Distance: Distance from observation point to observed point. Range 0(inclusive) ~ ∞

```
mCapturePlayerView.setConstraint(float minFov, float maxFov, float defaultFov, float minDistance, float maxDistance, float defaultDistance);
```

if you want to switch between `Plane Mode` and others, you must restart preview first.

```
if (current mode is Normal && new mode is Plane){
    InstaCameraManager.getInstance().closePreviewStream();
    InstaCameraManager.getInstance().startPreviewStream(resolution, previewType);
}
```

You can set `PlayerGestureListener` to observe gesture operation.

```
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

```
List<WorkWrapper> list = WorkUtils.getAllCameraWorks(
    InstaCameraManager.getInstance().getCameraHttpPrefix(),
    InstaCameraManager.getInstance().getCameraInfoMap(),
    InstaCameraManager.getInstance().getAllUrlList(),
    InstaCameraManager.getInstance().getRawUrlList());
```

Scan from local directory

> Note: This is a time-consuming operation and needs to be processed in a child thread.

```
List<WorkWrapper> list = WorkUtils.getAllLocalWorks(String dirPath);
```

or if you have urls of the media file, you can also create `WorkWrapper` by yourself.

```
String[] urls = {img1.insv, img2.insv, img3.insv};
WorkWrapper workWrapper = new WorkWrapper(urls);
```

You can get the media info based on the `workWrapper`

```
int width = workWrapper.getWidth();
int height = workWrapper.getHeight();
int bitrate = workWrapper.getBitrate();
double fps = workWrapper.getFps();
```

You can determine whether it is a video or an image based on the `workWrapper`

```
boolean isPhoto = workWrapper.isPhoto();
boolean isVideo = workWrapper.isVideo();
```

```
boolean isHDRPhoto = workWrapper.isHDRPhoto();
boolean isHDRVideo = workWrapper.isHDRVideo();
```

When you need a `Uniquely Identify` of `WorkWrapper`, such as DiskCacheKey of Glide. You can get it by

```
String id = workWrapper.getIdenticalKey();
```

### Image Player

You can play an image by `InstaImagePlayerView`.

Put `InstaImagePlayerView` in your xml file

```
<com.arashivision.sdkmedia.player.image.InstaImagePlayerView
    android:id="@+id/player_image"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

Bind lifecycle when your activity created

```
@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_preview);
    mImagePlayerView.setLifecycle(getLifecycle());
}
```

Build parameters and play

```
mImagePlayerView.prepare(workWrapper, new ImageParamsBuilder());
mImagePlayerView.play();
```

Release `InstaImagePlayerView` when activity is destory

```
@Override
protected void onDestroy() {
    super.onDestroy();
    mImagePlayerView.destroy();
}
```

You can configure more options by `ImageParamsBuilder`

```
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

```
mImagePlayerView.switchNormalMode();
```

Switch to Fisheye Mode

```
mImagePlayerView.switchFisheyeMode();
```

Switch to Perspective Mode

```
mImagePlayerView.switchPerspectiveMode();
```

or You can custom customize the player angle by yourself

> Fov: Viewing width. Range 0(inclusive) ~ 180(exclusive)
>
> Distance: Distance from observation point to observed point. Range 0(inclusive) ~ ∞

```
mImagePlayerView.setConstraint(float minFov, float maxFov, float defaultFov, float minDistance, float maxDistance, float defaultDistance);
```

You can set `PlayerViewListener` to observe player status changed.

```
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

```
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

```
<com.arashivision.sdkmedia.player.video.InstaVideoPlayerView
    android:id="@+id/player_video"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

Bind lifecycle when your activity created

```
@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_preview);
    mVideoPlayerView.setLifecycle(getLifecycle());
}
```

Build parameters and play

```
mVideoPlayerView.prepare(workWrapper, new VideoParamsBuilder());
mVideoPlayerView.play();
```

Release `InstaVideoPlayerView` when activity is destory

```
@Override
protected void onDestroy() {
    super.onDestroy();
    mVideoPlayerView.destroy();
}
```

You can configure more options by `VideoParamsBuilder`

```
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

```
mVideoPlayerView.switchNormalMode();
```

Switch to Fisheye Mode

```
mVideoPlayerView.switchFisheyeMode();
```

Switch to Perspective Mode

```
mVideoPlayerView.switchPerspectiveMode();
```

or You can custom customize the player angle by yourself

> Fov: Viewing width. Range 0(inclusive) ~ 180(exclusive)
>
> Distance: Distance from observation point to observed point. Range 0(inclusive) ~ ∞

```
mVideoPlayerView.setConstraint(float minFov, float maxFov, float defaultFov, float minDistance, float maxDistance, float defaultDistance);
```

You can set `PlayerViewListener` to observe player status changed.

```
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

```
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

```
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

```
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

```
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

> Note: Exporting videos has high requirements on mobile phone performance. If you encounter oom or app being forcibly killed by the system during export, please set a smaller width and height.

```
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

```
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

```
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

> Note: Exporting video has high requirements on mobile phone performance. If you encounter oom or app being forcibly killed by the system when exporting 5.7k, please set a smaller width and height or increase app priority.

```
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

```
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

```
ExportUtils.stopExport(int exportId);
```



## <a name="MediaSDK生成HDR图像" />Generate HDR Image

If you have a `WorkWrapper` of HDR Image, you can generate it to one image file by `HDR Stiching`.

> Note: This is a time-consuming operation and needs to be processed in a child thread.

> Note: Only local file could be generate, please download the files from camera first.

```
boolean isSuccessful = StitchUtils.generateHDR(WorkWrapper workWrapper, String hdrOutputPath);
```

After the generate is successful, the outputPath can be played as a proxy or set as export url.

```
ImageParamsBuilder builder = new ImageParamsBuilder()
        // If HDR stitching is successful then set it as the playback proxy
        .setUrlForPlay(isSuccessful ? hdrOutputPath : null);
mImagePlayerView.prepare(workWrapper, builder);
mImagePlayerView.play();
```

```
ExportImageParamsBuilder builder = new ExportImageParamsBuilder()
        .setExportMode(ExportUtils.ExportMode.PANORAMA)
        .setTargetPath(exportPath)
        .setUrlForExport(hdrOutputPath);
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
