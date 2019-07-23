# ogrgdalcheatbook
Example commands for geospatial tasks with OGR, GDAL, PostGIS, ...


Get info:
`ogrinfo -so -al input.geojson`

Convert formats:
`ogr2ogr out.geojson in.shp`

Add field:
`ogrinfo -sql "ALTER TABLE lyr ADD COLUMN field NUMERIC(1000,2)" lyr.shp `

Drop/remove/delete field:
`ogrinfo -sql "ALTER TABLE lyr DROP COLUMN field" lyr.shp  `

Rename field:
`ogrinfo -sql "ALTER TABLE input RENAME COLUMN SUM(hectares) TO hectares" in.shp  `

Reorder fields:
`ogr2ogr new.shp old.shp -select "second_fld, first_fld" `

Replace string:
`ogrinfo in.geojson -sql "UPDATE TABLE SET field = REPLACE( field, 'bad', 'good')"`

Merge files:
`ogr2ogr merged.shp in1.shp; ogr2ogr -update -append merged.shp in2.shp -nln merged  `

Reproject (while defining unknown source spatial ref system?):
`ogr2ogr out_3857.geojson in_4236.geojson -s_srs 'GEOGCS["WGS 84",DATUM["WGS_1984",SPHEROID["WGS 84",6378137,298.257223563,AUTHORITY["EPSG","7030"]],AUTHORITY["EPSG","6326"]],PRIMEM["Greenwich",0,AUTHORITY["EPSG","8901"]],UNIT["degree",0.01745329251994328,AUTHORITY["EPSG","9122"]],AUTHORITY["EPSG","4326"]]'
 -t_srs 'PROJCS["WGS 84 / Pseudo-Mercator",GEOGCS["WGS 84",DATUM["WGS_1984",SPHEROID["WGS 84",6378137,298.257223563,AUTHORITY["EPSG","7030"]],AUTHORITY["EPSG","6326"]],PRIMEM["Greenwich",0,AUTHORITY["EPSG","8901"]],UNIT["degree",0.0174532925199433,AUTHORITY["EPSG","9122"]],AUTHORITY["EPSG","4326"]],PROJECTION["Mercator_1SP"],PARAMETER["central_meridian",0],PARAMETER["scale_factor",1],PARAMETER["false_easting",0],PARAMETER["false_northing",0],UNIT["metre",1,AUTHORITY["EPSG","9001"]],AXIS["X",EAST],AXIS["Y",NORTH],EXTENSION["PROJ4","+proj=merc +a=6378137 +b=6378137 +lat_ts=0.0 +lon_0=0.0 +x_0=0.0 +y_0=0 +k=1.0 +units=m +nadgrids=@null +wktext +no_defs"],AUTHORITY["EPSG","3857"]]'
`

Clip by bounding box:
`ogr2ogr out.geojson in.geojson -clipsrc <x_min> <y_min> <x_max> <y_max>`

Clip by vector:
`ogr2ogr out.geojson in.geojson -clipsrc clipping_shp.geojson (must be same srs; dissolve clipsrc first if desired)  `

Union/dissolve/merge-by-attribute:
`ogr2ogr union.shp in.shp -dialect sqlite -sql "SELECT ST_Union(GEOMETRY),union_field FROM in_lyr GROUP BY union_field" in_lyr.shp `


Convert field type (cast) ":
`  `
"
Rename field  ":
`  `
"
Rename & reorder fields ":
`ogr2ogr new.shp old.shp -sql "SELECT second_fld as lynx, first_fld as drinks from old"  `
"
Add a new field & calculate all the areas ":
`ogr2ogr new.shp old.shp -sql "SELECT *, ST_AREA(GEOMETRY) as area from old" `
"
Reorder features/shapes ":
`  `
"
Set/update/calculate field/values ":
`ogrinfo -dialect sqlite -sql "UPDATE table_name SET field='value'"  `
"
Look-Up/pseudo-join/calculate ":
`update states set pop2014=(select pop from pops2014 where name=name); (?) `
"
Insert/add record ":
`ogrinfo -sql "INSERT INTO table (field1,field2) value1, value2" `
"
Delete/remove record/feature  ":
`ogrinfo -dialect sqlite -sql "DELETE FROM table WHERE field=value"  `
"
  ":
