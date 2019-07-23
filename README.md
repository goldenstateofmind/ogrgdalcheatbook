# ogrgdalcheatbook
Example commands for geospatial tasks with OGR, GDAL, PostGIS, ...

Get info:
`ogrinfo -so -al input.geojson`
```
Layer name: input
Geometry: Polygon
Feature Count: 40465
Extent: (-122.401443, 37.785243) - (-122.399679, 37.786270)
Layer SRS WKT:
...
    AUTHORITY["EPSG","4326"]]
val: Integer (0.0)
hex: String (0.0)
```

Convert formats:

`ogr2ogr output.geojson input.shp`

Add field:

`ogrinfo -sql "ALTER TABLE input ADD COLUMN field NUMERIC(1000,2)" input.shp `

Drop/remove/delete field:

`ogrinfo -sql "ALTER TABLE input DROP COLUMN field" input.shp  `

Rename field:

`ogrinfo -sql "ALTER TABLE input RENAME COLUMN SUM(hectares) TO hectares" input.shp  `

Reorder fields:

`ogr2ogr new.shp old.shp -select "second_fld, first_fld" `

Optimize field widths ":

`ogrinfo input.shp -sql "RESIZE input"`

Replace string:

`ogrinfo in.geojson -sql "UPDATE TABLE SET field = REPLACE( field, 'bad', 'good')"`

Merge files:

`ogr2ogr merged.shp in1.shp; ogr2ogr -update -append merged.shp in2.shp -nln merged  `

Reproject / change coordinate system:

`ogr2ogr out_3857.geojson in_4236.geojson -t_srs 'epsg:3857' -s_srs 'epsg:4326'`

Reproject (while defining unknown source spatial ref system):
```
ogr2ogr out_3857.geojson in_4236.geojson \
-s_srs 'GEOGCS["WGS 84",DATUM["WGS_1984",SPHEROID["WGS 84",6378137,298.257223563,AUTHORITY["EPSG","7030"]],AUTHORITY["EPSG","6326"]],PRIMEM["Greenwich",0,AUTHORITY["EPSG","8901"]],UNIT["degree",0.01745329251994328,AUTHORITY["EPSG","9122"]],AUTHORITY["EPSG","4326"]]' \
-t_srs 'PROJCS["WGS 84 / Pseudo-Mercator",GEOGCS["WGS 84",DATUM["WGS_1984",SPHEROID["WGS 84",6378137,298.257223563,AUTHORITY["EPSG","7030"]],AUTHORITY["EPSG","6326"]],PRIMEM["Greenwich",0,AUTHORITY["EPSG","8901"]],UNIT["degree",0.0174532925199433,AUTHORITY["EPSG","9122"]],AUTHORITY["EPSG","4326"]],PROJECTION["Mercator_1SP"],PARAMETER["central_meridian",0],PARAMETER["scale_factor",1],PARAMETER["false_easting",0],PARAMETER["false_northing",0],UNIT["metre",1,AUTHORITY["EPSG","9001"]],AXIS["X",EAST],AXIS["Y",NORTH],EXTENSION["PROJ4","+proj=merc +a=6378137 +b=6378137 +lat_ts=0.0 +lon_0=0.0 +x_0=0.0 +y_0=0 +k=1.0 +units=m +nadgrids=@null +wktext +no_defs"],AUTHORITY["EPSG","3857"]]'
```

Clip by bounding box:

`ogr2ogr out.geojson in.geojson -clipsrc <x_min> <y_min> <x_max> <y_max>`

Clip by vector:

`ogr2ogr out.geojson in.geojson -clipsrc clipping_shp.geojson (must be same srs; dissolve clipsrc first if desired)  `

Union/dissolve/merge-by-attribute:

`ogr2ogr union.shp input.shp -dialect sqlite -sql "SELECT ST_Union(GEOMETRY),union_field FROM input GROUP BY union_field" input.shp `

Convert field type (cast) ":
`  `

Add a new field & calculate all the areas:

`ogr2ogr new.shp old.shp -sql "SELECT *, ST_AREA(GEOMETRY) as area from old" `

Reorder features/shapes:
`  `

Set/update/calculate field/values:

`ogrinfo -dialect sqlite -sql "UPDATE table_name SET field='value'"  `

Look-Up/pseudo-join/calculate:

`update states set pop2014=(select pop from pops2014 where name=name); (?) `

Insert/add record:

`ogrinfo -sql "INSERT INTO table (field1,field2) value1, value2" `

Delete/remove record/feature:

`ogrinfo -dialect sqlite -sql "DELETE FROM table WHERE field=value"  `

Create serial / unique id ":

`ogrinfo -sql "ALTER TABLE input ADD COLUMN uid integer primary key autoincrement" `

Create index (non-spatial):

`ogrinfo -sql "CREATE INDEX ON `

Concatenate:

`ogrinfo input.shp -dialect sqlite -sql "SELECT field_a || ':' || field_b AS info FROM input LIMIT 10" `


Join:
`  `

Explode ":
`  `

Buffer:
`ogrinfo input.shp -dialect sqlite -sql "INSERT INTO input (field1, geometry) SELECT 'buff', ST_Buffer(GEOMETRY,1000) FROM input WHERE field1='foo' " `

Simplify (point-remove) ":
`ogr2ogr -simplify 10 …  `

Simplify/Generalize ":
`alter table input add column the_geom_simp geometry; 
update input set the_geom_simp = ST_SimplifyPreserveTopology(the_geom,5);  `

Count vertices:
`select st_npoints(the_geom) as npoints from input order by npoints; select sum(st_npoints(the_geom)) from input;  `

Centroid (within) ":
`ogr2ogr -f GeoJSON -t_srs epsg:4326 -lco COORDINATE_PRECISION=5 output.geojson input.shp -dialect sqlite -sql "SELECT ST_PointOnSurface(GEOMETRY) AS GEOMETRY,other_fields FROM input" `

"spatial join" point-in-poly two shapefiles (one/same folder/directory) ":
`ogrinfo input.shp -dialect sqlite -sql "UPDATE input SET county2 = (SELECT NAME FROM 'counties_4326.shp'.counties_4326 WHERE ST_Intersects(geometry, ST_PointOnSurface(input.geometry)))"`


Merge ":
`  `

Erase ":
`ST_Difference `

Union ":
`  `

Create/build spatial index:
`ogrinfo -sql "CREATE SPATIAL INDEX ON input" input.shp  `

Summarize table attribue/field:
`ogrinfo input.shp -dialect sqlite -sql "SELECT dataagg, COUNT(*) AS cnt FROM input GROUP BY dataagg ORDER BY cnt DESC" `

Summarize to file ":
`ogr2ogr input.shp -f csv sum_dataagg.csv -dialect sqlite -sql "SELECT dataagg, COUNT(*) AS cnt FROM input GROUP BY dataagg ORDER BY cnt DESC" `

Explode (multipart to singlepart) ":
`ST_Dump `

Get a list of fields:
`ogr2ogr -f "CSV" scc_fields.csv -dialect sqlite -sql "select * from scc_acquisitions_4326_b where 1=0" scc_acquisitions_4326_b.shp  `

extrude to 3d kml ":
`ogr2ogr out.kml -f kml input.shp -zfield pop -dsco AltitudeMode=absolute -dsco extrude=1 -dsco tessellate=1  `

snap / coordinate_precision ":
`ST_SnapToGrid (requires gdal 1.11 ?)  `


bbox min max:
`ogrinfo -dialect sqlite -sql "SELECT MbrMinX(ST_Union(GEOMETRY)), MbrMinY(ST_Union(GEOMETRY)), MbrMaxX(ST_Union(GEOMETRY)), MbrMaxY(ST_Union(GEOMETRY)) FROM cb_2014_us_county_500k WHERE STATEFP = '06' GROUP BY STATEFP" cb_2014_us_county_500k.shp`


sql data types:
`INTEGER`
`REAL`
`TEXT`

