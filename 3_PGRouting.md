# 3. PGRouting

This is a Minimum Working Example of using PGRouting for graph analysis.

## Create a new database and extensions

First create a new database:

`CREATE DATABASE pgr_mwe;`

Then add the 'pgrouting' and 'postgis' extensions to it:

`CREATE EXTENSION pgrouting CASCADE;`

Here, `CASCADE` also installs the 'postgis' extension which 'pgrouting' depends on.

### Check the installed version

`SELECT * FROM pgr_version();`
 
This example was tested against version 3.8.

## Create a table of edges

There is no need to create nodes first, they are implied by the 'source' and 'target' values. The reverse_cost is not necessary unless you want the forward and reverse costs to be different. Running it undirected is done by setting a parameter in the function call.

```
CREATE TABLE edges(                                         
    id SERIAL PRIMARY KEY,
    source INTEGER NOT NULL,
    target INTEGER NOT NULL,
    cost DOUBLE PRECISION NOT NULL,
    reverse_cost DOUBLE PRECISION
);
```

After the table has been defined, data can be added to it like this:

```
INSERT INTO edges (source, target, cost, reverse_cost) VALUES
  (1, 2, 1.0, 1.0),
  (2, 4, 1.0, 1.0),
  (1, 3, 2.5, 2.5),
  (3, 4, 0.2, 0.2)
;
```

## Create a view of the nodes

Viewing the nodes can be done using the following code. Note that this creates a view of the nodes in the edges table, it doesn't create a table of the nodes. `UNION` will remove duplicate values:

```
CREATE VIEW nodes AS
SELECT source AS id FROM edges
UNION
SELECT target AS id FROM edges;
```

To list all tables in the database use `\dt`, to list all views use `dv` and to list all relations use `\d`.

## Run a basic Dijkstra analysis

Running a basic Dijkstra shortest path analysis can be run in a similar way to the following. There are many variations to this, so this is just shown as an example:

```
SELECT * FROM pgr_dijkstra(
  'SELECT id, source, target, cost, reverse_cost FROM edges',
  ARRAY[1,3], ARRAY[2,4],
  directed := false
);
```

In this case shortest paths from 1 & 3 to 2 & 4 will be found, treating the network as undirected.