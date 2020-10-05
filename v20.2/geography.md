---
title: GEOGRAPHY
summary: The GEOGRAPHY data type stores spatial data about geometric objects in the 2-d plane mapped onto a sphere (really, an ellipsoid).
toc: true
---

The GEOGRAPHY data type stores spatial data about geometric objects in the 2-d plane mapped onto a sphere (really, an ellipsoid).  It is used for calculations on the Earth's surface, and is more accurate since it takes the curvature of the Earth into effect.

CockroachDB supports indexing `GEOGRAPHY` columns with [inverted indexes](inverted-indexes.html). This permits accelerating the following types of queries:

- Filtering with a spatial predicate function such as `ST_Covers`.
- [Joins](joins.html) between spatial and non-spatial data. 

For a list of functions that operate on the `GEOGRAPHY` data type, see the [documentation on spatial functions](functions-and-operators.html#geospatial-functions).

{{site.data.alerts.callout_info}}
CockroachDB does not support 3-dimensional geographies.
{{site.data.alerts.end}}

## Syntax

The easiest way to generate a value of type `GEOGRAPHY` is using the `ST_GeogFromText` function on a string of [Well-known Text](well-known-text.html):

{% include copy-clipboard.html %}
~~~ sql
SELECT st_geogfromtext('POINT(-74 41)');
~~~

~~~
                   st_geogfromtext
------------------------------------------------------
  0101000020E610000000000000008052C00000000000804440
(1 row)
~~~

## Size

XXX: find out about this

## See also

- [`GEOMETRY`](geometry.html)
- [Well-Known Text](well-known-text.html)
- [Well-Known Binary](well-known-binary.html)
- [Spatial Functions](functions-and-operators.html#geospatial-functions)
- [Data Types](data-types.html)
- [Inverted Indexes](inverted-indexes.html)
