## [SQLiteOpenHelper](http://developer.android.com/intl/zh-cn/reference/android/database/sqlite/SQLiteOpenHelper.html)
* A helper class that contain cool stuff to,
  * Create the database
  * help us handle database versioning
  * Helps us modify our tables
* For many apps, upgrading to a new version without dataloss is critical
* For sunshine, we just want to make sure that upgrades in that involved database schema changes happen smoothly

## Code

[Source](https://github.com/udacity/Sunshine-Version-2/blob/sunshine_master/app/src/main/java/com/example/android/sunshine/app/data/WeatherDbHelper.java)

``` java
public class WeatherDbHelper extends SQLiteOpenHelper {

    // If you change the database schema, you must increment the database version.
    private static final int DATABASE_VERSION = 2;

    static final String DATABASE_NAME = "weather.db";

    public WeatherDbHelper(Context context) {
     super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }
    ..
    ..
    ..
```
* Contains constants  `DATABASE_VERSION` and `DATABASE_NAME`
* `DATABASE_VERSION` :
  * Typically starts at version 1
  * Must be **manually incremented** each-time we release an updated APK with a new database schema
* `DATABASE_NAME` : Actual name of the database file in the file-system
*  These values are passed into the **constructor** to **initialise the database helper**

### onCreate

* **onCreate method:** Called when the database is created for the first time.
```java
    @Override
    public void onCreate(SQLiteDatabase sqLiteDatabase) {
        // Create a table to hold locations.  A location consists of the string supplied in the
        // location setting, the city name, and the latitude and longitude
        final String SQL_CREATE_LOCATION_TABLE = "CREATE TABLE " + LocationEntry.TABLE_NAME + " (" +
                LocationEntry._ID + " INTEGER PRIMARY KEY," +
                // unique location
                LocationEntry.COLUMN_LOCATION_SETTING + " TEXT UNIQUE NOT NULL, " +
                LocationEntry.COLUMN_CITY_NAME + " TEXT NOT NULL, " +
                LocationEntry.COLUMN_COORD_LAT + " REAL NOT NULL, " +
                LocationEntry.COLUMN_COORD_LONG + " REAL NOT NULL " +
                " );";

        final String SQL_CREATE_WEATHER_TABLE = "CREATE TABLE " + WeatherEntry.TABLE_NAME + " (" +
                // Why AutoIncrement here, and not above?
                // Unique keys will be auto-generated in either case.  But for weather
                // forecasting, it's reasonable to assume the user will want information
                // for a certain date and all dates *following*, so the forecast data
                // should be sorted accordingly.
                WeatherEntry._ID + " INTEGER PRIMARY KEY AUTOINCREMENT," +

                // the ID of the location entry associated with this weather data
                WeatherEntry.COLUMN_LOC_KEY + " INTEGER NOT NULL, " +
                WeatherEntry.COLUMN_DATE + " INTEGER NOT NULL, " +
                WeatherEntry.COLUMN_SHORT_DESC + " TEXT NOT NULL, " +
                WeatherEntry.COLUMN_WEATHER_ID + " INTEGER NOT NULL," +

                WeatherEntry.COLUMN_MIN_TEMP + " REAL NOT NULL, " +
                WeatherEntry.COLUMN_MAX_TEMP + " REAL NOT NULL, " +

                WeatherEntry.COLUMN_HUMIDITY + " REAL NOT NULL, " +
                WeatherEntry.COLUMN_PRESSURE + " REAL NOT NULL, " +
                WeatherEntry.COLUMN_WIND_SPEED + " REAL NOT NULL, " +
                WeatherEntry.COLUMN_DEGREES + " REAL NOT NULL, " +

                // Set up the location column as a foreign key to location table.
                " FOREIGN KEY (" + WeatherEntry.COLUMN_LOC_KEY + ") REFERENCES " +
                LocationEntry.TABLE_NAME + " (" + LocationEntry._ID + "), " +

                // To assure the application have just one weather entry per day
                // per location, it's created a UNIQUE constraint with REPLACE strategy
                " UNIQUE (" + WeatherEntry.COLUMN_DATE + ", " +
                WeatherEntry.COLUMN_LOC_KEY + ") ON CONFLICT REPLACE);";

        sqLiteDatabase.execSQL(SQL_CREATE_LOCATION_TABLE);
        sqLiteDatabase.execSQL(SQL_CREATE_WEATHER_TABLE);
    }
```
* Write the correct SQL string so that we can create the table (using the contract)
* Have the system execute this SQL by calling `db.execSQL(String)`
* **NULL constraints :**
  * Help prevent us from inserting records without columns being filled out into the database
  * This helps to prevent bugs
* **FOREIGN KEY constraint :**
  * We **cannot insert** a weather entry into the database until a location entry for the weather location has been inserted
  * We **cannot delete** location entries while there exist a weather entry that points to them
``` java
" FOREIGN KEY (" + WeatherEntry.COLUMN_LOC_KEY + ") REFERENCES " +
                LocationEntry.TABLE_NAME + " (" + LocationEntry._ID + "), " +
```

### onUpgrade

* called when the database has already been created and the version has been changed.
* **version change :**
  * should signify that the columns, tables or general structure of the database has been changed.
  * Make sure to change the version when you make changes to the database tables.
  * *Why care?* : Because your app wont have errors when you make changes to the database tables.
* **SQLiteOpenHelper** knows about this because the version we pass to its **constructor** has been changed
* If the data that the table contains is user-generated we would want to preserve it.
  * We would use [ALTER TABLE](https://www.sqlite.org/lang_altertable.html) to add new columns.
* In below example, we simply delete the table.
```java
@Override
public void onUpgrade(SQLiteDatabase sqLiteDatabase, int oldVersion, int newVersion) {
    // This database is only a cache for online data, so its upgrade policy is
    // to simply to discard the data and start over
    // Note that this only fires if you change the version number for your database.
    // It does NOT depend on the version number for your application.
    // If you want to update the schema without wiping data, commenting out the next 2 lines
    // should be your top priority before modifying this method.
    sqLiteDatabase.execSQL("DROP TABLE IF EXISTS " + LocationEntry.TABLE_NAME);
    sqLiteDatabase.execSQL("DROP TABLE IF EXISTS " + WeatherEntry.TABLE_NAME);
    onCreate(sqLiteDatabase);
}
```
