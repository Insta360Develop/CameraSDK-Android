中文版本请参看[这里](./readme_cn.md)

------



# Table of Contents
- [Camera SDK Function](#CameraSDK功能)
  - [Initialization](#CameraSDK初始化)
  - [Connect / Disconnect / Monitor Camera](#CameraSDK连接/断开/监听相机)
  - [Get other camera information](#CameraSDK获取相机其他信息)
  - [Preview](#CameraSDK预览)
  - [Capture](#CameraSDK拍摄)
  - [Settings](#CameraSDK属性设置)
  - [Others](#CameraSDK其他功能)
  - [OSC](#CameraSDKOSC)
- [Media SDK Function](#MediaSDK功能)
  - [Initialization](#MediaSDK初始化)
  - [Preview](#MediaSDK预览)
  - [WorkWrapper](#MediaSDKWorkWrapper)
  - [Get Media File List](#MediaSDK获取媒体文件列表)
  - [Player](#MediaSDK播放)
  - [Export](#MediaSDK导出)
  - [Generate HDR Image](#MediaSDK生成HDR图像)
  
# <a name="CameraSDK功能" />Camera SDK Function

## <a name="CameraSDK初始化" />Initialization

You need to initialize SDK in Application

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



## <a name="CameraSDK连接/断开/监听相机" />Connect / Disconnect / Listener Camera

### Connect

You can connect the camera by WIFI or USB

By WIFI

```
InstaCameraManager.getInstance().openCamera(InstaCameraManager.ConnectBy.WIFI);
```

By USB

```
InstaCameraManager.getInstance().openCamera(InstaCameraManager.ConnectBy.USB);
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

### Listener

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


## <a name="CameraSDK获取相机其他信息" />Get other camera information

> The following methods are only used as parameters for other interfaces to call. Developers don't need to pay much attention to them, and they will be explained later at the places where they need to be called.

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



## <a name="CameraSDK预览" />Preview

### Open Preview Stream

You can open the preview stream after the camera is successfully connected

```
InstaCameraManager.getInstance().startPreviewStream();
```

### Close Preview Stream

You need to close the preview when exiting the preview page. If the camera is passively disconnected or if you call `closeCamera` directly during the preview process, the SDK will automatically close the preview stream processing related state without the need to call` closePreviewStream`.

```
InstaCameraManager.getInstance().closePreviewStream();
```

### Listener Status Changed

```
InstaCameraManager.getInstance().setPreviewStatusChangedListener(IPreviewStatusListener);
```

IPreviewStatusListener

```
// Opening preview
onOpening();

// Preview stream is on and can be played
// See[Media SDK Function - Preview]
onOpened();

// Preview stopped
onIdle();

// Preview failed to open
onError();
```



## <a name="CameraSDK拍摄" />Capture

> You can call the Capture Interface after the camera is successfully connected

### Capture Mode

Normal Capture

```
// isRaw: If true then capture the original image in RAW format
InstaCameraManager.getInstance().startNormalCapture(boolean isRaw);

// Whether the camera is Normal Capturing
InstaCameraManager.getInstance().isNormalCapturing();
```

HDR Capture

```
// isRaw：If true then capture the original image in RAW format
InstaCameraManager.getInstance().startHDRCapture(false);

// Whether the camera is HDR Capturing
InstaCameraManager.getInstance().isHDRCapturing();
```

Interval Capture

```
// Start
InstaCameraManager.getInstance().startIntervalShooting();

// Stop
InstaCameraManager.getInstance().stopIntervalShooting();

// Whether the camera is Interval Capturing
InstaCameraManager.getInstance().isIntervalShooting();

// Set Interval Time，in ms
InstaCameraManager.getInstance().setIntervalTime(int intervalMs);
```

Normal Record

```
// Start
InstaCameraManager.getInstance().startNormalRecord();

// Stop
InstaCameraManager.getInstance().stopNormalRecord();

// Whether the camera is Normal Recording
InstaCameraManager.getInstance().isNormalRecording();
```

HDR Record

```
// Start
InstaCameraManager.getInstance().startHDRRecord();

// Stop
InstaCameraManager.getInstance().stopHDRRecord();

// Whether the camera is HDR Recording
InstaCameraManager.getInstance().isHDRRecording();
```

TimeLapse

```
// Start
InstaCameraManager.getInstance().startTimeLapse();

// Stop
InstaCameraManager.getInstance().stopTimeLapse();

// Whether the camera is TimeLapsing
InstaCameraManager.getInstance().isTimeLapsing();
```

### Listener Status Changed

```
InstaCameraManager.getInstance().setCaptureStatusListener(ICaptureStatusListener);
```

ICaptureStatusListener

```
// Capture Starting
onCaptureStarting();

// Capture Working...
onCaptureWorking();

// Capture Stopping...
onCaptureStopping();

// Capture Finished
onCaptureFinish();

// Record Duration，in ms
onCaptureTimeChanged(long captureTime);

// Interval shots
onCaptureCountChanged(int captureCount);
```



## <a name="CameraSDK属性设置" />Settings

### EV Value

```;
// Set the EV value of a mode
// funcMode: Select the Capture Mode when setting the EV value @InstaCameraManager.FunctionMode
// ev: Range -4 ~ 4
InstaCameraManager.getInstance().setExposureEV(int funcMode, float ev);

// Get the EV value of a mode
InstaCameraManager.getInstance().getExposureEV(int funcMode);
```

Supported setting modes: InstaCameraManager.FunctionMode

```
FUNCTION_MODE_CAPTURE_NORMAL：Normal Capture
FUNCTION_MODE_HDR_CAPTURE：HDR Capture
FUNCTION_MODE_INTERVAL_SHOOTING：Interval Capture
FUNCTION_MODE_RECORD_NORMAL：Normal Record
FUNCTION_MODE_HDR_RECORD：HDR Record
FUNCTION_MODE_BULLETTIME：BulletTime
FUNCTION_MODE_TIMELAPSE：TimeLapse
```

### Camera Beep

> Set whether there is a beep sound when the camera shoots

```
// Set Beep Enabled
InstaCameraManager.getInstance().setCameraBeepSwitch(boolean beep);

// Whether beep is enabled
InstaCameraManager.getInstance().isCameraBeep();
```



## <a name="CameraSDK其他功能" />Others

### Calibrate Gyro

> This function must be used when the camera is connected to WIFI
>
> Before correction, please stand the camera upright on a stable and level surface.

```
InstaCameraManager.getInstance().calibrateGyro(ICameraOperateCallback);
```

ICameraOperateCallback

```
// Operation Successful
onSuccessful();

// Operation Failed
onFailed();

// Failure due to incorrect camera connection
onCameraConnectError();
```

### Format SD card

```
InstaCameraManager.getInstance().formatStorage(ICameraOperateCallback);
```

ICameraOperateCallback

> See`Calibrate Gyro`



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

------



# <a name="MediaSDK功能" />Media SDK Function

## <a name="MediaSDK初始化" />Initialization

```
InstaMediaSDK.init(Application);
```



## <a name="MediaSDK预览" />Preview

InstaCapturePlayerView

```
// Bind Lifecycle
setLifecycle(Lifecycle);

// Set player status listener
setPlayerViewListener(PlayerViewListener);

// Set gesture listener
setGestureListener(PlayerGestureListener);

// Configuration parameters before playback
prepare(CaptureParamsBuilder);

// Play
play();

// Release
destroy();
```

```
// Switch Normal Mode. Available only when the rendering mode is `RENDER_MODE_AUTO`
switchNormalMode();

// Switch Fisheye Mode. Available only when the rendering mode is `RENDER_MODE_AUTO`
switchFisheyeMode();

// Switch Perspective Mode. Available only when the rendering mode is `RENDER_MODE_AUTO`
switchPerspectiveMode();

// Whether the player is loading
boolean isLoading();

// Parameters required for CameraSdk to connect with PlayerView
Object getPipeline();
```

PlayerViewListener

```
// Loading Status Changed
// isLoading: Whether the player is loading
onLoadingStatusChanged(boolean isLoading);

// Load Finish
onLoadingFinish();

// Load Failed
// desc: Error details
onFail(String desc);
```

PlayerGestureListener

```
// Touch Down
onDown(MotionEvent e);

// Click
onTap(MotionEvent e);

// Touch Up
onUp();

// Long Press
onLongPress(MotionEvent e);

// Scale
onZoom();
onZoomAnimation();
onZoomAnimationEnd();

// Scroll
onScroll();
onFlingAnimation();
onFlingAnimationEnd();
```

CaptureParamsBuilder

```
// (Must) Set Camera Type. Get from `InstaCameraManager.getInstance().getCameraType()`
setCameraType(String cameraType);

// (Must) Set Media Offset. Get from `InstaCameraManager.getInstance().getMediaOffset()`
setMediaOffset();

// (Optional) Set Render Model Type, the default is `RENDER_MODE_AUTO`
// RENDER_MODE_AUTO：Normal Mode. Could Switch to `Fisheye Mode` or `Perspective Mode`
// RENDER_MODE_PLANE_STITCH：Plane Mode
setRenderModelType(int renderModelType);

// (Optional) Set the aspect ratio of the screen, the default is full screen display (ie full canvas)
// If the rendering mode type is `RENDER_MODE_PLANE_STITCH`, the recommended setting ratio is 2:1
setScreenRatio(int ratioX, int ratioY);

// (Optional) Whether to allow gesture operations, the default is true
setGestureEnabled(boolean enabled);
```

Display preview in `IPreviewStatusListener.onOpened ()` callback of `CameraSdk`

```
@Override
public void onOpened() {
    InstaCameraManager.getInstance().setStreamEncode();
    mCapturePlayerView.setPlayerViewListener(new PlayerViewListener() {
        @Override
        public void onLoadingFinish() {
        	Object pipeline = mCapturePlayerView.getPipeline();
            InstaCameraManager.getInstance().setPipeline(pipeline);
        }
    });
    mCapturePlayerView.prepare(createParams());
    mCapturePlayerView.play();
}
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



## <a name="MediaSDK获取媒体文件列表" />Get Media File List

### Get from camera

```
// Parameters obtained from `CameraSDK`
List<WorkWrapper> WorkUtils.getAllCameraWorks(
    InstaCameraManager.getInstance().getCameraHttpPrefix(),
    InstaCameraManager.getInstance().getCameraInfoMap(),
    InstaCameraManager.getInstance().getAllUrlList(),
    InstaCameraManager.getInstance().getRawUrlList());
```

### Get from local directory

```
// dirPath: Local directory path
List<WorkWrapper> getAllLocalWorks(String dirPath);
```



## <a name="MediaSDK播放" />Player

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
