# ContentProvider

### Why ?
* Allows you to **share your data safely and efficiently across app boundaries** by **abstracting** the underlying datasource (in this case, SQLite).
* So that other apps can access it without really needing to understand how you stored it.
* In fact, the calendar, SMS, and contacts APIs work that way, using *shared content providers*.
* It is **also useful if you want to don't intend to share your data**. See next section.

## Why should we use it always?

* By using content providers, it's easier for you to potentially switch out the datasource. (like say you now store in files)
* And much easier for someone other than you to manage the UI layer code without them having to understand the depths of your data storage implementation.
* On the UI layer, its a generic mechanism that returns cursors. The same of those returned by SQLite databases.
* So if your data implementation changes, then your content provider is effected. Still, it's just you writing the code right now and that's a lot of boilerplate for the sake of following a neat design pattern.
* Well, as far as the framework is concerned, all data is handled through content providers. So if you want to interact with anything outside of your app, such as **sending data to a widget or returning search results from within your app**. You'll need a content provider for that too. In fact, that's how the Google play store and Gmail widgets work. As well as the ability to get search results from google play.
* Similarly there's a **bunch of APIs designed to optimize the process of synching and querying data, and update the UI accordingly. And all of them also expect content providers.**
* That **includes sync adapters and cursor loaders**. Which make your app able to efficiently **sync** with your server, **load data** in your UI layer, and which include built in content **observers** that will **update your UI automatically when the underlying data changes**.
* You could, of course built all of that yourself but **at a certain point the advantage you gained by not writing a content provider to begin with is lost in the process of having to recreate all of the clusters that utilize it**.

