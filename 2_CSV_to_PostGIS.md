# 2. CSV to PostGIS

To list the existing databases `\l` or `\list` and to create a new database:

`CREATE DATABASE test_gis;`

Then connect to it using:

`\c test_gis;`

Create the PostGIS extension within it:

`CREATE EXTENSION IF NOT EXISTS postgis;`

## Create a table to hold the raw WKT

```
DROP TABLE IF EXISTS street_segments_raw CASCADE;
CREATE TABLE street_segments_raw (
    wkt text
);
```

`\copy street_segments_raw FROM 'barnsbury_street_segments_27700.csv' WITH (FORMAT csv, HEADER true)`

```
DROP TABLE IF EXISTS shop_points_raw CASCADE;
CREATE TABLE shop_points_raw (
    wkt text
);
```

Copy the street segment csv into the table:

`\copy shop_points_raw FROM 'barnsbury_shop_points_27700.csv' WITH (FORMAT csv, HEADER true)`

## Convert the raw text table

```
DROP TABLE IF EXISTS street_segments CASCADE;
CREATE TABLE street_segments (
    id  integer PRIMARY KEY,
    geom geometry(LineString, 27700)
);
```

```
INSERT INTO public.lines (name, geom)
SELECT name, ST_SetSRID(ST_GeomFromText(wkt), 4326)
FROM public.lines_raw;
```