serial:
`autoincrementing integer  `

:
`gdalwarp -of GTiff -cutline DATA/area_of_interest.shp -cl area_of_interest -crop_to_cutline DATA/PCE_in_gw.asc data_masked7.tiff  `

:
`gdalwarp -co COMPRESS=DEFLATE -of GTiff -cutline tcb3_landscape_units_3310.shp -crop_to_cutline -cwhere "Name='American Canyon'" aet2010_2039_ave_ccsm4_rcp85.asc.tif aet_clip.tif -s_srs "EPSG:3310" -t_srs "EPSG:3310"  `

convert vector to raster:
`gdal_rasterize 10min_graticule_sierra_studyarea_CalTealeAlbers.shp teale.tif -tr 1000 1000 -te -211405 -279184 224593 388744 -a FIDTXT  `



alter table surveys add column user_group_id int references user_groups(id) on delete cascade;:

:
`  `

select duplicate ids: ":
`ogrinfo  input.shp -dialect sqlite -sql "SELECT name, COUNT(*) AS cnt FROM input GROUP BY name HAVING cnt>1"`

find unit holdings problems ":
`ogrinfo -dialect sqlite -sql "select project_na, count(project_na) as cnt from ( SELECT project_nu, project_na FROM input GROUP BY project_nu, project_na ) group by project_nu order by cnt" input.shp `



only in PostGIS 2.1? ST_Simplify:
`http://strk.keybit.net/blog/2013/03/08/on-the-fly-simplification-of-topologically-defined-geometries/ `

:
`http://postgis.net/docs/manual-dev/TP_ST_Simplify.html  `

not to be confused with: (same func name, diff param type):
`http://postgis.net/docs/manual-dev/ST_Simplify.html `

Forget not mapshaper! (can windows install w/ npm):
`mapshaper tl_2014_us_county_48_wgs84.shp -simplify 0.1 -o tl_2014_us_county_48_wgs84_simp100.shp  `

topology / topojson ":
`http://www.perrygeo.com/topological-simplification-of-simple-features.html  `

:
`  `

http://trac.osgeo.org/gdal/wiki/UserDocs/ReadInZip:
`  `

http://lists.osgeo.org/pipermail/gdal-dev/2012-September/034006.html:
`ogrinfo -ro /vsizip/vsicurl/http://www2.census.gov/geo/tiger/GENZ2010/gz_2010_us_040_00_20m.zip -dialect sqlite -sql "select NAME, STATE, ST_X(ST_PointOnSurface(geometry)) as x, ST_Y(ST_PointOnSurface(geometry)) as y from gz_2010_us_040_00_20m where NAME='California'"  `

:
`  `

:
`ogrinfo -dialect sqlite -sql "update data set Animal=trim(Animal)" data.dbf `

:
`ogrinfo -dialect sqlite -sql "update data set Animal=trim( upper(substr(Animal,1,1)) || lower(substr(Animal,2,79)) )" data.dbf  `

:
`  `

:
`ogrinfo -dialect sqlite -sql "select upper(substr(Animal,1,1)) || lower(substr(Animal,2,79)) from data" data.dbf  `

:
`  `

join to external dbf:

:
`ogrinfo -dialect sqlite -sql "SELECT a.field as afield, b.field as bfield FROM 'table_one.csv'.table_one as a LEFT JOIN table_two as b ON a.field = b.field" table_two.csv    `

:
`"ogrinfo -dialect sqlite -sql ""SELECT a.name , b.name FROM 'ne_10m_pp_ca.dbf'.ne_10m_pp_ca AS a, '01_b_ca_county_seats_and_largest_cities.dbf'.01_b_ca_county_seats_and_largest_cities AS b WHERE a.name = b.name"" 01_b_ca_county_seats_and_largest_cities.dbf  
" `