`add a column called state_abbr which is MT where state='Montana', 'WA' where state='Washington' `
"
create serial / unique id ":
`ogrinfo -sql "ALTER TABLE lyr ADD COLUMN uid integer primary key autoincrement" `
"
create index (non-spatial)  ":
`ogrinfo -sql "CREATE INDEX ON `
"
concatenate ":
`ogrinfo -dialect sqlite -sql "SELECT AGNCY_ID || ':' || UNIT_NAME AS SUPER_UNIT FROM CPAD_Units_nightly LIMIT 10" foo.shp `
"
postgis ":
`ogrinfo -so -al PG:"host=localhost user=mapcollab_coastal_conservancy dbname=mapcollab_coastal_conservancy" `
"
postgis: pg2shp ":
`ogr2ogr ca3310_test.shp PG:"host=localhost user=mapcollab_coastal_conservancy dbname=mapcollab_coastal_conservancy" srid_test  -sql "select polygon_test as geometry, srid from srid_test where polygon_test is not null and ST_IsValid(polygon_test)"  `
"
postgis: destroy db (make new)  ":
`ogr2ogr -overwrite -f "PostgreSQL" PG:"host=myhost user=myuser dbname=mydb password=mypass" city.shp  `
"
postgis: shp2pg (ALWAYS use -update -append; these refer to the db as a whole, not tables individually) ":
`ogr2ogr -f PostgreSQL -update -append PG:"dbname=my_db user=me password=pw" [-nln table_name] shp_name.shp  `
"
postgis: shp2pg (if db:user is in conf file and you want tablename=shpname) ":
`ogr2ogr -f PostgreSQL -update -append PG:"user=me" shp_name.shp `
"
postgis: csv2pg ??  ":
`ogr2ogr -f PostgreSQL -update -append PG:"dbname=my_db user=me password=pw" [-nln table_name] csv_name.csv  `
"
Join  ":
`  `
"
Explode ":
`  `
"
Buffer  ":
`ogrinfo -dialect sqlite -sql "INSERT INTO one_cpad (AGNCY_NAME,GEOMETRY) SELECT 'buff',ST_Buffer(GEOMETRY,1000) FROM one_cpad WHERE AGNCY_NAME='foo' " one_cpad.shp `
"
Simplify (point-remove) ":
`ogr2ogr -simplify 10 …  `
"
Simplify/Generalize ":
`alter table scc_dissolve add column the_geom_simp geometry; update scc_dissolve set the_geom_simp = ST_SimplifyPreserveTopology(the_geom,5);  `
"
Count vertices  ":
`select st_npoints(the_geom) as npoints from scc_dissolve order by npoints; select sum(st_npoints(the_geom)) from scc_dissolve;  `
"
Centroid (within) ":
`ogr2ogr -f GeoJSON -t_srs epsg:4326 -lco COORDINATE_PRECISION=5 out.js in.shp -dialect sqlite -sql "SELECT ST_PointOnSurface(GEOMETRY) AS GEOMETRY,other_fields FROM lyr" `
"
"spatial join" point-in-poly two shapefiles (one/same folder/directory) ":
`ptime ogrinfo -dialect sqlite -sql "UPDATE scc_acquisitions_4326_b SET county2 = (SELECT NAME FROM 'counties_4326.shp'.counties_4326 WHERE ST_Intersects(geometry, ST_PointOnSurface(scc_acquisitions_4326_b.geometry)))" scc_acquisitions_4326_b.shp `
"
Optimize field widths ":
`ogrinfo -sql "RESIZE lyr" lyr.shp `
"
Merge ":
`  `
"
Erase ":
`ST_Difference `
"
Union ":
`  `
"
Create/build spatial index  ":
`ogrinfo -sql "CREATE SPATIAL INDEX ON lyr" lyr.shp  `
"
Summarize table attribue/field  ":
`ogrinfo -dialect sqlite -sql "SELECT dataagg, COUNT(*) AS cnt FROM CCED_2014 GROUP BY dataagg ORDER BY cnt DESC" CCED_2014.shp  `
"
Summarize to file ":
`ogr2ogr -f csv sum_dataagg.csv -dialect sqlite -sql "SELECT dataagg, COUNT(*) AS cnt FROM CCED_2014 GROUP BY dataagg ORDER BY cnt DESC" CCED_2014.shp `
"
Explode (multipart to singlepart) ":
`ST_Dump `
"
Get a list of fields  ":
`ogr2ogr -f "CSV" scc_fields.csv -dialect sqlite -sql "select * from scc_acquisitions_4326_b where 1=0" scc_acquisitions_4326_b.shp  `
"
extrude to 3d kml ":
`ogr2ogr out.kml -f kml in.shp -zfield pop -dsco AltitudeMode=absolute -dsco extrude=1 -dsco tessellate=1  `
"
snap / coordinate_precision ":
`ST_SnapToGrid (requires gdal 1.11 ?)  `
"
bbox min max  ":
`"ogrinfo -dialect sqlite -sql ""SELECT MbrMinX(ST_Union(GEOMETRY)), MbrMinY(ST_Union(GEOMETRY)), MbrMaxX(ST_Union(GEOMETRY)), MbrMaxY(ST_Union(GEOMETRY)) FROM cb_2014_us_county_500k WHERE STATEFP = '06' GROUP BY STATEFP"" cb_2014_us_county_500k.shp
" `
"
  ":
