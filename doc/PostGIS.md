PostGIS
=======

* [Data types](#data-types)
* [Queries](#queries)
    * [Create an unique id](#create-an-unique-id)
    * [Set first character to uppercase](#set-first-character-to-uppercase)
    * [Replace all the text before a specific character](#replace-all-the-text-before-a-specific-character)
    * [Erase a string if found](#erase-a-string-if-found)
    * [Test if a value is an integer](#test-if-a-value-is-an-integer)
    * [Concatenate strings with possible null values](#concatenate-strings-with-possible-null-values)
    * [Split values into rows](#split-values-into-rows)
    * [Add an unit to a value](#add-an-unit-to-a-value)
    * [Convert numeric to string](#convert-numeric-to-string)
    * [Convert date to string](#convert-date-to-string)
    * [Order results by list](#order-results-by-list)
    * [Split string with a separator and get specific part](#split-string-with-a-separator-and-get-specific-part)
* [Spatial queries](#spatial-queries)
    * [Spatial join (point in polygon)](#spatial-join-point-in-polygon)
    * [Create a line between two points](#create-a-line-between-two-points)
    * [Return intersected features](#return-intersected-features)
    * [Merge polygons with an attribute](#merge-polygons-with-an-attribute)
    * [Check validity of geometries](#check-validity-of-geometries)
    * [Return features inside buffer around points](#return-features-inside-buffer-around-points)
* [Triggers](#triggers)
    * [Get parcel number](#get-parcel-number)
    * [Get parcels numbers](#get-parcels-numbers)
    * [Get absolute path](#get-absolute-path)
* [Sequences](#sequences)
    * [Set current value](#set-current-value)
* [Geometries](#geometries)
    * [Create column](#create-column)
    * [Create index](#create-index)
* [Information schema](#information-schema)
    * [Get all tables](#get-all-tables)
    * [Get all tables with type](#get-all-tables-with-type)
    * [Get all geometry tables](#get-all-geometry-tables)
    * [Get all views](#get-all-views)
* [Miscellaneous](#miscellaneous)
    * [Get PostGIS version](#get-postgis-version)
* [psql](#psql)
    * [Create schema and allow rights](#create-schema-and-allow-rights)
    * [Dump and restore database](#dump-and-restore-database)

Data types
----------

| Type          | Name                                       |
| ------------- | ------------------------------------------ |
| **Character** | `varchar(255)`, `varchar(50)`, `text`      |
| **Numeric**   | `int4`, `float8`                           |
| **Boolean**   | `bool`                                     |
| **Date/Time** | `date`, `timestamp`                        |
| **Geometry**  | `Point`, `MultiLineString`, `MultiPolygon` |

Queries
-------

### Create an unique id

```sql
SELECT ROW_NUMBER() OVER (ORDER BY <column>) AS id
FROM <table>;
```

### Set first character to uppercase

```sql
SELECT ((UPPER(SUBSTR(<column>, 1, 1)) || SUBSTR(<column>, 2))) :: varchar AS <column>
FROM <table>;
```

### Replace all the text before a specific character

```sql
SELECT REGEXP_REPLACE(<column>, '^[^<char> ]*<char> ', '') AS <column>
FROM <table>;
```

### Erase a string if found

```sql
SELECT
    CASE
        WHEN <column> ~ '.* <string> .*' THEN REGEXP_REPLACE(<column>, ' <new-string> ', '')
        ELSE <column>
    END
    AS <column>
FROM <table>;
```

### Test if a value is an integer

```sql
SELECT
    CASE
        WHEN <column> ~ '^[0-9]+$' THEN TRUE
        ELSE FALSE
    END
    AS is_integer
FROM <table>;
```

### Concatenate strings with possible null values

```sql
SELECT street || ' ' || num || COALESCE(suffix, '') AS address
FROM addresses;
```

### Split values into rows

```sql
SELECT id, REGEXP_SPLIT_TO_TABLE(no_parcelle, ';') AS no_parcelle
FROM <table>;
```

### Add an unit to a value

```sql
SELECT (ROUND(length :: numeric, 2) || ' m') :: varchar AS length
FROM <table>;
```

### Convert numeric to string

```sql
SELECT
    TRIM(TO_CHAR(p.numero :: numeric, '9999 999 9')) :: varchar(20) AS numero,
    TRIM(TO_CHAR(ST_X(p.geom), '999G999.99 m')) :: varchar(20) AS coord_y,
    TRIM(TO_CHAR(ST_Y(p.geom), '999G999.99 m')) :: varchar(20) AS coord_x,
    TRIM(TO_CHAR(p.geomalt, '9G999.99 m')) :: varchar(20) AS altitude
FROM mo.mo_pfp1 p;
```

### Convert date to string

```sql
SELECT to_char(<column>, 'DD.MM.YYYY') AS <column>
FROM <table>;
```

### Order results by list

```sql
SELECT *
FROM osm.osm_roads r
ORDER BY
    CASE
        WHEN r.highway IN ('motorway', 'motorway_link') THEN 1
        WHEN r.highway = 'primary' THEN 2
        WHEN r.highway = 'secondary' THEN 3
        WHEN r.highway = 'tertiary' THEN 4
        ELSE 5
    END DESC;
```

### Split string with a separator and get specific part

```sql
SELECT (REGEXP_SPLIT_TO_ARRAY(<attribute>, '/'))[3] AS <column>
FROM <table>;
```

Spatial queries
---------------

### Spatial join (point in polygon)

```sql
SELECT *
FROM <table1> a, <table2> b
WHERE ST_Within(a.geom, b.geom);
```

### Create a line between two points

```sql
SELECT ST_MakeLine(a.geom, b.geom) :: Geometry(LineString, 21781) AS geom
FROM <table1> a
JOIN <table2> b ON a.id = b.fk;
```

### Return intersected features

```sql
SELECT ST_Intersection(a.geom, b.geom) AS geom
FROM <table1> a, <table2> b
WHERE ST_Intersects(a.geom, b.geom);
```

### Merge polygons with an attribute

```sql
SELECT attribute, ST_Union(ST_SnapToGrid(geom, 0.0001)) :: Geometry(MultiPolygon, 21781) AS geom
FROM <table>
GROUP BY attribute;
```

### Check validity of geometries

```sql
SELECT *
FROM <table>
WHERE ST_IsValid(geom) = true;
```

### Return features inside buffer around points

```sql
SELECT a.*
FROM <table1> a
JOIN <table2> b ON ST_Contains(ST_Buffer(b.geom, 100), a.geom);
```

Triggers
--------

### Get parcel number

```sql
BEGIN
    SELECT INTO new.no_parcelle numero
    FROM mo.mo_par
    WHERE ST_Within(new.geom, geom);
    RETURN new;
END;
```

### Get parcels numbers

```sql
BEGIN
    SELECT INTO new.no_parcelle String_Agg(numero, ';' ORDER BY numero)
    FROM mo.mo_par
    WHERE ST_Intersects(ST_Buffer(new.geom, -0.1), geom)
    GROUP BY new.fid;
    RETURN new;
END;
```

### Get absolute path

```sql
BEGIN
    SELECT INTO new.file
    CASE
        WHEN new.file IS NOT NULL THEN replace(new.file, '<drive>', '<absolute-path>') --'X:\', '\\path\to\folder\'
        ELSE NULL
    END;
    RETURN new;
END;
```

Sequences
---------

### Set current value

```sql
SELECT setval('<schema>.<table>_<field>_seq', (SELECT MAX(<field>) FROM <schema>.<table>));
```

Geometries
----------

### Create column

```sql
SELECT AddGeometryColumn(
    'schema', '<table>',
    'geom', 21781,
    '<Point|MultiLineString|MultiPolygon>', 2
);
```

### Create index

```sql
CREATE INDEX table_geom_idx
ON schema.<table>
USING gist (geom);
```

Information schema
------------------

### Get all tables

```sql
SELECT table_schema, table_name
FROM information_schema.tables
WHERE table_type = 'BASE TABLE'
AND table_schema NOT IN ('information_schema', 'public', 'pg_catalog', 'topology')
ORDER BY table_schema, table_name;
```

### Get all tables with type

```sql
SELECT t.table_schema, t.table_name,
    CASE
        WHEN g.type IS NOT NULL THEN g.type
        ELSE 'TABLE'
    END AS table_type
FROM information_schema.tables t
LEFT JOIN geometry_columns g ON g.f_table_schema || '.' || g.f_table_name = t.table_schema || '.' || t.table_name
WHERE t.table_type = 'BASE TABLE'
AND t.table_schema NOT IN ('information_schema', 'public', 'pg_catalog', 'topology')
ORDER BY t.table_schema, t.table_name;
```

### Get all geometry tables

```sql
SELECT t.table_schema, t.table_name, g.type
FROM information_schema.tables t
JOIN geometry_columns g ON g.f_table_schema || '.' || g.f_table_name = t.table_schema || '.' || t.table_name
WHERE t.table_type = 'BASE TABLE'
AND t.table_schema NOT IN ('information_schema', 'public', 'pg_catalog', 'topology')
ORDER BY t.table_schema, t.table_name;
```

### Get all views

```sql
SELECT table_schema, table_name, view_definition
FROM information_schema.views
WHERE table_schema NOT IN ('information_schema', 'public', 'pg_catalog', 'topology')
ORDER BY table_schema, table_name;
```

Miscellaneous
-------------

### Get PostGIS version

```sql
SELECT PostGIS_full_version();
```

psql
----

### Create schema and allow rights

```bash
sudo -u postgres psql -c 'CREATE SCHEMA <schema>' <database>
sudo -u postgres psql -c 'GRANT ALL ON SCHEMA <schema> TO "<user>"' <database>
```

### Dump and restore database

```bash
sudo -u postgres pg_dump <source_database> > <source_database>.out
sudo -u postgres dropdb <target_database>
sudo -u postgres createdb <target_database>
sudo -u postgres psql -d <target_database> -f <source_database>.out
```