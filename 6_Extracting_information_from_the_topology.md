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


