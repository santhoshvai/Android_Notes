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
* **onCreate method:** Called when the database is created for the first time.
```java
    @Override
    public void onCreate(SQLiteDatabase sqLiteDatabase) {
        // Create a table to hold locations.  A location consists of the string supplied in the
        // location setting, the city name, and the latitude and longitude
        final String SQL_CREATE_LOCATION_TABLE = "CREATE TABLE " + LocationEntry.TABLE_NAME + " (" +
                LocationEntry._ID + " INTEGER PRIMARY KEY," +
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

## Test
* Android has a built-in testing framework that allows us to create a test APK that executes a JUnit test that call into classes in our main APK.
* **JUnit :** a [testing framework](http://www.tutorialspoint.com/junit/junit_quick_guide.htm) that allows us to run automated test suites
* **Small example :**

```java
package com.example.android.sunshine.app.data;

import android.test.AndroidTestCase;

public class TestPractice extends AndroidTestCase {
    /*
        This gets run before every test.
     */
    @Override
    protected void setUp() throws Exception {
        super.setUp();
    }

    public void testThatDemonstratesAssertions() throws Throwable {
        int a = 5;
        int b = 3;
        int c = 5;
        int d = 10;

        assertEquals("X should be equal", a, c);
        assertTrue("Y should be true", d > a);
        assertFalse("Z should be false", a == b);

        if (b > d) {
            fail("XX should never happen");
        }
    }

    @Override
    protected void tearDown() throws Exception {
        super.tearDown();
    }
}
```
* `extends AndroidTestCase` :
  * `setUp` : Run before each test
  * `tearDown` : Run after each test
* Add new methods in the class with prefix `test` : similar to JUnit
  * Like `testThatDemonstratesAssertions`

* **FullTestSuite.java**

``` java
package com.example.android.sunshine.app;

import android.test.suitebuilder.TestSuiteBuilder;

import junit.framework.Test;
import junit.framework.TestSuite;

public class FullTestSuite extends TestSuite {
    public static Test suite() {
        return new TestSuiteBuilder(FullTestSuite.class)
                .includeAllPackagesUnderHere().build();
    }

    public FullTestSuite() {
        super();
    }
}
```
* Contains code to include all of the java test classes in its package into a suite of tests that JUnit will run
* Each test should have atleast one check that uses an assert to see if the program supplies the correct output
* Easily add additional tests
  * By just adding additional java class files to our test directory
* We will likely have a class like in each project we make
  * They arent typically project specific ( except package name, duh! )
  * So, literally this code can be *copy-pasted*

### Running Tests

**Option 1**

  Right click on the folder androidTest and select Run > Tests in ‘com.exampl…. Make sure the select the Android logo (as shown below) and not the Gradle logo.

  ![Imgur](http://i.imgur.com/oClkDZ2.png)

  **Option 2**

  Right next to the Run button, click on the drop down and select Edit Configurations.

  ![Imgur](http://i.imgur.com/aicNLGt.png)

  You’re going to make a “testing” configuration. To do this, click the + button and select Android Tests.

  ![Imgur](http://i.imgur.com/44WROfU.png)

  Make sure the module is app and name it something related, such as “Test Android”.

  ![Imgur](http://i.imgur.com/LEtylib.png)

  Then you can hit OK. Finally, press the run button, with your test configuration selected in the drop down, to run the tests.

  ![Run the tests](http://i.imgur.com/N8NIqYm.png)

  If you see any error messages when you try to run your tests, make sure there aren't any JUnit run configurations (we only want Android Tests). If there's anything under the JUnit dropdown in the left section, try deleting it (select it and click the - button).