`  `
"
  ":
`  `
"
spatial join / poly-in-poly / point in polygon  ":
`ogrinfo -dialect sqlite -sql "UPDATE cced SET county2 = (SELECT name_pcase FROM counties WHERE ST_Intersects(geometry, cced.geometry))" temp2.sqlite  `
"
  ":
`ogrinfo -dialect sqlite -sql "UPDATE Acquisitions_working_SD SET COUNTY_NAM = ( SELECT NAME FROM 'N:\California\Administrative\Counties\TIGER-Census\tl_2013_ca_county_3310.shp'.tl_2013_ca_county_3310 as two WHERE ST_Intersects(geometry, ST_PointOnSurface(Acquisitions_working_SD.geometry)) )"  Acquisitions_working_SD.shp   `
"
sql data types: ":
`  `
"
int ":
`#ERROR! `
"
varchar(n)  ":
`  `
"
date  ":
`  `
"
numeric ":
`Real (24.15)  `
"
numeric(0,2)  ":
`  `
"
numeric(1000) ":
`integer?  `
"
numeric(10,2) ":
`up to 10^8, with 2 decimals `
"
text  ":
`varchar `
"
serial  ":
`autoincrementing integer  `
"
float ":
`the only number type for sqlite?  `
"
  ":
`  `
"
  ":
`  `
"
  ":
`gdalwarp -of GTiff -cutline DATA/area_of_interest.shp -cl area_of_interest -crop_to_cutline DATA/PCE_in_gw.asc data_masked7.tiff  `
"
  ":
`gdalwarp -co COMPRESS=DEFLATE -of GTiff -cutline tcb3_landscape_units_3310.shp -crop_to_cutline -cwhere "Name='American Canyon'" aet2010_2039_ave_ccsm4_rcp85.asc.tif aet_clip.tif -s_srs "EPSG:3310" -t_srs "EPSG:3310"  `
"
convert vector to raster  ":
`gdal_rasterize 10min_graticule_sierra_studyarea_CalTealeAlbers.shp teale.tif -tr 1000 1000 -te -211405 -279184 224593 388744 -a FIDTXT  `
"
  ":
`  `
"
  ":
`  `
"
select addgeometrycolumn('collab_points', 'the_geom_simp', 3857, 'MULTIPOINT', 2);  ":
`  `
"
select addgeometrycolumn('collab_lines', 'the_geom_simp', 3857, 'MULTILINESTRING', 2);  ":
`  `
"
select addgeometrycolumn('collab_polygons', 'the_geom_simp', 3857, 'MULTIPOLYGON', 2);  ":
`  `
"
  ":
`  `
"
"create table surveys_updates (
  id serial primary key,
  user_id int references users(id) on delete cascade,
  survey_id int references surveys(id) on delete cascade,
  dt_update timestamp with time zone default now()
 );"  ":
`  `
"
alter table surveys add column user_group_id int references user_groups(id) on delete cascade;  ":
`  `
"
  ":
`  `
"
  ":
