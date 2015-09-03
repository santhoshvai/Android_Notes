## Activity lifecycle - [Video](https://www.youtube.com/watch?t=68&v=85MppyLJHz0)
![Imgur](http://i.imgur.com/3ZWKSjU.png)

## Active and visible lifetimes

* **Active :**  Activity is in the foreground and has focus
  * `onPause` is called as soon as the activity is partially obscured. Like in [this example of Permissions dialog](http://www.androidpolice.com/wp-content/uploads/2014/06/nexusae0_wm_Screenshot_2014-05-15-11-20-031.png)
  * Same thing happens when another activity is called to fulfill an implicit intent and the user needs to make a seclection
  * To make efficient use of resources, use the `onPause` signal to adjust your app's resource burden.
  * So most updates through a UI can be paused when `onPause` is called.
  * **Caution :** The app is still visible, so we shouldnt pause any processes that are drwaing your UI

* **Visible :** Continues whenever the app is at all visible, ends when it is completely obsucured by another app. (like user opening another app)
  * At this point app is moved to the background (after `onStop`)

## Sequence during device rotation

`onPause` -> `onStop` -> `onDestroy` -> `onCreate` -> `onStart` -> `onResume`

## Activity Termination

* Since Honeycomb (3.0) (API: 11), you can rely on `onStop` being called before the app is at the risk of being terminated
* For pre-honeycomb, you have to rely on `onPause`

## How to prepare for Termination

* Android doesnt announce changes in app state
  * Doesnt announce when it is low in memory
  * It doesnt ask users to close apps to free up resources

* Android does everything it can to make the resource limitation of the device invisible to the user
  * Means keeping the foreground app running smoothly
  * And switching between apps seamless
  * And that means killing apps in the background
  * It does that because it needs the resources used by the background app to keep the foreground app running smoothly.

* As soon as the app is invisible, it is likely to die without notice but ready to return from the dead
  * `onPause` and `onStop` are singmals that our app may be killed imminently
  * So clean up any resources that need an orderly teardown.
  * Some examples of listeners or updates to stop/close/disconnect are,
    * Any open connections or sockets 
    * Sensor listeners
    * Location updates
    * Dynamic broadcast receivers
    * Games physics engines
  * Cleaning up is needed to be considered only when you are adding things like sensor listeners or location listeners, because the default components will handle most of the things for you

## Maintaining state
* `onSaveInstanceState` is called immediately before `onPause`, so soon as your app is no longer active
  * Save a bundle of information here 
* `onRestoreInstanceState` is called immediately after `onCreate` but only if the app is being restarted after having been terminated by the system
  * Read the bundle of information saved here 

