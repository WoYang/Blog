[点击查看文档](https://docs.google.com/presentation/d/1b-W_cMQsGYY6ujtT_JzpLwouQdTECzMIrYcwc-mwZAY/edit?usp=sharing)

# 1.为什么想要开始使用Flutter

1.Flutter是Fuchsia系统主要的UI开发框架

Fuchsia不同于Android，Android基于Linux内核，对内存、性能硬件要求比较高；Zircon 微内核是为 Fuchsia OS 提供支持的核心平台，无论是对存储还是内存之类的硬件要求都大幅降低，在这种前提下Fuchsia甚至可以适用各种物联网设备。更多关于Fuchsia:[https://fuchsia-china.com/fuchsia-os-intro-slide/](https://fuchsia-china.com/fuchsia-os-intro-slide/)

2.Flutter优势是跨平台且性能高

a.它在原生系统上使用Skia绘制

b.使用Dart binding，避免了React Native等跨平台方案通过桥接器与Javascript通讯导致效率低下的问题

3.目前已知：

阿里的闲鱼APP已经使用Flutter在移动终端混合开发并商用。

腾讯在NOW直播上已经开始投入使用。

京东在京东金融上已经投入使用。

美团也在使用Flutter技术混合开发。

更多了解:https://flutter.dev/showcase

4.基于Flutter的第三方集成正在增长，越来越多的技术会在Flutter上得以实现，是个挑战也是个机遇。

# 2.使用中将要面临的问题

1.能跨平台的是Flutter，其他资源比如播放器库，不能保证Android平台上的so能在IOS上正常运行，所以我们理解为UI能跨平台应该是更准确。

2.Flutter是用基于Skia来绘制控件，虽然能最大程度发挥性能，但也意味着要有很长的路要走。

3.Flutter是正在开源的项目，处于快速发展且不稳定阶段，有些需求还不支持，在使用Flutter实现客户化定制时，成本较大。

# 3.如何解决问题

  目前大多数的需求开发都需要投入很大精力在造轮子的事情上，这是现状。
  
要把Flutter方案直接投入大规模商业使用，对于任何公司都是一个巨大挑战。

参考目前其他厂商的方案，将Flutter适用在某个特定的业务下通过数据来反馈业务的好坏。比如闲鱼APP商品详情页就使用了Flutter接口和功能，并且已经线上验证OK，达到生产稳定性的要求。闲鱼APP使用到Flutter的主要功能包括视频播放、图片、ListView、键盘、浮层、动画、截屏。

----------------------------------------------------------------------------------------------------------------------------------------
**Flutter**尝鲜：
https://docs.google.com/presentation/d/1GrMfH7YWwVHtG-B0WzIIGiP9T47fn_Yew8GyGW3rY7E/edit?copiedFromTrash#slide=id.g24d7daddcc_0_5

**Flutter**的野心：
https://developers.googleblog.com/2019/05/Flutter-io19.html

![](https://upload-images.jianshu.io/upload_images/3613947-f52cd28b7ba8dce8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**Flutter**开发环境：

1.环境搭建、学习推荐

[Flutter中文网](https://flutterchina.club/flutter-for-android/)

[Flutter官网](https://flutter.dev/)

![Flutter框架总览](https://upload-images.jianshu.io/upload_images/3613947-6ea575133a5f74f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

[Flutter Web](https://flutter.dev/web)

Flutter源码：

**framework**:https://github.com/flutter/flutter

**engine**:https://github.com/flutter/engine

---------------------------------------------------------------------------------------------------------------------------------------

###Flutter TV端探索：

把Flutter适用到TV端，首先考虑到的就是操作体验。TV上通过遥控器实现操作，那么在Flutter中对遥控器的支持如何呢？

经过简单的尝试后，很遗憾的发现Flutter上的Widget对Touch事件的做了全面支持，但是在KeyEvent事件上并不友好。

那么就以遥控器**KeyEvent**事件源为切入点，来分析Flutter遥控器事件的处理，并抛出使用遥控器时面临的新的问题。

首先参考Android输入事件的传递机制如下：
![Android事件传递](https://upload-images.jianshu.io/upload_images/3613947-6bb9bf2f3da78a41.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Android View视图构成](https://upload-images.jianshu.io/upload_images/3613947-aba446d4d3600de9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Android View事件分发](https://upload-images.jianshu.io/upload_images/3613947-6669aadd9f38f372.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Flutter在Android上的构成大致可以拆分如下：
![Android Flutter构成](https://upload-images.jianshu.io/upload_images/3613947-7d515ff91dc887e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从Flutter的结构图上可以看到，Android的输入事件将通过SurfaceView转发到Flutter引擎中。

以下是对结构图作出源码分析：

Flutter在Android应用的入口是继承FlutterActivity的MainActivity，比如以最简单的[Flutter SDK](https://github.com/flutter/flutter)中[hello_world](https://github.com/flutter/flutter/tree/master/examples/hello_world)为例:
~~~Java
package io.flutter.examples.hello_world;

import android.os.Bundle;
import io.flutter.app.FlutterActivity;
import io.flutter.plugins.GeneratedPluginRegistrant;

public class MainActivity extends FlutterActivity {
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    GeneratedPluginRegistrant.registerWith(this);
  }
}
~~~
~~~Java
package io.flutter.plugins;

import io.flutter.plugin.common.PluginRegistry;

/**
 * Generated file. Do not edit.
 */
public final class GeneratedPluginRegistrant {
  public static void registerWith(PluginRegistry registry) {
    if (alreadyRegisteredWith(registry)) {
      return;
    }
  }

  private static boolean alreadyRegisteredWith(PluginRegistry registry) {
    final String key = GeneratedPluginRegistrant.class.getCanonicalName();
    if (registry.hasPlugin(key)) {
      return true;
    }
    registry.registrarFor(key);
    return false;
  }
}
~~~
可以看到GeneratedPluginRegistrant.java是通过工具自动生成的，添加依赖后构建工具自动生成，比如添加了对fluttertoast的依赖后自动生成如下：
~~~Java
package io.flutter.plugins;

import io.flutter.plugin.common.PluginRegistry;
import io.github.ponnamkarthik.toast.fluttertoast.FluttertoastPlugin;

/**
 * Generated file. Do not edit.
 */
public final class GeneratedPluginRegistrant {
  public static void registerWith(PluginRegistry registry) {
    if (alreadyRegisteredWith(registry)) {
      return;
    }
    FluttertoastPlugin.registerWith(registry.registrarFor("io.github.ponnamkarthik.toast.fluttertoast.FluttertoastPlugin"));
  }

  private static boolean alreadyRegisteredWith(PluginRegistry registry) {
    final String key = GeneratedPluginRegistrant.class.getCanonicalName();
    if (registry.hasPlugin(key)) {
      return true;
    }
    registry.registrarFor(key);
    return false;
  }
}
~~~
图中[**flutter engine**](https://github.com/flutter/engine) 是什么? 如何和Android产生联系？仅从FlutterActivity并不能看出什么端倪。
但是从应用的[AndroidManifest.xml](https://github.com/flutter/flutter/blob/master/examples/hello_world/android/app/src/main/AndroidManifest.xml)可以看到声明为[FlutterApplication](https://github.com/flutter/engine/blob/70a1106b509ea3f34febca59967ed7a76c05ce33/shell/platform/android/io/flutter/app/FlutterApplication.java)：
~~~Java
// Copyright 2013 The Flutter Authors. All rights reserved.
// Use of this source code is governed by a BSD-style license that can be
// found in the LICENSE file.

package io.flutter.app;

import android.app.Activity;
import android.app.Application;
import android.support.annotation.CallSuper;

import io.flutter.view.FlutterMain;

/**
 * Flutter implementation of {@link android.app.Application}, managing
 * application-level global initializations.
 */
public class FlutterApplication extends Application {
    @Override
    @CallSuper
    public void onCreate() {
        super.onCreate();
        FlutterMain.startInitialization(this);
    }

    private Activity mCurrentActivity = null;
    public Activity getCurrentActivity() {
        return mCurrentActivity;
    }
    public void setCurrentActivity(Activity mCurrentActivity) {
        this.mCurrentActivity = mCurrentActivity;
    }
}
~~~
在onCreat中完成了[FlutterMain.java](https://github.com/flutter/engine/blob/master/shell/platform/android/io/flutter/view/FlutterMain.java)初始化:
~~~Java
    /**
     * Starts initialization of the native system.
     * @param applicationContext The Android application context.
     */
    public static void startInitialization(Context applicationContext) {
        startInitialization(applicationContext, new Settings());
    }

    /**
     * Starts initialization of the native system.
     * @param applicationContext The Android application context.
     * @param settings Configuration settings.
     */
    public static void startInitialization(Context applicationContext, Settings settings) {
        if (Looper.myLooper() != Looper.getMainLooper()) {
          throw new IllegalStateException("startInitialization must be called on the main thread");
        }
        // Do not run startInitialization more than once.
        if (sSettings != null) {
          return;
        }

        sSettings = settings;

        long initStartTimestampMillis = SystemClock.uptimeMillis();
        initConfig(applicationContext);
        initAot(applicationContext);
        initResources(applicationContext);

        if (sResourceUpdater == null) {
            System.loadLibrary("flutter");
        } else {
            sResourceExtractor.waitForCompletion();
            File lib = new File(PathUtils.getDataDirectory(applicationContext), DEFAULT_LIBRARY);
            if (lib.exists()) {
                System.load(lib.getAbsolutePath());
            } else {
                System.loadLibrary("flutter");
            }
        }

        // We record the initialization time using SystemClock because at the start of the
        // initialization we have not yet loaded the native library to call into dart_tools_api.h.
        // To get Timeline timestamp of the start of initialization we simply subtract the delta
        // from the Timeline timestamp at the current moment (the assumption is that the overhead
        // of the JNI call is negligible).
        long initTimeMillis = SystemClock.uptimeMillis() - initStartTimestampMillis;
        nativeRecordStartTimestamp(initTimeMillis);
    }
~~~
完成了资源配置及libflutter.so的加载。其中libflutter.so就是**Flutter engine**模块。
接下来来看看入口[FlutterActivity.java](https://github.com/flutter/engine/blob/master/shell/platform/android/io/flutter/app/FlutterActivity.java)：
~~~Java
public class FlutterActivity extends Activity implements FlutterView.Provider, PluginRegistry, ViewFactory {
    private static final String TAG = "FlutterActivity";
    
    private final FlutterActivityDelegate delegate = new FlutterActivityDelegate(this, this);

    // These aliases ensure that the methods we forward to the delegate adhere
    // to relevant interfaces versus just existing in FlutterActivityDelegate.
    private final FlutterActivityEvents eventDelegate = delegate;
    private final FlutterView.Provider viewProvider = delegate;
    private final PluginRegistry pluginRegistry = delegate;
    ...
~~~
统一使用[FlutterActivityDelegate](https://github.com/flutter/engine/blob/master/shell/platform/android/io/flutter/app/FlutterActivityDelegate.java)来代理。
~~~Java
    @Override
    public void onCreate(Bundle savedInstanceState) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            Window window = activity.getWindow();
            window.addFlags(LayoutParams.FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS);
            window.setStatusBarColor(0x40000000);
            window.getDecorView().setSystemUiVisibility(PlatformPlugin.DEFAULT_SYSTEM_UI);
        }

        String[] args = getArgsFromIntent(activity.getIntent());
        FlutterMain.ensureInitializationComplete(activity.getApplicationContext(), args);

        flutterView = viewFactory.createFlutterView(activity);
        if (flutterView == null) {
            FlutterNativeView nativeView = viewFactory.createFlutterNativeView();
            flutterView = new FlutterView(activity, null, nativeView);
            flutterView.setLayoutParams(matchParent);
            activity.setContentView(flutterView);
            launchView = createLaunchView();
            if (launchView != null) {
                addLaunchView();
            }
        }

        if (loadIntent(activity.getIntent())) {
            return;
        }

        String appBundlePath = FlutterMain.findAppBundlePath(activity.getApplicationContext());
        if (appBundlePath != null) {
            runBundle(appBundlePath);
        }
    }
~~~
在onCreat阶段通过ensureInitializationComplete完成初始化：
~~~Java
 /**
     * Blocks until initialization of the native system has completed.
     * @param applicationContext The Android application context.
     * @param args Flags sent to the Flutter runtime.
     */
    public static void ensureInitializationComplete(Context applicationContext, String[] args) {
        if (Looper.myLooper() != Looper.getMainLooper()) {
          throw new IllegalStateException("ensureInitializationComplete must be called on the main thread");
        }
        if (sSettings == null) {
          throw new IllegalStateException("ensureInitializationComplete must be called after startInitialization");
        }
        if (sInitialized) {
            return;
        }
        try {
            sResourceExtractor.waitForCompletion();

            List<String> shellArgs = new ArrayList<>();

            shellArgs.add("--icu-symbol-prefix=_binary_icudtl_dat");
            ApplicationInfo applicationInfo = applicationContext.getPackageManager().getApplicationInfo(
                applicationContext.getPackageName(), PackageManager.GET_META_DATA);
            shellArgs.add("--icu-native-lib-path=" + applicationInfo.nativeLibraryDir + File.separator + DEFAULT_LIBRARY);

            if (args != null) {
                Collections.addAll(shellArgs, args);
            }
            if (sIsPrecompiledAsSharedLibrary) {
                shellArgs.add("--" + AOT_SHARED_LIBRARY_PATH + "=" +
                    new File(PathUtils.getDataDirectory(applicationContext), sAotSharedLibraryPath));
            } else {
                if (sIsPrecompiledAsBlobs) {
                    shellArgs.add("--" + AOT_SNAPSHOT_PATH_KEY + "=" +
                        PathUtils.getDataDirectory(applicationContext));
                } else {
                    shellArgs.add("--cache-dir-path=" +
                        PathUtils.getCacheDirectory(applicationContext));

                    shellArgs.add("--" + AOT_SNAPSHOT_PATH_KEY + "=" +
                        PathUtils.getDataDirectory(applicationContext) + "/" + sFlutterAssetsDir);
                }
                shellArgs.add("--" + AOT_VM_SNAPSHOT_DATA_KEY + "=" + sAotVmSnapshotData);
                shellArgs.add("--" + AOT_VM_SNAPSHOT_INSTR_KEY + "=" + sAotVmSnapshotInstr);
                shellArgs.add("--" + AOT_ISOLATE_SNAPSHOT_DATA_KEY + "=" + sAotIsolateSnapshotData);
                shellArgs.add("--" + AOT_ISOLATE_SNAPSHOT_INSTR_KEY + "=" + sAotIsolateSnapshotInstr);
            }

            if (sSettings.getLogTag() != null) {
                shellArgs.add("--log-tag=" + sSettings.getLogTag());
            }

            String appBundlePath = findAppBundlePath(applicationContext);
            String appStoragePath = PathUtils.getFilesDir(applicationContext);
            String engineCachesPath = PathUtils.getCacheDirectory(applicationContext);
            nativeInit(applicationContext, shellArgs.toArray(new String[0]),
                appBundlePath, appStoragePath, engineCachesPath);

            sInitialized = true;
        } catch (Exception e) {
            Log.e(TAG, "Flutter initialization failed.", e);
            throw new RuntimeException(e);
        }
    }

    private static native void nativeInit(Context context, String[] args, String bundlePath, String appStoragePath, String engineCachesPath);
~~~
nativeInit方法涉及到JNI，在Flutter_main.cc中如下:
~~~cpp
bool FlutterMain::Register(JNIEnv* env) {
  static const JNINativeMethod methods[] = {
      {
          .name = "nativeInit",
          .signature = "(Landroid/content/Context;[Ljava/lang/String;Ljava/"
                       "lang/String;Ljava/lang/String;Ljava/lang/String;)V",
          .fnPtr = reinterpret_cast<void*>(&Init),
      },
      {
          .name = "nativeRecordStartTimestamp",
          .signature = "(J)V",
          .fnPtr = reinterpret_cast<void*>(&RecordStartTimestamp),
      },
  };

  jclass clazz = env->FindClass("io/flutter/view/FlutterMain");

  if (clazz == nullptr) {
    return false;
  }

  return env->RegisterNatives(clazz, methods, arraysize(methods)) == 0;
}
~~~
初始化完毕后创建[FlutterView](https://github.com/flutter/engine/blob/master/shell/platform/android/io/flutter/view/FlutterView.java)，**activity.setContentView(flutterView)**说明整个flutter的UI都在FlutterView内部了：
~~~Java
/**
 * An Android view containing a Flutter app.
 */
public class FlutterView extends SurfaceView implements BinaryMessenger, TextureRegistry {
   ...
   public FlutterView(Context context, AttributeSet attrs, FlutterNativeView nativeView) {
        super(context, attrs);

        Activity activity = (Activity) getContext();
        if (nativeView == null) {
            mNativeView = new FlutterNativeView(activity.getApplicationContext());
        } else {
            mNativeView = nativeView;
        }

        dartExecutor = mNativeView.getDartExecutor();
        flutterRenderer = new FlutterRenderer(mNativeView.getFlutterJNI());
        mIsSoftwareRenderingEnabled = FlutterJNI.nativeGetIsSoftwareRenderingEnabled();
        mMetrics = new ViewportMetrics();
        mMetrics.devicePixelRatio = context.getResources().getDisplayMetrics().density;
        setFocusable(true);
        setFocusableInTouchMode(true);

        mNativeView.attachViewAndActivity(this, activity);

        mSurfaceCallback = new SurfaceHolder.Callback() {
            @Override
            public void surfaceCreated(SurfaceHolder holder) {
                assertAttached();
                mNativeView.getFlutterJNI().onSurfaceCreated(holder.getSurface());
            }

            @Override
            public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {
                assertAttached();
                mNativeView.getFlutterJNI().onSurfaceChanged(width, height);
            }

            @Override
            public void surfaceDestroyed(SurfaceHolder holder) {
                assertAttached();
                mNativeView.getFlutterJNI().onSurfaceDestroyed();
            }
        };
        getHolder().addCallback(mSurfaceCallback);

        mActivityLifecycleListeners = new ArrayList<>();
        mFirstFrameListeners = new ArrayList<>();

        // Create all platform channels
        navigationChannel = new NavigationChannel(dartExecutor);
        keyEventChannel = new KeyEventChannel(dartExecutor);
        lifecycleChannel = new LifecycleChannel(dartExecutor);
        localizationChannel = new LocalizationChannel(dartExecutor);
        platformChannel = new PlatformChannel(dartExecutor);
        systemChannel = new SystemChannel(dartExecutor);
        settingsChannel = new SettingsChannel(dartExecutor);

        // Create and setup plugins
        PlatformPlugin platformPlugin = new PlatformPlugin(activity, platformChannel);
        addActivityLifecycleListener(platformPlugin);
        mImm = (InputMethodManager) getContext().getSystemService(Context.INPUT_METHOD_SERVICE);
        mTextInputPlugin = new TextInputPlugin(this, dartExecutor);
        androidKeyProcessor = new AndroidKeyProcessor(keyEventChannel, mTextInputPlugin);
        androidTouchProcessor = new AndroidTouchProcessor(flutterRenderer);

        // Send initial platform information to Dart
        sendLocalesToDart(getResources().getConfiguration());
        sendUserPlatformSettingsToDart();
    }

    @Override
    public boolean onKeyUp(int keyCode, KeyEvent event) {
        if (!isAttached()) {
            return super.onKeyUp(keyCode, event);
        }
        androidKeyProcessor.onKeyUp(event);
        return super.onKeyUp(keyCode, event);
    }

    @Override
    public boolean onKeyDown(int keyCode, KeyEvent event) {
        if (!isAttached()) {
            return super.onKeyDown(keyCode, event);
        }
        androidKeyProcessor.onKeyDown(event);
        return super.onKeyDown(keyCode, event);
    }
    ...
}
~~~
Android上FlutterView继承自SurfaceView，所有和Android相关的处理都在FlutterView中完成。
其中绑定了[NavigationChannel](https://github.com/flutter/engine/blob/master/shell/platform/android/io/flutter/embedding/engine/systemchannels/NavigationChannel.java)、[KeyEventChannel](https://github.com/flutter/engine/blob/master/shell/platform/android/io/flutter/embedding/engine/systemchannels/KeyEventChannel.java)、[LifecycleChannel]()、[LocalizationChannel]()、[PlatformChannel]()、[SystemChannel]()、[SettingsChannel]()等事件通道。
最关心的KeyEvent事件最终通过[AndroidKeyProcessor](https://github.com/flutter/engine/blob/master/shell/platform/android/io/flutter/embedding/android/AndroidKeyProcessor.java)转化为FlutterKeyEvent在[KeyEventChannel](https://github.com/flutter/engine/blob/master/shell/platform/android/io/flutter/embedding/engine/systemchannels/KeyEventChannel.java)中实现转发。
~~~Java
public class KeyEventChannel {
    @NonNull
    public final BasicMessageChannel<Object> channel;

    public KeyEventChannel(@NonNull DartExecutor dartExecutor) {
        this.channel = new BasicMessageChannel(dartExecutor, "flutter/keyevent", JSONMessageCodec.INSTANCE);
    }

  public void keyUp(@NonNull FlutterKeyEvent keyEvent) {
    Map<String, Object> message = new HashMap<>();
    message.put("type", "keyup");
    message.put("keymap", "android");
    encodeKeyEvent(keyEvent, message);

    channel.send(message);
  }

  public void keyDown(@NonNull FlutterKeyEvent keyEvent) {
    Map<String, Object> message = new HashMap<>();
    message.put("type", "keydown");
    message.put("keymap", "android");
    encodeKeyEvent(keyEvent, message);

    channel.send(message);
  }
  ...
}

public final class BasicMessageChannel<T> {
    private static final String TAG = "BasicMessageChannel#";
    private final BinaryMessenger messenger;
    private final String name;
    private final MessageCodec<T> codec;

    public BasicMessageChannel(BinaryMessenger messenger, String name, MessageCodec<T> codec) {
        assert messenger != null;

        assert name != null;

        assert codec != null;

        this.messenger = messenger;
        this.name = name;
        this.codec = codec;
    }

    public void send(T message) {
        this.send(message, (BasicMessageChannel.Reply)null);
    }

    public void send(T message, BasicMessageChannel.Reply<T> callback) {
        this.messenger.send(this.name, this.codec.encodeMessage(message), callback == null ? null : new BasicMessageChannel.IncomingReplyHandler(callback));
    }
    ...
}
~~~
可以看到事件转发的原理其实是通过[DartExecutor](https://github.com/flutter/engine/blob/master/shell/platform/android/io/flutter/embedding/engine/dart/DartExecutor.java)实现的:
~~~Java
public class DartExecutor implements BinaryMessenger {
    private static final String TAG = "DartExecutor";
    @NonNull
    private final FlutterJNI flutterJNI;
    @NonNull
    private final DartMessenger messenger;
    private boolean isApplicationRunning = false;

    public DartExecutor(@NonNull FlutterJNI flutterJNI) {
        this.flutterJNI = flutterJNI;
        this.messenger = new DartMessenger(flutterJNI);
    }

    public void onAttachedToJNI() {
        this.flutterJNI.setPlatformMessageHandler(this.messenger);
    }

    public void onDetachedFromJNI() {
        this.flutterJNI.setPlatformMessageHandler((PlatformMessageHandler)null);
    }

    public boolean isExecutingDart() {
        return this.isApplicationRunning;
    }
  
    ...

    public void send(@NonNull String channel, @Nullable ByteBuffer message) {
        this.messenger.send(channel, message, (BinaryReply)null);
    }

    public void send(@NonNull String channel, @Nullable ByteBuffer message, @Nullable BinaryReply callback) {
        this.messenger.send(channel, message, callback);
    }
    ...
}
~~~
实际转发到[DartMessenger](https://github.com/flutter/engine/blob/master/shell/platform/android/io/flutter/embedding/engine/dart/DartMessenger.java)中：
~~~Java
class DartMessenger implements BinaryMessenger, PlatformMessageHandler {
    private static final String TAG = "DartMessenger";
    @NonNull
    private final FlutterJNI flutterJNI;
    @NonNull
    private final Map<String, BinaryMessageHandler> messageHandlers;
    @NonNull
    private final Map<Integer, BinaryReply> pendingReplies;
    private int nextReplyId = 1;

    DartMessenger(@NonNull FlutterJNI flutterJNI) {
        this.flutterJNI = flutterJNI;
        this.messageHandlers = new HashMap();
        this.pendingReplies = new HashMap();
    }

    public void setMessageHandler(@NonNull String channel, @Nullable BinaryMessageHandler handler) {
        if (handler == null) {
            this.messageHandlers.remove(channel);
        } else {
            this.messageHandlers.put(channel, handler);
        }

    }

    public void send(@NonNull String channel, @NonNull ByteBuffer message) {
        this.send(channel, message, (BinaryReply)null);
    }

    public void send(@NonNull String channel, @Nullable ByteBuffer message, @Nullable BinaryReply callback) {
        int replyId = 0;
        if (callback != null) {
            replyId = this.nextReplyId++;
            this.pendingReplies.put(replyId, callback);
        }

        if (message == null) {
            this.flutterJNI.dispatchEmptyPlatformMessage(channel, replyId);
        } else {
            this.flutterJNI.dispatchPlatformMessage(channel, message, message.position(), replyId);
        }

    }
    ...
}
~~~
在[FlutterView](https://github.com/flutter/engine/blob/master/shell/platform/android/io/flutter/view/FlutterView.java)中创建[DartExecutor](https://github.com/flutter/engine/blob/master/shell/platform/android/io/flutter/embedding/engine/dart/DartExecutor.java)，在FlutterNativeView初始化时就创建了[FlutterJNI](https://github.com/flutter/engine/blob/master/shell/platform/android/io/flutter/embedding/engine/FlutterJNI.java)，[FlutterJNI](https://github.com/flutter/engine/blob/master/shell/platform/android/io/flutter/embedding/engine/FlutterJNI.java)就是Android与Dart的入口：
~~~Java
    @UiThread
    public void dispatchPlatformMessage(String channel, ByteBuffer message, int position, int responseId) {
        this.ensureAttachedToNative();
        this.nativeDispatchPlatformMessage(this.nativePlatformViewId, channel, message, position, responseId);
    }
~~~
[FlutterJNI](https://github.com/flutter/engine/blob/master/shell/platform/android/io/flutter/embedding/engine/FlutterJNI.java)对应的JNI代码位于[platform_view_android_jni.cc](https://github.com/flutter/engine/blob/master/shell/platform/android/platform_view_android_jni.cc)：
~~~cpp
#define ANDROID_SHELL_HOLDER \
  (reinterpret_cast<shell::AndroidShellHolder*>(shell_holder))
...
static void DispatchEmptyPlatformMessage(JNIEnv* env,
                                         jobject jcaller,
                                         jlong shell_holder,
                                         jstring channel,
                                         jint responseId) {
  ANDROID_SHELL_HOLDER->GetPlatformView()->DispatchEmptyPlatformMessage(
      env,                                         //
      fml::jni::JavaStringToString(env, channel),  //
      responseId                                   //
  );
}
static void DispatchPlatformMessage(JNIEnv* env,
                                    jobject jcaller,
                                    jlong shell_holder,
                                    jstring channel,
                                    jobject message,
                                    jint position,
                                    jint responseId) {
  ANDROID_SHELL_HOLDER->GetPlatformView()->DispatchPlatformMessage(
      env,                                         //
      fml::jni::JavaStringToString(env, channel),  //
      message,                                     //
      position,                                    //
      responseId                                   //
  );
}
...
bool RegisterApi(JNIEnv* env) {
  static const JNINativeMethod flutter_jni_methods[] = {
    ...
    {
          .name = "nativeDispatchEmptyPlatformMessage",
          .signature = "(JLjava/lang/String;I)V",
          .fnPtr =
              reinterpret_cast<void*>(&shell::DispatchEmptyPlatformMessage),
    },
    {
          .name = "nativeDispatchPlatformMessage",
          .signature = "(JLjava/lang/String;Ljava/nio/ByteBuffer;II)V",
          .fnPtr = reinterpret_cast<void*>(&shell::DispatchPlatformMessage),
    },
    ...
}

~~~
C++代码层调用至[**android_shell_holder**](https://github.com/flutter/engine/blob/master/shell/platform/android/android_shell_holder.cc)：
~~~cpp
namespace shell {

AndroidShellHolder::AndroidShellHolder(
    blink::Settings settings,
    fml::jni::JavaObjectWeakGlobalRef java_object,
    bool is_background_view)
    : settings_(std::move(settings)), java_object_(java_object) {
  static size_t shell_count = 1;
  auto thread_label = std::to_string(shell_count++);

  FML_CHECK(pthread_key_create(&thread_destruct_key_, ThreadDestructCallback) ==
            0);

  if (is_background_view) {
    thread_host_ = {thread_label, ThreadHost::Type::UI};
  } else {
    thread_host_ = {thread_label, ThreadHost::Type::UI | ThreadHost::Type::GPU |
                                      ThreadHost::Type::IO};
  }

  // Detach from JNI when the UI and GPU threads exit.
  auto jni_exit_task([key = thread_destruct_key_]() {
    FML_CHECK(pthread_setspecific(key, reinterpret_cast<void*>(1)) == 0);
  });
  thread_host_.ui_thread->GetTaskRunner()->PostTask(jni_exit_task);
  if (!is_background_view) {
    thread_host_.gpu_thread->GetTaskRunner()->PostTask(jni_exit_task);
  }

  fml::WeakPtr<PlatformViewAndroid> weak_platform_view;
  Shell::CreateCallback<PlatformView> on_create_platform_view =
      [is_background_view, java_object, &weak_platform_view](Shell& shell) {
        std::unique_ptr<PlatformViewAndroid> platform_view_android;
        if (is_background_view) {
          platform_view_android = std::make_unique<PlatformViewAndroid>(
              shell,                   // delegate
              shell.GetTaskRunners(),  // task runners
              java_object              // java object handle for JNI interop
          );

        } else {
          platform_view_android = std::make_unique<PlatformViewAndroid>(
              shell,                   // delegate
              shell.GetTaskRunners(),  // task runners
              java_object,             // java object handle for JNI interop
              shell.GetSettings()
                  .enable_software_rendering  // use software rendering
          );
        }
        weak_platform_view = platform_view_android->GetWeakPtr();
        return platform_view_android;
      };

  Shell::CreateCallback<Rasterizer> on_create_rasterizer = [](Shell& shell) {
    return std::make_unique<Rasterizer>(shell.GetTaskRunners());
  };

  // The current thread will be used as the platform thread. Ensure that the
  // message loop is initialized.
  fml::MessageLoop::EnsureInitializedForCurrentThread();
  fml::RefPtr<fml::TaskRunner> gpu_runner;
  fml::RefPtr<fml::TaskRunner> ui_runner;
  fml::RefPtr<fml::TaskRunner> io_runner;
  fml::RefPtr<fml::TaskRunner> platform_runner =
      fml::MessageLoop::GetCurrent().GetTaskRunner();
  if (is_background_view) {
    auto single_task_runner = thread_host_.ui_thread->GetTaskRunner();
    gpu_runner = single_task_runner;
    ui_runner = single_task_runner;
    io_runner = single_task_runner;
  } else {
    gpu_runner = thread_host_.gpu_thread->GetTaskRunner();
    ui_runner = thread_host_.ui_thread->GetTaskRunner();
    io_runner = thread_host_.io_thread->GetTaskRunner();
  }
  ...
fml::WeakPtr<PlatformViewAndroid> AndroidShellHolder::GetPlatformView() {
  FML_DCHECK(platform_view_);
  return platform_view_;
}
~~~
[PlatformViewAndroid](https://github.com/flutter/engine/blob/master/shell/platform/android/platform_view_android.cc)：
~~~cpp
namespace shell {

PlatformViewAndroid::PlatformViewAndroid(
    PlatformView::Delegate& delegate,
    blink::TaskRunners task_runners,
    fml::jni::JavaObjectWeakGlobalRef java_object,
    bool use_software_rendering)
    : PlatformView(delegate, std::move(task_runners)),
      java_object_(java_object),
      android_surface_(AndroidSurface::Create(use_software_rendering)) {
  FML_CHECK(android_surface_)
      << "Could not create an OpenGL, Vulkan or Software surface to setup "
         "rendering.";
}
PlatformViewAndroid::PlatformViewAndroid(
    PlatformView::Delegate& delegate,
    blink::TaskRunners task_runners,
    fml::jni::JavaObjectWeakGlobalRef java_object)
    : PlatformView(delegate, std::move(task_runners)),
      java_object_(java_object),
      android_surface_(nullptr) {}
...
void PlatformViewAndroid::DispatchPlatformMessage(JNIEnv* env,
                                                  std::string name,
                                                  jobject java_message_data,
                                                  jint java_message_position,
                                                  jint response_id) {
  uint8_t* message_data =
      static_cast<uint8_t*>(env->GetDirectBufferAddress(java_message_data));
  std::vector<uint8_t> message =
      std::vector<uint8_t>(message_data, message_data + java_message_position);

  fml::RefPtr<blink::PlatformMessageResponse> response;
  if (response_id) {
    response = fml::MakeRefCounted<PlatformMessageResponseAndroid>(
        response_id, java_object_, task_runners_.GetPlatformTaskRunner());
  }

  PlatformView::DispatchPlatformMessage(
      fml::MakeRefCounted<blink::PlatformMessage>(
          std::move(name), std::move(message), std::move(response)));
}

void PlatformViewAndroid::DispatchEmptyPlatformMessage(JNIEnv* env,
                                                       std::string name,
                                                       jint response_id) {
  fml::RefPtr<blink::PlatformMessageResponse> response;
  if (response_id) {
    response = fml::MakeRefCounted<PlatformMessageResponseAndroid>(
        response_id, java_object_, task_runners_.GetPlatformTaskRunner());
  }

  PlatformView::DispatchPlatformMessage(
      fml::MakeRefCounted<blink::PlatformMessage>(std::move(name),
                                                  std::move(response)));
}
...
}
~~~
抽象为平台无关[PlatformView](https://github.com/flutter/engine/blob/master/shell/common/platform_view.cc)：
~~~cpp
namespace shell {

PlatformView::PlatformView(Delegate& delegate, blink::TaskRunners task_runners)
    : delegate_(delegate),
      task_runners_(std::move(task_runners)),
      size_(SkISize::Make(0, 0)),
      weak_factory_(this) {}

PlatformView::~PlatformView() = default;
...
void PlatformView::DispatchPlatformMessage(
    fml::RefPtr<blink::PlatformMessage> message) {
  delegate_.OnPlatformViewDispatchPlatformMessage(std::move(message));
}
...
}
~~~
最终通过delegate_代理将事件发送给Dart VM，这里delegate_是Jni初始化时的[Shell](https://github.com/flutter/engine/blob/master/shell/common/shell.cc):
~~~cpp
// |PlatformView::Delegate|
void Shell::OnPlatformViewDispatchPlatformMessage(
    fml::RefPtr<PlatformMessage> message) {
  FML_DCHECK(is_setup_);
  FML_DCHECK(task_runners_.GetPlatformTaskRunner()->RunsTasksOnCurrentThread());

  task_runners_.GetUITaskRunner()->PostTask(
      [engine = engine_->GetWeakPtr(), message = std::move(message)] {
        if (engine) {
          engine->DispatchPlatformMessage(std::move(message));
        }
      });
}
~~~
到[Engine](https://github.com/flutter/engine/blob/master/shell/common/engine.cc)中实现与Dart VM交互：
~~~cpp
Engine::Engine(Delegate& delegate,
               blink::DartVM& vm,
               fml::RefPtr<blink::DartSnapshot> isolate_snapshot,
               fml::RefPtr<blink::DartSnapshot> shared_snapshot,
               blink::TaskRunners task_runners,
               blink::Settings settings,
               std::unique_ptr<Animator> animator,
               fml::WeakPtr<blink::SnapshotDelegate> snapshot_delegate,
               fml::WeakPtr<blink::IOManager> io_manager)
    : delegate_(delegate),
      settings_(std::move(settings)),
      animator_(std::move(animator)),
      activity_running_(false),
      have_surface_(false),
      weak_factory_(this) {
  // Runtime controller is initialized here because it takes a reference to this
  // object as its delegate. The delegate may be called in the constructor and
  // we want to be fully initilazed by that point.
  runtime_controller_ = std::make_unique<blink::RuntimeController>(
      *this,                                 // runtime delegate
      &vm,                                   // VM
      std::move(isolate_snapshot),           // isolate snapshot
      std::move(shared_snapshot),            // shared snapshot
      std::move(task_runners),               // task runners
      std::move(snapshot_delegate),          // snapshot delegate
      std::move(io_manager),                 // io manager
      settings_.advisory_script_uri,         // advisory script uri
      settings_.advisory_script_entrypoint,  // advisory script entrypoint
      settings_.idle_notification_callback   // idle notification callback
  );
}
...
void Engine::DispatchPlatformMessage(
    fml::RefPtr<blink::PlatformMessage> message) {
  if (message->channel() == kLifecycleChannel) {
    if (HandleLifecyclePlatformMessage(message.get()))
      return;
  } else if (message->channel() == kLocalizationChannel) {
    if (HandleLocalizationPlatformMessage(message.get()))
      return;
  } else if (message->channel() == kSettingsChannel) {
    HandleSettingsPlatformMessage(message.get());
    return;
  }

  if (runtime_controller_->IsRootIsolateRunning() &&
      runtime_controller_->DispatchPlatformMessage(std::move(message))) {
    return;
  }

  // If there's no runtime_, we may still need to set the initial route.
  if (message->channel() == kNavigationChannel)
    HandleNavigationPlatformMessage(std::move(message));
}
~~~
[Engine](https://github.com/flutter/engine/blob/master/shell/common/engine.cc)已经和DartVM紧密相关了。在[runtime](https://github.com/flutter/engine/tree/master/runtime)内部通过[runtime_controller](https://github.com/flutter/engine/blob/master/runtime/runtime_controller.cc)转发事件：
~~~cpp
bool RuntimeController::DispatchPlatformMessage(
    fml::RefPtr<PlatformMessage> message) {
  if (auto* window = GetWindowIfAvailable()) {
    TRACE_EVENT1("flutter", "RuntimeController::DispatchPlatformMessage",
                 "mode", "basic");
    window->DispatchPlatformMessage(std::move(message));
    return true;
  }
  return false;
}
~~~
[Window](https://github.com/flutter/engine/blob/master/lib/ui/window/window.cc)：
~~~cpp
void Window::DispatchPlatformMessage(fml::RefPtr<PlatformMessage> message) {
  std::shared_ptr<tonic::DartState> dart_state = library_.dart_state().lock();
  if (!dart_state)
    return;
  tonic::DartState::Scope scope(dart_state);
  Dart_Handle data_handle =
      (message->hasData()) ? ToByteData(message->data()) : Dart_Null();
  if (Dart_IsError(data_handle))
    return;

  int response_id = 0;
  if (auto response = message->response()) {
    response_id = next_response_id_++;
    pending_responses_[response_id] = response;
  }

  tonic::LogIfError(
      tonic::DartInvokeField(library_.value(), "_dispatchPlatformMessage",
                             {tonic::ToDart(message->channel()), data_handle,
                              tonic::ToDart(response_id)}));
}
~~~

![](https://upload-images.jianshu.io/upload_images/3613947-3a25dc5fe0c4a82b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在[runtime_controller](https://github.com/flutter/engine/blob/master/runtime/runtime_controller.cc)中，[Flutter引擎通过tonic实现了C++和dart接口的相互调用。](https://zhuanlan.zhihu.com/p/48858491)
DartInvokeField会执行Dart_Invoke方法，[hooks](https://github.com/flutter/engine/blob/master/lib/ui/hooks.dart)已经可以把Engine和Dart Framework联系起来：
~~~dart
@pragma('vm:entry-point')
void _dispatchPlatformMessage(String name, ByteData data, int responseId) {
  if (window.onPlatformMessage != null) {
    _invoke3<String, ByteData, PlatformMessageResponseCallback>(
      window.onPlatformMessage,
      window._onPlatformMessageZone,
      name,
      data,
      (ByteData responseData) {
        window._respondToPlatformMessage(responseId, responseData);
      },
    );
  } else {
    window._respondToPlatformMessage(responseId, null);
  }
}

/// Invokes [callback] inside the given [zone] passing it [arg1], [arg2] and [arg3].
void _invoke3<A1, A2, A3>(void callback(A1 a1, A2 a2, A3 a3), Zone zone, A1 arg1, A2 arg2, A3 arg3) {
  if (callback == null)
    return;

  assert(zone != null);

  if (identical(zone, Zone.current)) {
    callback(arg1, arg2, arg3);
  } else {
    zone.runGuarded(() {
      callback(arg1, arg2, arg3);
    });
  }
}
~~~
Dart Framework层[window](https://github.com/flutter/engine/blob/master/lib/ui/window.dart)处理:
~~~dart
  /// Called whenever this window receives a message from a platform-specific
  /// plugin.
  ///
  /// The `name` parameter determines which plugin sent the message. The `data`
  /// parameter is the payload and is typically UTF-8 encoded JSON but can be
  /// arbitrary data.
  ///
  /// Message handlers must call the function given in the `callback` parameter.
  /// If the handler does not need to respond, the handler should pass null to
  /// the callback.
  ///
  /// The framework invokes this callback in the same zone in which the
  /// callback was set.
  PlatformMessageCallback get onPlatformMessage => _onPlatformMessage;
  PlatformMessageCallback _onPlatformMessage;
  Zone _onPlatformMessageZone;
  set onPlatformMessage(PlatformMessageCallback callback) {
    _onPlatformMessage = callback;
    _onPlatformMessageZone = Zone.current;
  }
~~~
在[binding](https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/services/binding.dart)时注册了回调接口：
~~~dart
mixin ServicesBinding on BindingBase {
  @override
  void initInstances() {
    super.initInstances();
    _instance = this;
    window
      ..onPlatformMessage = BinaryMessages.handlePlatformMessage;
    initLicenses();
  }
~~~
[BinaryMessages](https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/services/platform_messages.dart)消息分发如下：
~~~dart
  /// Calls the handler registered for the given channel.
  ///
  /// Typically called by [ServicesBinding] to handle platform messages received
  /// from [Window.onPlatformMessage].
  ///
  /// To register a handler for a given message channel, see [setMessageHandler].
  static Future<void> handlePlatformMessage(
        String channel, ByteData data, ui.PlatformMessageResponseCallback callback) async {
    ByteData response;
    try {
      final _MessageHandler handler = _handlers[channel];
      if (handler != null)
        response = await handler(data);
    } catch (exception, stack) {
      FlutterError.reportError(FlutterErrorDetails(
        exception: exception,
        stack: stack,
        library: 'services library',
        context: 'during a platform message callback',
      ));
    } finally {
      callback(response);
    }
  }
~~~
到这里看到通过句柄channel获取到对应的_MessageHandler，最终消息得到处理：
~~~dart
 /// Set a callback for receiving messages from the platform plugins on the
  /// given channel, without decoding them.
  ///
  /// The given callback will replace the currently registered callback for that
  /// channel, if any. To remove the handler, pass null as the `handler`
  /// argument.
  ///
  /// The handler's return value, if non-null, is sent as a response, unencoded.
  static void setMessageHandler(String channel, Future<ByteData> handler(ByteData message)) {
    if (handler == null)
      _handlers.remove(channel);
    else
      _handlers[channel] = handler;
  }
~~~
看看[KeyEvent](https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/services/raw_keyboard.dart)事件的注册:
~~~dart
class RawKeyboard {
  RawKeyboard._() {
    SystemChannels.keyEvent.setMessageHandler(_handleKeyEvent);
  }

  /// The shared instance of [RawKeyboard].
  static final RawKeyboard instance = RawKeyboard._();

  final List<ValueChanged<RawKeyEvent>> _listeners = <ValueChanged<RawKeyEvent>>[];

  /// Calls the listener every time the user presses or releases a key.
  ///
  /// Listeners can be removed with [removeListener].
  void addListener(ValueChanged<RawKeyEvent> listener) {
    _listeners.add(listener);
  }

  /// Stop calling the listener every time the user presses or releases a key.
  ///
  /// Listeners can be added with [addListener].
  void removeListener(ValueChanged<RawKeyEvent> listener) {
    _listeners.remove(listener);
  }

  Future<dynamic> _handleKeyEvent(dynamic message) async {
    final RawKeyEvent event = RawKeyEvent.fromMessage(message);
    if (event == null) {
      return;
    }
    if (event is RawKeyDownEvent) {
      _keysPressed.add(event.logicalKey);
    }
    if (event is RawKeyUpEvent) {
      _keysPressed.remove(event.logicalKey);
    }
    if (_listeners.isEmpty) {
      return;
    }
    for (ValueChanged<RawKeyEvent> listener in List<ValueChanged<RawKeyEvent>>.from(_listeners)) {
      if (_listeners.contains(listener)) {
        listener(event);
      }
    }
  }

  final Set<LogicalKeyboardKey> _keysPressed = Set<LogicalKeyboardKey>();

  /// Returns the set of keys currently pressed.
  Set<LogicalKeyboardKey> get keysPressed {
    return _keysPressed.toSet();
  }
}
~~~
[SystemChannels](https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/services/system_channels.dart):
~~~dart
  static const BasicMessageChannel<dynamic> keyEvent = BasicMessageChannel<dynamic>(
      'flutter/keyevent',
      JSONMessageCodec(),
  );
~~~
其中RawKeyEvent根据平台不同分别区分为[RawKeyEventDataAndroid](https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/services/raw_keyboard_android.dart),[RawKeyEventDataFuchsia](https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/services/raw_keyboard_fuchsia.dart)，[RawKeyEventDataMacOs](https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/services/raw_keyboard_macos.dart),[RawKeyEventDataLinux](https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/services/raw_keyboard_linux.dart)。


只有在注册了listener才能真正的完成事件分发，Flutter Framework中实现了一个[RawKeyboardListener](https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/widgets/raw_keyboard_listener.dart)，可以顺便了解下[StatefulWidget](https://docs.flutter.io/flutter/widgets/StatefulWidget-class.html)和[StatelessWidget](https://docs.flutter.io/flutter/widgets/StatelessWidget-class.html)：

![Widget](https://upload-images.jianshu.io/upload_images/3613947-284ede325da8aefb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

~~~dart
class RawKeyboardListener extends StatefulWidget {
  /// Creates a widget that receives raw keyboard events.
  ///
  /// For text entry, consider using a [EditableText], which integrates with
  /// on-screen keyboards and input method editors (IMEs).
  const RawKeyboardListener({
    Key key,
    @required this.focusNode,
    @required this.onKey,
    @required this.child,
  }) : assert(focusNode != null),
       assert(child != null),
       super(key: key);

  /// Controls whether this widget has keyboard focus.
  final FocusNode focusNode;

  /// Called whenever this widget receives a raw keyboard event.
  final ValueChanged<RawKeyEvent> onKey;

  /// The widget below this widget in the tree.
  ///
  /// {@macro flutter.widgets.child}
  final Widget child;

  @override
  _RawKeyboardListenerState createState() => _RawKeyboardListenerState();

  @override
  void debugFillProperties(DiagnosticPropertiesBuilder properties) {
    super.debugFillProperties(properties);
    properties.add(DiagnosticsProperty<FocusNode>('focusNode', focusNode));
  }
}
~~~
按键事件通常也需要和Focus关联起来，一般都是在Widget获取焦点后才分发需要处理的按键事件:
~~~dart
  void _handleFocusChanged() {
    if (widget.focusNode.hasFocus)
      _attachKeyboardIfDetached();
    else
      _detachKeyboardIfAttached();
  }

  bool _listening = false;

  void _attachKeyboardIfDetached() {
    if (_listening)
      return;
    RawKeyboard.instance.addListener(_handleRawKeyEvent);
    _listening = true;
  }

  void _detachKeyboardIfAttached() {
    if (!_listening)
      return;
    RawKeyboard.instance.removeListener(_handleRawKeyEvent);
    _listening = false;
  }

  void _handleRawKeyEvent(RawKeyEvent event) {
    if (widget.onKey != null)
      widget.onKey(event);
  }
~~~
这里的焦点事件都是通过[FocusNode](https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/widgets/focus_manager.dart)来管理：
~~~dart
class FocusNode extends ChangeNotifier {
  FocusScopeNode _parent;
  FocusManager _manager;
  bool _hasKeyboardToken = false;

  /// Whether this node has the overall focus.
  ///
  /// A [FocusNode] has the overall focus when the node is focused in its
  /// parent [FocusScopeNode] and [FocusScopeNode.isFirstFocus] is true for
  /// that scope and all its ancestor scopes.
  ///
  /// To request focus, find the [FocusScopeNode] for the current [BuildContext]
  /// and call the [FocusScopeNode.requestFocus] method:
  ///
  /// ```dart
  /// FocusScope.of(context).requestFocus(focusNode);
  /// ```
  ///
  /// This object notifies its listeners whenever this value changes.
  bool get hasFocus => _manager?._currentFocus == this;

  /// Removes the keyboard token from this focus node if it has one.
  ///
  /// This mechanism helps distinguish between an input control gaining focus by
  /// default and gaining focus as a result of an explicit user action.
  ///
  /// When a focus node requests the focus (either via
  /// [FocusScopeNode.requestFocus] or [FocusScopeNode.autofocus]), the focus
  /// node receives a keyboard token if it does not already have one. Later,
  /// when the focus node becomes focused, the widget that manages the
  /// [TextInputConnection] should show the keyboard (i.e., call
  /// [TextInputConnection.show]) only if it successfully consumes the keyboard
  /// token from the focus node.
  ///
  /// Returns whether this function successfully consumes a keyboard token.
  bool consumeKeyboardToken() {
    if (!_hasKeyboardToken)
      return false;
    _hasKeyboardToken = false;
    return true;
  }

  /// Cancels any outstanding requests for focus.
  ///
  /// This method is safe to call regardless of whether this node has ever
  /// requested focus.
  void unfocus() {
    _parent?._resignFocus(this);
    assert(_parent == null);
    assert(_manager == null);
  }

  @override
  void dispose() {
    _manager?._willDisposeFocusNode(this);
    _parent?._resignFocus(this);
    assert(_parent == null);
    assert(_manager == null);
    super.dispose();
  }

  void _notify() {
    notifyListeners();
  }

  @override
  String toString() => '${describeIdentity(this)}${hasFocus ? '(FOCUSED)' : ''}';
}
~~~
[ChangeNotifier](https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/foundation/change_notifier.dart)：
~~~dart
class ChangeNotifier implements Listenable {
  ObserverList<VoidCallback> _listeners = ObserverList<VoidCallback>();

  bool _debugAssertNotDisposed() {
    assert(() {
      if (_listeners == null) {
        throw FlutterError(
          'A $runtimeType was used after being disposed.\n'
          'Once you have called dispose() on a $runtimeType, it can no longer be used.'
        );
      }
      return true;
    }());
    return true;
  }

  /// Whether any listeners are currently registered.
  ///
  /// Clients should not depend on this value for their behavior, because having
  /// one listener's logic change when another listener happens to start or stop
  /// listening will lead to extremely hard-to-track bugs. Subclasses might use
  /// this information to determine whether to do any work when there are no
  /// listeners, however; for example, resuming a [Stream] when a listener is
  /// added and pausing it when a listener is removed.
  ///
  /// Typically this is used by overriding [addListener], checking if
  /// [hasListeners] is false before calling `super.addListener()`, and if so,
  /// starting whatever work is needed to determine when to call
  /// [notifyListeners]; and similarly, by overriding [removeListener], checking
  /// if [hasListeners] is false after calling `super.removeListener()`, and if
  /// so, stopping that same work.
  @protected
  bool get hasListeners {
    assert(_debugAssertNotDisposed());
    return _listeners.isNotEmpty;
  }

  /// Register a closure to be called when the object changes.
  ///
  /// This method must not be called after [dispose] has been called.
  @override
  void addListener(VoidCallback listener) {
    assert(_debugAssertNotDisposed());
    _listeners.add(listener);
  }

  /// Remove a previously registered closure from the list of closures that are
  /// notified when the object changes.
  ///
  /// If the given listener is not registered, the call is ignored.
  ///
  /// This method must not be called after [dispose] has been called.
  ///
  /// If a listener had been added twice, and is removed once during an
  /// iteration (i.e. in response to a notification), it will still be called
  /// again. If, on the other hand, it is removed as many times as it was
  /// registered, then it will no longer be called. This odd behavior is the
  /// result of the [ChangeNotifier] not being able to determine which listener
  /// is being removed, since they are identical, and therefore conservatively
  /// still calling all the listeners when it knows that any are still
  /// registered.
  ///
  /// This surprising behavior can be unexpectedly observed when registering a
  /// listener on two separate objects which are both forwarding all
  /// registrations to a common upstream object.
  @override
  void removeListener(VoidCallback listener) {
    assert(_debugAssertNotDisposed());
    _listeners.remove(listener);
  }

  /// Discards any resources used by the object. After this is called, the
  /// object is not in a usable state and should be discarded (calls to
  /// [addListener] and [removeListener] will throw after the object is
  /// disposed).
  ///
  /// This method should only be called by the object's owner.
  @mustCallSuper
  void dispose() {
    assert(_debugAssertNotDisposed());
    _listeners = null;
  }

  /// Call all the registered listeners.
  ///
  /// Call this method whenever the object changes, to notify any clients the
  /// object may have. Listeners that are added during this iteration will not
  /// be visited. Listeners that are removed during this iteration will not be
  /// visited after they are removed.
  ///
  /// Exceptions thrown by listeners will be caught and reported using
  /// [FlutterError.reportError].
  ///
  /// This method must not be called after [dispose] has been called.
  ///
  /// Surprising behavior can result when reentrantly removing a listener (i.e.
  /// in response to a notification) that has been registered multiple times.
  /// See the discussion at [removeListener].
  @protected
  @visibleForTesting
  void notifyListeners() {
    assert(_debugAssertNotDisposed());
    if (_listeners != null) {
      final List<VoidCallback> localListeners = List<VoidCallback>.from(_listeners);
      for (VoidCallback listener in localListeners) {
        try {
          if (_listeners.contains(listener))
            listener();
        } catch (exception, stack) {
          FlutterError.reportError(FlutterErrorDetails(
            exception: exception,
            stack: stack,
            library: 'foundation library',
            context: 'while dispatching notifications for $runtimeType',
            informationCollector: (StringBuffer information) {
              information.writeln('The $runtimeType sending notification was:');
              information.write('  $this');
            }
          ));
        }
      }
    }
  }
}
~~~
焦点事件的触发也可以按以上思路分析。

总体来看，按键事件是直接绑定到已注册的Widget视图上，在TV端使用Flutter实现视图之间的焦点移动特效，没有预料中那么顺畅，Flutter的Widget并不像Android的framework可以完成事件分发。

为解决这个问题目前的两个思路：

1.每个视图都绑定RawKeyEventListener

   优点：符合Flutter编码方式，可以快速完成Flutter代码编写，适用于Flutter SDK的迭代更新
   
   缺点：需要提前预置上下左右焦点的节点，代码混乱且冗余
   
参考代码:https://github.com/WoYang/tv_demo

实现效果：

效果预览:https://upload-images.jianshu.io/upload_images/3613947-f20df9cc84e9f793.gif?imageMogr2/auto-orient/strip

2.抽象一个Root Widget作为父视图，参考Android View事件分发机制实现

优点：框架比较清晰，焦点逻辑的抽离利于项目部署
    
缺点：需要重写大部分的Widget，内容较多，Flutter SDK的更新可能导致兼容性问题

###结束语

今天对Flutter在Android上的构成作了一个大致的分析，主要针对Flutter在TV端的使用作了一次尝试，也抛出了一些将要面临的问题，当然在实际使用Flutter开发时还会面临更多问题。没有一个标准去衡量Flutter的好坏，需要大家一起集思广益，多思考一些可以使用的场景，这次分享也是和大家一起了解一下Flutter的面貌，抛砖引玉，希望能一起在Flutter上做更多的事情。
