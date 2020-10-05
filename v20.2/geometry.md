---
title: GEOMETRY
summary: The GEOMETRY data type stores spatial data about objects in the 2-dimensional plane
toc: true
---

The `GEOMETRY` data type stores spatial data about geometric objects in the 2-dimensional plane.

 CockroachDB supports indexing spatial columns with [inverted indexes](inverted-indexes.html). This permits accelerating the following types of queries:

- Filtering with a spatial predicate function such as `ST_Covers`.
- [Joins](joins.html) between spatial and non-spatial data. 

{{site.data.alerts.callout_info}}
CockroachDB does not support 3-dimensional geometries.
{{site.data.alerts.end}}

## Syntax

The easiest way to generate a value of type `GEOMETRY` is using the `ST_GeomFromText` function on a string of [Well-known Text](well-known-text.html):

{% include copy-clipboard.html %}
~~~ sql
SELECT st_geomfromtext('POINT(-74 41)');
~~~

~~~
               st_geomfromtext
----------------------------------------------
  010100000000000000008052C00000000000804440
(1 row)
~~~

## Size

XXX: find out about this

## Examples

{{site.data.alerts.callout_success}}
For a list of functions that operate on the `GEOMETRY` data type, see the [documentation on spatial functions](functions-and-operators.html#geospatial-functions).
{{site.data.alerts.end}}

### Creating a geometry column from well-known text



## See also

- [Spatial Functions](functions-and-operators.html#geospatial-functions)
- [Data Types](data-types.html)
- [Inverted Indexes](inverted-indexes.html)
