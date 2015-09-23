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
* `SQLiteQueryBuilder` is a helper to compose SQL. They are used when we need to create joins, unions etc. If there is only one table then use `SQLiteDatabase.query()`. Because querybuilder uses an extra object. We miust use it only when needed. [Source](http://stackoverflow.com/questions/17230049/sqlitequerybuilder-query-vs-sqlitedatabase-query)

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
