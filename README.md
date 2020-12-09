# sample-notebook-snowflake-folium

This is a sample Jupyter Notebook for visualization of Snowflake geospatial data with Folium, written for https://zenn.dev/indigo13love/articles/b505bc77b11957 (Japanese article).

To use this notebook, you have to populate a sample table on Snowflake as the article, or the steps below:

## 1. Download the sample dataset

Access [Shinjuku Ward Open Data Catalog](http://www.city.shinjuku.lg.jp/opendata/opendata_category_00012.html), download `新宿区道路線網`, and unzip.

## 2. Convert the shapefile to GeoJSON

Convert `認定.shp` to new-line delimited GeoJSON file as below:

```bash
$ ogr2ogr -f geojsonseq shinjuku-roads.geojson 認定.shp
```

## 3. Create a table and a stage on Snowflake

```sql
create or replace table shinjuku_roads (
  id int,
  name varchar,
  start_date date,
  start_address varchar,
  end_address varchar,
  length double,
  average_width double,
  area double,
  geo geography
);
```

```sql
create or replace stage stage_shinjuku_roads
file_format = (type = json);
```

## 4. Upload the GeoJSON file and load to the table

`PUT` is not supported in Snowflake Web UI, so you have to use SnowSQL or other clients using a connetor.

```sql
put file:///path/to/shinjuku-roads.geojson @stage_shinjuku_roads
```

After uploaded, you can load the GeoJSON file to the table:

```sql
copy into shinjuku_roads
from (select
  $1:properties:SAFIELD000 id,
  $1:properties:SAFIELD001 name,
  to_date($1:properties:SAFIELD004::varchar, 'yyyy年mm月dd日') start_date,
  $1:properties:SAFIELD005 start_address,
  $1:properties:SAFIELD006 end_address,
  $1:properties:SAFIELD012 length,
  $1:properties:SAFIELD013 average_width,
  $1:properties:SAFIELD014 area,
  to_geography($1)
from @shinjuku_roads);
```
