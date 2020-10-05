---
title: Spatial Indexes
summary: How CockroachDB uses spatial indexes for efficiently storing and querying spatial data.
toc: true
---

{% include {{page.version.version}}/sql/spatial-support-new.md %}

This page describes CockroachDB's approach to indexing spatial data, including:

- What spatial indexing is
- How spatial indexing works

## What is a spatial index?

A spatial index is just like any other [index](indexes.html).  Its purpose in life is to improve your database's performance by helping SQL locate data without having to look through every row of a table.

Spatial indexes are mainly used for the same tasks as any other index type, namely:

- Fast filtering of objects based on spatial predicate functions, such as `ST_Contains`.

- Speeding up joins between spatial and non-spatial data.

It differs from other indexes as follows:

- Its inner workings are specialized to operate on 2-dimensional `GEOMETRY` and `GEOGRAPHY` data types.

- It is stored by CockroachDB as a special type of [inverted index](inverted-indexes.html).  For more details, see [How spatial indexing works](#how-spatial-indexing-works) below.

## How spatial indexing works

At a high level, CockroachDB takes a "divide the space" approach to spatial indexing that works by decomposing the space being indexed into buckets of various sizes.  This approach is necessary to preserve CockroachDB's ability to scale horizontally.

CockroachDB uses the [S2 library](https://s2geometry.io/) to divide the space being indexed into a [quadtree](https://en.wikipedia.org/wiki/Quadtree) data structure with a set number of levels and a data-independent shape. Each node in the quad tree (really, S2 cell) represents some part of the indexed space and is divided once horizontally and once vertically to produce 4 child cells in the next level. The nodes are numbered using a [Hilbert space-filling curve](https://en.wikipedia.org/wiki/Hilbert_curve) which preserves locality; the leaf nodes of the quadtree measure 1cm across the Earth's surface.  This means that spatial accuracy of your indexes is tunable down to 1cm (with tradeoffs of accuracy vs. speed during index creation -- see below).

When indexing an object, a covering is computed using some number of the predefined cells. The number of covering cells can vary per indexed object. There is an important tradeoff in the number of cells used to represent an object in the index: fewer cells use less space but create a looser covering. A looser covering retrieves more false positives from the index, which is expensive because the exact answer computation that's run after the index query is expensive. However, at some point the benefits of retrieving fewer false positives is outweighed by how long it takes to scan a large index.

Because the space is divided beforehand, it must be finite. This means that CockroachDB's spatial indexing works for (spherical) {{GEOGRAPHY}} and for finite {{GEOMETRY}} (planar) but not for infinite {{GEOMETRY}}.

Advantages of CockroachDB's "divide the space" approach to spatial indexing include:

+ Easy to scale horizontally.
+ No balancing operations are required, unlike [R-tree indexes](https://en.wikipedia.org/wiki/R-tree).
+ Inserts require no locking.
+ Bulk ingest is simpler to implement than other approaches.
+ Allows a per-object tradeoff between index size and false positives during index creation.

Disadvantages of the "divide the space" approach include:

+ Does not support indexing infinite {{GEOMETRY}} types.
+ Includes more false positives in the index by default, which must then be filtered out by the SQL execution layer.  This filtering can be slow, and thus tuning spatial indexes becomes more important to get good performance.

## Index storage

CockroachDB stores spatial indexes as a special type of [inverted index](inverted-indexes.html).  The spatial inverted index maps from a location to one or more shapes whose [coverings](spatial-glossary.html#covering) include that location.  Since a location can be used in the covering for multiple shapes, and each shape can have multiple locations in its covering, there is a many-to-many relationship between locations and shapes.

As such, a row in the index might look like:

| Key                           | Value |
|-------------------------------+-------|
| 'POINT(-74.147896 41.679517)' |       |

To control how many duplicates  configure the index to use 

## Examples

{% include copy-clipboard.html %}
~~~ sql
-- Taken from SQL logictests
CREATE INDEX geom_idx_1 ON geo_table USING GIST(geom) WITH (geometry_min_x=0, s2_max_level=15)
CREATE INDEX geom_idx_3 ON geo_table USING GIST(geom) WITH (s2_max_level=10)
CREATE INDEX geog_idx_1 ON geo_table USING GIST(geog) WITH (s2_level_mod=2)

CREATE INDEX geom_idx_1 ON geo_table USING GIST(geom) WITH (geometry_min_x=0, s2_max_level=15);
CREATE INDEX geog_idx_1 ON geo_table USING GIST(geog) WITH (s2_level_mod=3);

CREATE TABLE public.geo_table (
   id INT8 NOT NULL,
   geog GEOGRAPHY(GEOMETRY,4326) NULL,
   geom GEOMETRY(GEOMETRY,3857) NULL,
   CONSTRAINT "primary" PRIMARY KEY (id ASC),
   INVERTED INDEX geom_idx_1 (geom) WITH (s2_max_level=15, geometry_min_x=0),
   INVERTED INDEX geom_idx_2 (geom) WITH (geometry_min_x=0),
   INVERTED INDEX geom_idx_3 (geom) WITH (s2_max_level=10),
   INVERTED INDEX geom_idx_4 (geom),
   INVERTED INDEX geog_idx_1 (geog) WITH (s2_level_mod=2),
   INVERTED INDEX geog_idx_2 (geog),
   FAMILY fam_0_geog (geog),
   FAMILY fam_1_geom (geom),
   FAMILY fam_2_id (id)
)
~~~

To see the S2 Coverings:

{% include copy-clipboard.html %}
~~~ sql
ST_AsEWKT(ST_S2Covering(geom::geometry, 's2_max_cells=2'))
ST_AsEWKT(ST_S2Covering(geog::geography, 's2_max_cells=2'))
~~~

## See also

- [Inverted Indexes](inverted-indexes.html)
- [Indexes](indexes.html)
- [Spatial Features](spatial-features.html)
- [Working with Spatial Data](spatial-data.html)
- [Spatial and GIS Glossary of Terms](spatial-glossary.html)
- [Spatial functions](functions-and-operators.html#geospatial-functions)
- [Migrate from Shapefiles](migrate-from-shapefiles.html)
- [Migrate from GeoJSON](migrate-from-geojson.html)
- [Migrate from GeoPackage](migrate-from-geopackage.html)
- [Migrate from OpenStreetMap](migrate-from-openstreetmap.html)
