# Contract
* It is an agreement between data-model, storage, and views, presentation, **describing how information is accessed.**
* Implemented in a contract class that contains,
  * a list of constants,
  * typically database column names, that are used to associate data from the data source within the UI of an application.

![Imgur](http://i.imgur.com/jVQwoI6.png?1)

* Each column in the data source represents an UI element
* [Example - contactscontract](http://developer.android.com/intl/zh-cn/reference/android/provider/ContactsContract.html)
  * It has a column for the display name
  * Flags for the contact
  * columns for IDs that are used to link the contact to data in other tables
  * Columns for URIs that point to images associated with the contact
* In this example, ([udacity- sunshine](https://github.com/udacity/Sunshine-Version-2))
  * Contract is used to both create and interact with database tables
  * Need to define it at the start of building our database
* Find what columns you need from the [wireframe prototypes](http://developer.android.com/images/training/app-navigation-wireframing-wires-phone.png) of your app.

### Separate Tables

![Imgur](http://i.imgur.com/ZRpLnlc.png)

* **Why? :**
  * *Avoid duplicates:* As there will be many weather entries with same location
  * Update location without updating weather every time
* **Location ID :** Defined with a constraint that is identified as a [foreign key](http://www.w3schools.com/sql/sql_foreignkey.asp).
  * Which means its a unique ID from another table.
* **What does this look like?**

![Imgur](http://i.imgur.com/VGjtS3d.png)

* **Inner Join:**
  * We can use this location ID to perform an operation known as an [inner join](http://www.tutorialspoint.com/sqlite/sqlite_using_joins.htm)
  * Inner joins combine the data queried from two rows in different tables into one row in the result
  * We can get any combination of columns from both tables

![Imgur](http://i.imgur.com/lT62lbO.png)

* `_ID` is the primary key of the location table and should be set to [autoincrement](http://www.tutorialspoint.com/sqlite/sqlite_using_autoincrement.htm)
* `FOREIGN KEY` constraint enforces that the `LOCATION ID` is a valid `_ID`
  * In order to insert in weather table we have to insert in the location table

**Code**

``` java
public static final class LocationEntry implements BaseColumns {
        public static final String TABLE_NAME = "location";
        // The location setting string is what will be sent to openweathermap
        // as the location query.
        public static final String COLUMN_LOCATION_SETTING = "location_setting";

        // Human readable location string, provided by the API.  Because for styling,
        // "Mountain View" is more recognizable than 94043.
        public static final String COLUMN_CITY_NAME = "city_name";

        // In order to uniquely pinpoint the location on the map when we launch the
        // map intent, we store the latitude and longitude as returned by openweathermap.
        public static final String COLUMN_COORD_LAT = "coord_lat";
        public static final String COLUMN_COORD_LONG = "coord_long";

    }

    /* Inner class that defines the table contents of the weather table */
    public static final class WeatherEntry implements BaseColumns {

        public static final String TABLE_NAME = "weather";

        // Column with the foreign key into the location table.
        public static final String COLUMN_LOC_KEY = "location_id";
        // Date, stored as long in milliseconds since the epoch
        public static final String COLUMN_DATE = "date";
        // Weather id as returned by API, to identify the icon to be used
        public static final String COLUMN_WEATHER_ID = "weather_id";

        // Short description and long description of the weather, as provided by API.
        // e.g "clear" vs "sky is clear".
        public static final String COLUMN_SHORT_DESC = "short_desc";

        // Min and max temperatures for the day (stored as floats)
        public static final String COLUMN_MIN_TEMP = "min";
        public static final String COLUMN_MAX_TEMP = "max";

        // Humidity is stored as a float representing percentage
        public static final String COLUMN_HUMIDITY = "humidity";

        // Humidity is stored as a float representing percentage
        public static final String COLUMN_PRESSURE = "pressure";

        // Windspeed is stored as a float representing windspeed  mph
        public static final String COLUMN_WIND_SPEED = "wind";

        // Degrees are meteorological degrees (e.g, 0 is north, 180 is south).  Stored as floats.
        public static final String COLUMN_DEGREES = "degrees";
    }
```
* We miss a column defining the unique ID
  * we are implementing the class `BaseColumns`
  * which contains a constant called `_id`
  * By adding an `_id` to our table as a primary key,
    * we can utilize various android helper classes that rely on the `_id` column being created
* `COLUMN_LOC_KEY` is the **FOREIGN KEY** that points to records in location table
