---
layout: with-sidebar
title: Control File Configuration
bodyclass: homepage
---

### Contents
- [Basic control file setup](#setup-control)
    - [Header row / column list](#header-row)
    - [Date/time formatting](#date-time)
    - [Location column and geocoding configuration](#location-geocoding)
    - [Ignoring columns](#ignore-columns)
- [Complete control file settings](#complete-control)


{#setup-control}

<div class="well">
<strong>NOTICE:</strong> this guide pertains to using the replace method via FTP or HTTP (and not via Soda2).
</div>

### Basic control file setup

The control file is a JSON-formatted file that is used to configure a Standard DataSync job that uses the 'replace via FTP' or 'replace via HTTP' method. Control files are specific to the dataset you are updating.

An example of a typical control file:
```json
{
  "action" : "Replace",
  "csv" :
    {
      "useSocrataGeocoding" : true,
      "columns" : null,
      "skip" : 0,
      "fixedTimestampFormat" : ["ISO8601","MM/dd/yy","MM/dd/yyyy"],
      "floatingTimestampFormat" : ["ISO8601","MM/dd/yy","MM/dd/yyyy"],
      "timezone" : "UTC",
      "separator" : ",",
      "quote" : "\"",
      "encoding" : "utf-8",
      "emptyTextIsNull" : true,
      "trimWhitespace" : true,
      "trimServerWhitespace" : true,
      "overrides" : {}
    }
}
```

This guide will describe how to use the different options within the control file.

{#header-row}<p>&nbsp;</p>

#### Header row/column list

The `columns` and `skip` options enable configuration of how the columns within the CSV/TSV align with those of the dataset.

`columns`: List of column names in the following format `["col_id1","col_id2",..]`. If it’s `null` then the first line of the CSV/TSV after any skipped records is used. If specified, it must be an array of strings, and must not contain nulls.
**IMPORTANT NOTE:** the column names, whether provided in “columns” or in the first row of the CSV/TSV, must be column identifiers (API field names), not the display name of the columns.

`skip`: Specifies the number of rows to skip before reaching the header.

**Common combinations of `columns` and `skip`:**

If the first line of the CSV/TSV is the list of column identifiers:
```
"columns": null,
"skip": 0,
```

If the first line of the CSV is the columns incorrectly formatted, for example with human-readable names instead of column identifiers, for example:
```
"columns": ["first_name","last_name","age"],
"skip": 1,
```

If the first line of the CSV/TSV is data (there is no header row), for example you would use:
```
"columns": ["first_name","last_name","age"],
"skip": 0,
```

{#date-time}<p>&nbsp;</p>

#### Date/time formatting

##### Timestamp Format Options

The `floatingTimestampFormat` and `fixedTimestampFormat` options specify how date/time data is formatted in the CSV/TSV file. `floatingTimestampFormat` applies to ("Date & Time" datatype columns) and `fixedTimestampFormat` functions in the same way but applies to Fixed Timestamps ("Date & Time (with timezone)" datatype columns). If the format does not specify a time zone, the zone may be given via the `timezone` option.  If no zone information is provided, UTC is assumed.

Both `floatingTimestampFormat` and `fixedTimestampFormat` accept a string (e.g. "ISO8601") or a JSON-formatted list of formats including "ISO8601" and any date/time "Joda time" format-string. Joda time syntax is documented in detail here: [http://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html](http://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html)

Example syntax to accept four of the most common date/time formats:
```
"fixedTimestampFormat" : ["ISO8601","MM/dd/yy","MM/dd/yyyy","dd-MMM-yyyy"],
"floatingTimestampFormat" : ["ISO8601","MM/dd/yy","MM/dd/yyyy","dd-MMM-yyyy"],
```

This would accept any of the following example date/time data values: "2014-04-22", "2014-04-22T05:44:38", "04/22/2014", "4/22/2014", "4/22/14", and "22-Apr-2014".

If you want to allow a date with or without a time value (e.g. both "2014-04-22" and "2014-04-22 9:30:00"), you would use:
```
"fixedTimestampFormat" : ["yyyy-MM-dd", "yyyy-MM-dd HH:mm:ss"],
"floatingTimestampFormat" : ["yyyy-MM-dd", "yyyy-MM-dd HH:mm:ss"],
```

##### Timezone option
`timezone` specifies the timezones for FixedTimestamps ("Date & Time (with timezone)" columns). This only has an effect if the timestamp format does not specify a time zone.

You can set this to one of the following:
1. "UTC"
2. A timezone name (e.g. "US/Pacific").  The list of accepted names can be found at [http://joda-time.sourceforge.net/timezones.html](http://joda-time.sourceforge.net/timezones.html). *Please avoid the 3-letter variants as these are ambiguous (e.g. MST is both Mountain Standard Time and Malaysia Standard Time).*

{#location-geocoding}<p>&nbsp;</p>

#### Location column and geocoding configuration

The `syntheticLocations` option allows configuring a Location datatype column to populate from address, city, state, zipcode or latitude/longitude data within existing columns of the CSV/TSV.

For example:
```
 "syntheticLocations" : {
   "location_col_id" : {
     "address" : "address_col_id",
     "city" : "city_col_id",
     "state" : "state_col_id",
     "zip" : "zipcode_col_id",
     "latitude" : "lat_col_id",
     "longitude" : "lng_col_id"
   }
 }
```

All of the following are optional: "address", "city", "state", "zip", "latitude", and "longitude".
Those that are are not provided are not filled in on the generated location.  The values are
field names of columns that must exist in the CSV. In the above example, a Location datatype column with the identifier `location_col_id` would pull in the "address" from the column with identifier `address_col_id`, the "city" from column with identifier `city_col_id`, and so on.

The synthetic location `location_col_id` should not be present in the CSV.  If it is, you can ignore this column using the `ignoreColumns option`.

When you provide any combination of location information but do not fill in latitude or longitude then Socrata will do geocoding automatically to generate the latitude and longitude values. Read [this guide](http://support.socrata.com/entries/27849363-Location-Information-Data-which-can-be-geocoded) for more detailed information on the Location column.

<div class="well">
<strong>IMPORTANT:</strong> If you are providing the latitude and longitude values directly to the Location column (i.e. you are NOT using Socrata's geocoding), you need to set the `useSocrataGeocoding` option to `false`.  If you are not providing the latitude and longitude, you need to set this to `true`.  This will minimize the number of perceived changes to the dataset, decreasing the time it takes to complete your job.  If you are constructing multiple Location columns and they require different `useSocrataGeocoding` settings, you may use the `overrides` option.
</div>

{#ignore-columns}<p>&nbsp;</p>

#### Ignoring Columns

The `ignoreColumns` options you to exclude columns within the CSV/TSV.  This may be necessary if the dataset lacks a column within the CSV or if a synthetic location is provided in the CSV, but you would still like it constructed from individual address fields.

`ignoreColumns`: List of column names in the following format `["col_id1","col_id2",..]`. These must be present in `columns`.


{#complete-control}<p>&nbsp;</p>

### Complete control file settings

The control file is comprised of the
- [Action setting](#action-setting)
- [File type settings](#filetype-settings)

{#action-setting}<p>&nbsp;</p>

#### Actions

The action is given by one of the following strings:

| Option    | Explanation
| ------------- | ------------------------------
| Replace |  Use if the the CSV/TSV represents the desired new state for the dataset. Delta Importer will calculate the minimal number of upserts necessary and make it so
| Append | Use if the CSV/TSV contains rows to append to append to the dataset. If the dataset does not have a RowID, then all rows in the CSV are added, even if they duplicate existing rows. If the datset does have a RowID, then matching row IDs actually cause an Update operation and missing and new row IDs cause the addition of new rows.  *This is not yet supported in DataSync.*
| Delete | Use if the CSV/TSV contains row IDs of rows to delete. This option requires that a [row indentifier](http://dev.socrata.com/docs/row-identifiers.html) be set on the dataset.


{#filetype-settings}<p>&nbsp;</p>

#### CSV or TSV Settings

The following are options available to both CSV files or TSV files within the `csv` or `tsv` object:

| Option    | Explanation
| ------------- | ------------------------------
| encoding | "utf-8", or any other encoding that the JVM understands.  The list is available at /datasync/charsets.json from your socrata domain (e.g. https://opendata.socrata.com/datasync/charsets.json).
| separator | Field separator. Typically "," or "\t".
| quote | Used to quote values which contain the separator character. Separators between quotes will be treated as part of the value. Typical values are "\"" for double-quotes, "'" for single-quotes and "\u0000" for no quote character.
| escape | Used to specify the escape character. Typically this is "\\", a single backslash. Note that in CSV’s you can always escape a double-quote by doubling it up (e.g. "This is just one "" string.").
| columns | JSON list of column names. If null then the first line of the csv after any skipped records is used. If specified, it must be an array of strings and must not contain nulls. Note that the column names, whether provided in “columns” or in the first row of the CSV, must match the API field name, not the display name of the columns.
| ignoreColumns | Specifies any columns in the CSV/TSV file that are to be ignored. These must be given as an array of strings and must be present in `columns`.
| skip | Specifies the number of rows to skip. The first row that will be read is `skip` + 1; whether this is treated as data or a header row depends on how `columns` is set.  If `columns` is null, this row will be treated as the header; otherwise it is treated as data.
| trimWhitespace | Trims leading and trailing whitespace before inserting the data into the dataset. Note that it also trims quoted values so " Foo" would be converted to "Foo".
| trimServerWhitespace | Trims leading and trailing whitespace that already exists in the dataset. This flag is generally only necessary if data was previously added to the dataset with whitespace (due to trimWhitespace being false).
| useSocrataGeocoding | This is relevant only to Location columns and controls how comparisons are made between values in the CSV/TSV file and the data we have stored in our servers.  If you are not providing the latitude and longitude in the synthetic location (e.g. Socrata will geocode the location), set this to "true" to minimize perceived changes to your data. If you are providing the latitude and longitude in the synthetic location, set this to "false".
| emptyTextIsNull | For old backend datasets, set this to “true”.  The old backend converts empty strings to null and if “false”, every empty string will be viewed as a change to your dataset, slowing down the upsert time considerably. For new backend datasets, this will affect how data is imported into text fields. If true, then empty text (not whitespace) will be treated as NULL. If false, it will be treated as the empty string.
| floatingTimestampFormat | Specifies how Floating Timestamps (“Date & Time” columns) are interpreted. Typical values are "ISO8601" or "yyyy-MM-dd".  Any [joda-formated string](http://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html) is acceptable. If you want to allow multiple formats to be accepted, then you can specify a list of values rather than a single value (e.g. ["ISO8601", "MM.dd.yyyy"]).
| fixedTimestampFormat | Same as floatingTimestampFormat but for Fixed Timestamps (“Date & Time (with timezone)”).  If the format does not specify a time zone, the zone named by the `timezone` field is used.
| dropUninterpretbleRows | If specified limits the amount of uninterpretable data allowable in each row. (Vaguely speaking these are the values grayed out in the UI). Options are "Never", in which case a job will fail if the CSV/TSV has any uninterpretable values, and "TenPercent", in which case 10% of values can be uninterpretable before the job fails.
| timezone | Specifies the timezones for FixedTimestamps (“Date & Time (with timezone)” columns).  This only has an effect if the timestamp format does not specify a time zone. Typical values are "UTC" or "US/Pacific".  A list of accepted names is at [http://joda-time.sourceforge.net/timezones.html](http://joda-time.sourceforge.net/timezones.html). *Please avoid the 3-letter variants as these are ambiguous (e.g. MST is both Mountain Standard Time and Malaysia Standard Time).
| syntheticLocations | Allows transformation of multiple columns into one or more Location columns during insert. See See the [Location column and geocoding configuration](#location-geocoding) section for an example.
| overrides | A map whose keys are field names, and whose values are objects containing per-column overrides for the `timestampFormat`, `timezone`, `emptyTextIsNull`, `trimWhitespace`, `trimServerWhitespace` and `useSocrataGeocoding` settings.  Note that “timestampFormat” applies to both fixed and floating timestamps. For an example, see below:


Example of using column-level overrides:

```json
{
  "action" : "Replace",
  "csv" : {
    "useSocrataGeocoding" : true,
    "fixedTimestampFormat" : "ISO8601",
    "floatingTimestampFormat" : "ISO8601",
    "timezone" : "UTC",
    "overrides" : {
      "my_time_column" : {
        "timestampFormat" : "YYYY-MM-dd HH:mm:ss",
        "timezone" : "US/Central"
      },
      "my_text_column" : {
        "emptyTextIsNull" : true
      }
      "my_location_column" {
        "useSocrataGeocoding" : false
      }
    }
  }
}
```




