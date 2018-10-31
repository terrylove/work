[nDraw suspend] device does not suspend again when FL toggle in suspend state

onUserActivityLocked　PowerManagerEx.java
android_server_PowerManagerService_wakeUp　frameworks/base/services/jni/com_android_server_power_PowerManagerService.cpp　
NativeInputManager::interceptKeyBeforeQueueing　frameworks/base/services/jni/com_android_server_input_InputManagerService.cpp

Use PowerManagerEx Static variable wakeupkey to record Keycode in interceptKeyBeforeQueueing()

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
