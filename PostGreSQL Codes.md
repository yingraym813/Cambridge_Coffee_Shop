## Data Loading
Create new extension (PostGis) using query tool:
```javascript
CREATE EXTENSION postgis
  VERSION "3.0.2";
```
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
OR using the `COPY` statement (if the server has the access permission):
```javascript
COPY coffee_shops
FROM '/my/file.csv'
DELIMITER ','
CSV HEADER;
```
Now that the data has been loaded into the 'coffee_shops' table.
```javascript
SELECT * FROM coffee_shops;
```
Take a look at the table and realize that some observations contain NULL value in lon and lat, which may cause problems when calculating geometry for the shop; however, the numbe of shops with NULL lat/lon is small, so it is safe to omit them.\
\
In order not to make changes to the original table, shops with complete lat & lon information store in a new table **'coffee_shops_new'**.
```javascript
CREATE TABLE coffee_shops_new AS
  SELECT * FROM coffee_shops 
  WHERE (lon IS NOT NULL 
         and 
         lat IS NOT NULL);
```
## Create Geometry Field
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

```javascript
CREATE INDEX coffee_shops_gist
  ON coffee_shops
  USING gist (geom);
```