FULL OUTER JOIN!:
`ogr2ogr -f csv schema_full_outer_join.csv -dialect sqlite -sql "SELECT c.field as cfield, n.field as nfield FROM 'tcc.csv'.tcc as c LEFT JOIN tnc as n ON c.field = n.field UNION ALL SELECT c.field as cfield, n.field as nfield FROM tnc as n LEFT JOIN 'tcc.csv'.tcc as c ON cfield=nfield WHERE cfield IS NULL" tnc.csv `

:
`  `

"Transfer Attributes" aka "Two Themes" aka Census Zips (see ET Geo for good description and equations):
`  `

including overlapping buffers (state parks factfinder):
`  `

:
`ogr2ogr intersect1.shp -dialect sqlite -sql "SELECT ST_Area(buffer.geometry) AS buff_m2, ST_Area(acs_albers.geometry) AS acs_m2, buffer.DID as DID, acs_albers.B01001e1, acs_albers.B19013e1, ST_Intersection(acs_albers.geometry,buffer.geometry) AS GEOMETRY, CAST (ST_Area(ST_Intersection(acs_albers.geometry,buffer.geometry)) AS numeric) AS split_m2 FROM 'buffer.shp'.buffer as buffer,'acs_albers.shp'.acs_albers as acs_albers WHERE ST_Intersects(acs_albers.geometry,buffer.geometry)" buffer.shp acs_albers.shp  `

:
`ogr2ogr out4.shp -dialect sqlite -sql "SELECT DID, ST_Union(geometry) AS geometry, ST_Area(ST_Union(geometry)) as st_area, sum(B01001e1 * split_m2 / acs_m2) as sum_b0, sum( (B19013e1 * split_m2)/2033845.4 ) as ave_b1 FROM intersect1 GROUP BY DID" intersect1.shp -overwrite  `

http://books.google.com/books?id=zCaxAgAAQBAJ&pg=PT182&lpg=PT182&dq=postgis+Using+polygon+overlays+for+proportional+census+estimates&source=bl&ots=fxIP1SNotq&sig=9dtlp6EfKIye3qJLDLo793QsDn4&hl=en&sa=X&ei=H-RiU4qkJYWBogTBrIK4Aw&ved=0CEsQ6AEwBA#v=onepage&q=postgis%20Using%20polygon%20overlays%20for%20proportional%20census%20estimates&f=false ":
`  `


numeric field overflow: use -lco PRECISION=NO ":
`ogr2ogr -f Postgresql pg:"dbname=dbname user=postgres password=xyz" input.shp -lco PRECISION=no -nlt MULTIPOLYGON -update -overwrite `


"ogr2ogr -f geojson us50_topo_current_grid.geojson \
-dialect sqlite -sql \
""select \
\""W Long\"" as lon_w, \
\""S Lat\"" as lat_s, \
\""E Long\"" as lon_e, \
\""N Lat\"" as lat_n, \
\""Map Name\"" as map_name, \
\""Download GeoPDF\"" as url, \
\""Primary State\"" as state_p, \
CAST(\""Date On Map\"" as int) as year, \
CAST(\""Aerial Photo Year\"" as int) as year_photo, \
\""Datum\"" as datum, \
\""Projection\"" as projection, \
 ST_MakePolygon( ST_GeomFromText( 'LINESTRING( ' \
 || \""W Long\"" || ' ' || \""S Lat\"" || ', ' \
 || \""W Long\"" || ' ' || \""N Lat\"" || ', ' \
 || \""E Long\"" || ' ' || \""N Lat\"" || ', ' \
 || \""E Long\"" || ' ' || \""S Lat\"" || ', ' \
 || \""W Long\"" || ' ' || \""S Lat\"" || ')' ) ) \
 as geometry \
FROM topomaps_all WHERE Version = 'Current' "" \
topomaps_all.csv
" ":
`  `

:
`  `

alias ogr2pg='ogr2ogr -f postgresql -progress --config PG_USE_COPY YES pg:"host=localhost dbname=our_db user=me" -update -append -skipfailures ':


http://download.geofabrik.de/north-america/us/california.html ":
`  `

http://download.geofabrik.de/north-america/us/california-latest.osm.pbf ":
`  `

