# 6. Extracting information from the topology

## 6.1 The basic topology layers faces, edges, nodes

The basic elements of the topogeometry can be loaded into QGIS.

### node

### edge

### edge_data

### face

Contains a `face_id` attribute, and the geomtry is the bounding box of each face.

## 6.2 Creating face geometries

The basic topology doesn't store the face geometries. It stores the minimum amount of information needed to create the face geometries from the nodes and edges (and a bounding box representation).

To get the geometry of the faces into QGIS, the function `topology.ST_GetFaceGeometry()` creates them.

### Using a view

So that the geometries of the faces reflect the current state of the underlying topogeometry, we will create a ['live'] view of the faces which will update as the topogeometry updates:

```
CREATE VIEW my_topo_faces AS
SELECT 
  face_id,
  topology.ST_GetFaceGeometry('street_segment_topo', face_id) AS geom
FROM street_segment_topo.face
WHERE face_id > 0; -- exclude the unbounded face
```

To subsequently check the SQL query behind an existing view we can use:

```
SELECT  pg_get_viewdef('my_topo_faces', true); -- where my_topo_faces is the name of the view
```

## 6.3 Measuring and storing information about the edges

The topology itself is not the place to store attribute information, even about the elements of the topologies themselves.

It is best to store that information in a separate attribute table linked to the elements in the topology.

In this case the topology contains:

- street_segment_topo.node
- street_segment_topo.edge_data
- street_segment_topo.face

### 6.3.1 Create a table to hold the attributes

```
CREATE TABLE street_segment_edge_attrs (
  edge_id integer PRIMARY KEY,
  length double precision,
  bearing double precision
);
```

### 6.3.2 Link it by foreign key to the topology edges

The foreign key column is the column of values in the child table that point to values in the parent table:

```
ALTER TABLE street_segment_edge_attrs  -- modify an existing table
  ADD CONSTRAINT fk_edge               -- add a constraint to the table (the name is arbitrary)
  FOREIGN KEY (edge_id)                -- the column that will hold the foreign key
  REFERENCES street_segment_topo.edge_data (edge_id); -- the parent column the foreign key must be in
```

This is doing a couple of things:

1. It makes it a requirement that every value in the foreign key column must exist in the parent table's target column. Values that do not exist in the target column will be rejected.
2. It also means that prior to deleting the parent column (e.g. `street_segment_topo.edge_data`) PostgreSQL will check for any child tables that have a foreign key relationship to prevent breaking them.

### 6.3.3 Compute and Populate the Attributes

Note, this is not a view, this is a table, that would need updating if the topology changes.

```
INSERT INTO street_segment_edge_attrs (edge_id, length, bearing)
SELECT
  e.edge_id,
  ST_Length(e.geom) AS length,
  degrees(ST_Azimuth(ST_StartPoint(e.geom), ST_EndPoint(e.geom))) AS bearing
FROM street_segment_topo.edge_data AS e;
````

To create it as a table (which would update with changes to the topology):

```
CREATE OR REPLACE VIEW edge_metrics_v AS
SELECT
  e.edge_id,
  ST_Length(e.geom)::double precision                        AS length_m,
  degrees(ST_Azimuth(ST_StartPoint(e.geom), ST_EndPoint(e.geom))) AS bearing_deg
FROM street_segment_topo.edge_data e;
```

There is an issue with this which is that the two tables are still separate - the topology geometries and the attributes. To combine them, a solution is to create another view based on a join e.g.:

```
CREATE OR REPLACE VIEW city_edge_view AS
SELECT
  e.edge_id,
  e.geom,
  a.length,
  a.bearing
FROM city_topo.edge_data AS e
JOIN city_edge_attrs AS a
  ON e.edge_id = a.edge_id;
```

However, to avoid this stacking of views for this simple example, a better approach is just to include the geometry in the attribute view to start with and then load that into QGIS (for example).

```
CREATE OR REPLACE VIEW street_segment_edge_attrs AS
SELECT
  e.edge_id,
  e.geom,
  ST_Length(e.geom) AS length,
  degrees(ST_Azimuth(ST_StartPoint(e.geom), ST_EndPoint(e.geom))) AS bearing
FROM street_segment_topo.edge_data AS e;
```
