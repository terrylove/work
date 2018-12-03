[nDraw suspend] device does not suspend again when FL toggle in suspend state

onUserActivityLocked　PowerManagerEx.java
android_server_PowerManagerService_wakeUp　frameworks/base/services/jni/com_android_server_power_PowerManagerService.cpp　
NativeInputManager::interceptKeyBeforeQueueing　frameworks/base/services/jni/com_android_server_input_InputManagerService.cpp

1.Use PowerManagerEx Static variable wakeupkey to record Keycode in interceptKeyBeforeQueueing()

```java
            case KeyEvent.KEYCODE_BUTTON_A: //FrontLight Key
            case KeyEvent.KEYCODE_BUTTON_B: //FrontLight Key
            case KeyEvent.KEYCODE_BUTTON_8: //Tap Key
            case KeyEvent.KEYCODE_HOME: {
                if (down) {
                    PowerManagerEx.wakeupkey = keyCode;
                    if (mPstate.get() == PowerState.POWER_STATE_SUSPEND)
                    {
                        Slog.i(TAG, "==>Suspend, wake up....");
                        result |= ACTION_WAKE_UP;
                        // For 2sec Suspend, Home Key WakeUp Android and send event to APP
                        if (keyCode == KeyEvent.KEYCODE_HOME) {
                            if (KeyCharacterMap.deviceHasKey(KeyEvent.KEYCODE_HOME)
                                && !KeyCharacterMap.deviceHasKey(KeyEvent.KEYCODE_BACK)) {
                                new Thread(new Runnable() {
                                    public void run() {
                                    sendDownAndUpKeyEvents(KeyEvent.KEYCODE_BACK);
                                   }
                                }).start();
                            }
                        }
                    }
                }
                break;
            }
```
2.check wakeupkey in onUserActivityLocked if FL key equal to wakeupkey, set SUSPEND alarm
```java
	public void onUserActivityLocked(int event) {
		Slog.d(LOG_TAG,LOG_TAG, new Throwable());
		Slog.d(LOG_TAG, "=========   onUserActivityLocked event:" + event + "========");
		if ( mPstate.get() == PowerState.POWER_STATE_RUN && mPstate.is_EnhanceFeatureOn()) {
			//mPstate.SuspendAlarmClear();// this will cause deadlock
			if ( event == PowerManager.USER_ACTIVITY_EVENT_TOUCH || event == PowerManager.USER_ACTIVITY_EVENT_OTHER
					|| wakeupkey == KeyEvent.KEYCODE_BUTTON_A
					|| wakeupkey == KeyEvent.KEYCODE_BUTTON_B) {
				wakeupkey = -1;
				mPmExHandler.sendEmptyMessage(SUSPEND_ALARM_CLEAR);
				mPmExHandler.sendEmptyMessage(SUSPEND_ALARM_SET);
			}
		}
	}
```


NDrawSurface
-----------
nDraw permission

```c
void PermissionCache::cache(const String16& permission,
        uid_t uid, bool granted)
	
granted = android::checkPermission(permission, pid, uid);	
	
```

IPermissionControl server end
com/android/server/am/ActivityManagerService.java

```
static class PermissionController extends IPermissionController.Stub

```


```java
private void grantPermissionsLPw(PackageParser.Package pkg, boolean replace) {
```




