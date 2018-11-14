[nDraw] Bugfix: TCE underrun when toggle nNote tool icon 
nDraw hover out 1 sec will turn off EPD power but sometimes epd screen is still updating. That will be panic

[ndraw sync] ndraw stroke disappear when toast dismiss

nDraw sync
Need to use View invalidate to sync nDraw and app

GLConsumer::updateAndReleaseLocked

opengl draw
===========

eglGetDisplay frameworks/native/opengl/libagl/egl.cpp
egl_display_t::get_display frameworks/native/opengl/libagl/egl.cpp
egl_display_t gDisplays[NUM_DISPLAYS]

eglInitialize frameworks/native/opengl/libagl/egl.cpp


EGLNativeWindowType
selectConfigForNativeWindow
EGLUtils::selectConfigForNativeWindow development/egl_draw/include/EGLUtils.h
EGLUtils::selectConfigForPixelFormat development/egl_draw/include/EGLUtils.h

eglGetConfigs frameworks/native/opengl/libagl/egl.cpp




glTexImage2D frameworks/native/opengl/libagl/texture.cpp
```cpp
void glTexImage2D(
        GLenum target, GLint level, GLint internalformat,
        GLsizei width, GLsizei height, GLint border,
        GLenum format, GLenum type, const GLvoid *pixels)
{
...

    if (pixels) {
        const int32_t formatIdx = convertGLPixelFormat(format, type);
        const GGLFormat& pixelFormat(c->rasterizer.formats[formatIdx]);
        const int32_t align = c->textures.unpackAlignment-1;
        const int32_t bpr = ((width * pixelFormat.size) + align) & ~align;
        const size_t size = bpr * height;
        const int32_t stride = bpr / pixelFormat.size;

        GGLSurface userSurface;
        userSurface.version = sizeof(userSurface);
        userSurface.width  = width;
        userSurface.height = height;
        userSurface.stride = stride;
        userSurface.format = formatIdx;
        userSurface.compressedFormat = 0;
        userSurface.data = (GLubyte*)pixels;

        int err = copyPixels(c, *surface, 0, 0, userSurface, 0, 0, width, height);
        if (err) {
            ogles_error(c, err);
            return;
        }
        generateMipmap(c, level);
    }
}
```

glDrawTexiOES frameworks/native/opengl/libagl/texture.cpp
Wcr Hcr ?

createTextureSurface frameworks/native/opengl/libagl/texture.cpp
sp<EGLTextureObject> tex
getAndBindActiveTextureObject frameworks/native/opengl/libagl/texture.cpp
ogles_context_t* c
  
userSurface

glDrawArrays frameworks/native/opengl/libagl/array.cpp


opengl draw to FB
========
```
trap.cpp:767                  trianglex_big
trap.cpp:375                  linex
trap.cpp:477                  trianglex_debug
primitives.cpp:632            triangle
array.cpp:682                 drawPrimitivesTriangleFanOrStrip
array.cpp:1374                glDrawArrays
GLES11RenderEngine.cpp:233    android::GLES11RenderEngine::drawMesh(android::Mesh const&)
Layer.cpp:612                 android::Layer::drawWithOpenGL(android::sp<android::DisplayDevice const> const&, android::Region const&) const
Layer.cpp:555                 android::Layer::onDraw(android::sp<android::DisplayDevice const> const&, android::Region const&) const
Layer.cpp:457                 android::Layer::draw(android::sp<android::DisplayDevice const> const&, android::Region const&) const
SurfaceFlinger.cpp:2125       android::SurfaceFlinger::doComposeSurfaces(android::sp<android::DisplayDevice const> const&, android::Region const&)
SurfaceFlinger.cpp:2005       android::SurfaceFlinger::doDisplayComposition(android::sp<android::DisplayDevice const> const&, android::Region const&)
SurfaceFlinger.cpp:1224       android::SurfaceFlinger::doComposition()
SurfaceFlinger.cpp:966        android::SurfaceFlinger::handleMessageRefresh()
SurfaceFlinger.cpp:942        android::SurfaceFlinger::onMessageReceived(int)
Looper.cpp:302                android::Looper::pollInner(int)
Looper.cpp:191                android::Looper::pollOnce(int, int*, int*, void**)
Looper.h:176                  android::Looper::pollOnce(int)
SurfaceFlinger.cpp:834 (discriminator 1)android::SurfaceFlinger::run()
main_surfaceflinger.cpp:55    main
libc_init_dynamic.cpp:112     __libc_init
```

