# Loaders

* Loaders are awesome.
* They were introduced in honeycomb but are available as part of the support library. Loaders are essentially the **best practice implementation for asynchronous data loading within an activity or fragment.**
* So when you create a loader it creates an async task to load data on the background thread. It then syncs with the UI thread when the initial loading is complete, and can be set up to monitor the underlying data, and deliver any updates to the UI thread as well.
* Best still, all that work you did adding a content provider to your database pays off right now, with the **cursor loader**.
* The cursor loader is an implementation of the async task loader, specifically designed to query content providers, and return a cursor, which you can then bind directly to the UI.
* It will automatically update that cursor, whenever the underlying content provider changes, based on changes to the underlying database.
* And it will reconnect to the last returned cursor after being recreated, along with the host activity, after config change. That means you won't have to requery data, just because the device was rotated.
* so the **cursor loader handles all of your cursor management and background thread creation, UI thread synchronization, and data source monitoring**.
* If you chose not to use content providers, you chose poorly. But still, you can take advantage of loaders, you just need to create your own loader by extending `AsyncTaskLoader` directly. Note: [AsyncTaskLoader](http://developer.android.com/reference/android/content/AsyncTaskLoader.html) is a Loader that uses an AsyncTask behind the scenes to do its work.

![cursor loaders](http://i.imgur.com/hm6vqtw.png)

## What are they?

![loaders](http://i.imgur.com/uAJ3v2W.png)

* Loaders provide a framework to perform a synchronous loading of data.
* They are registered by ID with a loader manager, which allows them to live beyond the life cycle of the activity or fragment they are associated with.
* In addition to loading data, loaders include mechanisms to monitor the source of their data, and deliver new results when the contents change.

### AsyncTask Lifecycle

![AsyncTask Lifecycle](http://i.imgur.com/VWIRQJc.png)

* Until now, async task was used to load the data from the internet in sunshine.
* We typically create our AsyncTask in the `onCreate()` method of our activity. It starts a thread which begins a background task.
* If we rotate the device, or do something else that requires the activity to be restarted, it will create another AsyncTask to do the background operation.
* We'll create extra network usage, or file system usage as both threads run in parallel. And, it'll take a longer time for the user to see the result of the load.

### AsyncTaskloader Lifecycle

![AsyncTaskLoader lifecycle](http://i.imgur.com/LzKdA3V.png)

* Same functionality as AsyncTask, but its a loader its lifecycle is different.
* With an Async Task Loader, once we rotate the device, the loader manager will make sure that the old loader is connected to the Async Task Loader equivalent of `onPostExecute`, the `onLoadFinished` function.
* The loader thread keeps running in the `loadInBackground` function, and once it finishes the activity gets notified through `onLoadFinished`.

### CursorLoader lifecycle

![CursorLoader lifecycle](http://i.imgur.com/Xm5j46f.png)

* It is derived from `AsyncTaskLoader` but has an **additional optimization**.
* If the cursor has already finished loading and load in background before the activity restarts, the CursorLoader recognizes this during the loader initialization.
* And the cursor is immediately delivered to the UI through `onLoadFinished`

### What are the disadvantages in querying the database from the UI code rather than a loader?

> Early versions of android didn't have the loader pattern it was added to avoid directly querying the database from the UI code.

* **The query could take a long time**
  * Early versions of android don't have cursor loaders. Instead, the cursor adapter re-queried the database on the main UI thread, when data from a content provider changed. This caused frame rate drops in many applications.    
* **The activity could stop before it completes**
  * We noted how things like async tasks are bound to the UI. So something as little as an orientation change could cause the query to complete after the activity stops.
* **The cursor is tied to the activity, so if the activity restarts, the data must be re-queried.**
  * The cursor being tied to the activity also means that the data would generally have to be re-queried when the screen rotates.
