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
  lon numeric
);
```
`type(zip)=varchar(10)` because of the 'XXXXX-XXXX' zip code format in the US.\
`type(lat)=type(lon)=numeric` for exact precision.\
\
Load data from local CSV file to the **'coffee_shops'** table using import.\
```javascript
SELECT * FROM coffee_shops;
```
Take a look at the table and realize that some observations contain NULL value in lon and lat, which may cause problems when calculating geometry for the shop; however, the numbe of shops with NULL lat/lon is small, so it is safe to omit them.\
\
In order not to make changes to the original table, shops with complete lat & lon information store in a new table **'coffee_shops_new'**.
```javascript
CREATE TABLE coffee_shops_new AS
  SELECT * FROM coffee_shops WHERE (lon IS NOT NULL and lat IS NOT NULL);
```