GLConsumer::bindTextureImageLocked()
void glEGLImageTargetTexture2DOES(GLenum target, GLeglImageOES image) frameworks/native/opengl/libagl/texture.cpp

mSurfaceFlingerConsumer->updateTexImage(&r)








analysis Toast


```
D/Choreographer( 3560): xxxx
D/Choreographer( 3560): java.lang.Throwable
D/Choreographer( 3560): 	at android.view.Choreographer.postCallbackDelayedInternal(Choreographer.java:321)
D/Choreographer( 3560): 	at android.view.Choreographer.postCallbackDelayed(Choreographer.java:315)
D/Choreographer( 3560): 	at android.view.Choreographer.postCallback(Choreographer.java:289)
D/Choreographer( 3560): 	at android.view.ViewRootImpl.scheduleTraversals(ViewRootImpl.java:1046)
D/Choreographer( 3560): 	at android.view.ViewRootImpl.requestLayout(ViewRootImpl.java:831)
D/Choreographer( 3560): 	at android.view.ViewRootImpl.setView(ViewRootImpl.java:498)
D/Choreographer( 3560): 	at android.view.WindowManagerGlobal.addView(WindowManagerGlobal.java:259)
D/Choreographer( 3560): 	at android.view.WindowManagerImpl.addView(WindowManagerImpl.java:69)
D/Choreographer( 3560): 	at android.widget.Toast$TN.handleShow(Toast.java:405)
D/Choreographer( 3560): 	at android.widget.Toast$TN$1.run(Toast.java:313)
D/Choreographer( 3560): 	at android.os.Handler.handleCallback(Handler.java:733)
D/Choreographer( 3560): 	at android.os.Handler.dispatchMessage(Handler.java:95)
D/Choreographer( 3560): 	at android.os.Looper.loop(Looper.java:137)
D/Choreographer( 3560): 	at android.app.ActivityThread.main(ActivityThread.java:5017)
D/Choreographer( 3560): 	at java.lang.reflect.Method.invokeNative(Native Method)
D/Choreographer( 3560): 	at java.lang.reflect.Method.invoke(Method.java:515)
D/Choreographer( 3560): 	at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:779)
D/Choreographer( 3560): 	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:595)
D/Choreographer( 3560): 	at dalvik.system.NativeStart.main(Native Method)
```


check FrameBufferSurface and FB relation

FrameBufferSurface 
============

FramebufferSurface frameworks/native/services/surfaceflinger/DisplayHardware/FramebufferSurface.cpp

```
    // initialize our non-virtual displays
    for (size_t i=0 ; i<DisplayDevice::NUM_BUILTIN_DISPLAY_TYPES ; i++) {
        DisplayDevice::DisplayType type((DisplayDevice::DisplayType)i);
        // set-up the displays that are already connected
        if (mHwc->isConnected(i) || type==DisplayDevice::DISPLAY_PRIMARY) {
            // All non-virtual displays are currently considered secure.
            bool isSecure = true;
            createBuiltinDisplayLocked(type);
            wp<IBinder> token = mBuiltinDisplays[i];

            sp<BufferQueue> bq = new BufferQueue(new GraphicBufferAlloc());
            sp<FramebufferSurface> fbs = new FramebufferSurface(*mHwc, i, bq);
            sp<DisplayDevice> hw = new DisplayDevice(this,
                    type, allocateHwcDisplayId(type), isSecure, token,
                    fbs, bq,
                    mEGLConfig);
            if (i > DisplayDevice::DISPLAY_PRIMARY) {
                // FIXME: currently we don't get blank/unblank requests
                // for displays other than the main display, so we always
                // assume a connected display is unblanked.
                ALOGD("marking display %d as acquired/unblanked", i);
                hw->acquireScreen();
            }
            mDisplays.add(token, hw);
        }
    }

```
set FrameBufferSurface to DisplayDevice

