## Data Loading
Create new extension (PostGis) using query tool:
```javascript
CREATE EXTENSION postgis
  VERSION "3.0.2";
```
### Loading CSV files through SQL or pdAdmin 4 GUI.
Create a new table **'coffee_shops'** to store data:
```javascript
CREATE TABLE coffee_shops(
  id integer NOT NULL,
  name text NOT NULL,  
  address text,
  city text,
  state text,
  zip varchar(10),
  lat numeric,
  lon numeric);
```
`type(zip)=varchar(10)` because of the 'XXXXX-XXXX' zip code format in the US.\
`type(lat)=type(lon)=numeric` for exact precision.\
\
Load data from local CSV file to the **'coffee_shops'** table by selecting Import/Export feature in the pgAdmin menu.\
OR using the `\COPY` statement on psql:
```javascript
\COPY coffee_shops FROM '/my/file.csv' DELIMITER ',' CSV HEADER;
```
Now that the data has been loaded into the 'coffee_shops' table.
```javascript
SELECT * FROM coffee_shops;
```
Take a look at the table and realize that some observations contain NULL value in lon and/or lat fields, which may cause problems when calculating geometry for the shop; however, the numbe of shops with NULL lat/lon is small, so it is safe to omit them.\
\
In order not to make changes to the original table, shops with complete lat & lon information store in a new table **'coffee_shops_new'**.
```javascript
CREATE TABLE coffee_shops_new AS
  SELECT * FROM coffee_shops 
  WHERE (lon IS NOT NULL 
         and 
         lat IS NOT NULL);
```
### Loading Shapefile (.sp) through QGIS-LTR
Connect a QGIS new project to the database and by its built-in DB Manager, import Cambridge Neighbourhoods information table to the database.
```javascript
SELECT * FROM cambridge_neighborhoods;
```
## Create Geometry Field and Index
Add a new column to the **'coffee_shops_new'** that stores geometry information for each coffee shop in Cambridge, MA.\
The new column `geom` has data type geometry(POINT,4326).\
Type of geometry is POINT since each coffee shop is a point in 2D space specified by its X and Y (longitude and latitude) coordinates.\
Coordinate system 4326 is the **EPSG Code** that corresponds to **World Geodetic System 1984 (WGS84)**.
```javascript
ALTER TABLE coffee_shops_new
  ADD COLUMN geom geometry(POINT,4326);
```
Update the `geom` column by each store's longitude and latitude:
```javascript
UPDATE coffee_shops_new 
SET geom = ST_SetSRID(ST_MakePoint(lon,lat),4326);
```
Create Index for table **'coffee_shops_new'** for easier searching:
```javascript
CREATE INDEX coffee_shops_gist
  ON coffee_shops
  USING gist (geom);
```
## Some Queries
1. Count the number of coffee shops within each neighborhood in Cambridge, MA.
```javascript
SELECT nbs.name as name, 
       count(*) as shops_in_the_area
FROM coffee_shops_new as shops, 
     cambridge_neighborhoods as nbs
WHERE ST_Intersects(shops.geom, nbs.geom)
GROUP BY nbs.name
ORDER BY shops_in_the_area desc;
```

2. Select coffee shops base on their distance to MIT and Havard Square.
  - To get more accurate distance in meters, table **'coffee_shops_new'** needs to be reprojected to the local coordinate system (UTM 19N, equivalent EPSG 32619).
```javascript
ALTER TABLE coffee_shops_new
  ADD COLUMN geom_utm geometry(POINT, 32619);

UPDATE coffee_shops_new
  SET geom_utm = ST_Transform(geom, 32619);
```
  - Looking up for MIT and Havard Square's longitude and latitude via [gps-coordinates](https://www.gps-coordinates.net)
  - MIT (-71.096646,42.3582393)
  - Havard Square (-71.12015533447266, 42.37255859375)
  - Select coffee shops near MIT order by distances.
```javascript
SELECT name, address, geom <-> ST_SetSRID(ST_MakePoint(-71.096646,42.3582393),4326) as dist_to_MIT
FROM coffee_shops_new
ORDER BY dist_to_MIT;
```
  - Select coffee shops within 2 KMs to Havard Square
```javascript
SELECT 
  name, 
  address,
  geom_utm <-> ST_Transform(ST_SetSRID(ST_MakePoint(-71.12015533447266, 42.37255859375),
                                       4326),
                            32619) as dist_to_HS
FROM coffee_shops_new
WHERE ST_DWithin(geom_utm,
                 ST_Transform(ST_SetSRID(ST_MakePoint(-71.12015533447266, 42.37255859375),
                                         4326),
                              32619),
                 2000)
ORDER BY dist_to_HS desc;
```
