# 6. Extracting information from the topology

## 6.1 The basic topology layers faces, edges, nodes

## 6.2 True face geometry

As described above the basic topology doesn't store the geometry of the faces, it stores the minimum amount of information needed to create the face geometries from the nodes and edges, and also a bounding box representation.

To get the geometry of the faces into QGIS we need to use the function `topology.ST_GetFaceGeometry()` to create them.

To start with, the idea is that we don't want to generate a permanent set of Polygons based on the current state of the topolgy. Ideally the Poloygons will be created on-the-fly ans so will reflect any changes
