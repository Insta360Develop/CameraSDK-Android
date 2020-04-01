中文版本请参看[这里](./readme_cn.md)

------



# Table of Contents
- [Camera SDK Function](#CameraSDK功能)
  - [Initialization](#CameraSDK初始化)
  - [Connect / Disconnect / Obersve Camera](#CameraSDK连接/断开/监听相机)
  - [Preview](#CameraSDK预览)
  - [Capture](#CameraSDK拍摄)
  - [Settings](#CameraSDK属性设置)
  - [Other Function](#CameraSDK其他功能)
  - [Get other camera information](#CameraSDK获取相机其他信息)
  - [OSC](#CameraSDKOSC)
- [Media SDK Function](#MediaSDK功能)
  - [Initialization](#MediaSDK初始化)
  - [Preview](#MediaSDK预览)
  - [Player](#MediaSDK播放)
  - [Export](#MediaSDK导出)
  - [Generate HDR Image](#MediaSDK生成HDR图像)
  
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
    implementation 'com.arashivision.sdk:sdkcamera:1.0.0'
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



## <a name="CameraSDK预览" />Preview

After the camera is successfully connected, you can manipulate the camera preview stream like this

```
public class PreviewActivity extends BaseObserveCameraActivity implements IPreviewStatusListener {

    @Override
    protected void onResume() {
        super.onResume();
        // Auto open preview after page gets focus
        InstaCameraManager.getInstance().setPreviewStatusChangedListener(this);
        InstaCameraManager.getInstance().startPreviewStream();
    }

    @Override
    protected void onStop() {
        super.onStop();
        // Auto close preview after page loses focus
        InstaCameraManager.getInstance().setPreviewStatusChangedListener(null);
        InstaCameraManager.getInstance().closePreviewStream();
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

If you want to display the preview content, please see [Media SDK Function - Preview](#MediaSDK预览)



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

You can Set interval time first (in ms)

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

And the result is one of `CAPTURE_TYPE_NORMAL_CAPTURE`, `CAPTURE_TYPE_HDR_CAPTURE`, `CAPTURE_TYPE_INTERVAL_SHOOTING`,
`CAPTURE_TYPE_NORMAL_RECORD`, `CAPTURE_TYPE_HDR_RECORD`, `CAPTURE_TYPE_TIMELAPSE`, `CAPTURE_TYPE_UNKNOWN`, `CAPTURE_TYPE_IDLE`

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
            public void onCaptureFinish() {
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



## <a name="CameraSDKOSC" />OSC

> OSC Official Document: [Open Spherical Camera API](https://developers.google.cn/streetview/open-spherical-camera/guides/osc)

IOscRequestDelegate

> Use your favorite network library to implement `Get / Post` synchronization request and return the result to `OscManager` for use

```
/**
 * Send a Http network request by Get
 *
 * @param url       Request address
 * @param headerMap HTTP request headers to use
 * @return Network Request Response Body or Error Message
 */
OSCResult sendRequestByGet(String url, Map<String, String> headerMap);

/**
 * Send a Http network request by Post
 *
 * @param url       Request address
 * @param content   osc command content
 * @param headerMap HTTP request headers to use
 * @return Network Request Response Body or Error Message
 */
OSCResult sendRequestByPost(String url, String content, Map<String, String> headerMap);

/**
 * Request result
 *
 * @param isSuccessful 
 * @param result       Success is response.body().string()，Failure is response.message()      
OSCResult(boolean isSuccessful, String result)
```

OscManager

```
// Set Options
// OscOptions for parameter details
// If successful, callback returns null
setOptions(OscOptions options, IOscCallback callback);

// Take Picture
// OscOptions: Set the Options first, and then take a photo automatically. If the current CaptureMode is image, select it, otherwise it is required
// If successful, callback returns file address (String[] urls), could be downloaded to local
takePicture(OscOptions options, IOscCallback callback);

// Start Record
// OscOptions: Set the Options first, and then take a photo automatically. If the current CaptureMode is video, select it, otherwise it is required
// If successful, callback returns null
startRecord(OscOptions options, IOscCallback callback);

// Stop Record
// If successful, callback returns file address (String[] urls), could be downloaded to local
startRecord(IOscCallback callback);

// Optional OSC Request
// oscApi：such as /osc/info, /osc/state
// content：Content of RequestBody. If it is null, use GET request, otherwise use POST request
customRequest(String oscApi, String content, IOscCallback callback);
```

OscOptions

> ONE X Camera Supported Options

```
OscOptions.Builder builder = new OscOptions.Builder();
OscOptions oscOptions = builder.build();

// set Capture Mode: OscOptions.CAPTURE_MODE_IMAGE/CAPTURE_MODE_VIDEO
setCaptureMode(@CaptureMode String captureMode)

// set Exposure Mode
setExposureProgram(@ExposureProgram int exposureProgram);

// Set ISO Value
setISO(@ISO int iso);

// set Shutter Speed
setShutterSpeed(@ShutterSpeed int speed);

// set White Balance
setWhiteBalance(@WhiteBalance String whiteBalance);

// Set whether to enable HDR Capture
setHDREnabled(boolean enabled);

// Set HDR Capture Parameters
setExposureBracketParams(int shots, int increment);
```

IOscCallback

```
// Start sending OSC requests
onStartRequest();

// Request Successful
onSuccessful(Object object);

// Request failed
onError(String message);
```


## <a name="CameraSDK获取相机其他信息" />Get other camera information

The following methods are only used as parameters for other interfaces to call. Developers don't need to pay much attention to them, and they will be explained later at the places where they need to be called.

Camera Type

```
InstaCameraManager.getInstance().getCameraType();
```

Camera Media Offset

```
InstaCameraManager.getInstance().getMediaOffset();
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
    implementation 'com.arashivision.sdk:sdkmedia:1.0.0'
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



## <a name="MediaSDK预览" />Preview

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
            .setMediaOffset(InstaCameraManager.getInstance().getMediaOffset());
    return builder;
}
```

Release `InstaCapturePlayerView` when preview is closed

```
@Override
protected void onStop() {
    super.onStop();
    // Auto close preview after activity loses focus
    InstaCameraManager.getInstance().setPreviewStatusChangedListener(null);
    InstaCameraManager.getInstance().closePreviewStream();
    mCapturePlayerView.destroy();
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
            // Must. The parameters are fixed
            .setCameraType(InstaCameraManager.getInstance().getCameraType())
            // Must. The parameters are fixed
            .setMediaOffset(InstaCameraManager.getInstance().getMediaOffset())
            // Optional. Whether to allow gesture operations, the default is true
            .setGestureEnabled(true);
    if (You want to preview as plane mode) {
        // Plane Mode
        // Optional. Set Render Model Type, the default is `RENDER_MODE_AUTO`
        builder.setRenderModelType(CaptureParamsBuilder.RENDER_MODE_PLANE_STITCH)
                // Optional. Set the aspect ratio of the screen, the default is full screen display (ie full canvas)
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

if you want to switch between `Plane Mode` and others, you must restart preview first.

```
if (current mode is Normal && new mode is Plane){
    InstaCameraManager.getInstance().closePreviewStream();
    InstaCameraManager.getInstance().startPreviewStream();
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



## <a name="MediaSDKWorkWrapper" />WorkWrapper

Data model to be transmitted by interfaces such as play and export

```
// Build from media file path
WorkWrapper workWrapper = new WorkWrapper(String url);
WorkWrapper workWrapper = new WorkWrapper(String[] urls);

// Get media file path
String[] getUrls();

// Determine the media file type
boolean isVideo();
boolean isPhoto();

// Get unique identification
String getIdenticalKey()
```




## <a name="MediaSDK播放" />Player

If you want to play an image or a video, you must create a `WorkWrapper` object first.

You can scan media files from camera or a local directory to get `List<WorkWrapper>`.

Scan from camera 

```
List<WorkWrapper> list = WorkUtils.getAllCameraWorks(
    InstaCameraManager.getInstance().getCameraHttpPrefix(),
    InstaCameraManager.getInstance().getCameraInfoMap(),
    InstaCameraManager.getInstance().getAllUrlList(),
    InstaCameraManager.getInstance().getRawUrlList());
```

Scan from local directory

```
List<WorkWrapper> list = WorkUtils.getAllLocalWorks(String dirPath);
```

or if you have urls of the media file, you can create `WorkWrapper` by yourself.

```
String[] urls = {img1.insv, img2.insv, img3.insv};
WorkWrapper workWrapper = new WorkWrapper(urls);
```

You can determine whether it is a video or a picture based on the `workWrapper`

```
boolean isPhoto = workWrapper.isPhoto();
boolean isVideo = workWrapper.isVideo();
```

When you need a `Uniquely Identify` of `WorkWrapper`, such as DiskCacheKey of Glide. You can get it by

```
String id = workWrapper.getIdenticalKey();
```

### Image Player

InstaImagePlayerView

```
// Bind Lifecycle
setLifecycle(Lifecycle);

// Set player status listener
setPlayerViewListener(PlayerViewListener);

// Set gesture listener
setGestureListener(PlayerGestureListener);

// Configuration parameters before playback
prepare(WorkWrapper, ImageParamsBuilder);

// Play
play();

// Release
destroy();
```

```
// Whether the player is loading
boolean isLoading();

// Get the data of the current playback angle for configuration parameters when exporting thumbnails
float getFov();
float getDistance();
float getYaw();
float getPitch();
```

PlayerViewListener

> See`Media SDK Function - Preview`

PlayerGestureListener

> See`Media SDK Function - Preview`

ImageParamsBuilder

```
// (Optional) Whether to enable dynamic stitching, the default is true.
setDynamicStitch(boolean dynamicStitch);

// (Optional) Set playback proxy file, such as HDR.jpg generated by stitching, the default is null
setUrlForPlay(String url);

// (Optional) Set the aspect ratio of the screen, the default is full screen display (ie full canvas)
setScreenRatio(int ratioX, int ratioY);

// (Optional) Whether to allow gesture operations, the default is true
setGestureEnabled(boolean enabled);

// (Optional) Cache Folder，the default is getCacheDir() + "/work_thumbnail"
setCacheWorkThumbnailRootPath(String path);

// (Optional) Cache Folder，the default is getCacheDir() + "/stabilizer"
setStabilizerCacheRootPath(String path);

// (Optional) Cache Folder，the default is getCacheDir() + "/cut_scene"
setCacheCutSceneRootPath(String path);
```

### Video Player

InstaVideoPlayerView

```
// Bind Lifecycle
setLifecycle(Lifecycle);

// Set player status listener
setPlayerViewListener(PlayerViewListener);

// Set gesture listener
setGestureListener(PlayerGestureListener);

// Set video status listener
setVideoStatusListener(VideoStatusListener);

// Configuration parameters before playback
prepare(WorkWrapper, ImageParamsBuilder);

// Play
play();

// Release
destroy();
```

```
// Whether the player is loading
boolean isLoading();

// Get the data of the current playback angle for configuration parameters when exporting thumbnails
float getFov();
float getDistance();
float getYaw();
float getPitch();

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

PlayerViewListener

> See`Media SDK Function - Preview`

PlayerGestureListener

> See`Media SDK Function - Preview`

VideoStatusListener

```
// Player progress changed
// position: the current playback position
// length: video total duration
onProgressChanged(long position, long length);

// Player state changed
onPlayStateChanged(boolean isPlaying);

// Seek Complete
onSeekComplete();

// Play Complete
onCompletion();
```

VideoParamsBuilder

```
// (Optional) Loading icon, default is none
setLoadingImageResId(int resId);

// (Optional) Background color when loading, default is black
setLoadingBackgroundColor(int color);

// (Optional) Whether to play automatically after preparation, the default is true
setAutoPlayAfterPrepared(boolean autoPlayAfterPrepared);

// (Optional) Whether to loop playback, the default is true
setIsLooping(boolean isLooping);

// (Optional) Set the aspect ratio of the screen, the default is full screen display (ie full canvas)
setScreenRatio(int ratioX, int ratioY);

// (Optional) Whether to allow gesture operations, the default is true
setGestureEnabled(boolean enabled);

// (Optional) Cache Folder，the default is getCacheDir() + "/work_thumbnail"
setCacheWorkThumbnailRootPath(String path);

// (Optional) Cache Folder，the default is getCacheDir() + "/cut_scene"
setCacheCutSceneRootPath(String path);
```



## <a name="MediaSDK导出" />Export

ExportImageParamsBuilder

```
// (Must) Set Camera Type. Get from `InstaCameraManager.getInstance().getCameraType()`
setCameraType(String cameraType);

// (Must) Set the export file path
setTargetPath(String path);

// (Must) Set the width of the exported image. It must be a power of 2.
setWidth(int width);

// (Must) Set the height of the exported image. It must be a power of 2.
setHeight(int height);

// (Optional) Set export mode, default is `PANORAMA`
// ExportMode.PANORAMA: Use when exporting panorama media
// ExportMode.SPHERE: Used when exporting flat thumbnails
setExportMode(ExportUtils.ExportMode mode);

// (Optional) Whether to enable dynamic stitching, the default is true.
setDynamicStitch(boolean dynamicStitch);

// (Optional) Set the camera angle. It is recommended to use when exporting thumbnails. The currently displayed angle parameters can be obtained from `PlayerView`.
setFov(float fov);
setDistance(float distance);
setYaw(float yaw);
setPitch(float pitch);

// (Optional) Cache Folder，the default is getCacheDir() + "/work_thumbnail"
setCacheWorkThumbnailRootPath(String path);

// (Optional) Cache Folder，the default is getCacheDir() + "/stabilizer"
setStabilizerCacheRootPath(String path);

// (Optional) Cache Folder，the default is getCacheDir() + "/cut_scene"
setCacheCutSceneRootPath(String path);
```

ExportVideoParamsBuilder

```
// (Must) Set Camera Type. Get from `InstaCameraManager.getInstance().getCameraType()`
setCameraType(String cameraType);

// (Must) Set the export file path
setTargetPath(String path);

// (Must) Set the width of the exported video. It must be a power of 2.
setWidth(int width);

// (Must) Set the height of the exported video. It must be a power of 2.
setHeight(int height);

// (Must) Set the bitrate of the exported video.
setBitrate(int bitrate);

// (Optional) Set export mode, default is `PANORAMA`
// ExportMode.PANORAMA: Use when exporting panorama media
// ExportMode.SPHERE: Used when exporting flat thumbnails
setExportMode(ExportUtils.ExportMode mode);

// (Optional) Whether to enable dynamic stitching, the default is true.
setDynamicStitch(boolean dynamicStitch);

// (Optional) Cache Folder，the default is getCacheDir() + "/work_thumbnail"
setCacheWorkThumbnailRootPath(String path);

// (Optional) Cache Folder，the default is getCacheDir() + "/cut_scene"
setCacheCutSceneRootPath(String path);
```

IExportCallback

```
// Export Successful
onSuccess();

// Export Failed
onFail();

// Stop Export
onCancel();

// Export progress, callback only when exporting video
onProgress(float progress);
```

### Export Image

Export Panorama Media

```
ExportImageParamsBuilder builder = new ExportImageParamsBuilder()
        .setExportMode(ExportUtils.ExportMode.PANORAMA)
        .setCameraType(InstaCameraManager.getInstance().getCameraType())
        .setTargetPath(path)
        .setWidth(2048)
        .setHeight(1024);
int exportId = ExportUtils.exportImage(WorkWrapper, builder, IExportCallback);
```

Export Thumbnail

```
ExportImageParamsBuilder builder = new ExportImageParamsBuilder()
        .setExportMode(ExportUtils.ExportMode.SPHERE)
        .setCameraType(InstaCameraManager.getInstance().getCameraType())
        .setTargetPath(path)
        .setWidth(512)
        .setHeight(512)
        .setFov(mImagePlayerView.getFov())
        .setDistance(mImagePlayerView.getDistance())
        .setYaw(mImagePlayerView.getYaw())
        .setPitch(mImagePlayerView.getPitch());
int exportId = ExportUtils.exportImage(WorkWrapper, builder, IExportCallback);
```

### Export Video

Export Panorama Media

```
ExportVideoParamsBuilder builder = new ExportVideoParamsBuilder()
        .setExportMode(ExportUtils.ExportMode.PANORAMA)
        .setCameraType(InstaCameraManager.getInstance().getCameraType())
        .setTargetPath(path)
        .setWidth(2048)
        .setHeight(1024)
        .setBitrate(20 * 1024 * 1024);
int exportId = ExportUtils.exportVideo(WorkWrapper, builder, IExportCallback);
```

Export Thumbnail

```
ExportImageParamsBuilder builder = new ExportImageParamsBuilder()
        .setExportMode(ExportUtils.ExportMode.SPHERE)
        .setCameraType(InstaCameraManager.getInstance().getCameraType())
        .setTargetPath(path)
        .setWidth(512)
        .setHeight(512)
        .setFov(mVideoPlayerView.getFov())
        .setDistance(mVideoPlayerView.getDistance())
        .setYaw(mVideoPlayerView.getYaw())
        .setPitch(mVideoPlayerView.getPitch());
int exportId = ExportUtils.exportVideoToImage(WorkWrapper, builder, IExportCallback);
```

### Stop Export

```
ExportUtils.stopExport(int exportId);
```



## <a name="MediaSDK生成HDR图像" />Generate HDR Image

```
boolean StitchUtils.generateHDR(WorkWrapper workWrapper, String outputPath);
```

> After the generate is successful, the outputPath can be played as a proxy. `ImageParamsBuilder.setUrlForPlay(outputPath);`
