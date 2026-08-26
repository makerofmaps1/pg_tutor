## Basic SQL
Select the core columns from `sites` to see the base table structure.

```sql
SELECT site_id, name, measurement, geom, metadata
FROM sites;
```

## CTE
Use a CTE to define a temporary result and query it immediately.

```sql
WITH s AS (
  SELECT site_id
  FROM sites
)
SELECT site_id
FROM s;
```

## Function
Use a simple PostgreSQL function to transform text values.

```sql
SELECT UPPER(name)
FROM sites;
```

## JSONB
Use JSONB to extract a value, filter by a value, and insert a JSON document.

```sql
SELECT metadata->>'status'
FROM sites;

SELECT site_id
FROM sites
WHERE metadata->>'status' = 'active';

INSERT INTO sites (site_id, name, measurement, geom, metadata)
VALUES (1, 'A', 1, ST_SetSRID(ST_MakePoint(0, 0), 4326), '{"status":"active"}'::jsonb);
```

## Load a Point
Insert one point into `sites` using SRID 4326.

```sql
INSERT INTO sites (site_id, name, measurement, geom, metadata)
VALUES (2, 'B', 2, ST_SetSRID(ST_MakePoint(-73.99, 40.73), 4326), '{}'::jsonb);
```

## Load a Polygon
Insert one polygon into `boundaries` using SRID 4326.

```sql
INSERT INTO boundaries (boundary_id, name, geom)
VALUES (
  1,
  'Zone 1',
  ST_SetSRID(
    ST_GeomFromText('POLYGON((-74.00 40.70, -74.00 40.75, -73.95 40.75, -73.95 40.70, -74.00 40.70))'),
    4326
  )
);
```

## Points Within a Polygon
Find site points that fall inside boundary polygons.

```sql
SELECT s.site_id, b.boundary_id
FROM sites s
JOIN boundaries b
  ON ST_Within(s.geom, b.geom);
```

## Intersecting Geometries (ST_DWithin)
Find geometries that are within a small distance threshold.

```sql
SELECT s.site_id, b.boundary_id
FROM sites s
JOIN boundaries b
  ON ST_DWithin(s.geom, b.geom, 0.001);
```

## Calculate Area
Compute polygon area from `boundaries` geometries.

```sql
SELECT boundary_id, ST_Area(geom::geography)
FROM boundaries;
```

## Putting It All Together
Use a CTE with a spatial join and boundary area calculation in one query.

```sql
WITH s AS (
  SELECT site_id, geom
  FROM sites
)
SELECT s.site_id, b.boundary_id, ST_Area(b.geom::geography)
FROM s
JOIN boundaries b
  ON ST_Within(s.geom, b.geom);
```

## Cheat Sheet
- `SELECT`, `WITH`, `UPPER`, `metadata->>'key'`, `INSERT`, `ST_SetSRID`, `ST_MakePoint`, `ST_GeomFromText`, `ST_Within`, `ST_DWithin`, `ST_Area`