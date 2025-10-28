# PostGIS to PostGIS topology

This example takes an existing PostGIS table of LineStrings and converts them directly to a PostGIS topology. Note that this version does not involve the creation of a topogeometry column in the LineStrings table.

### 1. Start the PostgreSQL shell

`psql`

List the available databases:

`\l`

### Connect to a database

`\c test_gis`

### List the tables in the database

`\dt`

### 2. Look at a sample of the LineString table

The actual geometries are held in EWKB binary blobs (?) so just running a `SELECT * FROM ...` will show that. Here we ask for a sample of up to three rows:

`SELECT * FROM street_segments LIMIT 3;`

To see a text representation of the geometries with their full decimal places we can use: 

`SELECT id, ST_AsText( geom ) FROM street_segments LIMIT 3;`

### Snap LineString geometries

First we'll snap the lines to a grid - what this amounts to is essentially rounding the coordinate values to to a coarser resolution. This is just to make sure that line end-points that look as if they coincide, do in fact coincide.

```
-- Optional: light cleanup/snap to grid (set grid to your tolerance scale)
UPDATE street_segments
SET geom = ST_SnapToGrid(geom, 0.1);  -- ~10cm in projected meters
```

Check this worked using the same text representation as above.

### Check for duplicate lines

### Check for zero length lines

```
SELECT ST_Length(geom) AS length
FROM street_segments
ORDER BY length
LIMIT 5;
```

### Add a spatial index

Speeds up identification of near or intersecting geometries when building the topology. Without a spatial index, each check requires a full table scan.

```
CREATE INDEX street_segments_gix
ON street_segments
USING gist (geom);
```

Calculate spatial extent and distribution statistics among others to enable the PostgreSQL planner to choose an appropriate plan:

```
ANALYZE street_segments;
```

To see these stats and geometry specific stats use:

```
SELECT *
FROM pg_stats
WHERE tablename = 'street_segments';
```

or

```
SELECT *
FROM geometry_columns
WHERE f_table_name = 'street_segments';
```

## Create a topology

### Check the PostGIS Topology extension is installed in the database

`CREATE EXTENSION IF NOT EXISTS postgis_topology;`

### Create the topology

```
-- name, srid, precision (aka tolerance), hasZ
SELECT topology.CreateTopology('street_segment_topo', 27700, 0.1, false);
```

### Check the topologysummary

Inspect the topology to check (note that the topology gets created within a schema that specifically stores topologies):

```
SELECT topology.TopologySummary('street_segment_topo');
```

## Create the elements of the topology based on the street_segment geometries

There seem to be a couple (?) of ways of doing this. The version described here just creates the basic topology from the given geometries, there aren't any attributes associated with the topology elements. The alternative version is more complete, adding references to the elements of the topology as a topogeometry column within the original table of the geometries used to create it. If it is just the topology that you are interested in, the distinction probably seems unnecessary. Considering attributes though (e.g. overlapping street names), or situations where a street segment also functions as a plot boundary (for example), the power of separating the attributes from the elements of the topology perhaps becomes a bit clearer.

### Option 1: Just build the topology (don't link it back to the geometry table)

```
-- make sure the tolerance (0.1) matches the tolerance the topology was created with
SELECT topology.TopoGeo_AddLineString('street_segment_topo', geom, 0.1)
FROM street_segments;
```

This outputs the number of rows that were processed. In this case, comparing the number of entries in the original geometry table with the number of eges in the topologysummary shows that they both number 125.

```
SELECT COUNT(*) FROM street_segments;
```

compared with

```
SELECT topology.TopologySummary('street_segment_topo');
```

## Validate the results

```
SELECT topology.ValidateTopology('street_topo'); -- should return no errors
```

As listed in the output this will:
- Check coincident nodes
- Check edges crossing nodes
- Check invalid or not-simple edges
- Check crossing edges
- Check edges start_node mismatch
- Check edges end_node mismatch
- Check faces without edges
- Check edge linking
- Build edge rings
- Construct geometry of all faces