ogrinfo -so -al 'http://download.geofabrik.de/north-america/us/california-latest.osm.pbf'

ogrinfo -ro -al -so /vsizip//vsicurl/https://raw.githubusercontent.com/OSGeo/gdal/master/autotest/ogr/data/poly.zip
ogrinfo -ro -al -so /vsizip//vsicurl/http//www.naturalearthdata.com/download/50m/cultural/ne_50m_admin_0_map_subunits.zip
ogrinfo -ro -al -so /vsizip//vsicurl/https://www.naturalearthdata.com/http//www.naturalearthdata.com/download/50m/cultural/ne_50m_admin_0_map_subunits.zip
ogrinfo -ro -al -so /vsizip//vsicurl/https://www.naturalearthdata.com/http//www.naturalearthdata.com/download/50m/cultural/ne_50m_admin_0_map_subunits.zip/ne_50m_admin_0_map_subunits/ne_50m_admin_0_map_subunits.shp

ogrinfo -ro -al -so "/vsizip/vsicurl/https://www.naturalearthdata.com/http//www.naturalearthdata.com/download/10m/cultural/ne_10m_admin_0_countries.zip" 
ogrinfo -ro -al -so "/vsizip/vsicurl/https://www.naturalearthdata.com/http//www.naturalearthdata.com/download/10m/cultural/ne_10m_admin_0_countries.zip/ne_10m_admin_0_countries.shp" 

ogrinfo -ro -al -so /vsizip/{/vsicurl/https://www.naturalearthdata.com/http//www.naturalearthdata.com/download/10m/cultural/ne_10m_admin_0_countries.zip}/ne_10m_admin_0_countries.shp
ogrinfo -ro -al -so /vsizip//vsicurl/https://www.naturalearthdata.com/http//www.naturalearthdata.com/download/10m/cultural/ne_10m_admin_0_countries.zip/ne_10m_admin_0_countries.shp



ogrinfo -ro -al -so /vsizip//vsicurl/http://www2.census.gov/geo/tiger/GENZ2010/gz_2010_us_040_00_20m.zip/gz_2010_us_040_00_20m/gz_2010_us_040_00_20m.shp
ogrinfo -ro -al -so http://www2.census.gov/geo/tiger/GENZ2010/gz_2010_us_040_00_20m.zip
ogrinfo -ro -al -so /vsizip//vsicurl/gz_2010_us_040_00_20m.zip
ogrinfo -ro -al -so /vsizip/gz_2010_us_040_00_20m.zip

ogrinfo -ro -al -so /vsizip/gz_2010_us_040_00_20m.zip


ogrinfo -ro /vsizip//vsicurl/http://www2.census.gov/geo/tiger/GENZ2010/gz_2010_us_040_00_20m.zip -dialect sqlite -sql "select NAME, STATE, ST_X(ST_PointOnSurface(geometry)) as x, ST_Y(ST_PointOnSurface(geometry)) as y from gz_2010_us_040_00_20m where NAME='California'"

ogrinfo -dialect sqlite -sql "select natural from multipolygons group by natural order by natural" california-latest.osm.pbf:
`  `

ogr2ogr -dialect sqlite -sql "SELECT osm_id, osm_way_id, name, type, natural, other_tags FROM multipolygons WHERE natural IN ('bay', 'glacier', 'lake', 'marsh', 'pond', 'river', 'water', 'wetland')" osm_calif_water.shp california-latest.osm.pbf:
`  `

ogr2ogr -f GeoJSON output.geojson "http:/example.com/arcgis/rest/services/SERVICE/LAYER/MapServer/0/query?f=json&returnGeometry=true&etc=..." OGRGeoJSON
ogr2ogr -f 'GeoJSON' dest.geojson /vsizip/archive.zip/zipped_dir/in.geojson


Add an index on an attribute:

ogrinfo example.shp -sql "CREATE INDEX ON example USING fieldname"

Add a spatial index:

ogrinfo example.shp -sql "CREATE SPATIAL INDEX ON example"
