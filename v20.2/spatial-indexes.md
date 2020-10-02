---
title: Spatial Indexes
summary: How CockroachDB uses spatial indexes for efficiently storing and querying spatial data.
toc: true
---

{% include {{page.version.version}}/misc/spatial-support-new.md %}

This page describes CockroachDB's approach to indexing spatial data, including:

- What spatial indexing is
- How spatial indexing works

## What is a spatial index?

A spatial index is just like any other [index](indexes.html).  Its purpose in life is to improve your database's performance by helping SQL locate data without having to look through every row of a table.

It differs from other indexes as follows:

- Its inner workings are specialized to operate on 2-dimensional `GEOMETRY` and `GEOGRAPHY` data types.

- It is stored by CockroachDB as a special type of [inverted index](inverted-indexes.html).  For more details, see [How spatial indexing works](#how-spatial-indexing-works) below.

Spatial indexes are mainly used for the same tasks as any other index type, namely:

- Fast filtering of objects based on spatial predicate functions, such as `ST_Contains`.

- Speeding up joins between spatial and non-spatial data.

## How spatial indexing works

At a high level, CockroachDB takes a "divide the space" approach to spatial indexing.  This works by decomposing the space being indexed into buckets of various sizes.  This is necessary to preserve CockroachDB's horizontal scalability, since under the hood CockroachDB stores data in a [distributed key-value store](architecture/overview.html) that is spread across the nodes that make up a cluster.

More specifically, CockroachDB uses the [S2 library](https://s2geometry.io/) to divide the space into a [quadtree](https://en.wikipedia.org/wiki/Quadtree) data structure with a set number of levels and a data-independent shape. Each node in the quad tree (a "cell" in S2 parlance) represents some part of the indexed space and is divided once horizontally and once vertically to produce 4 child nodes in the next level. Each node in the quadtree is "content addressable", meaning that mapping from the node to its unique ID and back is possible without external information.  The cells are numbered using a [Hilbert space-filling curve](https://en.wikipedia.org/wiki/Hilbert_curve) which preserves cell locality; each cell is represented by a 64-bit integer and the leaf cells of the quadtree measure 1cm across the Earth's surface.

This representation results in several nice properties that can be exploited by the implementation:

+ The IDs of all ancestors of a cell are enumerable.
+ Getting the IDs of all descendants of a cell is a range query.
+ The cells of nearby IDs are spatially near (thanks to the Hilbert curve enumeration).
+ Spatial accuracy is tunable down to 1cm (with tradeoffs of accuracy vs. speed during index creation -- see below).

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

CockroachDB stores spatial indexes as a special type of [inverted index](inverted-indexes.html).  Unlike an inverted index on a [`JSONB`](jsonb.html) column, under the hood it maps from a location to one or more shapes whose [coverings](spatial-glossary.html#covering) include that location.  Since a location can be used in the covering for multiple shapes, and each shape can have multiple locations in its covering, there is a many-to-many relationship between locations and shapes.

## Examples

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
