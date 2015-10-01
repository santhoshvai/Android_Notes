# Loaders

* Loaders are awesome.
* They were introduced in honeycomb but are available as part of the support library. Loaders are essentially the **best practice implementation for asynchronous data loading within an activity or fragment.**
* So when you create a loader it creates an async task to load data on the background thread. It then syncs with the UI thread when the initial loading is complete, and can be set up to monitor the underlying data, and deliver any updates to the UI thread as well.
* Best still, all that work you did adding a content provider to your database pays off right now, with the **cursor loader**.
* The cursor loader is an implementation of the `asyncTaskLoader`, specifically designed to query content providers, and return a cursor, which you can then bind directly to the UI.
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

## Use CursorLoader with WeatherProvider

### How does it currently work (without loader) ?

![Current process](http://i.imgur.com/igRGuCq.png)

* Our UI, first, builds a URI, using the weatherContract.
* The UI then calls a method in the content Resolver.
* Which ultimately forwards the request to our weatherProvider.
* Our weatherProvider uses WeatherDBHelper to get an instance of SQLite database. Creating, or updating the tables of our database as necessary. We then pass the SQL query to the SQLiteDatabase class, which sends our query off to our SQLite database.

### With Cursor Loader

* Cursor Loader takes the URI and calls the content resolver on our behalf, inside of an AsyncTask.

![cursor loader working](http://i.imgur.com/8oQFY32.png)

* Ultimately the cursor gets returned to the Android UI, and it will be used by a cursor adapter to populate the list view (in sunshine's case), from our database cursor. Similar to the way our array adapter populated the list view from an array.

![cursor adapter working](http://i.imgur.com/SG1EMmX.png)

* For our next task, we are going to replace the array adapter we have currently in the forecast fragment, with a cursor adapter. We'll eventually connect it to our database using a cursor loader.

### Using cursor adapter

We’re going to be adding a CursorAdapter to Sunshine called ForecastAdapter. Why? Well, our loader will be making the calls on our content provider to get a Cursor and we’ll want to take the data from that cursor and put it into our UI. **This is what adapters are meant for, associating data with UI components.**

For now, we’re not going to worry about the Loader and we're just going to change our code to use a more appropriate adapter. Currently sunshine uses an ArrayAdapter. This ArrayAdapter is populated only when we’re syncing with OpenWeatherMapAPI. Basically we get the JSON, put it in the content provider, take it out again, and change it to an array. This is not ideal. Let’s fix all of this.

About making an adapter, will be done in another chapter, for now, we get code with quick overview.

* [GH link to forecast adapter](https://github.com/udacity/Sunshine-Version-2/blob/4.18_cursor_adapter/app/src/main/java/com/example/android/sunshine/app/ForecastAdapter.java)
  * `formatHighLows` and `convertCursorRowToUXFormat` are two formatting methods specific to Sunshine.
  * The other two methods are necessary to override whenever you’re extending a cursor adapter.
  * `newView` - Remember that adapters work with listviews to populate them. They create duplicates of the same layout to put into the list view. This is where you return what layout is going to be duplicated.

``` java
View view = LayoutInflater.from(context).inflate(R.layout.list_item_forecast, parent, false);
return view;
```
In our case, we’re inflating our listview layout, list_item_forecast, and then returning it.

  * `bindView` - This is where the exciting bit occurs. As the name suggests you are binding the values in the cursor to the view.

``` java
TextView tv = (TextView)view;
tv.setText(convertCursorRowToUXFormat(cursor));
```

The View passed into bindView is the View returned from newView. We know it’s a TextView, so we cast it. Then we take the Cursor, run it through our custom made formatting function, and set the text of the TextView.

* [GH link to utility.java](https://github.com/udacity/Sunshine-Version-2/blob/4.18_cursor_adapter/app/src/main/java/com/example/android/sunshine/app/Utility.java)

### Refactoring to use ForecastAdapter with Cursors and from the Fragment (TODO: Just copied and pasted this section, need edits)

* **Change mForecastAdapter's type**

Change `mForecastAdapter`, to be an instance of `ForecastAdapter`.

* **Get Data from the Database**

Let’s go to where we first need to populate the ForecastFragment with data and do so by getting the data from the database. Go to `onCreateView`. Use WeatherProvider to query the database the same way you are in FetchWeatherTask:

``` java
     String locationSetting = Utility.getPreferredLocation(getActivity());

        // Sort order:  Ascending, by date.
        String sortOrder = WeatherContract.WeatherEntry.COLUMN_DATE + " ASC";
        Uri weatherForLocationUri = WeatherContract.WeatherEntry.buildWeatherLocationWithStartDate(
                locationSetting, System.currentTimeMillis());

        Cursor cur = getActivity().getContentResolver().query(weatherForLocationUri,
                null, null, null, sortOrder);
```

* **Make a new ForecastAdapter**

Still in `onCreateView`, we have a Cursor cur, so let’s use our new ForecastAdapter. Create a new ForecastAdapter with the new cursor. The list will be empty the first time we run.

``` java
mForecastAdapter = new ForecastAdapter(getActivity(), cur, 0);
```

* **Delete OnItemClickListener**

Because we changed the adapter, the OnItemClickListener in `ForecastFragment` for the ListView won’t work. Specifically this line `String forecast = mForecastAdapter.getItem(position);` is problematic because getItem with a CursorAdapter doesn’t return a string.

Go ahead and remove or comment out this for now.

We’ll talk more about this and correct this soon enough. Until then, our code will compile and run but not have access to our DetailView.

* **Clean up**

Inside of `FetchWeatherTask`, we’re going to remove the formatting code and anything for updating the adapter. You can remove:

* Any reference to `mForecastAdapter`
* `getReadableDateString`, `formatHighLows`, `convertContentValuesToUXFormat`. These are all formatting functions and we’ve moved them to the ForecastAdapter.
* The lines in `getWeatherDataFromJson` where we requery the database after the insert.
* `PostExecute`

**Note:** To keep your tests working, you'll need to modify line 42 of TestFetchWeatherTask to be:

``` java
FetchWeatherTask fwt = new FetchWeatherTask(getContext());
```

To see all of the changes in the codebase, check out this [diff](https://github.com/udacity/Sunshine-Version-2/compare/4.17_bulkinserts_with_contentprovider...4.18_cursor_adapter).

### Leveraging Loaders

**Create a cursorLoader in three easy steps**

1. Create an Integer constant for a Loader ID.
  * A single activity can use multiple Loaders and they are differentiated by these IDs.
  * `private static final int MY_LOADER_ID = [MY ID]`
2. Fill in Loader callbacks
  * They are a generic interface class with one parameterized type
  * When we use the loader callbacks with a cursor loader, the parameterized type we use is cursor.
  * `Loader<cursor> onCreateLoader(int i, Bundle bundle)`.
    * The most interesting one for us.
    * Returns a loader with parameterized type, cursor. This is where we create and return our cursorLoader.
    * cursorLoader has a constructor that takes all of the standard content provider query parameters, and will call the content provider on our behalf, when it is executed by the the loaderManager. `return new cursorLoader([context], [URI], [Projection], [Selection], [SelectionArgs], [SortOrder])`
    * since cursorLoader derives an Async TaskLoader, it will be executed in a *background thread.*
  * `void onLoadFinished(Loader<cursor> cursorLoader, Cursor cursor)`
    * called when the loader completes, and our data is ready.
    * To use the data in our cursor adapter, we just call `cursorAdapter swapCursor(cursor);`.
    * But this is where we would also perform any other UI updates we would want to make when the data is ready.
  * `void onLoaderReset(Loader<cursor> cursorLoader, Cursor cursor)`
    * Typically only called when our loader is being destroyed.
    * It means we need to remove all references to the loader data.
    * to do that we call swap cursor on our cursor adapter with null. `cursorAdapter swapCursor(null);`
3. Initialize Loader with LoaderManager
  * `getLoaderManager().initLoader([loaderID], [Bundle], [loaderCallback]);`
  * **when loading in a fragment we should do this in the `onActivitycreated` method**

[Documentation on loaders](http://developer.android.com/guide/components/loaders.html)
[Documentation on CursorLoader](http://developer.android.com/reference/android/content/CursorLoader.html)

* **NOTE:** We will want to pass zero in for the flags when you create the cursor adapter as the loader will now be responsible for requiring when data changes.
* Use support library

``` java
import android.support.v4.app.Fragment;
import android.support.v4.app.LoaderManager;
import android.support.v4.content.Loader;
import android.support.v4.content.CursorLoader;
```

**CODE**

``` java
public class ForecastFragment extends Fragment implements LoaderManager.LoaderCallbacks<Cursor> {
private ForecastAdapter mForecastAdapter;
private static final int MY_LOADER_ID = 7;

@Override
public void onActivityCreated(Bundle savedInstanceState) {
    super.onActivityCreated(savedInstanceState);
    getLoaderManager().initLoader(MY_LOADER_ID, null, this);
}

@Override
public View onCreateView(LayoutInflater inflater, ViewGroup container,
                         Bundle savedInstanceState) {

    // The CursorAdapter will take data from our cursor and populate the ListView
    // However, we cannot use FLAG_AUTO_REQUERY since it is deprecated, so we will end
    // up with an empty list the first time we run.
    mForecastAdapter = new ForecastAdapter(getActivity(), null, 0);

    View rootView = inflater.inflate(R.layout.fragment_main, container, false);

    // Get a reference to the ListView, and attach this adapter to it.
    ListView listView = (ListView) rootView.findViewById(R.id.listview_forecast);
    listView.setAdapter(mForecastAdapter);

    return rootView;
}

@Override  
public Loader<Cursor> onCreateLoader(int id, Bundle args) {
      String locationSetting = Utility.getPreferredLocation(getActivity());
      // Sort order:  Ascending, by date.      
      String sortOrder = WeatherContract.WeatherEntry.COLUMN_DATE + " ASC";      
      Uri weatherForLocationUri = WeatherContract.WeatherEntry.buildWeatherLocationWithStartDate(              locationSetting, System.currentTimeMillis());
      return new CursorLoader(getActivity(), weatherForLocationUri,  null, null, null, sortOrder);
}
@Override
public void onLoadFinished(Loader<Cursor> loader, Cursor data) {
    // Swap the new cursor in.  (The framework will take care of closing the
    // old cursor once we return.)
    mForecastAdapter.swapCursor(data);
}

@Override
public void onLoaderReset(Loader<Cursor> loader) {
    // This is called when the last Cursor provided to onLoadFinished()
    // above is about to be closed.  We need to make sure we are no
    // longer using it.
    mForecastAdapter.swapCursor(null);
}
}