![Why Content Providers Matter](http://i.imgur.com/KaKBsGh.png)

## 4 Steps to Building a ContentProvider

![4 Steps to Building a ContentProvider](http://i.imgur.com/MFBPf3m.png)

### Determine URIs
* A content provider allows us to think of the data associated with our views in terms of universal resource identifiers or URIs.
* They identify a resource in our case a row, or rows, in a database.

Example, `CONTENT://COM.EXAMPLE.ANDROID.SUNSHINE.APP/WEATHER/64111`

**Scheme**

`CONTENT:`

* A scheme is the first part of a URI, that proceeds the colon, and it identifies the protocol that the URI will be using.
* We are used to seeing schemes such as HTTP and FTP in URIs.
* Content colon implies that the uri refers to a content provider

**Authority**

`COM.EXAMPLE.ANDROID.SUNSHINE.APP`

* A content authority, which is a unique string used to locate your content provider
* it should almost always be the package name of the application.

**Location**

`WEATHER`

* A location which typically points to a database table within the application

**Query**

`64111` or `64111?DATE=1435104000`

* Optional query. This can either be part of the URI path or it can take the form of a traditional URI query following a question mark.

* URIs are the primary piece of data that are passed in the intents.
* Another thing that these URIs is used for in android is to **notify the user interface that a piece of the data it is displaying has changed.**
* The code registers an Android construct called `content observer` during our initial query, against the URI that is being displayed. When running a database update routine, likely on another thread. That routine will tell Android that the data associated with the URI has changed. And ultimately, that the URI should requery the database and refresh the display. This will cause to show the latest data. Even though the URI hasnt changed. Content providers can return all sorts of data. But the abstract functions return either a cursor containing a list of items or a single item.
* How these URIs are structured is up to each application. But android provides support, for appending an ID to the URI path, to indicate that the application is interested in a specific record. Rather than a range of records.

**Chosen URIs for sunshine example**

![Sunshine URIs](http://i.imgur.com/BvrICOK.png)
* Types,
  * `DIR` - return many items
  * `ITEM` - return 1 item
* Some URIs exist because the UI has a view that needs to expose that particular data. (like weather with location & data)
* Others primarily used for writing the database (like WEATHER (dir) )

### Update Contract

* In addition to the column names for our database, the contract is also a great place to define the URIs that our application will be using to access its data.

``` java
public class WeatherContract {

    // The "Content authority" is a name for the entire content provider, similar to the
    // relationship between a domain name and its website.  A convenient string to use for the
    // content authority is the package name for the app, which is guaranteed to be unique on the
    // device.
    public static final String CONTENT_AUTHORITY = "com.example.android.sunshine.app";

    // Use CONTENT_AUTHORITY to create the base of all URI's which apps will use to contact
    // the content provider.
    public static final Uri BASE_CONTENT_URI = Uri.parse("content://" + CONTENT_AUTHORITY);

    // Possible paths (appended to base content URI for possible URI's)
    // For instance, content://com.example.android.sunshine.app/weather/ is a valid path for
    // looking at weather data.
    public static final String PATH_WEATHER = "weather";
    public static final String PATH_LOCATION = "location";
    ..
    ..
    ..

    /* Inner class that defines the table contents of the location table */
 public static final class LocationEntry implements BaseColumns {

    // represents base location for each table.
     public static final Uri CONTENT_URI =
             BASE_CONTENT_URI.buildUpon().appendPath(PATH_LOCATION).build();

     public static final String CONTENT_TYPE =
             ContentResolver.CURSOR_DIR_BASE_TYPE + "/" + CONTENT_AUTHORITY + "/" + PATH_LOCATION;
     public static final String CONTENT_ITEM_TYPE =
             ContentResolver.CURSOR_ITEM_BASE_TYPE + "/" + CONTENT_AUTHORITY + "/" + PATH_LOCATION;

    ..
    ..
    ..

    // functions to help build the Content Provider queries.
     public static Uri buildLocationUri(long id) {
         return ContentUris.withAppendedId(CONTENT_URI, id);
     }
 }

 /* Inner class that defines the table contents of the weather table */
   public static final class WeatherEntry implements BaseColumns {

       public static final Uri CONTENT_URI =
               BASE_CONTENT_URI.buildUpon().appendPath(PATH_WEATHER).build();

       public static final String CONTENT_TYPE =
               ContentResolver.CURSOR_DIR_BASE_TYPE + "/" + CONTENT_AUTHORITY + "/" + PATH_WEATHER;
       public static final String CONTENT_ITEM_TYPE =
               ContentResolver.CURSOR_ITEM_BASE_TYPE + "/" + CONTENT_AUTHORITY + "/" + PATH_WEATHER;

        ...
        ...
        ...

       public static Uri buildWeatherUri(long id) {
           return ContentUris.withAppendedId(CONTENT_URI, id);
       }

       public static Uri buildWeatherLocation(String locationSetting) {
           return CONTENT_URI.buildUpon().appendPath(locationSetting).build();
       }

       /* WEATHER/[locationSetting]?DATE=[startDate] */
       public static Uri buildWeatherLocationWithStartDate(
               String locationSetting, long startDate) {
           long normalizedDate = normalizeDate(startDate);
           return CONTENT_URI.buildUpon().appendPath(locationSetting)
                   .appendQueryParameter(COLUMN_DATE, Long.toString(normalizedDate)).build();
       }

       public static Uri buildWeatherLocationWithDate(String locationSetting, long date) {
           return CONTENT_URI.buildUpon().appendPath(locationSetting)
                   .appendPath(Long.toString(normalizeDate(date))).build();
       }

       /* Functions to retrieve knowledge from URIs */
       public static String getLocationSettingFromUri(Uri uri) {
           return uri.getPathSegments().get(1);
       }

       public static long getDateFromUri(Uri uri) {
           return Long.parseLong(uri.getPathSegments().get(2));
       }

       public static long getStartDateFromUri(Uri uri) {
           String dateString = uri.getQueryParameter(COLUMN_DATE);
           if (null != dateString && dateString.length() > 0)
               return Long.parseLong(dateString);
           else
               return 0;
       }
   }
```

* `ContentUris`
  * Utility methods useful for working with Uri objects that use the "content" scheme. Content URIs have the syntax `content://authority/path/id`
  * `id`
    * A unique numeric identifier for a single row in the subset of data identified by the preceding path part. Most providers recognize content URIs that contain an id part and give them special handling. A table that contains a column named _ID often expects the id part to be a particular value for that column.
  * `public static Uri withAppendedId (Uri contentUri, long id)`
    * Appends the given ID to the end of the path.

* Cursors returned from a content provider have unique types based upon their content and the base path used for the query. Android uses a form similar to the internet media type or mime type to describe the type returned by the URI.

* Cursors that can,
  * return more than one item - prefixed by the `CURSOR_DIR_BASE_TYPE` (`ContentResolver.CURSOR_DIR_BASE_TYPE`)
  * return only a single item - prefoxed by `CURSOR_ITEM_BASE_TYPE` (`ContentResolver.CURSOR_ITEM_BASE_TYPE`)

## Content Provider Code

``` java
// The URI Matcher used by this content provider.
    private static final UriMatcher sUriMatcher = buildUriMatcher();
    private WeatherDbHelper mOpenHelper;

    static final int WEATHER = 100;
    static final int WEATHER_WITH_LOCATION = 101;
    static final int WEATHER_WITH_LOCATION_AND_DATE = 102;
    static final int LOCATION = 300;
   ...
   ...
   ...

```
* Content providers implement functionality based upon URIs passed to them.
* This content provider will implement 4 types of URIs. Each URI performs different operations against the underlying SQLiteDatabases.
* For ease of implementation content providers typically tie each URI type internally to an **integer constant**.

``` java
static final int WEATHER = 100;
static final int WEATHER_WITH_LOCATION = 101;
static final int WEATHER_WITH_LOCATION_AND_DATE = 102;
static final int LOCATION = 300;
```
### URIMatcher

* Android provides a `URIMatcher` class to help match incoming URIs to help match incoming URIs to the content provider integer constants. This is important because we need to have a way of knowing which type of URI is passed into our content provider so we can perform the requested operation. Once we have the integer constants, we can easily use them in switch statements.
* UriMatcher provides for an **expression syntax** to match various URIs that works a bit like regular expressions.

![URIMatcher](http://i.imgur.com/xMiJC3n.png)

* Hash symbol - match a number (#)
* Asterix - match any string (*)

``` java
static UriMatcher buildUriMatcher() {

     UriMatcher sURIMatcher = new UriMatcher(UriMatcher.NO_MATCH);

     sURIMatcher.addURI (WeatherContract.CONTENT_AUTHORITY, WeatherContract.PATH_WEATHER, WEATHER);
     sURIMatcher.addURI(WeatherContract.CONTENT_AUTHORITY, WeatherContract.PATH_WEATHER + "/*",WEATHER_WITH_LOCATION);
     sURIMatcher.addURI(WeatherContract.CONTENT_AUTHORITY, WeatherContract.PATH_WEATHER + "/*/#",WEATHER_WITH_LOCATION_AND_DATE);
     sURIMatcher.addURI(WeatherContract.CONTENT_AUTHORITY, WeatherContract.PATH_LOCATION,LOCATION);

     return sURIMatcher;
 }
```

* The URIMatcher is central to coding our Content Provider.
* It will be used in the implementation of core Content Provider methods.

### Android Manifest

* [Docs](http://developer.android.com/intl/zh-cn/guide/topics/manifest/provider-element.html)
* We need to register our content provider class in the manifest so that the platform knows about it.

``` xml
<provider
  android:authorities="[CONTENT AUTHORITY]"
  android:name=[CONTENT PROVIDER CLASS] />
```

* In `android:name`, we don't need to specify the full package name. The additional package of the class is all we need.

Like this,

``` xml
<provider
    android:authorities="com.example.android.sunshine.app"
    android:name=".data.WeatherProvider" />
```

### ContentResolver

* Once the weather provider has been registered with the package manager, we can use an Android utility class called the `ContentResolver` to refer to it.
* The ContentResolver locates our class using the Content Authority and makes direct calls to the weather provider on our behalf.

### Implement functions of content provider

![Functions of content provider](http://i.imgur.com/XpwjU2M.png)

* This is the final and most involved step in implementing our content provider.

**onCreate()**

* An instance variable for our database helper is already created.
* It is initialised here.
* This database helper will be used in most of the content provider methods.

``` java
private WeatherDbHelper mOpenHelper;
  @Override
  public boolean onCreate() {
     mOpenHelper = new WeatherDbHelper(getContext());
     return true;
  }
```

**getType()**

* Nice way to review what URIs we are going to handle
* Only to find what type of data is given by the returned database cursor. (ITEM or DIR)

``` java
@Override
public String getType(Uri uri) {

    final int match = sUriMatcher.match(uri);

    switch (match) {
        case WEATHER_WITH_LOCATION_AND_DATE:
            return WeatherContract.WeatherEntry.CONTENT_ITEM_TYPE;
        case WEATHER_WITH_LOCATION:
            return WeatherContract.WeatherEntry.CONTENT_TYPE;
        case WEATHER:
            return WeatherContract.WeatherEntry.CONTENT_TYPE;
        case LOCATION:
            return WeatherContract.LocationEntry.CONTENT_TYPE;
        default:
            throw new UnsupportedOperationException("Unknown uri: " + uri);
    }
}
```

**query()**

* It will be the most complex of the required content provider methods.
* Use the `sURIMatcher` to switch on the type of URI
* Each response from this function will return a cursor that correspons to the incoming query as defined by the URI.
* Setting the `notification URI` of the cursor to the one that was passed causes the cursor to *register a content observer*, to watch for changes that happen to that URI and any of its descendants. This allows the content provider to easily tell the UI when the cursor changes, on operations like database insert or update.

``` java
    private static final SQLiteQueryBuilder sWeatherByLocationSettingQueryBuilder;

    static{
        sWeatherByLocationSettingQueryBuilder = new SQLiteQueryBuilder();

        //This is an inner join which looks like
        //weather INNER JOIN location ON weather.location_id = location._id
        sWeatherByLocationSettingQueryBuilder.setTables(
                WeatherContract.WeatherEntry.TABLE_NAME + " INNER JOIN " +
                        WeatherContract.LocationEntry.TABLE_NAME +
                        " ON " + WeatherContract.WeatherEntry.TABLE_NAME +
                        "." + WeatherContract.WeatherEntry.COLUMN_LOC_KEY +
                        " = " + WeatherContract.LocationEntry.TABLE_NAME +
                        "." + WeatherContract.LocationEntry._ID);
    }

    //location.location_setting = ?
    private static final String sLocationSettingSelection =
            WeatherContract.LocationEntry.TABLE_NAME+
                    "." + WeatherContract.LocationEntry.COLUMN_LOCATION_SETTING + " = ? ";

    //location.location_setting = ? AND date >= ?
    private static final String sLocationSettingWithStartDateSelection =
            WeatherContract.LocationEntry.TABLE_NAME+
                    "." + WeatherContract.LocationEntry.COLUMN_LOCATION_SETTING + " = ? AND " +
                    WeatherContract.WeatherEntry.COLUMN_DATE + " >= ? ";

    //location.location_setting = ? AND date = ?
    private static final String sLocationSettingAndDaySelection =
            WeatherContract.LocationEntry.TABLE_NAME +
                    "." + WeatherContract.LocationEntry.COLUMN_LOCATION_SETTING + " = ? AND " +
                    WeatherContract.WeatherEntry.COLUMN_DATE + " = ? ";

    private Cursor getWeatherByLocationSetting(Uri uri, String[] projection, String sortOrder) {
        String locationSetting = WeatherContract.WeatherEntry.getLocationSettingFromUri(uri);
        long startDate = WeatherContract.WeatherEntry.getStartDateFromUri(uri);

        String[] selectionArgs;
        String selection;

        if (startDate == 0) {
            selection = sLocationSettingSelection;
            selectionArgs = new String[]{locationSetting};
        } else {
            selectionArgs = new String[]{locationSetting, Long.toString(startDate)};
            selection = sLocationSettingWithStartDateSelection;
        }

        return sWeatherByLocationSettingQueryBuilder.query(mOpenHelper.getReadableDatabase(),
                projection,
                selection,
                selectionArgs,
                null,
                null,
                sortOrder
        );
    }

    private Cursor getWeatherByLocationSettingAndDate(
            Uri uri, String[] projection, String sortOrder) {
        String locationSetting = WeatherContract.WeatherEntry.getLocationSettingFromUri(uri);
        long date = WeatherContract.WeatherEntry.getDateFromUri(uri);

        return sWeatherByLocationSettingQueryBuilder.query(mOpenHelper.getReadableDatabase(),
                projection,
                sLocationSettingAndDaySelection,
                new String[]{locationSetting, Long.toString(date)},
                null,
                null,
                sortOrder
        );
    }

    @Override
   public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs,
                       String sortOrder) {
       // Here's the switch statement that, given a URI, will determine what kind of request it is,
       // and query the database accordingly.
       Cursor retCursor;
       switch (sUriMatcher.match(uri)) {
           // "weather/*/*"
           case WEATHER_WITH_LOCATION_AND_DATE:
           {
               retCursor = getWeatherByLocationSettingAndDate(uri, projection, sortOrder);
               break;
           }
           // "weather/*"
           case WEATHER_WITH_LOCATION: {
               retCursor = getWeatherByLocationSetting(uri, projection, sortOrder);
               break;
           }
           // "weather"
           case WEATHER: {
               retCursor = mOpenHelper.getReadableDatabase().query(
                       WeatherContract.WeatherEntry.TABLE_NAME,
                       projection,
                       selection,
                       selectionArgs,
                       null,
                       null,
                       sortOrder
               );
               break;
           }
           // "location"
           case LOCATION: {
               retCursor = mOpenHelper.getReadableDatabase().query(
                       WeatherContract.LocationEntry.TABLE_NAME,
                       projection,
                       selection,
                       selectionArgs,
                       null,
                       null,
                       sortOrder
               );
               break;
           }

           default:
               throw new UnsupportedOperationException("Unknown uri: " + uri);
       }
       retCursor.setNotificationUri(getContext().getContentResolver(), uri);
       return retCursor;
   }
```
*SQLiteQueryBuilder*

* `SQLiteQueryBuilder` is a helper to compose SQL. They are used when we need to create joins, unions etc. If there is only one table then use `SQLiteDatabase.query()`. Because querybuilder uses an extra object. We must use it only when needed. [Source](http://stackoverflow.com/questions/17230049/sqlitequerybuilder-query-vs-sqlitedatabase-query)

* in this case, it is initialized in the static constructor, describing the join between both tables.
* `setTables` fills out the part in the front part of the SQL query. Note, since both tables have a field with `_ID`, we must explicitly use the table name to indicate specifically which ID we are talking about in the join.
* The selection is defined using the question mark replacement syntax. The selection parameters will replace these values. Note that we fetch the parameters from the URI and built a string array so that they can substituted into our query.

**insert()**

* This function takes advantage of the URI matcher code. But it only requires the base URIs. That's because inserts are fundamentally simpler. We simply want to make sure that the right record winds up in the right table.
* The data that is contained in the other URI, such as location and date, will actually be in the content values used in the insert. Note that if we wanted to, we could support all of the URIs here. But it makes the implementation of the insert function far more complicated.
* when we insert into our database, we wanted to notify every content observer that might have data modified by our insert. It turns out that cursors register themselves as notify for descendants. Which means that notifying the root URI will also notify all descendants of the URI, ones that contain additional path information. Just like with calling our content providers. We can use a content resolver to notify our content observer.
* As you can see, the root URI for each table in sunshine just contains the content scope, the authority, and the table name. consider a content URI that contains a date, it is a descendant of the plain weather content URI. If we notify based on anything other than the root URI, then a cursor listening on the root URI will not get notified of a change that would certainly impact it. So we have to be very careful when doing that.

``` java
@Override
  public Uri insert(Uri uri, ContentValues values) {
      final SQLiteDatabase db = mOpenHelper.getWritableDatabase();
      final int match = sUriMatcher.match(uri);
      Uri returnUri;

      switch (match) {
          case WEATHER: {
              normalizeDate(values);
              long _id = db.insert(WeatherContract.WeatherEntry.TABLE_NAME, null, values);
              if ( _id > 0 )
                  returnUri = WeatherContract.WeatherEntry.buildWeatherUri(_id);
              else
                  throw new android.database.SQLException("Failed to insert row into " + uri);
              break;
          }
          default:
              throw new UnsupportedOperationException("Unknown uri: " + uri);
      }
      getContext().getContentResolver().notifyChange(uri, null);
      return returnUri;
  }
```

* For weather, we just passed the parameters that came into the content provider into the database insert call. We should throw an exception if the insert fails. The only trick here is to make sure we return the correct value, which is a URI. A function to build these URIs was already made while making the contract, it contained the weather path followed by an ID.
* If we were being complete in our content provider implementation, we should also implement these URI types in the contract URI matcher and query function. But the beautiful thing about implementing a content provider, especially if its only being used by your application, is that you only need to implement the features you use.

**update() and delete()**

``` java
@Override
public int delete(Uri uri, String selection, String[] selectionArgs) {
    final SQLiteDatabase db = mOpenHelper.getWritableDatabase();
    final int match = sUriMatcher.match(uri);
    int rows;
    // if selection is null it will delete all rows.
    // passing 1 will remove all rows and return number of rows deleted
    if (selection == null) selection = "1";
    switch (match) {
        case WEATHER: {
            // if selection is null it will delete all rows.
            rows = db.delete(WeatherContract.WeatherEntry.TABLE_NAME, selection, selectionArgs);
            break;
        }
        case LOCATION: {
            rows = db.delete(WeatherContract.LocationEntry.TABLE_NAME, selection, selectionArgs);
            break;
        }
        default:
            throw new UnsupportedOperationException("Unknown uri: " + uri);
    }
    if (rows != 0)
        getContext().getContentResolver().notifyChange(uri, null);
    return rows;
}

@Override
public int update(
        Uri uri, ContentValues values, String selection, String[] selectionArgs) {
    final SQLiteDatabase db = mOpenHelper.getWritableDatabase();
    final int match = sUriMatcher.match(uri);
    int rows;

    switch (match) {
        case WEATHER: {
            rows = db.update(WeatherContract.WeatherEntry.TABLE_NAME, values, selectionselectionArgs);
            break;
        }
        case LOCATION: {
            rows = db.update(WeatherContract.LocationEntry.TABLE_NAME, values, selectionselectionArgs);
            break;
        }
        default:
            throw new UnsupportedOperationException("Unknown uri: " + uri);
    }
    if (rows != 0)
        getContext().getContentResolver().notifyChange(uri, null);
    return rows;
}
```

**bulkInsert()**

* Optional method in the content provider
* In SQLite, putting a bunch of inserts into a single transaction is much faster than inserting them individually. BulkInsert allows us to do that.
* The default implementation just calls insert a bunch of times. But, we can wrap it in a transaction, if we implement ourselves.
* In this case, only for weather we need to insert in bulk.
``` java
@Override
public int bulkInsert(Uri uri, ContentValues[] values) {
   final SQLiteDatabase db = mOpenHelper.getWritableDatabase();
   final int match = sUriMatcher.match(uri);
   switch (match) {
       case WEATHER:
           db.beginTransaction();
           int returnCount = 0;
           try {
               for (ContentValues value : values) {
                   normalizeDate(value);
                   long _id = db.insert(WeatherContract.WeatherEntry.TABLE_NAME, null, value);
                   if (_id != -1) {
                       returnCount++;
                   }
               }
               db.setTransactionSuccessful();
           } finally {
               db.endTransaction();
           }
           getContext().getContentResolver().notifyChange(uri, null);
           return returnCount;
       default:
           return super.bulkInsert(uri, values);
   }
}
```

* Note: If we do not set the transaction to be successful, the records will not be commited when we call `endTransaction()`.


### Recap

1. We started off by defining the URI's that our ContentProvider will support.
2. We then updated our contract to reflect these URIs
3.  We built a URIMatcher that matches these URIs to constants we use in switch statements in all of the other required content provider functions.
4. Then we implemented the functions,
  1. `getType()` function to return the type of cursor returned for each URI.
  2.  We then implemented the contentProvider query functions, followed by the write operations insert, update, and delete.
  3. finally, the bulkInsert function was implemented to make updates to our database in a single transaction. This performs much faster, and causes less wear and tear on the flash chip, compared to updating the database in multiple transaction. **There are a lot of libraries out there in the open-source world to help us build ContentProviders**. If we want to use them, we can use them with the confidence knowing how they they work and what they are doing.

### Use them

So, all queries and updates to the database can now be done through the contentProvider interface. But we are still not using them in our code. We need to change our code to store our weather information  in the database using the content provider we just built.  

``` java
/**
    * Helper method to handle insertion of a new location in the weather database.
    *
    * @param locationSetting The location string used to request updates from the server.
    * @param cityName A human-readable city name, e.g "Mountain View"
    * @param lat the latitude of the city
    * @param lon the longitude of the city
    * @return the row ID of the added location.
    */
   long addLocation(String locationSetting, String cityName, double lat, double lon) {
       // Students: First, check if the location with this city name exists in the db
       Cursor cur = mContext.getContentResolver().query(WeatherContract.LocationEntry.CONTENT_URI,
              new String[]{WeatherContract.LocationEntry._ID},
               WeatherContract.LocationEntry.COLUMN_LOCATION_SETTING + " = ?",
               new String[]{locationSetting}, null);
       long locationId;
       // If it exists, return the current ID
       if (cur.moveToFirst()) {
           locationId = cur.getLong(0);
       } else {
           // Otherwise, insert it using the content resolver and the base URI
           ContentValues testValues = new ContentValues();
           testValues.put(WeatherContract.LocationEntry.COLUMN_LOCATION_SETTING, locationSetting);
           testValues.put(WeatherContract.LocationEntry.COLUMN_CITY_NAME, cityName);
           testValues.put(WeatherContract.LocationEntry.COLUMN_COORD_LAT, lat);
           testValues.put(WeatherContract.LocationEntry.COLUMN_COORD_LONG, lon);
           Uri uri = mContext.getContentResolver().insert(WeatherContract.LocationEntry.CONTENT_URI, testValues);
           locationId = ContentUris.parseId(uri);
       }
       cur.close();
       return locationId;
   }
```

## But

We really need to be querying things before requesting them from the internet. After all, the main point is to have a responsive application. We are still not updating the screen until after we have pulled data down from the web. We also want to avoid doing our query on the UI thread, because doing operations like queries on the UI thread cause Android to not be able to draw frames fast enough, which introduces frame jitter. Fortunately android offers a pattern for that known as **Loaders**.