```
BufferQueue.cpp:237           android::BufferQueue::requestBuffer(int, android::sp<android::GraphicBuffer>*)
Surface.cpp:206               android::Surface::dequeueBuffer(ANativeWindowBuffer**, int*)
Surface.cpp:99                android::Surface::hook_dequeueBuffer(ANativeWindow*, ANativeWindowBuffer**, int*)
egl.cpp:388                   android::egl_window_surface_v2_t::connect()
egl.cpp:1853                  eglMakeCurrent
egl_display.cpp:336           android::egl_display_t::makeCurrent(android::egl_context_t*, android::egl_context_t*, void*, void*, void*, void*, void*, void*)
eglApi.cpp:763                eglMakeCurrent
DisplayDevice.cpp:291         android::DisplayDevice::makeCurrent(void*, void*) const
SurfaceFlinger.cpp:641        android::SurfaceFlinger::init()
main_surfaceflinger.cpp:48    main
libc_init_dynamic.cpp:112     __libc_init
```
getDefaultDisplayDevice()->makeCurrent(mEGLDisplay, mEGLContext);

DisplayDevice::DisplayDevice
const sp<IGraphicBufferProducer>& producer,

gralloc module
------------
location: hardware/imx/mx6/libgralloc_wrapper/gralloc.cpp
```
struct private_module_t HAL_MODULE_INFO_SYM = {
    base: {
        common: {
            tag: HARDWARE_MODULE_TAG,
            version_major: 1,
            version_minor: 0,
            id: GRALLOC_HARDWARE_MODULE_ID,
            name: "Graphics Memory Allocator Module",
            author: "The Android Open Source Project",
            methods: &gralloc_module_methods,
            dso: NULL,
            reserved: {0}
        },
        registerBuffer: gralloc_register_buffer,
        unregisterBuffer: gralloc_unregister_buffer,
        lock: gralloc_lock,
        unlock: gralloc_unlock,
        perform: 0,
        lock_ycbcr: 0,
        reserved_proc: {0}
    },
    framebuffer: 0,
    numBuffers: 0,
    bufferMask: 0,
    lock: PTHREAD_MUTEX_INITIALIZER,
    currentBuffer: 0,
    info: {0},
    finfo: {{0},0},
    xdpi: 0.0,
    ydpi: 0.0,
    fps: 0.0,
    flags: 0,
    master_phys: 0,
    gpu_device: 0,
    gralloc_viv: 0,
    priv_dev: 0,
    external_module: 0,
    primary_fd: 0,
    closeDevice: false,
    ion_fd: -1
};

```
```
    if (!m->priv_dev) {
        gralloc_context_t *dev;
        dev = (gralloc_context_t*)malloc(sizeof(*dev));

        /* initialize our state here */
        memset(dev, 0, sizeof(*dev));

        /* initialize the procs */
        dev->device.common.tag = HARDWARE_DEVICE_TAG;
        dev->device.common.version = 0;
        dev->device.common.module = const_cast<hw_module_t*>(module);
        dev->device.common.close = gralloc_close;

        dev->device.alloc   = gralloc_alloc;
        dev->device.free    = gralloc_free;

        gralloc_context_t* ext;
        ext = (gralloc_context_t*)malloc(sizeof(*ext));
        memset(ext, 0, sizeof(*ext));
        memcpy(ext, dev, sizeof(*ext));
        dev->ext_dev = (alloc_device_t*)ext;

        m->priv_dev = (alloc_device_t*)dev;
        m->closeDevice = false;
    }
```

gralloc alloc
-----

gralloc framebuffer or buffer, it depend on usage

```c
int fsl_gralloc_alloc(alloc_device_t* dev,
        int w, int h, int format, int usage,
        buffer_handle_t* pHandle, int* pStride)
{
    ...

    private_handle_t* hnd = NULL;
    if (usage & GRALLOC_USAGE_HW_FBX) {
        gralloc_context_t *ctx = (gralloc_context_t *)dev;
        if (ctx->ext_dev == NULL) {
            ALOGE("ctx->ext_dev == NULL");
            return -EINVAL;
        }

        err = fsl_gralloc_alloc_framebuffer(ctx->ext_dev, size, usage, (buffer_handle_t*)&hnd);
    }
    else if (usage & GRALLOC_USAGE_HW_FB) {
        ALOGD("[%s %d] alloc framebuffer",__func__, __LINE__);
        err = fsl_gralloc_alloc_framebuffer(dev, size, usage, (buffer_handle_t*)&hnd);
    }
    else {
        ALOGD("[%s %d] alloc buffer",__func__, __LINE__);
        err = fsl_gralloc_alloc_buffer(dev, size, usage, (buffer_handle_t*)&hnd);
    }

    ...
}
```

FrameBufferSurface producer enqueue
---------

