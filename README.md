# 目录
- [Camera SDK功能](#CameraSDK功能)
  - [初始化](#CameraSDK初始化)
  - [连接/断开/监听相机](#CameraSDK连接/断开/监听相机)
  - [获取相机状态属性](#CameraSDK获取相机状态属性)
  - [获取相机其他信息](#CameraSDK获取相机其他信息)
  - [预览](#CameraSDK预览)
  - [拍摄](#CameraSDK拍摄)
  - [属性设置](#CameraSDK属性设置)
  - [其他功能](#CameraSDK其他功能)
  - [OSC](#CameraSDKOSC)
- [Media SDK功能](#MediaSDK功能)
  - [初始化](#MediaSDK初始化)
  - [预览](#MediaSDK预览)
  - [WorkWrapper](#MediaSDKWorkWrapper)
  - [获取媒体文件列表](#MediaSDK获取媒体文件列表)
  - [播放](#MediaSDK播放)
  - [导出](#MediaSDK导出)
  - [HDR拼接](#MediaSDKHDR拼接)
  
# <a name="CameraSDK功能" />Camera SDK功能

## <a name="CameraSDK初始化" />初始化

```
// IOscRequestDelegate：如果项目中使用SDK提供的OSC管理类，需创建一个Delegate实例，否则可传null
InstaCameraSDK.init(Application, @Nullable IOscRequestDelegate);
```



## <a name="CameraSDK连接/断开/监听相机" />连接/断开/监听相机

### 连接相机

```
// 通过USB连接相机
InstaCameraManager.getInstance().openCamera(InstaCameraManager.ConnectBy.USB);
// 通过WIFI连接相机
InstaCameraManager.getInstance().openCamera(InstaCameraManager.ConnectBy.WIFI);
```

### 断开相机

```
InstaCameraManager.getInstance().closeCamera();
```

### 相机状态监听器

可在多个页面`注册/反注册`监听

```
InstaCameraManager.getInstance().registerCameraChangedCallback(ICameraChangedCallback);
InstaCameraManager.getInstance().unregisterCameraChangedCallback(ICameraChangedCallback);
```

ICameraChangedCallback

```
// 相机状态发生改变
// enabled：当前相机是否可用
onCameraStatusChanged(boolean enabled);

// 相机连接失败
// 常见的情况是已经有其他手机或者本手机的其他应用跟这台相机建立了连接，导致此次建立失败，需要其他手机先断开与此相机的连接
onCameraConnectError();

// SD卡插拔通知
// enabled：当前SD卡是否可用
onCameraSDCardStateChanged(boolean enabled);

// SD卡存储状态发生改变
// freeSpace：当前可用大小
// totalSpace：总大小
onCameraStorageChanged(long freeSpace, long totalSpace);

// 相机电量低通知
void onCameraBatteryLow();

// 相机电量发生改变通知
// batteryLevel：当前电量（0-100，充电时会始终返回100）
// isCharging：是否正在充电中
onCameraBatteryUpdate(int batteryLevel, boolean isCharging);
```



## <a name="CameraSDK获取相机状态属性" />获取相机状态属性

当前相机是否已连接

```
InstaCameraManager.getInstance().isCameraConnected();
```

当前是否为WIFI连接

```
InstaCameraManager.getInstance().isCameraConnectedByWifi();
```

当前是否为USB连接

```
InstaCameraManager.getInstance().isCameraConnectedByUsb();
```



## <a name="CameraSDK获取相机其他信息" />获取相机其他信息

> 下列方法只被作为参数供其他接口调用，开发者无需重点关注，后边会在需要调用处进行相应说明。

获取相机型号名

```
InstaCameraManager.getInstance().getCameraType();
```

获取相机文件尾

```
InstaCameraManager.getInstance().getMediaOffset();
```

获取相机Host地址

```
InstaCameraManager.getInstance().getCameraHttpPrefix();
```

获取相机文件信息列表

```
InstaCameraManager.getInstance().getAllUrlList();
InstaCameraManager.getInstance().getRawUrlList();
InstaCameraManager.getInstance().getCameraInfoMap();
```



## <a name="CameraSDK预览" />预览

### 打开预览流

相机连接成功后即可打开预览流

```
InstaCameraManager.getInstance().startPreviewStream();
```

### 关闭预览流

退出预览页面时需关闭预览，如果相机被动断开了连接或者在预览过程中直接调用了`closeCamera`，SDK会自动关闭预览流处理相关状态，不需要额外调用`closePreviewStream`

```
InstaCameraManager.getInstance().closePreviewStream();
```

### 监听预览状态

```
InstaCameraManager.getInstance().setPreviewStatusChangedListener(IPreviewStatusListener);
```

IPreviewStatusListener

```
// 正在开启预览
onOpening();

// 预览流已开启，可播放显示
// 播放操作见[MediaSDK功能 - 预览]
onOpened();

// 预览已停止
onIdle();

// 预览开启失败
onError();
```



## <a name="CameraSDK拍摄" />拍摄

> 相机连接成功后即可调用拍摄接口

### 拍摄模式

普通拍照

```
// isRaw：如果为true则拍摄RAW格式原图
InstaCameraManager.getInstance().startNormalCapture(boolean isRaw);

// 判断状态
InstaCameraManager.getInstance().isNormalCapturing();
```

HDR拍照

```
// isRaw：如果为true则拍摄RAW格式原图
InstaCameraManager.getInstance().startHDRCapture(false);

// 判断状态
InstaCameraManager.getInstance().isHDRCapturing();
```

间隔拍照

```
// 开始
InstaCameraManager.getInstance().startIntervalShooting();

// 停止
InstaCameraManager.getInstance().stopIntervalShooting();

// 判断状态
InstaCameraManager.getInstance().isIntervalShooting();

// 设置间隔时间，单位ms
InstaCameraManager.getInstance().setIntervalTime(int intervalMs);
```

普通录像

```
// 开始
InstaCameraManager.getInstance().startNormalRecord();

// 停止
InstaCameraManager.getInstance().stopNormalRecord();

// 判断状态
InstaCameraManager.getInstance().isNormalRecording();
```

HDR录像

```
// 开始
InstaCameraManager.getInstance().startHDRRecord();

// 停止
InstaCameraManager.getInstance().stopHDRRecord();

// 判断状态
InstaCameraManager.getInstance().isHDRRecording();
```

延时摄影

```
// 开始
InstaCameraManager.getInstance().startTimeLapse();

// 停止
InstaCameraManager.getInstance().stopTimeLapse();

// 判断状态
InstaCameraManager.getInstance().isTimeLapsing();
```

### 监听拍摄状态

```
InstaCameraManager.getInstance().setCaptureStatusListener(ICaptureStatusListener);
```

ICaptureStatusListener

```
// 开始拍摄
onCaptureStarting();

// 拍摄中...
onCaptureWorking();

// 拍摄停止中...
onCaptureStopping();

// 拍摄结束
onCaptureFinish();

// 录像时长，单位ms
onCaptureTimeChanged(long captureTime);

// 间隔拍摄张数
onCaptureCountChanged(int captureCount);
```



## <a name="CameraSDK属性设置" />属性设置

### EV值

```;
// 设置某模式EV值
// funcMode：设置EV值时要先选择拍摄模式 @InstaCameraManager.FunctionMode
// ev：范围 -4 ~ 4
InstaCameraManager.getInstance().setExposureEV(int funcMode, float ev);

// 获取某模式EV值
InstaCameraManager.getInstance().getExposureEV(int funcMode);
```

所支持的设置模式：InstaCameraManager.FunctionMode

```
FUNCTION_MODE_CAPTURE_NORMAL：普通拍照
FUNCTION_MODE_HDR_CAPTURE：HDR拍照
FUNCTION_MODE_INTERVAL_SHOOTING：间隔拍照
FUNCTION_MODE_RECORD_NORMAL：普通录像
FUNCTION_MODE_HDR_RECORD：HDR录像
FUNCTION_MODE_BULLETTIME：子弹时间
FUNCTION_MODE_TIMELAPSE：延时摄影
```

### 相机提示音

> 设置相机拍摄时是否有提示音

```
// 设置
InstaCameraManager.getInstance().setCameraBeepSwitch(boolean beep);

// 判断
InstaCameraManager.getInstance().isCameraBeep();
```



## <a name="CameraSDK其他功能" />其他功能

### 陀螺仪矫正

> 必须在WIFI连接相机时才能使用此功能
>
> 矫正前请将相机竖直立在平稳的水平面上

```
InstaCameraManager.getInstance().calibrateGyro(ICameraOperateCallback);
```

ICameraOperateCallback

```
// 操作成功
onSuccessful();

// 操作失败
onFailed();

// 相机连接错误导致的失败
onCameraConnectError();
```

### 格式化SD卡

```
InstaCameraManager.getInstance().formatStorage(ICameraOperateCallback);
```

ICameraOperateCallback

> 详情见`陀螺仪矫正`

------



## <a name="CameraSDKOSC" />OSC

IOscRequestDelegate

> 用您喜欢的网络库实现Get/Post同步请求，并将结果返回给OscManager使用

```
/**
 * 以Get的方式发送一个Http网络请求
 *
 * @param url       请求地址
 * @param headerMap 需要使用的HTTP请求头
 * @return 网络请求响应正文(body)或错误信息(message)
 */
OSCResult sentRequestByGet(String url, Map<String, String> headerMap);

/**
 * 以Post的方式发送一个Http网络请求
 *
 * @param url       请求地址
 * @param content   osc命令内容
 * @param headerMap 需要使用的HTTP请求头
 * @return 网络请求响应正文(body)或错误信息(message)
 */
OSCResult sentRequestByPost(String url, String content, Map<String, String> headerMap);

/**
 * 请求结果
 *
 * @param isSuccessful 是否成功
 * @param result       成功为response.body().string()，失败为response.message()      
OSCResult(boolean isSuccessful, String result)
```

OscManager

```
// 发送设置相机参数命令
// OscOptions为参数详情
// 若成功，callback返回null
setOptions(OscOptions options, IOscCallback callback);

// 发送拍照命令
// OscOptions：先设置相机参数，之后自动拍照，如果当前CaptureMode为image则选填，否则必填
// 若成功，callback返回拍摄文件地址（String[] urls），可下载到本地
takePicture(OscOptions options, IOscCallback callback);

// 发送开始录像命令
// OscOptions：可先设置相机参数，之后自动开始录像，如果当前CaptureMode为video则选填，否则必填
// 若成功，callback返回null
startRecord(OscOptions options, IOscCallback callback);

// 发送停止录像命令
// 若成功，callback返回拍摄文件地址（String[] urls），可下载到本地
startRecord(IOscCallback callback);

// 自定义OSC命令请求
// oscApi：如/osc/info、/osc/state等
// content：RequestBody的内容，如果为null则使用GET请求，否则使用POST请求
customRequest(String oscApi, String content, IOscCallback callback);
```

OscOptions

> ONE X相机所支持的设置选项

```
OscOptions.Builder builder = new OscOptions.Builder();
OscOptions oscOptions = builder.build();

// 设置拍摄模式：OscOptions.CAPTURE_MODE_IMAGE/CAPTURE_MODE_VIDEO
setCaptureMode(@CaptureMode String captureMode)

// 设置曝光模式
setExposureProgram(@ExposureProgram int exposureProgram);

// 设置ISO
setISO(@ISO int iso);

// 设置快门速度
setShutterSpeed(@ShutterSpeed int speed);

// 设置白平衡
setWhiteBalance(@WhiteBalance String whiteBalance);

// 设置是否启动HDR拍摄
setHDREnabled(boolean enabled);

// 设置HDR拍摄参数
setExposureBracketParams(int shots, int increment);
```

IOscCallback

```
// 开始发送OSC命令请求
onStartRequest();

// 请求成功
onSuccessful(Object object);

// 请求失败
onError(String message);
```



# <a name="MediaSDK功能" />Media SDK功能

## <a name="MediaSDK初始化" />初始化

```
InstaMediaSDK.init(Application);
```



## <a name="MediaSDK预览" />预览

InstaCapturePlayerView

```
// 绑定生命周期
setLifecycle(Lifecycle);

// 设置播放器状态监听
setPlayerViewListener(PlayerViewListener);

// 设置手势操作监听
setGestureListener(PlayerGestureListener);

// 播放前配置参数
prepare(CaptureParamsBuilder);

// 播放
play();

// 释放
destroy();
```

```
// 切换到普通模式，仅限渲染模式为RENDER_MODE_AUTO时可用
switchNormalMode();

// 切换到鱼眼模式，仅限渲染模式为RENDER_MODE_AUTO时可用
switchFisheyeMode();

// 切换到透视模式，仅限渲染模式为RENDER_MODE_AUTO时可用
switchPerspectiveMode();

// 判断播放器是否正在加载中
boolean isLoading();

// CameraSdk与PlayerView连接所需的参数
Object getPipeline();
```

PlayerViewListener

```
// 加载状态发生改变
// isLoading：是否处于加载中
onLoadingStatusChanged(boolean isLoading);

// 加载完成
onLoadingFinish();

// 加载失败
// desc：错误详情
onFail(String desc);
```

PlayerGestureListener

```
// 按下
onDown(MotionEvent e);

// 点击
onTap(MotionEvent e);

// 抬起
onUp();

// 长按
onLongPress(MotionEvent e);

// 缩放
onZoom();
onZoomAnimation();
onZoomAnimationEnd();

// 滑动
onScroll();
onFlingAnimation();
onFlingAnimationEnd();
```

CaptureParamsBuilder

```
// （必须）配置相机型号，从InstaCameraManager.getInstance().getCameraType()获取
setCameraType(String cameraType);

// （必须）配置文件尾参数，从InstaCameraManager.getInstance().getMediaOffset()获取
setMediaOffset();

// （可选）设置渲染模式，默认为RENDER_MODE_AUTO
// RENDER_MODE_AUTO：普通模式，可切换鱼眼、透视预览样式
// RENDER_MODE_PLANE_STITCH：平铺模式
setRenderModelType(int renderModelType);

// （可选）设置画面横纵比，默认为全屏显示（即充满画布）
// 如果渲染模式为[平铺模式]，推荐设置比例为2:1
setScreenRatio(int ratioX, int ratioY);

// （可选）是否允许手势操作，默认为true
setGestureEnabled(boolean enabled);
```

在`CameraSdk`的`IPreviewStatusListener.onOpened()`回调中播放显示预览

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

播放、导出等接口需要传递的数据模型

```
// 通过媒体文件路径构建
WorkWrapper workWrapper = new WorkWrapper(String url);
WorkWrapper workWrapper = new WorkWrapper(String[] urls);

// 获取媒体文件路径
String[] getUrls();

// 判断媒体文件类型
boolean isVideo();
boolean isPhoto();

// 获取唯一标识ID
String getIdenticalKey()
```



## <a name="MediaSDK获取媒体文件列表" />获取媒体文件列表

### 从相机获取

```
// 参数从CameraSDK获取
List<WorkWrapper> WorkUtils.getAllCameraWorks(
    InstaCameraManager.getInstance().getCameraHttpPrefix(),
    InstaCameraManager.getInstance().getCameraInfoMap(),
    InstaCameraManager.getInstance().getAllUrlList(),
    InstaCameraManager.getInstance().getRawUrlList());
```

### 从本地目录获取

```
// dirPath：本地目录路径
List<WorkWrapper> getAllLocalWorks(String dirPath);
```



## <a name="MediaSDK播放" />播放

### 图片

InstaImagePlayerView

```
// 绑定生命周期
setLifecycle(Lifecycle);

// 设置播放器状态监听
setPlayerViewListener(PlayerViewListener);

// 设置手势操作监听
setGestureListener(PlayerGestureListener);

// 播放前配置参数
prepare(WorkWrapper, ImageParamsBuilder);

// 播放
play();

// 释放
destroy();
```

```
// 判断播放器是否正在加载中
boolean isLoading();

// 获取当前播放角度的数据，供导出缩略图时配置参数使用
float getFov();
float getDistance();
float getYaw();
float getPitch();
```

PlayerViewListener

> 详情见`Media SDK功能 - 预览`

PlayerGestureListener

> 详情见`Media SDK功能 - 预览`

ImageParamsBuilder

```
// （可选）是否启用动态拼接，默认为true
setDynamicStitch(boolean dynamicStitch);

// （可选）设置播放代理文件，例如拼接生成的HDR.jpg，默认为null
setUrlForPlay(String url);

// （可选）设置画面横纵比，默认为全屏显示（即充满画布）
setScreenRatio(int ratioX, int ratioY);

// （可选）是否允许手势操作，默认为true
setGestureEnabled(boolean enabled);

// （可选）缓存文件目录，默认为getCacheDir() + "/work_thumbnail"
setCacheWorkThumbnailRootPath(String path);

// （可选）缓存文件目录，默认为getCacheDir() + "/stabilizer"
setStabilizerCacheRootPath(String path);

// （可选）缓存文件目录，默认为getCacheDir() + "/cut_scene"
setCacheCutSceneRootPath(String path);
```

### 视频

InstaVideoPlayerView

```
// 绑定生命周期
setLifecycle(Lifecycle);

// 设置播放器状态监听
setPlayerViewListener(PlayerViewListener);

// 设置手势操作监听
setGestureListener(PlayerGestureListener);

// 设置视频状态监听
setVideoStatusListener(VideoStatusListener);

// 播放前配置参数
prepare(WorkWrapper, ImageParamsBuilder);

// 播放
play();

// 释放
destroy();
```

```
// 判断播放器是否正在加载中
boolean isLoading();

// 获取当前播放角度的数据，供导出缩略图时配置参数使用
float getFov();
float getDistance();
float getYaw();
float getPitch();

// 播放状态
isPlaying();

// 循环播放
setLooping(boolean isLooping);
isLooping();

// 设置音量(0-1)
setVolume(float volume);

// 暂停
pause();

// 恢复
resume();

// 跳转
seekTo(long position);
isSeeking();

// 获取当前播放位置
getVideoCurrentPosition();

// 获取视频总时长
getVideoTotalDuration();
```

PlayerViewListener

> 详情见`Media SDK功能 - 预览`

PlayerGestureListener

> 详情见`Media SDK功能 - 预览`

VideoStatusListener

```
// 播放进度改变
// position：当前播放位置
// length：视频总时长
onProgressChanged(long position, long length);

// 播放状态改变
onPlayStateChanged(boolean isPlaying);

// 跳转完成
onSeekComplete();

// 播放完毕
onCompletion();
```

VideoParamsBuilder

```
// （可选）加载图标，默认为无
setLoadingImageResId(int resId);

// （可选）加载时的背景色，默认为黑色
setLoadingBackgroundColor(int color);

// （可选）准备完成后是否自动播放，默认为true
setAutoPlayAfterPrepared(boolean autoPlayAfterPrepared);

// （可选）是否循环播放，默认为true
setIsLooping(boolean isLooping);

// （可选）设置画面横纵比，默认为全屏显示（即充满画布）
setScreenRatio(int ratioX, int ratioY);

// （可选）是否允许手势操作，默认为true
setGestureEnabled(boolean enabled);

// （可选）缓存文件目录，默认为getCacheDir() + "/work_thumbnail"
setCacheWorkThumbnailRootPath(String path);

// （可选）缓存文件目录，默认为getCacheDir() + "/cut_scene"
setCacheCutSceneRootPath(String path);
```



## <a name="MediaSDK导出" />导出

ExportImageParamsBuilder

```
// （必须）设置相机型号名，从InstaCameraManager.getInstance().getCameraType()获取
setCameraType(String cameraType);

// （必须）设置导出文件路径
setTargetPath(String path);

// （必须）设置导出图片宽度，需为2的幂数
setWidth(int width);

// （必须）设置导出图片高度，需为2的幂数
setHeight(int height);

// （可选）设置导出模式，默认为PANORAMA
// ExportMode.PANORAMA：导出全景素材时使用
// ExportMode.SPHERE：导出平面缩略图时使用
setExportMode(ExportUtils.ExportMode mode);

// （可选）是否启用动态拼接，默认为true
setDynamicStitch(boolean dynamicStitch);

// （可选）设置镜头角度，推荐在导出缩略图时使用，可从PlayerView中获取当前显示的角度参数
setFov(float fov);
setDistance(float distance);
setYaw(float yaw);
setPitch(float pitch);

// （可选）缓存文件目录，默认为getCacheDir() + "/work_thumbnail"
setCacheWorkThumbnailRootPath(String path);

// （可选）缓存文件目录，默认为getCacheDir() + "/stabilizer"
setStabilizerCacheRootPath(String path);

// （可选）缓存文件目录，默认为getCacheDir() + "/cut_scene"
setCacheCutSceneRootPath(String path);
```

ExportVideoParamsBuilder

```
// （必须）设置相机型号名，从InstaCameraManager.getInstance().getCameraType()获取
setCameraType(String cameraType);

// （必须）设置导出文件路径
setTargetPath(String path);

// （必须）设置导出视频宽度，需为2的幂数
setWidth(int width);

// （必须）设置导出视频高度，需为2的幂数
setHeight(int height);

// （必须）设置导出视频比特率
setBitrate(int bitrate);

// （可选）设置导出模式，默认为PANORAMA
// ExportMode.PANORAMA：导出全景素材时使用
// ExportMode.SPHERE：导出平面缩略图时使用
setExportMode(ExportUtils.ExportMode mode);

// （可选）是否启用动态拼接，默认为true
setDynamicStitch(boolean dynamicStitch);

// （可选）缓存文件目录，默认为getCacheDir() + "/work_thumbnail"
setCacheWorkThumbnailRootPath(String path);

// （可选）缓存文件目录，默认为getCacheDir() + "/cut_scene"
setCacheCutSceneRootPath(String path);
```

IExportCallback

```
// 导出成功
onSuccess();

// 导出失败
onFail();

// 停止导出
onCancel();

// 导出进度，仅导出视频时有进度回调
onProgress(float progress);
```

### 图片

导出全景图

```
ExportImageParamsBuilder builder = new ExportImageParamsBuilder()
        .setExportMode(ExportUtils.ExportMode.PANORAMA)
        .setCameraType(InstaCameraManager.getInstance().getCameraType())
        .setTargetPath(path)
        .setWidth(2048)
        .setHeight(1024);
int exportId = ExportUtils.exportImage(WorkWrapper, builder, IExportCallback);
```

导出缩略图

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

### 视频

导出全景视频

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

导出缩略图

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

### 停止导出

```
ExportUtils.stopExport(int exportId);
```



## <a name="MediaSDKHDR拼接" />HDR拼接

```
boolean StitchUtils.generateHDR(WorkWrapper workWrapper, String outputPath);
```

> 拼接成功后可将outputPath作为代理播放，ImageParamsBuilder.setUrlForPlay(outputPath);
