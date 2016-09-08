## Async tasks - bad choice

[Super SO answer](http://stackoverflow.com/questions/12797550/android-asynctask-for-long-running-operations)

* They are poorly tied to the host activity life cycle
  * The VM will hold on to the activity object as long as the asynctask is running even after `onDestroy` is called and expect to be discarded
  * **Rotation of Phone :** The behavior is to destroy activity and instantiate a new one. Now there are two async task threads trying to perform the same update.
* They create memory leaks easily
  * It is very convenient to create AsyncTasks as inner classes of your Activities. As the AsyncTask will need to manipulate the views of the Activity when the task is complete or in progress, using an inner class of the Activity seems convenient : inner classes can access directly any field of the outer class.
  * Nevertheless, it means the inner class will hold an invisible reference on its outer class instance : the Activity.
  * On the long run, this produces a memory leak : if the AsyncTask lasts for long, it keeps the activity "alive" whereas Android would like to get rid of it as it can no longer be displayed. The activity *can't be garbage collected* and that's a central mechanism for Android to preserve resources on the device.
* It is not the best pattern for a potentially very long background operation like fetching from webservices.
* If you leave the app, the async task will run as long as your app is kept alive but will run at a low priority and the your process will be the first one to be killed if your device needs more resources.
* There is a bigger problem, your app has to be visible and running in the foreground to instantiate the task in the first place.  In weather apps, this can have undesirable behavior if the weather changes rapidly.

### what we want

* We want the app to get regular updates in the foreground
* we want regular updates in the background as well, with minimal battery drain. (especially for notifications)

# Services

* start service much like you do an activity - by passing an intent. `startService(Intent)` and also `stopService(Intent)`
* unlike activities, services have no UI and they run at a higher priority than background activities. this means that an app with a running service is less likely to be killed at runtime. In fact, by default the system will by default try to restart services that are terminated before they are stopped from within the app.
* They have a simple lifecycle, unlike activities the services are designed to execute longer running tasks that should not be interrupted. Typically we only need to override the `onStartCommand` handler, which is where you begin the background task you wish to execute. Notice that there are no handlers to reflect that the app has moved to background, this is because the running service itself sends a signal to the framework, that the containing app should be considered higher priority than other apps in the background that dont have running services.

```
onCreate
  |
  V
onStartCommand
  |
  V
onDestroy  
```

*  In some cases your service may be performing a task while not having a UI cant be interrupted without interfering with the user experience.  Like *playing music or maps app*.
* In these cases, we can specify that the service should be considered to be running in the foreground by calling `startForeground(notification)`. This call takes in a notification. The notification will be displayed and cant be dismissed until the service has stopped or you call `stopForeground`.
* A foreground service runs at the same priority as a foreground activity. Makes it hard for the runtime to kill inorder to free up resources. If many foreground service runs at the same time, can make system difficult to manage resources ultimately leading to a worse user experience. Therefore must be stopped ASAP.

![service priority](http://i.imgur.com/ztGvqh5.png)

* like activities and receivers, services run on the main thread. So we ll need to use a backgrd thread or an async task to execute the long running tasks that you wish to do within your service. To make life easier you can use the intent service class. Which implements the common best practice pattern, for using intents, which are executed within a service. It creates a queue of incoming intents, passed in when start service is called. These are then processed sequentially on a background thread. Within the `onHandleIntent` handler within your intent service implementation. When the queue is empty, the service self-terminates until a new intent is received, and the process begins again.

![intent service](http://i.imgur.com/PzoWp1h.png)

* Framework alternative to rolling your own service.

```
Service --------- IntentService (for executing background tasks)
           |
           |
           ------ SyncAdapter (perfect for performing data synchronisation)
```

## Alarm manager

* Allows you to tell the system that you want it to wake a component of your application up after a period of time and do some processing in the background. You can even have it to wake up your application periodically.
* **What do we wake up in the backgroud?:** That would be a *broadcast receiver.*
  * A BroadcastReceiver is a special class that is used to receive intent broadcasts often from other applications. Typically a BroadcastReceiver will register an *intent filter* to register these broadcasts. Its also one way the application will listen in on **alarms**.
* Alarms take advantage of a new kind of intent, called a **PendingIntent**. A PendingIntent is a special kind of intent that is handed from one application to another. The big difference between a PendingIntent and a regular intent is that a PendingIntent gives permission for the app using it to send data with the same permissions and application identity as the app that created it. In android this allows the system process to call your application back in a specific asynchronous way without compromising the Android security model. In alarms, a PendingIntent is used by the alarm manager to talk to the BroadcastReceiver we create.

### Transferring data without draining battery

* basically cell radio drains batter life. So the more time it goes to idle state it is better. So you want to fetch most data you can in one shot! (balance is needed with what data you ll use and what you fetch)
* [Transferring Data Without Draining the Battery ](http://developer.android.com/training/efficient-downloads/index.html)
* [DevBytes: Efficient Data Transfers](https://www.youtube.com/watch?v=cSIB2pDvH3E&list=PLWz5rJ2EKKc-VJS9WQlj9xM_ygPopZ-Qd)

```
(Taken from developer docs -- Transferring Data Without Draining the Battery)

Every time you initiate a connection—irrespective of the size of the associated data transfer—you potentially cause the radio to draw power for nearly 20 seconds when using a typical 3G wireless radio. An app that pings the server every 20 seconds, just to acknowledge that the app is running and visible to the user, will keep the radio powered on indefinitely, resulting in a significant battery cost for almost no actual data transfer.

With that in mind it's important to bundle your data transfers and create a pending transfer queue. Done correctly, you can effectively phase-shift transfers that are due to occur within a similar time window, to make them all happen simultaneously—ensuring that the radio draws power for as short a duration as possible.

The underlying philosophy of this approach is to transfer as much data as possible during each transfer session in an effort to limit the number of sessions you require.

That means you should batch your transfers by queuing delay tolerant transfers, and preempting scheduled updates and prefetches, so that they are all executed when time-sensitive transfers are required. Similarly, your scheduled updates and regular prefetching should initiate the execution of your pending transfer queue.
```

* There's a lot to learn with making background transactions efficient, but the good news is that Android gives you the SyncManager framework that implements many of these best practices.You utilize that framework by implementing a SyncAdapter. The framework, originally introduced in Android 2.0 Eclair or Android API level 5, allows Android applications to leverage the same basic framework that Google apps use for efficient synchronization. Ultimately, it's a centralized place to put all of the device data transfers in one place. So that they all be scheduled intelligently by the OS. 
* Android **SyncManager** handles synchronization requests using **SyncAdapters**. The SyncManager batches and time shifts these requests, when possible, to allow your data transfers to be scheduled with transfers from other apps, **all working towards the goal of reducing the number of times the system has to switch on the radio**. If your device has less memory, it will schedule fewer simultaneous synchs. The SyncManager also takes care of things like checking for network connectivity before initiating transfers and retrying downloads when connectivity is dropped. 
* The synchronization framework works with content providers for two way synchronization and leverages the Android Account Manager to provide synchronization services that are tied to accounts. 
* Many of the normal use cases for sync adapters involve syncing the app's local data with information from an online account. For example, in an email app, a sync adapter might be used to pull down a user’s emails at regular intervals. Of course to do this the email app would need to store the user’s account information, such as a username and password, so that the app could log in and grab the new messages. To manage this, sync adapters each have a concept of a user’s account, tied with the sync adapter.
* Even if our application will do neither of these things, but we'll still have to deal with some of the complexity of these features. This can make SyncAdapter seem daunting at first.
* [Android developer training](http://developer.android.com/training/sync-adapters/index.html)
* use **AbstractThreadedSyncAdapter** class
* For a sync to happen successfully at periodic intervals in the background you must..
  * Have a content provider marked as syncable
  * Enable automatic sync for the syncAdapter
  * Set the interval in seconds

### Inexact repeating alarms

* infinitely better than exact repeating alarms but far from ideal.
* the problem with any kind of repeating alarm is that its *still polling your server to check for updates*. The more frequently you poll, the fresher the data you can display. But the higher the cost in battery life.
* *Google cloud messaging* lets your server notify your app directly when there is data ready to be downloaded. Or even include the new data in the message payload itself. Using GCM, you can send data from your server to any installed instance of your app via the google cloud.
* These messages can be simple tickles that trigger a sync adapter by notifying your app that there is new data ready to download. Or you can include the new data in the message payload.
* [Developer guide for Google Cloud Messaging](http://developer.android.com/google/gcm/index.html)
