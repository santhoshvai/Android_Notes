## Main thread VS Background Thread

* **Main thread** is the UI thread.
  * Handles all user input and output
  * Avoid long operations ( else UI will stutter )

* Kick off **Background thread** for long operations. For example, ( By async task )
  * Network calls
  * Decoding bitmaps
  * Reading & writing from the database
  
* **Async task is not always optimal**
  * You should always seek to eliminate the refresh button
  * A good app should **gives you what you want before you ask for it**

* ISSUE
  * The transfer is on a thread whose lifetime is tied to a UI component
    * If its tied to an activity and if the screen rotates the transfer will be terminated

### Better ways to sync
  
  * Where to use AsyncTask
    * Lifecycle is tied to the host activity
    * Expected to run for only a second or two
    * **Never use on network calls** because it is unwise to assume that it is gonna happen quickly
  
  
  * **Service** - A better alternative
    * Its an application component without an UI thats less likely to be interrupted
    * Can be scheduled using an `inexact repeating alarm
    * **Sync adapter** - designed especially to schedule your background data syncs as efficiently as possible
    * **Google cloud messaging** - Even better!
      * Lets you notify your sync adapter of changes in the server side
      * So you only initiate network requests, when you know that you have to do so!

## Intent
  * Example, When **Main activity** has to call **Detail activity**, `startActivity(Intent)` is used.
  * It is used when, App components need to, (Intents identify the target destination)
    * communicate with each other
    * or the system
  * **Explicit Intents**
    * Intent explicitly indicates the target, using the context and a class name of the activity.
  * Intents are like `envelopes`
    * TO: Who do you want the deliver to (The activity class name)
    * Message: small amount of data can be delivered by packaging them as extras.
  * **Implicit Intents**
    * You dont specify the name of the activity you wanna call
    * Rather specify what `action` you want to perform ( and on what data it has to be performed ).
    * Examples, [For more, see common intents page](http://developer.android.com/intl/zh-cn/guide/components/intents-common.html)
      * Browsing contacts
      * Making phone calls
      * Choose a contact from the address book
    * How do the android system know which app will handle?
      * For example, if you want to show a location, Google maps activity, will have an intent filter like the one below,
        ![Imgur](http://i.imgur.com/QvvOpeU.png)
      * Indicates that the Maps app, can handle the `VIEW` action on all data which has a `geo` scheme
    * this if statement takes care when there is no receiving apps in the user mobile.
```java
   if (intent.resolveActivity(getPackageManager()) != null) {
            startActivity(intent);
        } else {
            Log.d(LOG_TAG, "Couldn't call " + location + ", no receiving apps installed!");
        }
   }
```

  * [Broadcast Intents](https://www.youtube.com/watch?v=dvwjBQ5blnY) - sent to all apps
    * Example, device is charging or low battery ( we can also send ourselves by using `sendBroadcast()` )
    * If you want to recieve it, use a `BroadcastReceiver` with intent filter, to tell system that you wish to receive that specific broadcast
