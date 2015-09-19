# Test
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

## Steps

![Imgur](http://i.imgur.com/GBu7hyX.png)

## Code

  ``` java
  package com.example.android.sunshine.app.data;

  import android.content.ContentValues;
  import android.database.Cursor;
  import android.database.sqlite.SQLiteDatabase;
  import android.test.AndroidTestCase;

  import java.util.HashSet;

  public class TestDb extends AndroidTestCase {

      public static final String LOG_TAG = TestDb.class.getSimpleName();

      // Since we want each test to start with a clean slate
      void deleteTheDatabase() {
          mContext.deleteDatabase(WeatherDbHelper.DATABASE_NAME);
      }

      /*
          This function gets called before each test is executed to delete the database.  This makes
          sure that we always have a clean test.
       */
      public void setUp() {
          deleteTheDatabase();
      }

      /*
          Note that this only tests that the Location table has the correct columns, since we
          give you the code for the weather table.  This test does not look at the
       */
      public void testCreateDb() throws Throwable {
          // build a HashSet of all of the table names we wish to look for
          // Note that there will be another table in the DB that stores the
          // Android metadata (db version information)
          final HashSet<String> tableNameHashSet = new HashSet<String>();
          tableNameHashSet.add(WeatherContract.LocationEntry.TABLE_NAME);
          tableNameHashSet.add(WeatherContract.WeatherEntry.TABLE_NAME);

          mContext.deleteDatabase(WeatherDbHelper.DATABASE_NAME);
          SQLiteDatabase db = new WeatherDbHelper(
                  this.mContext).getWritableDatabase();
          assertEquals(true, db.isOpen());

          // have we created the tables we want?
          Cursor c = db.rawQuery("SELECT name FROM sqlite_master WHERE type='table'", null);

          assertTrue("Error: This means that the database has not been created correctly",
                  c.moveToFirst());

          // verify that the tables have been created
          do {
              tableNameHashSet.remove(c.getString(0));
          } while( c.moveToNext() );

          // if this fails, it means that your database doesn't contain both the location entry
          // and weather entry tables
          assertTrue("Error: Your database was created without both the location entry and weather entry tables",
                  tableNameHashSet.isEmpty());

          // now, do our tables contain the correct columns?
          c = db.rawQuery("PRAGMA table_info(" + WeatherContract.LocationEntry.TABLE_NAME + ")",
                  null);

          assertTrue("Error: This means that we were unable to query the database for table information.",
                  c.moveToFirst());

          // Build a HashSet of all of the column names we want to look for
          final HashSet<String> locationColumnHashSet = new HashSet<String>();
          locationColumnHashSet.add(WeatherContract.LocationEntry._ID);
          locationColumnHashSet.add(WeatherContract.LocationEntry.COLUMN_CITY_NAME);
          locationColumnHashSet.add(WeatherContract.LocationEntry.COLUMN_COORD_LAT);
          locationColumnHashSet.add(WeatherContract.LocationEntry.COLUMN_COORD_LONG);
          locationColumnHashSet.add(WeatherContract.LocationEntry.COLUMN_LOCATION_SETTING);

          int columnNameIndex = c.getColumnIndex("name");
          do {
              String columnName = c.getString(columnNameIndex);
              locationColumnHashSet.remove(columnName);
          } while(c.moveToNext());

          // if this fails, it means that your database doesn't contain all of the required location
          // entry columns
          assertTrue("Error: The database doesn't contain all of the required location entry columns",
          locationColumnHashSet.isEmpty());
          db.close();
      }

      public void testLocationTable() {
          insertLocation();
      }

      public void testWeatherTable() {
          // First insert the location, and then use the locationRowId to insert
          // the weather. Make sure to cover as many failure cases as you can.

          // Instead of rewriting all of the code we've already written in testLocationTable
          // we can move this code to insertLocation and then call insertLocation from both
          // tests. Why move it? We need the code to return the ID of the inserted location
          // and our testLocationTable can only return void because it's a test.

          long locationRowId = insertLocation();

          // Make sure we have a valid row ID.
          assertFalse("Error: Location Not Inserted Correctly", locationRowId == -1L);

          // First step: Get reference to writable database
          // If there's an error in those massive SQL table creation Strings,
          // errors will be thrown here when you try to get a writable database.
          WeatherDbHelper dbHelper = new WeatherDbHelper(mContext);
          SQLiteDatabase db = dbHelper.getWritableDatabase();

          // Second Step (Weather): Create weather values
          ContentValues weatherValues = TestUtilities.createWeatherValues(locationRowId);

          // Third Step (Weather): Insert ContentValues into database and get a row ID back
          long weatherRowId = db.insert(WeatherContract.WeatherEntry.TABLE_NAME, null, weatherValues);
          assertTrue(weatherRowId != -1);

          // Fourth Step: Query the database and receive a Cursor back
          // A cursor is your primary interface to the query results.
          Cursor weatherCursor = db.query(
                  WeatherContract.WeatherEntry.TABLE_NAME,  // Table to Query
                  null, // leaving "columns" null just returns all the columns.
                  null, // cols for "where" clause
                  null, // values for "where" clause
                  null, // columns to group by
                  null, // columns to filter by row groups
                  null  // sort order
          );

          // Move the cursor to the first valid database row and check to see if we have any rows
          assertTrue( "Error: No Records returned from location query", weatherCursor.moveToFirst() );

          // Fifth Step: Validate the location Query
          TestUtilities.validateCurrentRecord("testInsertReadDb weatherEntry failed to validate",
                  weatherCursor, weatherValues);

          // Move the cursor to demonstrate that there is only one record in the database
          assertFalse( "Error: More than one record returned from weather query",
                  weatherCursor.moveToNext() );

          // Sixth Step: Close cursor and database
          weatherCursor.close();
          dbHelper.close();
      }

      public long insertLocation() {
          // First step: Get reference to writable database
          // If there's an error in those massive SQL table creation Strings,
          // errors will be thrown here when you try to get a writable database.
          WeatherDbHelper dbHelper = new WeatherDbHelper(mContext);
          SQLiteDatabase db = dbHelper.getWritableDatabase();

          // Second Step: Create ContentValues of what you want to insert
          // (you can use the createNorthPoleLocationValues if you wish)
          ContentValues testValues = TestUtilities.createNorthPoleLocationValues();

          // Third Step: Insert ContentValues into database and get a row ID back
          long locationRowId;
          locationRowId = db.insert(WeatherContract.LocationEntry.TABLE_NAME, null, testValues);

          // Verify we got a row back.
          assertTrue(locationRowId != -1);

          // Data's inserted.  IN THEORY.  Now pull some out to stare at it and verify it made
          // the round trip.

          // Fourth Step: Query the database and receive a Cursor back
          // A cursor is your primary interface to the query results.
          Cursor cursor = db.query(
                  WeatherContract.LocationEntry.TABLE_NAME,  // Table to Query
                  null, // all columns
                  null, // Columns for the "where" clause
                  null, // Values for the "where" clause
                  null, // columns to group by
                  null, // columns to filter by row groups
                  null // sort order
          );

          // Move the cursor to a valid database row and check to see if we got any records back
          // from the query
          assertTrue( "Error: No Records returned from location query", cursor.moveToFirst() );

          // Fifth Step: Validate data in resulting Cursor with the original ContentValues
          // (you can use the validateCurrentRecord function in TestUtilities to validate the
          // query if you like)
          TestUtilities.validateCurrentRecord("Error: Location Query Validation Failed",
                  cursor, testValues);

          // Move the cursor to demonstrate that there is only one record in the database
          assertFalse( "Error: More than one record returned from location query",
                  cursor.moveToNext() );

          // Sixth Step: Close Cursor and Database
          cursor.close();
          db.close();
          return locationRowId;
      }
  }
  ```
* By using `AndroidTestCase`, the field `mContext` will provide you with a valid Context to pass to SQLiteOpenHelper.
* `Cursor` -  [Docs](http://developer.android.com/intl/zh-cn/reference/android/database/Cursor.html)
  * `moveToFirst()`
    * Move the cursor to the first row.
    * Return false if the cursor is empty. (Returns - whether the move succeeded)
  * `moveToNext()`
    * Move the cursor to the next row.
    * Return false if the cursor is already past the last entry in the result set. (Returns - whether the move succeeded.)
  * `getColumnIndex(String columnName)`
    * Returns the zero-based index for the given column name, or -1 if the column doesn't exist.
  * `getString(int columnIndex)`
    * Returns the value of the requested column as a String.

  ![Imgur](http://i.imgur.com/q2v9t09.png)
  
* `PRAGMA table_info(table-name);`

This pragma returns one row for each column in the named table. Columns in the result set include the column name, data type, whether or not the column can be NULL, and the default value for the column.

### SQLiteDatabase

[DOCS](http://developer.android.com/reference/android/database/sqlite/SQLiteDatabase.htm)

* `db.rawQuery` : `public Cursor rawQuery (String sql, String[] selectionArgs)`
  * Runs the provided SQL and returns a Cursor over the result set.
  * *selectionArgs usage*
    * `rawQuery("SELECT id, name FROM people WHERE name = ? AND id = ?", new String[] {"Santhosh", "7"});`
    * if you dont want it, use `null`

* `db.insert` : `public long insert (String table, String nullColumnHack, ContentValues values)`
  * Convenience method for inserting a row into the database.
  * **Returns :**
    * the row ID of the newly inserted row, or -1 if an error occurred
  * **Parameters :**
    * `table`	the table to insert the row into
    * `nullColumnHack`	optional; may be null. SQL doesn't allow inserting a completely empty row    without naming at least one column name. If your provided values is empty, no column names are known and an empty row can't be inserted. If not set to null, the nullColumnHack parameter provides the name of nullable column name to explicitly insert a NULL into in the case where your values is empty.
    * `values`	this map contains the initial column values for the row. The keys should be the column names and the values the column values
* `public Cursor query (String table, String[] columns, String selection, String[] selectionArgs, String groupBy, String having, String orderBy)`
  * **Returns**
    * A Cursor object, which is positioned before the first entry.
  * **Parameters**
    * `table`	The table name to compile the query against.
    * `columns`	A list of which columns to return. Passing null will return all columns, which is discouraged to prevent reading data from storage that isn't going to be used.
    * `selection`	A filter declaring which rows to return, formatted as an SQL WHERE clause (excluding the WHERE itself). Passing null will return all rows for the given table.
    * `selectionArgs`	You may include ?s in selection, which will be replaced by the values from selectionArgs, in order that they appear in the selection. The values will be bound as Strings.
    * `groupBy`	A filter declaring how to group rows, formatted as an SQL GROUP BY clause (excluding the GROUP BY itself). Passing null will cause the rows to not be grouped.
    * `having`	A filter declare which row groups to include in the cursor, if row grouping is being used, formatted as an SQL HAVING clause (excluding the HAVING itself). Passing null will cause all row groups to be included, and is required when row grouping is not being used.
    * `orderBy`	How to order the rows, formatted as an SQL ORDER BY clause (excluding the ORDER BY itself). Passing null will use the default sort order, which may be unordered.
  * [SO example](http://stackoverflow.com/questions/10600670/sqlitedatabase-query-method)
  * Another example

  ![Imgur](http://i.imgur.com/zXEU8i9.png)   

### TestUtilities

**ContentValues**

``` java
static ContentValues createNorthPoleLocationValues() {
    // Create a new map of values, where column names are the keys
    ContentValues testValues = new ContentValues();
    testValues.put(WeatherContract.LocationEntry.COLUMN_LOCATION_SETTING, TEST_LOCATION);
    testValues.put(WeatherContract.LocationEntry.COLUMN_CITY_NAME, "North Pole");
    testValues.put(WeatherContract.LocationEntry.COLUMN_COORD_LAT, 64.7488);
    testValues.put(WeatherContract.LocationEntry.COLUMN_COORD_LONG, -147.353);

    return testValues;
}
static ContentValues createWeatherValues(long locationRowId) {
       ContentValues weatherValues = new ContentValues();
       weatherValues.put(WeatherContract.WeatherEntry.COLUMN_LOC_KEY, locationRowId);
       weatherValues.put(WeatherContract.WeatherEntry.COLUMN_DATE, TEST_DATE);
       weatherValues.put(WeatherContract.WeatherEntry.COLUMN_DEGREES, 1.1);
       weatherValues.put(WeatherContract.WeatherEntry.COLUMN_HUMIDITY, 1.2);
       weatherValues.put(WeatherContract.WeatherEntry.COLUMN_PRESSURE, 1.3);
       weatherValues.put(WeatherContract.WeatherEntry.COLUMN_MAX_TEMP, 75);
       weatherValues.put(WeatherContract.WeatherEntry.COLUMN_MIN_TEMP, 65);
       weatherValues.put(WeatherContract.WeatherEntry.COLUMN_SHORT_DESC, "Asteroids");
       weatherValues.put(WeatherContract.WeatherEntry.COLUMN_WIND_SPEED, 5.5);
       weatherValues.put(WeatherContract.WeatherEntry.COLUMN_WEATHER_ID, 321);

       return weatherValues;
}
```
**Validation**
``` java
static void validateCurrentRecord(String error, Cursor valueCursor, ContentValues expectedValues) {
    Set<Map.Entry<String, Object>> valueSet = expectedValues.valueSet();
    for (Map.Entry<String, Object> entry : valueSet) {
        String columnName = entry.getKey();
        int idx = valueCursor.getColumnIndex(columnName);
        assertFalse("Column '" + columnName + "' not found. " + error, idx == -1);
        String expectedValue = entry.getValue().toString();
        assertEquals("Value '" + entry.getValue().toString() +
                "' did not match the expected value '" +
                expectedValue + "'. " + error, expectedValue, valueCursor.getString(idx));
    }
}
```