`  `
"
  ":
`  `
"
  ":
`  `
"
  ":
`  `
"
  ":
`  `
"
  ":
`  `
"
select duplicate ids: ":
`ogrinfo -dialect sqlite -sql "SELECT SUPER_UNIT, COUNT(*) AS cnt FROM CPAD_2014a_SuperUnits GROUP BY SUPER_UNIT HAVING cnt>1" CPAD_2014a_SuperUnits.shp `
"
find unit holdings problems ":
`ogrinfo -dialect sqlite -sql "select project_na, count(project_na) as cnt from ( SELECT project_nu, project_na FROM Acquisitions_working_SD GROUP BY project_nu, project_na ) group by project_nu order by cnt" Acquisitions_working_SD.shp `
"
  ":
`  `
"
  ":
`ogrinfo -dialect sqlite -sql "SELECT UNIT_ID, COUNT(DISTINCT UNIT_NAME) AS cnt FROM CPAD_2014a_Holdings GROUP BY UNIT_ID,AGNCY_NAME,ACCESS_TYP,MNG_AGNCY,COUNTY HAVING cnt>1 ORDER BY cnt" CPAD_2014a_Holdings.shp  `
"
  ":
`ogrinfo -dialect sqlite -sql "SELECT UNIT_ID, COUNT(DISTINCT UNIT_NAME) AS cnt FROM CPAD_2014a_Holdings GROUP BY UNIT_ID HAVING cnt>1 ORDER BY cnt" CPAD_2014a_Holdings.shp `
"
  ":
`  `
"
  ":
`  `
"
only in PostGIS 2.1? ST_Simplify  ":
`http://strk.keybit.net/blog/2013/03/08/on-the-fly-simplification-of-topologically-defined-geometries/ `
"
  ":
`http://postgis.net/docs/manual-dev/TP_ST_Simplify.html  `
"
not to be confused with: (same func name, diff param type)  ":
`http://postgis.net/docs/manual-dev/ST_Simplify.html `
"
Forget not mapshaper! (can windows install w/ npm)  ":
`mapshaper tl_2014_us_county_48_wgs84.shp -simplify 0.1 -o tl_2014_us_county_48_wgs84_simp100.shp  `
"
topology / topojson ":
`http://www.perrygeo.com/topological-simplification-of-simple-features.html  `
"
  ":
`  `
"
http://trac.osgeo.org/gdal/wiki/UserDocs/ReadInZip  ":
`  `
"
http://lists.osgeo.org/pipermail/gdal-dev/2012-September/034006.html  ":
`ogrinfo -ro /vsizip/vsicurl/http://www2.census.gov/geo/tiger/GENZ2010/gz_2010_us_040_00_20m.zip -dialect sqlite -sql "select NAME, STATE, ST_X(ST_PointOnSurface(geometry)) as x, ST_Y(ST_PointOnSurface(geometry)) as y from gz_2010_us_040_00_20m where NAME='California'"  `
"
  ":
`  `
"
  ":
`ogrinfo -dialect sqlite -sql "update data set Animal=trim(Animal)" data.dbf `
"
  ":
`ogrinfo -dialect sqlite -sql "update data set Animal=trim( upper(substr(Animal,1,1)) || lower(substr(Animal,2,79)) )" data.dbf  `
"
  ":
`  `
"
  ":
`ogrinfo -dialect sqlite -sql "select upper(substr(Animal,1,1)) || lower(substr(Animal,2,79)) from data" data.dbf  `
"
  ":
`  `
"
join to external dbf  ":
`%ogr_version% %nightlyDir%\CPAD_Holdings_nightly_bot_join_agency.shp %nightlyDir%\CPAD_Holdings_nightly_INTERNAL_NoOverlaps.shp -sql "SELECT %non_agency_fields%, %agency_fields_aliased% FROM CPAD_Holdings_nightly_INTERNAL_NoOverlaps LEFT JOIN 'agency_list.dbf'.agency_list ON CPAD_Holdings_nightly_INTERNAL_NoOverlaps.AGNCY_ID = agency_list.AGNCY_ID WHERE (CFF <> 1 or CFF is null) and AGNCY_ID > 0 and AGNCY_ID < 9999 and AGNCY_NAME <> 'Trust for Public Land'" -overwrite  `
"
  ":
