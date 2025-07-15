#  pgRouting Basic Workshop

[FOSS4G 2025 Workshop Mostar (Bosnia-Herzegovina)](https://2025.europe.foss4g.org/)

![FOSS4G 2025 Workshop Mostar (Bosnia-Herzegovina)](img/foss4g-europe-2025.png ) ![](img/pgrouting.png)

[Workshop Program Link](https://talks.osgeo.org/foss4g-europe-2025-workshops/talk/3Y3HSG/)


[![Creative Commons License](http://i.creativecommons.org/l/by-sa/4.0/88x31.png)](https://creativecommons.org/licenses/by-sa/4.0/)


## Enock Seth Nyamador

* Craft OpenStreetMap contributor & advocate for using Free and Open Source Software
* https://en.osm.town/@Enock4seth


## 📋 What we will learn 
- Installing pgRouting and PostGIS
- Importing OSM road data
- Creating a routing topology
- Running shortest-path queries with pgRouting
- Writing advanced SQL for routing
- Creating a custom PL/pgSQL stored procedure
- Bonus: Exploring other use cases and your own data (e.g. FTTH, etc) 

---

## Requirements

- PostgreSQL
- PostGIS
- pgRouting
- osm2pgrouting
- pgAdmin4 or psql
- QGIS (optional for visualization)

---

## Software
- OSGeoLive
- Your own server
- Local server
- Create a virtualbox https://workshop.pgrouting.org/3.0/en/general-intro/osgeolive.html

## Install PostgreSQL and all
```bash
sudo apt install -y \
  osm2pgrouting \
  postgresql-17 \
  postgresql-17-postgis-3 \
  postgresql-17-postgis-3-scripts \
  postgresql-17-pgrouting
```
---
## Download OSM Data

```bash
CITY="MOSTAR_BA"
BBOX="17.7888,43.3334,17.8241,43.3579"
wget --progress=dot:mega -O "$CITY.osm" "http://www.overpass-api.de/api/xapi?*[bbox=${BBOX}][@meta]"

```
---
## Import OSM data
```bash
osm2pgrouting \
  --f data/MOSTAR_BA.osm \
  --conf config/mapconfig.xml \
  --dbname <database_name> \
  --username <username> \
  --password <password> \
  --port <port> \
  --clean
```
Clean up
```bash
psql -c 'DELETE FROM ways WHERE length_m IS NULL;' -d city_routing
```
---
### Tables in Database
```sql
-- Check row count
SELECT count(*) FROM ways;

-- Inspect source/target
SELECT gid, source, target FROM ways LIMIT 5;
```
---
## Identifiers for queries

- `5972936559`: Faculty of Humanities and Social Sciences
- `4275858799`: Spanish square
- `388965890`: Most Musala
- `8108606186`: Most Bunur

```sql
SELECT osm_id, id FROM ways_vertices_pgr
WHERE osm_id IN (5972936559, 4275858799, 388965890, 8108606186)
ORDER BY osm_id;
```

 Location |  osm_id   |  id  
----|------------|------
 Most Musala |388965890 |   49
 Spanish square|4275858799 | 1684
 Faculty of Humanities and Social Sciences|5972936559 | 2520
 Most Bunur|8108606186 | 3803
---
## Exercise 1: Single Pedestrian Routing
- Walk from `Most Bunur` to `Faculty of Humanities and Social Sciences`
```sql
SELECT * FROM pgr_dijkstra(
  '
    SELECT gid AS id,
      source,
      target,
      length AS cost
    FROM ways
  ',
  3803,
  2520,
  directed := false);
```
---
## Exercise 2: Many Pedestrians going to the same destination¶
- Walk from `Most Bunur` and `Most Musala` to `Faculty of Humanities and Social Sciences`
```sql
SELECT * FROM pgr_dijkstra(
  '
    SELECT gid AS id,
      source,
      target,
      length_m AS cost
    FROM ways
  ',
  ARRAY[3803,49],
  2520,
  directed := false);
```

---
## Exercise 3: Many Pedestrians going to different destinations
- - Walk from `Most Bunur` and `Most Musala` to `Faculty of Humanities and Social Sciences` and `Spanish Square`
- Cost: minutes
```sql
SELECT * FROM pgr_dijkstra(
  '
    SELECT gid AS id,
      source,
      target,
      length_m / 1.3 / 60 AS cost -- line 6
    FROM ways
  ',
  ARRAY[3803,49],
  ARRAY[2520, 12712], -- line 10
  directed := false);
  ```