```
BufferQueue.cpp:482           android::BufferQueue::queueBuffer(int, android::IGraphicBufferProducer::QueueBufferInput const&, android::IGraphicBufferProducer::QueueBufferOutput*)
Surface.cpp:291               android::Surface::queueBuffer(ANativeWindowBuffer*, int)
Surface.cpp:111               android::Surface::hook_queueBuffer(ANativeWindow*, ANativeWindowBuffer*, int)
egl.cpp:537                   android::egl_window_surface_v2_t::swapBuffers()
egl.cpp:1962                  eglSwapBuffers
eglApi.cpp:1108               eglSwapBuffers
DisplayDevice.cpp:255         android::DisplayDevice::swapBuffers(android::HWComposer&) const
SurfaceFlinger.cpp:2017       android::SurfaceFlinger::doDisplayComposition(android::sp<android::DisplayDevice const> const&, android::Region const&)
SurfaceFlinger.cpp:1224       android::SurfaceFlinger::doComposition()
SurfaceFlinger.cpp:966        android::SurfaceFlinger::handleMessageRefresh()
SurfaceFlinger.cpp:942        android::SurfaceFlinger::onMessageReceived(int)
Looper.cpp:302                android::Looper::pollInner(int)
Looper.cpp:191                android::Looper::pollOnce(int, int*, int*, void**)
Looper.h:176                  android::Looper::pollOnce(int)
SurfaceFlinger.cpp:834 (discriminator 1)android::SurfaceFlinger::run()
main_surfaceflinger.cpp:55    main
libc_init_dynamic.cpp:112     __libc_init
```

Network refresh signal icon affect nDraw
```
D/ViewRootImpl( 3999): ViewRootImpl
D/ViewRootImpl( 3999): java.lang.Throwable
D/ViewRootImpl( 3999):  at android.view.ViewRootImpl.scheduleTraversals(ViewRootImpl.java:1041)
D/ViewRootImpl( 3999):  at android.view.ViewRootImpl.requestLayout(ViewRootImpl.java:829)
D/ViewRootImpl( 3999):  at android.view.View.requestLayout(View.java:16850)
D/ViewRootImpl( 3999):  at android.view.View.requestLayout(View.java:16850)
D/ViewRootImpl( 3999):  at android.view.View.requestLayout(View.java:16850)
D/ViewRootImpl( 3999):  at android.view.View.requestLayout(View.java:16850)
D/ViewRootImpl( 3999):  at android.view.View.requestLayout(View.java:16850)
D/ViewRootImpl( 3999):  at android.view.View.requestLayout(View.java:16850)
D/ViewRootImpl( 3999):  at android.view.View.requestLayout(View.java:16850)
D/ViewRootImpl( 3999):  at android.view.View.requestLayout(View.java:16850)
D/ViewRootImpl( 3999):  at android.widget.ImageView.setImageResource(ImageView.java:370)
D/ViewRootImpl( 3999):  at com.android.systemui.statusbar.SignalClusterView.apply(SignalClusterView.java:166)
D/ViewRootImpl( 3999):  at com.android.systemui.statusbar.SignalClusterView.setWifiIndicators(SignalClusterView.java:104)
D/ViewRootImpl( 3999):  at com.android.systemui.statusbar.policy.NetworkController.refreshSignalCluster(NetworkController.java:287)
D/ViewRootImpl( 3999):  at com.android.systemui.statusbar.policy.NetworkController.refreshViews(NetworkController.java:1160)
D/ViewRootImpl( 3999):  at com.android.systemui.statusbar.policy.NetworkController.onReceive(NetworkController.java:362)
D/ViewRootImpl( 3999):  at android.app.LoadedApk$ReceiverDispatcher$Args.run(LoadedApk.java:768)
D/ViewRootImpl( 3999):  at android.os.Handler.handleCallback(Handler.java:733)
D/ViewRootImpl( 3999):  at android.os.Handler.dispatchMessage(Handler.java:95)
D/ViewRootImpl( 3999):  at android.os.Looper.loop(Looper.java:136)
D/ViewRootImpl( 3999):  at android.app.ActivityThread.main(ActivityThread.java:5017)
D/ViewRootImpl( 3999):  at java.lang.reflect.Method.invokeNative(Native Method)
D/ViewRootImpl( 3999):  at java.lang.reflect.Method.invoke(Method.java:515)
D/ViewRootImpl( 3999):  at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:779)
D/ViewRootImpl( 3999):  at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:595)
D/ViewRootImpl( 3999):  at dalvik.system.NativeStart.main(Native Method)

```




