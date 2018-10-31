[nDraw 2ss] Back key does not work when device in suspend state
=====================

back key  
injectInputEvent android/hardware/input/InputManager.java  
onKeyUp　android/app/Activity.java  

Normal onKeydown flow
==============

```
D/Activity( 6555): Activity
D/Activity( 6555): java.lang.Throwable
D/Activity( 6555):      at android.app.Activity.onKeyDown(Activity.java:2106)
D/Activity( 6555):      at android.view.KeyEvent.dispatch(KeyEvent.java:2640)
D/Activity( 6555):      at android.app.Activity.dispatchKeyEvent(Activity.java:2429)
D/Activity( 6555):      at com.android.internal.policy.impl.PhoneWindow$DecorView.dispatchKeyEvent(PhoneWindow.java:1962)
D/Activity( 6555):      at android.view.ViewRootImpl$ViewPostImeInputStage.processKeyEvent(ViewRootImpl.java:3991)
D/Activity( 6555):      at android.view.ViewRootImpl$ViewPostImeInputStage.onProcess(ViewRootImpl.java:3965)
D/Activity( 6555):      at android.view.ViewRootImpl$InputStage.deliver(ViewRootImpl.java:3538)
D/Activity( 6555):      at android.view.ViewRootImpl$InputStage.onDeliverToNext(ViewRootImpl.java:3588)
D/Activity( 6555):      at android.view.ViewRootImpl$InputStage.forward(ViewRootImpl.java:3557)
D/Activity( 6555):      at android.view.ViewRootImpl$AsyncInputStage.forward(ViewRootImpl.java:3664)
D/Activity( 6555):      at android.view.ViewRootImpl$InputStage.apply(ViewRootImpl.java:3565)
D/Activity( 6555):      at android.view.ViewRootImpl$AsyncInputStage.apply(ViewRootImpl.java:3721)
D/Activity( 6555):      at android.view.ViewRootImpl$InputStage.deliver(ViewRootImpl.java:3538)
D/Activity( 6555):      at android.view.ViewRootImpl$InputStage.onDeliverToNext(ViewRootImpl.java:3588)
D/Activity( 6555):      at android.view.ViewRootImpl$InputStage.forward(ViewRootImpl.java:3557)
D/Activity( 6555):      at android.view.ViewRootImpl$InputStage.apply(ViewRootImpl.java:3565)
D/Activity( 6555):      at android.view.ViewRootImpl$InputStage.deliver(ViewRootImpl.java:3538)
D/Activity( 6555):      at android.view.ViewRootImpl$InputStage.onDeliverToNext(ViewRootImpl.java:3588)
D/Activity( 6555):      at android.view.ViewRootImpl$InputStage.forward(ViewRootImpl.java:3557)
D/Activity( 6555):      at android.view.ViewRootImpl$AsyncInputStage.forward(ViewRootImpl.java:3697)
D/Activity( 6555):      at android.view.ViewRootImpl$ImeInputStage.onFinishedInputEvent(ViewRootImpl.java:3857)
D/Activity( 6555):      at android.view.inputmethod.InputMethodManager$PendingEvent.run(InputMethodManager.java:2010)
D/Activity( 6555):      at android.view.inputmethod.InputMethodManager.invokeFinishedInputEventCallback(InputMethodManager.java:1704)
D/Activity( 6555):      at android.view.inputmethod.InputMethodManager.finishedInputEvent(InputMethodManager.java:1695)
D/Activity( 6555):      at android.view.inputmethod.InputMethodManager$ImeInputEventSender.onInputEventFinished(InputMethodManager.java:1987)
D/Activity( 6555):      at android.view.InputEventSender.dispatchInputEventFinished(InputEventSender.java:141)
D/Activity( 6555):      at android.os.MessageQueue.nativePollOnce(Native Method)
D/Activity( 6555):      at android.os.MessageQueue.next(MessageQueue.java:138)
D/Activity( 6555):      at android.os.Looper.loop(Looper.java:124)
D/Activity( 6555):      at android.app.ActivityThread.main(ActivityThread.java:5017)
D/Activity( 6555):      at java.lang.reflect.Method.invokeNative(Native Method)
D/Activity( 6555):      at java.lang.reflect.Method.invoke(Method.java:515)
D/Activity( 6555):      at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:779)
D/Activity( 6555):      at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:595)
D/Activity( 6555):      at dalvik.system.NativeStart.main(Native Method)
```
when backkey keydown and keyup in suspend state,
dispatchKeyEvent com/android/internal/policy/impl/PhoneWindow.java
only show isDown 0, that mean KeyEvent Action does not ACTION_DOWN, so we forcus to InputReader InputDispatcher to see how to initial a KeyEvent

In InputDispatcher::notifyKey frameworks/base/services/input/InputDispatcher.cpp
we try to check interceptKeyBeforeQueueing to get correct policyFlag, we found that policyFlag does not set to ACTION_PASS_TO_USER, that mean backkey keydown does not pass to app. if keydown does not pass to app, 

In frameworks/base/core/java/android/app/Activity.java
```java
    public boolean onKeyDown(int keyCode, KeyEvent event)  {
        Log.d(TAG,TAG,new Throwable());
        if (keyCode == KeyEvent.KEYCODE_BACK) {
            if (getApplicationInfo().targetSdkVersion
                    >= Build.VERSION_CODES.ECLAIR) {
                event.startTracking();
            } else {
                onBackPressed();
            }
            return true;
        }
```

```java
    public boolean onKeyUp(int keyCode, KeyEvent event) {
        Log.d(TAG,TAG,new Throwable());

        if (getApplicationInfo().targetSdkVersion
                >= Build.VERSION_CODES.ECLAIR) {
            Log.d(TAG,"event.isTracking()" + event.isTracking());
            Log.d(TAG,"event.isCanceled()" + event.isCanceled());
            if (keyCode == KeyEvent.KEYCODE_BACK && event.isTracking()
                    && !event.isCanceled()) {
                onBackPressed();
                return true;
            }
        }
        return false;
    }
```

```java
    public void onBackPressed() {
        Log.d(TAG,"mFragments.popBackStackImmediate()" + mFragments.popBackStackImmediate());
        if (!mFragments.popBackStackImmediate()) {
            finish();
        }
    }
```

event.isTracking() will not be true, finish will not be executed


＊＊＊
[nDraw suspend] power key long press in suspend state, show and dismiss power dialog. device go to hibernation when release power key
===============


