# PostGIS to PostGIS topology

This example takes an existing PostGIS table of LineStrings and converts them directly to a PostGIS topology. Note that this version does not involve the creation of a topogeometry column in the LineStrings table.

## 1. Start `psql`

`psql`

List the available databases:

`\l`

## Connect to database

`\c test_gis`

## List the tables in the database

`\dt`

## 2. Look at a sample of the LineString table

The actual geometries are held in EWKB binary blobs (?) so just running a `SELECT * FROM ...` will show that. Here we ask for a sample of up to three rows:

`SELECT * FROM street_segments LIMIT 3;`

To see a text representation of the geometries with their full decimal places we can use: 

`SELECT id, ST_AsText( geom ) FROM street_segments LIMIT 3;`

## Snap LineString geometries

First we'll snap the lines to a grid - what this amounts to is essentially rounding the coordinate values to to a coarser resolution. This is just to make sure that line end-points that look as if they coincide, do in fact coincide.

```
-- Optional: light cleanup/snap to grid (set grid to your tolerance scale)
UPDATE street_segments
SET geom = ST_SnapToGrid(geom, 0.1);  -- ~10cm in projected meters
```

Check this worked using the same text representation as above.

Double check this hasn't created any zero length lines:

```
SELECT ST_Length(geom) AS length
FROM street_segments
ORDER BY length
LIMIT 5;
```


## Create a topogeometry schema

