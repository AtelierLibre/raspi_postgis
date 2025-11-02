# 6. Extracting information from the topology

## 6.1 The basic topology layers faces, edges, nodes

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