`ogrinfo -dialect sqlite -sql "SELECT a.field as afield, b.field as bfield FROM 'table_one.csv'.table_one as a LEFT JOIN table_two as b ON a.field = b.field" table_two.csv    `
"
  ":
`"ogrinfo -dialect sqlite -sql ""SELECT a.name , b.name FROM 'ne_10m_pp_ca.dbf'.ne_10m_pp_ca AS a, '01_b_ca_county_seats_and_largest_cities.dbf'.01_b_ca_county_seats_and_largest_cities AS b WHERE a.name = b.name"" 01_b_ca_county_seats_and_largest_cities.dbf  
" `
"
FULL OUTER JOIN!  ":
`ogr2ogr -f csv nced_cced_schema_full_outer_join.csv -dialect sqlite -sql "SELECT c.field as cfield, n.field as nfield FROM 'tcc.csv'.tcc as c LEFT JOIN tnc as n ON c.field = n.field UNION ALL SELECT c.field as cfield, n.field as nfield FROM tnc as n LEFT JOIN 'tcc.csv'.tcc as c ON cfield=nfield WHERE cfield IS NULL" tnc.csv `
"
  ":
`  `
"
"Transfer Attributes" aka "Two Themes" aka Census Zips (see ET Geo for good description and equations)  ":
`  `
"
including overlapping buffers (state parks factfinder)  ":
`  `
"
  ":
`ogr2ogr intersect1.shp -dialect sqlite -sql "SELECT ST_Area(buffer.geometry) AS buff_m2, ST_Area(acs_albers.geometry) AS acs_m2, buffer.DID as DID, acs_albers.B01001e1, acs_albers.B19013e1, ST_Intersection(acs_albers.geometry,buffer.geometry) AS GEOMETRY, CAST (ST_Area(ST_Intersection(acs_albers.geometry,buffer.geometry)) AS numeric) AS split_m2 FROM 'buffer.shp'.buffer as buffer,'acs_albers.shp'.acs_albers as acs_albers WHERE ST_Intersects(acs_albers.geometry,buffer.geometry)" buffer.shp acs_albers.shp  `
"
  ":
`ogr2ogr out4.shp -dialect sqlite -sql "SELECT DID, ST_Union(geometry) AS geometry, ST_Area(ST_Union(geometry)) as st_area, sum(B01001e1 * split_m2 / acs_m2) as sum_b0, sum( (B19013e1 * split_m2)/2033845.4 ) as ave_b1 FROM intersect1 GROUP BY DID" intersect1.shp -overwrite  `
"
http://books.google.com/books?id=zCaxAgAAQBAJ&pg=PT182&lpg=PT182&dq=postgis+Using+polygon+overlays+for+proportional+census+estimates&source=bl&ots=fxIP1SNotq&sig=9dtlp6EfKIye3qJLDLo793QsDn4&hl=en&sa=X&ei=H-RiU4qkJYWBogTBrIK4Aw&ved=0CEsQ6AEwBA#v=onepage&q=postgis%20Using%20polygon%20overlays%20for%20proportional%20census%20estimates&f=false ":
`  `
"
  ":
`  `
"
  ":
`  `
"
numeric field overflow: use -lco PRECISION=NO ":
`ogr2ogr -f Postgresql pg:"dbname=jk_test user=postgres password=ginfo116" "P:\proj_a_d\coastal_conservancy\AcquisitionMapping2013\data\Acquisitions_working.shp" -lco PRECISION=no -nlt MULTIPOLYGON -update -overwrite `
"
  ":
`  `
"
"    multipolygon to polygon:
    CREATE TABLE temp AS SELECT 
        a.wkb_geometry FROM (
            SELECT (ST_Dump(wkb_geometry)).geom AS wkb_geometry 
            FROM cpad_superunits_n_m_a
        ) AS a" ":
`  `
"
  ":
