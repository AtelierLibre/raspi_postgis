# 4. Creating a PostGIS topology

## Create the extension

***Note: the extensions have to be enabled within each database.***

Within a database:

`CREATE EXTENSION postgis_topology;`

To check the version of PostGIS:

`SELECT PostGIS_Full_Version();`

At the time of writing this was version 3.6.0.

### The topology schema

Adding the topology extension to a database will automatically create a new schema called 'topology' - this keeps track of the topologies.

You can list the schemas in the database with:

`\dn`

Running `SHOW search_path;` should also show that the commands from 'topology' will be accessible without having to prepend `topology.`.

## Creating a topology

Running the command:

`SELECT CreateTopology('test_topology',4326);`

will create a new schema within the current database called 'test_topology'. Each topology is created within its own schema and the SRID applies to all tables within the schema.

Each topology schema will also be registered in the 'topology' schema's 'topology' table.

### Tables within the topology schema

The 'test_topology' schema will have four tables:

- node
- edge_data
- face
- relation

You can list these by running `\dt test_topology.*`.

The 'edge_data' table is the one that holds all of the information for how the elements are linked to each other. In general this is interacted with via the 'edge' view.

The 'face' table contains the 'face_id' and the 'mbr' (minimum bounding rectangle) of each face. The actual geometry of a face is constructed according to the sequence of edges that make it up.

## Adding topogeometries

The code below adds three LineStrings to the topogeometry, their endpoints coincide and so should form a triangular face.

```
SELECT TopoGeo_AddLineString(
    'test_topology',
    ST_GeomFromText(
        'LINESTRING(
            -0.1276 51.5072,
            -2.2426 53.4808
        )',
        4326
    )
);

SELECT TopoGeo_AddLineString(
    'test_topology',
    ST_GeomFromText(
        'LINESTRING(
            -2.2426 53.4808,
            -2.5879 51.4545
        )',
        4326
    )
);

SELECT TopoGeo_AddLineString(
    'test_topology',
    ST_GeomFromText(
        'LINESTRING(
            -2.5879 51.4545,
            -0.1276 51.5072
        )',
        4326
    )
);
```

Check that a new face was created by running

`SELECT * FROM test_topology.face;`

It is also worth looking at the 'edge_data' table to see how it links together nodes and faces.