`  `
"
"
    // polygons converted to lines
    //var line_sql = 'SELECT ST_CollectionExtract(the_geom,2) FROM ' + TABLENAME + ' as lines_table';
    //SELECT ST_Transform(ST_CollectionExtract(the_geom,2),3857) AS the_geom_webmercator, cartodb_id FROM cpad_units_m_nightly_simplify10m

    //ogr2ogr cpad_lines.shp cpad_units_m_nightly_simplify10m.shp -dialect sqlite -sql ""SELECT Boundary(geometry) as geometry, unit_name FROM cpad_units_m_nightly_simplify10m "" -overwrite

    //SELECT ST_Transform(Boundary(the_geom),3857) AS the_geom_webmercator, cartodb_id FROM cpad_units_m_nightly_simplify10m
    //SELECT ST_Transform(ExteriorRing(the_geom),3857) AS the_geom_webmercator, cartodb_id FROM cpad_units_m_nightly_simplify10m" ":
`  `
"
  ":
`  `
"
"update column based on another table: UPDATE
    ""QuestionTrackings""
SET
    ""QuestionID"" = (SELECT ""QuestionID"" FROM ""Answers"" WHERE ""AnswerID""=""QuestionTrackings"".""AnswerID"")
WHERE
    ""QuestionID"" is NULL
AND ..."  ":
`  `
"
  ":
`  `
"
utf8 etc  ":
`  `
"
windows: 2 lines  ":
`SET PGCLIENTENCODING=LATIN1 `
"
  ":
`ogr2ogr pg:"host=ginserver dbname=cpad_2015 user=postgres password=ginfo116" stamen_final_list.csv -update -append  `
"
linux: include  options='-c client_encoding=latin1' inside connection params? ":
`PGCLIENTENCODING=utf-8 ogr2ogr pg:"host=ginserver dbname=cpad_2015 user=postgres password=ginfo116" stamen_final_list.csv -update -append `
"
within psql determine encoding: ":
`show SERVER_ENCODING; `
"
  ":
`  `
"
postgis to shp utf8 encoding  ":
`ogr2ogr cpad_2014b8_superunits_name_manager_access.shp  pg:"host=ginserver dbname=cpad_2015 user=postgres password=ginfo116" su_nma_unbuffer -overwrite -t_srs epsg:3310  -lco ENCODING=UTF-8 `
"
  ":
`  `
"
postgis backup  ":
`pg_dump -U comap_mapcollab -n public --schema-only comap_mapcollab > comap_mapcollab_schema_defs.dump `
"
  ":
`  `
"
shp2pg  ":
`ogr2ogr --config PGCLIENTENCODING LATIN1 -f postgresql  pg:"dbname=db user=user password=password options='-c client_encoding=latin1'" -nln new_name -update -append -t_srs epsg:4326 -nlt MultiPolygon -lco PRECISION=NO  file.shp `
"
pg2shp  ":
`ogr2ogr --config SHAPE_ENCODING "UTF-8" -lco ENCODING="UTF-8" file.shp  pg:"dbname=db user=user password=password" table  `
"
  ":
`Needs BOTH -lco ENCODING="UTF-8"  `
"
  ":
`AND --config SHAPE_ENCODING "UTF-8" `
"
  ":
`  `
"
"table name with periods/spaces:
ogrinfo -sql ""select count(*) from 'us48_osm_km_64.40.merge' where value >= 40 and value < 50"" us48_osm_km_64.40.merge.shp" ":
`  `
"
  ":
`  `
"
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
"
  ":
`  `
"
alias ogr2pg='ogr2ogr -f postgresql -progress --config PG_USE_COPY YES pg:"host=localhost dbname=mapple2 user=johnkelly" -update -append -skipfailures '  ":
`  `
"
  ":
`  `
"
http://download.geofabrik.de/north-america/us/california.html ":
`  `
"
http://download.geofabrik.de/north-america/us/california-latest.osm.pbf ":
`  `
"
ogrinfo -so -al california-latest.osm.pbf ":
`  `
"
ogrinfo -dialect sqlite -sql "select natural from multipolygons group by natural order by natural" california-latest.osm.pbf  ":
`  `
"
ogr2ogr -dialect sqlite -sql "SELECT osm_id, osm_way_id, name, type, natural, other_tags FROM multipolygons WHERE natural IN ('bay', 'glacier', 'lake', 'marsh', 'pond', 'river', 'water', 'wetland')" osm_calif_water.shp california-latest.osm.pbf  ":
`  `
"
