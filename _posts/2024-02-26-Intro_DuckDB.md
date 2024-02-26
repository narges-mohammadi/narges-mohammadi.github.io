# DuckDB Introductory Notebook 

* This notebook is heavily inspired by the introductory workshop on PostGIS.
* The same tasks are done with DuckDB.



```python
! pip install duckdb
```

    Requirement already satisfied: duckdb in /usr/local/lib/python3.10/dist-packages (0.9.2)



```python
import duckdb
import pandas as pd
```


```python
con = duckdb.connect()
```


```python
con.install_extension('spatial')
con.load_extension('spatial')
```

## Download the data


```python
!wget http://s3.cleverelephant.ca/postgis-workshop-2020.zip

```

    --2024-02-26 12:05:22--  http://s3.cleverelephant.ca/postgis-workshop-2020.zip
    Resolving s3.cleverelephant.ca (s3.cleverelephant.ca)... 16.182.71.216, 16.182.97.240, 16.182.67.160, ...
    Connecting to s3.cleverelephant.ca (s3.cleverelephant.ca)|16.182.71.216|:80... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 22547324 (22M) [application/zip]
    Saving to: ‘postgis-workshop-2020.zip’
    
    postgis-workshop-20 100%[===================>]  21.50M  50.9MB/s    in 0.4s    
    
    2024-02-26 12:05:23 (50.9 MB/s) - ‘postgis-workshop-2020.zip’ saved [22547324/22547324]
    



```python
!unzip postgis-workshop-2020.zip
```

    Archive:  postgis-workshop-2020.zip
       creating: postgis-workshop/
       creating: postgis-workshop/printing/
      inflating: postgis-workshop/printing/nyc_data_dictionary.pdf  
      inflating: postgis-workshop/printing/postgis-workshop-exercises.docx  
      inflating: postgis-workshop/printing/postgis-workshop-exercises.pdf  
      inflating: postgis-workshop/printing/nyc_data_dictionary.docx  
       creating: postgis-workshop/data/
      inflating: postgis-workshop/data/nyc_homicides.dbf  
      inflating: postgis-workshop/data/nyc_neighborhoods.shx  
      inflating: postgis-workshop/data/nyc_neighborhoods.shp  
      inflating: postgis-workshop/data/nyc_census_blocks.shx  
      inflating: postgis-workshop/data/nyc_streets.prj  
      inflating: postgis-workshop/data/nyc_subway_stations.dbf  
      inflating: postgis-workshop/data/nyc_census_blocks.shp  
      inflating: postgis-workshop/data/nyc_subway_stations.shx  
      inflating: postgis-workshop/data/nyc_census_blocks.dbf  
      inflating: postgis-workshop/data/nyc_subway_stations.shp  
      inflating: postgis-workshop/data/nyc_neighborhoods.dbf  
      inflating: postgis-workshop/data/nyc_homicides.shx  
      inflating: postgis-workshop/data/nyc_census_sociodata.sql  
       creating: postgis-workshop/data/2000/
      inflating: postgis-workshop/data/2000/nyc_census_blocks_2000.prj  
      inflating: postgis-workshop/data/2000/nyc_census_blocks_2000.shx  
      inflating: postgis-workshop/data/2000/nyc_census_blocks_2000.shp  
      inflating: postgis-workshop/data/2000/nyc_census_blocks_2000.dbf  
      inflating: postgis-workshop/data/2000/nyc_census_sociodata_2000.sql  
      inflating: postgis-workshop/data/nyc_homicides.shp  
      inflating: postgis-workshop/data/nyc_neighborhoods.prj  
      inflating: postgis-workshop/data/nyc_streets.shp  
      inflating: postgis-workshop/data/nyc_census_blocks.prj  
      inflating: postgis-workshop/data/nyc_streets.shx  
      inflating: postgis-workshop/data/nyc_subway_stations.prj  
      inflating: postgis-workshop/data/nyc_streets.dbf  
      inflating: postgis-workshop/data/nyc_data.backup  
      inflating: postgis-workshop/data/nyc_homicides.prj  



```python
!ls
```

    postgis-workshop  postgis-workshop-2020.zip  sample_data



```python
con.sql('SELECT * FROM ST_Read("postgis-workshop/data/nyc_neighborhoods.shp");')
```




    ┌───────────────┬──────────────────────┬───────────────────────────────────────────────────────────────────────────────┐
    │   BORONAME    │         NAME         │                                     geom                                      │
    │    varchar    │       varchar        │                                   geometry                                    │
    ├───────────────┼──────────────────────┼───────────────────────────────────────────────────────────────────────────────┤
    │ Brooklyn      │ Bensonhurst          │ POLYGON ((582771.4257198056 4495167.427365481, 584651.2943549604 4497541.64…  │
    │ Manhattan     │ East Village         │ POLYGON ((585508.7534890148 4509691.267208001, 586826.3570590394 4508984.18…  │
    │ Manhattan     │ West Village         │ POLYGON ((583263.2776595836 4509242.626023987, 583276.8199068634 4509378.82…  │
    │ The Bronx     │ Throggs Neck         │ POLYGON ((597640.0090688139 4520272.719938631, 597647.7457808304 4520617.82…  │
    │ The Bronx     │ Wakefield-Williams…  │ POLYGON ((595285.2053417757 4525938.79838847, 595348.5452399419 4526158.776…  │
    │ Queens        │ Auburndale           │ POLYGON ((600973.0089608189 4510338.8574458575, 601002.162100491 4510743.04…  │
    │ Manhattan     │ Battery Park         │ POLYGON ((583408.1010054761 4508093.1107926965, 583356.0476085746 4507665.1…  │
    │ Manhattan     │ Carnegie Hill        │ POLYGON ((588501.208387079 4515525.879866604, 588125.0295366715 4514806.769…  │
    │ Staten Island │ Mariners Harbor      │ POLYGON ((570300.1080794979 4497031.156416456, 570393.8363765916 4497227.42…  │
    │ Staten Island │ Rossville            │ POLYGON ((564664.9567825551 4489358.426550738, 564771.4570512844 4489415.86…  │
    │     ·         │     ·                │                                       ·                                       │
    │     ·         │     ·                │                                       ·                                       │
    │     ·         │     ·                │                                       ·                                       │
    │ Manhattan     │ Yorkville            │ POLYGON ((589221.3042189671 4515275.918404947, 589137.4128152014 4515181.03…  │
    │ Manhattan     │ Chinatown            │ POLYGON ((584212.0263544542 4507715.994306428, 584418.5960554194 4508039.30…  │
    │ Queens        │ Bayside              │ MULTIPOLYGON (((602775.3139520675 4516699.252888854, 602773.8062540573 4516…  │
    │ Brooklyn      │ Coney Island         │ MULTIPOLYGON (((586797.2142308942 4491782.743200868, 586797.6993674854 4491…  │
    │ Queens        │ Corona               │ POLYGON ((595483.2191909261 4513816.729664179, 595482.9622063976 4513818.69…  │
    │ Brooklyn      │ Red Hook             │ MULTIPOLYGON (((584212.898102789 4502321.474446659, 584306.8204918649 45023…  │
    │ Queens        │ Douglastown-Little…  │ POLYGON ((605082.2876993549 4513540.148431633, 605091.565767156 4513533.146…  │
    │ Queens        │ Whitestone           │ MULTIPOLYGON (((600138.4932375996 4516909.499424748, 600138.4432112409 4516…  │
    │ Queens        │ Steinway             │ MULTIPOLYGON (((593231.5525230656 4515088.539060092, 593306.6081032692 4515…  │
    │ Staten Island │ Rosebank             │ POLYGON ((579051.0303273386 4495284.647006003, 579062.1215490546 4495347.57…  │
    ├───────────────┴──────────────────────┴───────────────────────────────────────────────────────────────────────────────┤
    │ 129 rows (20 shown)                                                                                        3 columns │
    └──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘




```python
con.sql('CREATE TABLE IF NOT EXISTS nyc_neighborhoods AS SELECT * FROM ST_Read("postgis-workshop/data/nyc_neighborhoods.shp");')
```

## Simple SQL


```python
con.sql('SELECT * FROM nyc_neighborhoods;')
```




    ┌───────────────┬──────────────────────┬───────────────────────────────────────────────────────────────────────────────┐
    │   BORONAME    │         NAME         │                                     geom                                      │
    │    varchar    │       varchar        │                                   geometry                                    │
    ├───────────────┼──────────────────────┼───────────────────────────────────────────────────────────────────────────────┤
    │ Brooklyn      │ Bensonhurst          │ POLYGON ((582771.4257198056 4495167.427365481, 584651.2943549604 4497541.64…  │
    │ Manhattan     │ East Village         │ POLYGON ((585508.7534890148 4509691.267208001, 586826.3570590394 4508984.18…  │
    │ Manhattan     │ West Village         │ POLYGON ((583263.2776595836 4509242.626023987, 583276.8199068634 4509378.82…  │
    │ The Bronx     │ Throggs Neck         │ POLYGON ((597640.0090688139 4520272.719938631, 597647.7457808304 4520617.82…  │
    │ The Bronx     │ Wakefield-Williams…  │ POLYGON ((595285.2053417757 4525938.79838847, 595348.5452399419 4526158.776…  │
    │ Queens        │ Auburndale           │ POLYGON ((600973.0089608189 4510338.8574458575, 601002.162100491 4510743.04…  │
    │ Manhattan     │ Battery Park         │ POLYGON ((583408.1010054761 4508093.1107926965, 583356.0476085746 4507665.1…  │
    │ Manhattan     │ Carnegie Hill        │ POLYGON ((588501.208387079 4515525.879866604, 588125.0295366715 4514806.769…  │
    │ Staten Island │ Mariners Harbor      │ POLYGON ((570300.1080794979 4497031.156416456, 570393.8363765916 4497227.42…  │
    │ Staten Island │ Rossville            │ POLYGON ((564664.9567825551 4489358.426550738, 564771.4570512844 4489415.86…  │
    │     ·         │     ·                │                                       ·                                       │
    │     ·         │     ·                │                                       ·                                       │
    │     ·         │     ·                │                                       ·                                       │
    │ Manhattan     │ Yorkville            │ POLYGON ((589221.3042189671 4515275.918404947, 589137.4128152014 4515181.03…  │
    │ Manhattan     │ Chinatown            │ POLYGON ((584212.0263544542 4507715.994306428, 584418.5960554194 4508039.30…  │
    │ Queens        │ Bayside              │ MULTIPOLYGON (((602775.3139520675 4516699.252888854, 602773.8062540573 4516…  │
    │ Brooklyn      │ Coney Island         │ MULTIPOLYGON (((586797.2142308942 4491782.743200868, 586797.6993674854 4491…  │
    │ Queens        │ Corona               │ POLYGON ((595483.2191909261 4513816.729664179, 595482.9622063976 4513818.69…  │
    │ Brooklyn      │ Red Hook             │ MULTIPOLYGON (((584212.898102789 4502321.474446659, 584306.8204918649 45023…  │
    │ Queens        │ Douglastown-Little…  │ POLYGON ((605082.2876993549 4513540.148431633, 605091.565767156 4513533.146…  │
    │ Queens        │ Whitestone           │ MULTIPOLYGON (((600138.4932375996 4516909.499424748, 600138.4432112409 4516…  │
    │ Queens        │ Steinway             │ MULTIPOLYGON (((593231.5525230656 4515088.539060092, 593306.6081032692 4515…  │
    │ Staten Island │ Rosebank             │ POLYGON ((579051.0303273386 4495284.647006003, 579062.1215490546 4495347.57…  │
    ├───────────────┴──────────────────────┴───────────────────────────────────────────────────────────────────────────────┤
    │ 129 rows (20 shown)                                                                                        3 columns │
    └──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘




```python
con.sql('SELECT name FROM nyc_neighborhoods;')
```




    ┌──────────────────────────┐
    │           NAME           │
    │         varchar          │
    ├──────────────────────────┤
    │ Bensonhurst              │
    │ East Village             │
    │ West Village             │
    │ Throggs Neck             │
    │ Wakefield-Williamsbridge │
    │ Auburndale               │
    │ Battery Park             │
    │ Carnegie Hill            │
    │ Mariners Harbor          │
    │ Rossville                │
    │     ·                    │
    │     ·                    │
    │     ·                    │
    │ Yorkville                │
    │ Chinatown                │
    │ Bayside                  │
    │ Coney Island             │
    │ Corona                   │
    │ Red Hook                 │
    │ Douglastown-Little Neck  │
    │ Whitestone               │
    │ Steinway                 │
    │ Rosebank                 │
    ├──────────────────────────┤
    │   129 rows (20 shown)    │
    └──────────────────────────┘




```python
con.sql("SELECT name, ST_TRANSFORM(geom,'EPSG:26918','EPSG:4326') FROM nyc_neighborhoods;")

```




    ┌──────────────────────┬───────────────────────────────────────────────────────────────────────────────────────────────┐
    │         NAME         │                         st_transform(geom, 'EPSG:26918', 'EPSG:4326')                         │
    │       varchar        │                                           geometry                                            │
    ├──────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────┤
    │ Bensonhurst          │ POLYGON ((40.603177255343375 -74.02166685535772, 40.624372593164594 -73.99913147363611, 40.…  │
    │ East Village         │ POLYGON ((40.73372002435467 -73.9873400213324, 40.727213268064766 -73.9718363715135, 40.721…  │
    │ West Village         │ POLYGON ((40.729909197333484 -74.01398852243662, 40.731134619728365 -74.01381006286464, 40.…  │
    │ Throggs Neck         │ POLYGON ((40.827674726653015 -73.84204449268826, 40.830782040132526 -73.84189867432876, 40.…  │
    │ Wakefield-Williams…  │ POLYGON ((40.878984357327354 -73.86909835892438, 40.88095825479006 -73.86831299466589, 40.8…  │
    │ Auburndale           │ POLYGON ((40.737800825193474 -73.8041311108687, 40.74143762207188 -73.8037206850871, 40.747…  │
    │ Battery Park         │ POLYGON ((40.719540473403555 -74.01242682766723, 40.71569090909435 -74.01309999999982, 40.7…  │
    │ Carnegie Hill        │ POLYGON ((40.78595665284569 -73.95108078565387, 40.77952000911529 -73.95564003237227, 40.78…  │
    │ Mariners Harbor      │ POLYGON ((40.621119995798594 -74.168849994841, 40.6228799879877 -74.1677200221836, 40.62723…  │
    │ Rossville            │ POLYGON ((40.55246363625957 -74.23625454561993, 40.55297272805687 -74.23499090889992, 40.55…  │
    │     ·                │                                               ·                                               │
    │     ·                │                                               ·                                               │
    │     ·                │                                               ·                                               │
    │ Yorkville            │ POLYGON ((40.7836273516971 -73.94258327845486, 40.78278181818539 -73.94359090909067, 40.785…  │
    │ Chinatown            │ POLYGON ((40.71606184385847 -74.00296019678281, 40.718952888926566 -74.00047128061712, 40.7…  │
    │ Bayside              │ MULTIPOLYGON (((40.79486303864095 -73.78174435448365, 40.794885806020595 -73.78176180947467…  │
    │ Coney Island         │ MULTIPOLYGON (((40.57227710800682 -73.97455615126611, 40.572278255014275 -73.9745504023258,…  │
    │ Corona               │ POLYGON ((40.76978083335069 -73.86860581336617, 40.76979852067137 -73.86860855831928, 40.76…  │
    │ Red Hook             │ MULTIPOLYGON (((40.667471393754646 -74.00367396642184, 40.66781347939834 -74.0025576869529,…  │
    │ Douglastown-Little…  │ POLYGON ((40.76611879344588 -73.75493600167012, 40.766054544524884 -73.75482727272683, 40.7…  │
    │ Whitestone           │ MULTIPOLYGON (((40.797082383634994 -73.8129599183423, 40.79708758238439 -73.81296041872561,…  │
    │ Steinway             │ MULTIPOLYGON (((40.781494374848094 -73.89509156091411, 40.782179436469995 -73.8941907136972…  │
    │ Rosebank             │ POLYGON ((40.604597206467325 -74.06562040674635, 40.6051629552721 -74.06548143037406, 40.60…  │
    ├──────────────────────┴───────────────────────────────────────────────────────────────────────────────────────────────┤
    │ 129 rows (20 shown)                                                                                        2 columns │
    └──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘




```python
con.sql("SELECT name FROM nyc_neighborhoods WHERE boroname  = 'Brooklyn';")
```




    ┌──────────────────────────┐
    │           NAME           │
    │         varchar          │
    ├──────────────────────────┤
    │ Bensonhurst              │
    │ Bay Ridge                │
    │ Boerum Hill              │
    │ Cobble Hill              │
    │ Downtown                 │
    │ Sunset Park              │
    │ Borough Park             │
    │ East Brooklyn            │
    │ Flatbush                 │
    │ Park Slope               │
    │ Williamsburg             │
    │ Canarsie                 │
    │ Greenwood                │
    │ Gravesend-Sheepshead Bay │
    │ Dyker Heights            │
    │ Brownsville              │
    │ Bushwick                 │
    │ Fort Green               │
    │ Mapleton-Flatlands       │
    │ Bedford-Stuyvesant       │
    │ Carroll Gardens          │
    │ Coney Island             │
    │ Red Hook                 │
    ├──────────────────────────┤
    │         23 rows          │
    └──────────────────────────┘




```python
# Question: How many neighborhoods are in Brooklyn?
con.sql("SELECT COUNT(*) FROM nyc_neighborhoods WHERE boroname  = 'Brooklyn';")
```




    ┌──────────────┐
    │ count_star() │
    │    int64     │
    ├──────────────┤
    │           23 │
    └──────────────┘




```python
# What is the number of letters in the names of all the  neighborhoods in Brooklyn?
con.sql("SELECT length(name) FROM nyc_neighborhoods WHERE boroname  = 'Brooklyn';")
```




    ┌────────────────┐
    │ length("name") │
    │     int64      │
    ├────────────────┤
    │             11 │
    │              9 │
    │             11 │
    │             11 │
    │              8 │
    │             11 │
    │             12 │
    │             13 │
    │              8 │
    │             10 │
    │             12 │
    │              8 │
    │              9 │
    │             24 │
    │             13 │
    │             11 │
    │              8 │
    │             10 │
    │             18 │
    │             18 │
    │             15 │
    │             12 │
    │              8 │
    ├────────────────┤
    │    23 rows     │
    └────────────────┘




```python
# What is the average number of letters and standard deviation of number of letters in the names of all the neighborhoods in Brooklyn?
con.sql("SELECT avg(length(name)), stddev_pop(length(name)) FROM nyc_neighborhoods WHERE boroname  = 'Brooklyn';")
```




    ┌─────────────────────┬────────────────────────────┐
    │ avg(length("name")) │ stddev_pop(length("name")) │
    │       double        │           double           │
    ├─────────────────────┼────────────────────────────┤
    │   11.73913043478261 │         3.8246044558694345 │
    └─────────────────────┴────────────────────────────┘




```python
#What is the average number of letters in the names of all the neighborhoods in New York City, reported by borough?
con.sql("SELECT avg(length(name)),BORONAME  FROM nyc_neighborhoods GROUP BY BORONAME;")
```




    ┌─────────────────────┬───────────────┐
    │ avg(length("name")) │   BORONAME    │
    │       double        │    varchar    │
    ├─────────────────────┼───────────────┤
    │   11.73913043478261 │ Brooklyn      │
    │  12.041666666666666 │ The Bronx     │
    │  11.666666666666666 │ Queens        │
    │  11.821428571428571 │ Manhattan     │
    │  12.291666666666666 │ Staten Island │
    └─────────────────────┴───────────────┘




```python
# How many records are in the nyc_streets table?
con.sql('CREATE TABLE IF NOT EXISTS nyc_streets AS SELECT * FROM ST_Read("postgis-workshop/data/nyc_streets.shp");')
con.sql("SELECT COUNT(*)  FROM nyc_streets;")
```




    ┌──────────────┐
    │ count_star() │
    │    int64     │
    ├──────────────┤
    │        19090 │
    └──────────────┘




```python
con.sql("SELECT *  FROM nyc_streets;")
```




    ┌───────┬─────────────────┬─────────┬───────────────┬──────────────────────────────────────────────────────────────────┐
    │  ID   │      NAME       │ ONEWAY  │     TYPE      │                               geom                               │
    │ int64 │     varchar     │ varchar │    varchar    │                             geometry                             │
    ├───────┼─────────────────┼─────────┼───────────────┼──────────────────────────────────────────────────────────────────┤
    │     1 │ Shore Pky S     │ NULL    │ residential   │ LINESTRING (586785.4767897038 4492901.0014554765, 586898.23200…  │
    │     2 │ NULL            │ NULL    │ footway       │ LINESTRING (586645.0073625665 4504977.750360583, 586664.224899…  │
    │     3 │ Avenue O        │ NULL    │ residential   │ LINESTRING (586750.3019977848 4496109.72213903, 586837.3726880…  │
    │     4 │ Walsh Ct        │ NULL    │ residential   │ LINESTRING (586728.695515043 4497971.05313857, 586886.35822565…  │
    │     5 │ NULL            │ NULL    │ motorway_link │ LINESTRING (586587.0531467082 4510088.250402982, 586641.733985…  │
    │     6 │ Avenue Z        │ NULL    │ residential   │ LINESTRING (586792.1590947693 4493279.321965924, 586978.533576…  │
    │     7 │ Dank Ct         │ NULL    │ residential   │ LINESTRING (586794.7541421958 4493361.728529104, 586966.095369…  │
    │     8 │ Cumberland Walk │ NULL    │ footway       │ LINESTRING (586657.467661773 4505324.904212245, 586692.4890414…  │
    │     9 │ Cumberland Walk │ NULL    │ footway       │ LINESTRING (586670.7115426222 4505521.566761277, 586667.914896…  │
    │    10 │ NULL            │ NULL    │ residential   │ LINESTRING (586598.3257999844 4510424.446496345, 586602.313725…  │
    │     · │  ·              │  ·      │      ·        │                                ·                                 │
    │     · │  ·              │  ·      │      ·        │                                ·                                 │
    │     · │  ·              │  ·      │      ·        │                                ·                                 │
    │  9997 │ 90th Ave        │ NULL    │ residential   │ LINESTRING (602494.035355788 4507244.7209426155, 602605.227936…  │
    │  9998 │ 133rd Ave       │ NULL    │ residential   │ LINESTRING (602562.1305106754 4502705.253115755, 602696.344249…  │
    │  9999 │ Aberdeen Rd     │ NULL    │ residential   │ LINESTRING (602525.5525041247 4508387.533295875, 602521.719335…  │
    │ 10000 │ Bonnie Ln       │ NULL    │ residential   │ LINESTRING (602387.8547830804 4515676.392811145, 602386.229626…  │
    │ 10001 │ Phroane Ave     │ NULL    │ residential   │ LINESTRING (602535.6637386277 4505062.916847304, 602683.937529…  │
    │ 10002 │ 186th Ln        │ NULL    │ residential   │ LINESTRING (602538.8474735785 4510144.199364001, 602500.572119…  │
    │ 10003 │ NULL            │ NULL    │ motorway_link │ LINESTRING (602611.2730119578 4499970.983219131, 602631.903262…  │
    │ 10004 │ 23rd Rd         │ NULL    │ residential   │ LINESTRING (602407.4465591786 4514766.104999339, 602439.843192…  │
    │ 10005 │ JFK EXPWY       │ NULL    │ motorway      │ LINESTRING (602612.036052657 4500020.489844459, 602648.2995835…  │
    │ 10006 │ 116th Ave       │ NULL    │ residential   │ LINESTRING (602550.5036532616 4504502.428303754, 602552.862455…  │
    ├───────┴─────────────────┴─────────┴───────────────┴──────────────────────────────────────────────────────────────────┤
    │ ? rows (>9999 rows, 20 shown)                                                                              5 columns │
    └──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘




```python
con.sql("SELECT name, starts_with(name, 'B')  FROM nyc_streets;")
```




    ┌─────────────────┬──────────────────────────┐
    │      NAME       │ starts_with("name", 'B') │
    │     varchar     │         boolean          │
    ├─────────────────┼──────────────────────────┤
    │ Shore Pky S     │ false                    │
    │ NULL            │ NULL                     │
    │ Avenue O        │ false                    │
    │ Walsh Ct        │ false                    │
    │ NULL            │ NULL                     │
    │ Avenue Z        │ false                    │
    │ Dank Ct         │ false                    │
    │ Cumberland Walk │ false                    │
    │ Cumberland Walk │ false                    │
    │ NULL            │ NULL                     │
    │  ·              │  ·                       │
    │  ·              │  ·                       │
    │  ·              │  ·                       │
    │ 90th Ave        │ false                    │
    │ 133rd Ave       │ false                    │
    │ Aberdeen Rd     │ false                    │
    │ Bonnie Ln       │ true                     │
    │ Phroane Ave     │ false                    │
    │ 186th Ln        │ false                    │
    │ NULL            │ NULL                     │
    │ 23rd Rd         │ false                    │
    │ JFK EXPWY       │ false                    │
    │ 116th Ave       │ false                    │
    ├─────────────────┴──────────────────────────┤
    │ ? rows (>9999 rows, 20 shown)    2 columns │
    └────────────────────────────────────────────┘




```python
# How many streets in NYC start with ‘B’?
con.sql("SELECT COUNT(*)  FROM nyc_streets WHERE starts_with(name, 'B') = true;")
```




    ┌──────────────┐
    │ count_star() │
    │    int64     │
    ├──────────────┤
    │         1282 │
    └──────────────┘




```python
# How many streets in NYC start with ‘B’? (second method)
con.sql("SELECT Count(*)  FROM nyc_streets WHERE name LIKE 'B%';")
```




    ┌──────────────┐
    │ count_star() │
    │    int64     │
    ├──────────────┤
    │         1282 │
    └──────────────┘




```python
# Population of NYC; reading shapefile into table & printing the first rows of the table
con.sql('CREATE TABLE IF NOT EXISTS nyc_census_blocks AS SELECT * FROM ST_Read("postgis-workshop/data/nyc_census_blocks.shp");')
con.sql("SELECT * FROM nyc_census_blocks LIMIT 5;")
```




    ┌─────────────────┬────────────┬────────────┬───┬────────────┬────────────┬───────────────┬──────────────────────┐
    │      BLKID      │ POPN_TOTAL │ POPN_WHITE │ … │ POPN_ASIAN │ POPN_OTHER │   BORONAME    │         geom         │
    │     varchar     │   int64    │   int64    │   │   int64    │   int64    │    varchar    │       geometry       │
    ├─────────────────┼────────────┼────────────┼───┼────────────┼────────────┼───────────────┼──────────────────────┤
    │ 360850009001000 │         97 │         51 │ … │          5 │          8 │ Staten Island │ POLYGON ((577856.5…  │
    │ 360850020011000 │         66 │         52 │ … │          7 │          5 │ Staten Island │ POLYGON ((578620.7…  │
    │ 360850040001000 │         62 │         14 │ … │         25 │          3 │ Staten Island │ POLYGON ((577227.2…  │
    │ 360850074001000 │        137 │         92 │ … │         13 │         20 │ Staten Island │ POLYGON ((579037.0…  │
    │ 360850096011000 │        289 │        230 │ … │         32 │         27 │ Staten Island │ POLYGON ((577652.4…  │
    ├─────────────────┴────────────┴────────────┴───┴────────────┴────────────┴───────────────┴──────────────────────┤
    │ 5 rows                                                                                     9 columns (7 shown) │
    └────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘




```python
# What is the population of NYC?
con.sql("SELECT sum(POPN_TOTAL) AS population FROM nyc_census_blocks;")
```




    ┌────────────┐
    │ population │
    │   int128   │
    ├────────────┤
    │    8174377 │
    └────────────┘




```python
# What is the population of ‘The Bronx’?
con.sql("SELECT sum(POPN_TOTAL) FROM nyc_census_blocks WHERE BORONAME = 'The Bronx';")

```




    ┌─────────────────┐
    │ sum(POPN_TOTAL) │
    │     int128      │
    ├─────────────────┤
    │         1384453 │
    └─────────────────┘




```python
# How many neighborhoods are in each borough?
con.sql('SELECT COUNT(*),BORONAME  FROM nyc_neighborhoods GROUP BY BORONAME;')
```




    ┌──────────────┬───────────────┐
    │ count_star() │   BORONAME    │
    │    int64     │    varchar    │
    ├──────────────┼───────────────┤
    │           23 │ Brooklyn      │
    │           24 │ The Bronx     │
    │           30 │ Queens        │
    │           28 │ Manhattan     │
    │           24 │ Staten Island │
    └──────────────┴───────────────┘




```python
# For each borough in NYC, what is percentage of the population is “white”?
con.sql("SELECT BORONAME , 100.0 * sum(POPN_WHITE)/sum(POPN_TOTAL) FROM nyc_census_blocks GROUP BY BORONAME;")
```




    ┌───────────────┬───────────────────────────────────────────────┐
    │   BORONAME    │ ((100.0 * sum(POPN_WHITE)) / sum(POPN_TOTAL)) │
    │    varchar    │                    double                     │
    ├───────────────┼───────────────────────────────────────────────┤
    │ Brooklyn      │                             42.80117379326865 │
    │ The Bronx     │                            27.905822732877173 │
    │ Queens        │                             39.72207739459101 │
    │ Staten Island │                              72.8942034860154 │
    │ Manhattan     │                             57.44930394804628 │
    └───────────────┴───────────────────────────────────────────────┘



## Geometries


```python
# What is the area of the ‘West Village’ neighborhood?
con.sql("SELECT ST_Area(geom) FROM nyc_neighborhoods WHERE name = 'West Village';")
```




    ┌────────────────────┐
    │   st_area(geom)    │
    │       double       │
    ├────────────────────┤
    │ 1044614.5296485956 │
    └────────────────────┘




```python
# What is the geometry type of ‘Pelham St’? The length?
con.sql("SELECT ST_GeometryType(geom) AS geom_type, name, geom from nyc_streets WHERE name = 'Pelham St';")

```




    ┌──────────────────────┬───────────┬───────────────────────────────────────────────────────────────────────────────────┐
    │      geom_type       │   NAME    │                                       geom                                        │
    │ enum('point', 'lin…  │  varchar  │                                     geometry                                      │
    ├──────────────────────┼───────────┼───────────────────────────────────────────────────────────────────────────────────┤
    │ LINESTRING           │ Pelham St │ LINESTRING (585323.7613806158 4507140.08120791, 585318.1174625934 4507190.08686…  │
    └──────────────────────┴───────────┴───────────────────────────────────────────────────────────────────────────────────┘




```python
con.sql("SELECT ST_Length(geom) AS geom_length, name from nyc_streets WHERE name = 'Pelham St';")
```




    ┌───────────────────┬───────────┐
    │    geom_length    │   NAME    │
    │      double       │  varchar  │
    ├───────────────────┼───────────┤
    │ 50.32314951660229 │ Pelham St │
    └───────────────────┴───────────┘




```python
# create a table from nyc_subway_stations shapefile in the database
con.sql('CREATE TABLE IF NOT EXISTS nyc_subway_stations AS SELECT * FROM ST_Read("postgis-workshop/data/nyc_subway_stations.shp");')

con.sql('SELECT * from nyc_subway_stations LIMIT 3;')
```




    ┌──────────┬────────┬──────────────┬──────────┬───┬───────────┬─────────┬─────────┬─────────┬──────────────────────┐
    │ OBJECTID │   ID   │     NAME     │ ALT_NAME │ … │ TRANSFERS │  COLOR  │ EXPRESS │ CLOSED  │         geom         │
    │  double  │ double │   varchar    │ varchar  │   │  varchar  │ varchar │ varchar │ varchar │       geometry       │
    ├──────────┼────────┼──────────────┼──────────┼───┼───────────┼─────────┼─────────┼─────────┼──────────────────────┤
    │      1.0 │  376.0 │ Cortlandt St │ NULL     │ … │ R,W       │ YELLOW  │ NULL    │ NULL    │ POINT (583521.8544…  │
    │      2.0 │    2.0 │ Rector St    │ NULL     │ … │ 1         │ RED     │ NULL    │ NULL    │ POINT (583324.4866…  │
    │      3.0 │    1.0 │ South Ferry  │ NULL     │ … │ 1         │ RED     │ NULL    │ NULL    │ POINT (583304.1823…  │
    ├──────────┴────────┴──────────────┴──────────┴───┴───────────┴─────────┴─────────┴─────────┴──────────────────────┤
    │ 3 rows                                                                                      15 columns (9 shown) │
    └──────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘




```python
# What is the GeoJSON representation of ‘Broad St’ subway station?
con.sql("SELECT ST_AsGeoJSON(geom), name FROM nyc_subway_stations WHERE name = 'Broad St';")
```




    ┌──────────────────────────────────────────────────────────────────────┬──────────┐
    │                          st_asgeojson(geom)                          │   NAME   │
    │                               varchar                                │ varchar  │
    ├──────────────────────────────────────────────────────────────────────┼──────────┤
    │ {"type":"Point","coordinates":[583571.9059213118,4506714.341192182]} │ Broad St │
    └──────────────────────────────────────────────────────────────────────┴──────────┘




```python
con.sql("SELECT ST_Length(geom) / 1000 AS street_length_km, name from nyc_streets;")
```




    ┌──────────────────────┬─────────────────┐
    │   street_length_km   │      NAME       │
    │        double        │     varchar     │
    ├──────────────────────┼─────────────────┤
    │   0.7974398579946621 │ Shore Pky S     │
    │  0.14879975102119924 │ NULL            │
    │    2.312304024906042 │ Avenue O        │
    │  0.16039575340314952 │ Walsh Ct        │
    │   0.0877145235596008 │ NULL            │
    │   3.3619916036595883 │ Avenue Z        │
    │  0.17333277056192278 │ Dank Ct         │
    │  0.14735137946342827 │ Cumberland Walk │
    │  0.19715823234272525 │ Cumberland Walk │
    │ 0.024186507314031266 │ NULL            │
    │            ·         │  ·              │
    │            ·         │  ·              │
    │            ·         │  ·              │
    │    1.610084884554142 │ 90th Ave        │
    │  0.31142915293481516 │ 133rd Ave       │
    │   1.2715409645153237 │ Aberdeen Rd     │
    │  0.07416356832871748 │ Bonnie Ln       │
    │   0.2153371272826522 │ Phroane Ave     │
    │  0.20217578676797485 │ 186th Ln        │
    │    0.222220074446715 │ NULL            │
    │  0.03316768893455323 │ 23rd Rd         │
    │   0.2335251107822481 │ JFK EXPWY       │
    │ 0.013016351457638234 │ 116th Ave       │
    ├──────────────────────┴─────────────────┤
    │     ? rows (>9999 rows, 20 shown)      │
    └────────────────────────────────────────┘




```python
# What is the total length of streets (in kilometers) in New York City?
con.sql("SELECT sum(ST_Length(geom) / 1000) from nyc_streets;")
```




    ┌───────────────────────────────┐
    │ sum((st_length(geom) / 1000)) │
    │            double             │
    ├───────────────────────────────┤
    │            10418.265759654627 │
    └───────────────────────────────┘




```python
# What is the area of Manhattan in acres? (using nyc_census_blocks table)
con.sql('CREATE TABLE IF NOT EXISTS nyc_census_blocks AS SELECT * FROM ST_Read("postgis-workshop/data/nyc_census_blocks.shp");')

con.sql("SELECT sum(ST_Area(geom) / 4047) FROM nyc_census_blocks WHERE BORONAME = 'Manhattan';")
```




    ┌─────────────────────────────┐
    │ sum((st_area(geom) / 4047)) │
    │           double            │
    ├─────────────────────────────┤
    │          14601.398721554813 │
    └─────────────────────────────┘




```python
# What is the area of Manhattan in acres? (using nyc_neighborhoods table)
con.sql("SELECT sum(ST_Area(geom) / 4047) FROM nyc_neighborhoods WHERE BORONAME = 'Manhattan';")
```




    ┌─────────────────────────────┐
    │ sum((st_area(geom) / 4047)) │
    │           double            │
    ├─────────────────────────────┤
    │          13965.320122391191 │
    └─────────────────────────────┘




```python
# What is the most westerly subway station?
con.sql('DESCRIBE nyc_subway_stations;')

```




    ┌─────────────┬─────────────┬─────────┬─────────┬─────────┬───────┐
    │ column_name │ column_type │  null   │   key   │ default │ extra │
    │   varchar   │   varchar   │ varchar │ varchar │ varchar │ int32 │
    ├─────────────┼─────────────┼─────────┼─────────┼─────────┼───────┤
    │ OBJECTID    │ DOUBLE      │ YES     │ NULL    │ NULL    │  NULL │
    │ ID          │ DOUBLE      │ YES     │ NULL    │ NULL    │  NULL │
    │ NAME        │ VARCHAR     │ YES     │ NULL    │ NULL    │  NULL │
    │ ALT_NAME    │ VARCHAR     │ YES     │ NULL    │ NULL    │  NULL │
    │ CROSS_ST    │ VARCHAR     │ YES     │ NULL    │ NULL    │  NULL │
    │ LONG_NAME   │ VARCHAR     │ YES     │ NULL    │ NULL    │  NULL │
    │ LABEL       │ VARCHAR     │ YES     │ NULL    │ NULL    │  NULL │
    │ BOROUGH     │ VARCHAR     │ YES     │ NULL    │ NULL    │  NULL │
    │ NGHBHD      │ VARCHAR     │ YES     │ NULL    │ NULL    │  NULL │
    │ ROUTES      │ VARCHAR     │ YES     │ NULL    │ NULL    │  NULL │
    │ TRANSFERS   │ VARCHAR     │ YES     │ NULL    │ NULL    │  NULL │
    │ COLOR       │ VARCHAR     │ YES     │ NULL    │ NULL    │  NULL │
    │ EXPRESS     │ VARCHAR     │ YES     │ NULL    │ NULL    │  NULL │
    │ CLOSED      │ VARCHAR     │ YES     │ NULL    │ NULL    │  NULL │
    │ geom        │ GEOMETRY    │ YES     │ NULL    │ NULL    │  NULL │
    ├─────────────┴─────────────┴─────────┴─────────┴─────────┴───────┤
    │ 15 rows                                               6 columns │
    └─────────────────────────────────────────────────────────────────┘




```python
# What is the most westerly subway station?
con.sql('SELECT ST_X(geom), name from nyc_subway_stations ORDER BY ST_X(geom) LIMIT 1;')
```




    ┌───────────────────┬─────────────┐
    │    st_x(geom)     │    NAME     │
    │      double       │   varchar   │
    ├───────────────────┼─────────────┤
    │ 563292.1172580556 │ Tottenville │
    └───────────────────┴─────────────┘




```python
# What is the most easterly subway station?
con.sql('SELECT ST_X(geom), name from nyc_subway_stations ORDER BY ST_X(geom) DESC LIMIT 1;')
```




    ┌───────────────────┬──────────────┐
    │    st_x(geom)     │     NAME     │
    │      double       │   varchar    │
    ├───────────────────┼──────────────┤
    │ 605412.4548804507 │ Far Rockaway │
    └───────────────────┴──────────────┘



## Spatial Relationships


```python
con.sql("SELECT * from nyc_subway_stations;")
```




    ┌──────────┬────────┬───────────────────┬───┬──────────────┬─────────┬─────────┬──────────────────────┐
    │ OBJECTID │   ID   │       NAME        │ … │    COLOR     │ EXPRESS │ CLOSED  │         geom         │
    │  double  │ double │      varchar      │   │   varchar    │ varchar │ varchar │       geometry       │
    ├──────────┼────────┼───────────────────┼───┼──────────────┼─────────┼─────────┼──────────────────────┤
    │      1.0 │  376.0 │ Cortlandt St      │ … │ YELLOW       │ NULL    │ NULL    │ POINT (583521.8544…  │
    │      2.0 │    2.0 │ Rector St         │ … │ RED          │ NULL    │ NULL    │ POINT (583324.4866…  │
    │      3.0 │    1.0 │ South Ferry       │ … │ RED          │ NULL    │ NULL    │ POINT (583304.1823…  │
    │      4.0 │  125.0 │ 138th St          │ … │ GREEN        │ NULL    │ NULL    │ POINT (590250.1059…  │
    │      5.0 │  126.0 │ 149th St          │ … │ GREEN        │ express │ NULL    │ POINT (590454.7399…  │
    │      6.0 │   45.0 │ 149th St          │ … │ RED-GREEN    │ express │ NULL    │ POINT (590465.8934…  │
    │      7.0 │  127.0 │ 161st St          │ … │ GREEN-ORANGE │ NULL    │ NULL    │ POINT (590573.1694…  │
    │      8.0 │  208.0 │ 167th St          │ … │ ORANGE       │ NULL    │ NULL    │ POINT (591252.8314…  │
    │      9.0 │  128.0 │ 167th St          │ … │ GREEN        │ NULL    │ NULL    │ POINT (590946.3972…  │
    │     10.0 │  209.0 │ 170th St          │ … │ ORANGE       │ NULL    │ NULL    │ POINT (591583.6111…  │
    │       ·  │     ·  │    ·              │ · │  ·           │  ·      │  ·      │          ·           │
    │       ·  │     ·  │    ·              │ · │  ·           │  ·      │  ·      │          ·           │
    │       ·  │     ·  │    ·              │ · │  ·           │  ·      │  ·      │          ·           │
    │    481.0 │   99.0 │ Times Sq          │ … │ RED          │ express │ NULL    │ POINT (585513.1770…  │
    │    482.0 │  904.0 │ JFK Terminal 1    │ … │ AIR-BLUE     │ NULL    │ NULL    │ POINT (602356.9561…  │
    │    483.0 │  905.0 │ JFK Terminal 2-3  │ … │ AIR-BLUE     │ NULL    │ NULL    │ POINT (602526.1125…  │
    │    484.0 │  906.0 │ JFK Terminal 4    │ … │ AIR-BLUE     │ NULL    │ NULL    │ POINT (602985.6731…  │
    │    485.0 │  907.0 │ JFK Terminal 5-6  │ … │ AIR-BLUE     │ NULL    │ NULL    │ POINT (603245.7503…  │
    │    486.0 │  908.0 │ JFK Terminal 7    │ … │ AIR-BLUE     │ NULL    │ NULL    │ POINT (602880.3266…  │
    │    487.0 │  909.0 │ JFK Terminal 8    │ … │ AIR-BLUE     │ NULL    │ NULL    │ POINT (602433.3533…  │
    │    488.0 │  903.0 │ Federal Circle    │ … │ AIR-BLUE     │ NULL    │ NULL    │ POINT (600903.1428…  │
    │    489.0 │  902.0 │ Long Term Parking │ … │ AIR-BLUE     │ NULL    │ NULL    │ POINT (599552.1129…  │
    │    490.0 │  901.0 │ Howard Beach      │ … │ AIR-BLUE     │ NULL    │ NULL    │ POINT (598862.0205…  │
    ├──────────┴────────┴───────────────────┴───┴──────────────┴─────────┴─────────┴──────────────────────┤
    │ 490 rows (20 shown)                                                            15 columns (7 shown) │
    └─────────────────────────────────────────────────────────────────────────────────────────────────────┘




```python
# What is geometry of Broad Street subway station?
con.sql("SELECT geom, name from nyc_subway_stations WHERE name = 'Broad St';")
```




    ┌─────────────────────────────────────────────┬──────────┐
    │                    geom                     │   NAME   │
    │                  geometry                   │ varchar  │
    ├─────────────────────────────────────────────┼──────────┤
    │ POINT (583571.9059213118 4506714.341192182) │ Broad St │
    └─────────────────────────────────────────────┴──────────┘




```python
# What is geometry of Broad Street subway station?
con.sql("SELECT ST_GeometryType(geom), geom, name from nyc_subway_stations WHERE name = 'Broad St';")
```




    ┌─────────────────────────────────────────────────────────────┬─────────────────────────────────────────────┬──────────┐
    │                    st_geometrytype(geom)                    │                    geom                     │   NAME   │
    │ enum('point', 'linestring', 'polygon', 'multipoint', 'mul…  │                  geometry                   │ varchar  │
    ├─────────────────────────────────────────────────────────────┼─────────────────────────────────────────────┼──────────┤
    │ POINT                                                       │ POINT (583571.9059213118 4506714.341192182) │ Broad St │
    └─────────────────────────────────────────────────────────────┴─────────────────────────────────────────────┴──────────┘




```python
con.sql("SELECT typeof(geom) from nyc_subway_stations WHERE name = 'Broad St';")
```




    ┌──────────────┐
    │ typeof(geom) │
    │   varchar    │
    ├──────────────┤
    │ GEOMETRY     │
    └──────────────┘




```python
# What is the well-known binary (WKB) of Broad Street subway station?
con.sql("SELECT ST_AsHEXWKB(geom) from nyc_subway_stations WHERE name = 'Broad St';")
```




    ┌────────────────────────────────────────────┐
    │             st_ashexwkb(geom)              │
    │                  varchar                   │
    ├────────────────────────────────────────────┤
    │ 01010000000EEBD4CF27CF2141BC17D69516315141 │
    └────────────────────────────────────────────┘




```python
# What subway station record matches that geometry?
con.sql("SELECT name from nyc_subway_stations WHERE ST_Equals(geom, ST_GeomFromHEXWKB('01010000000EEBD4CF27CF2141BC17D69516315141'));")
```




    ┌──────────┐
    │   NAME   │
    │ varchar  │
    ├──────────┤
    │ Broad St │
    └──────────┘




```python
# What is the well-known text (WKT) of Broad Street subway station?
con.sql("SELECT ST_AsText(geom) from nyc_subway_stations WHERE name = 'Broad St';")
```




    ┌─────────────────────────────────────────────┐
    │               st_astext(geom)               │
    │                   varchar                   │
    ├─────────────────────────────────────────────┤
    │ POINT (583571.9059213118 4506714.341192182) │
    └─────────────────────────────────────────────┘




```python
# What is the type of well-known text (WKT) of Broad Street subway station?
con.sql("SELECT typeof(ST_AsText(geom)) from nyc_subway_stations WHERE name = 'Broad St';")
```




    ┌─────────────────────────┐
    │ typeof(st_astext(geom)) │
    │         varchar         │
    ├─────────────────────────┤
    │ VARCHAR                 │
    └─────────────────────────┘




```python
con.sql("SELECT * from nyc_neighborhoods;")
```




    ┌───────────────┬──────────────────────┬───────────────────────────────────────────────────────────────────────────────┐
    │   BORONAME    │         NAME         │                                     geom                                      │
    │    varchar    │       varchar        │                                   geometry                                    │
    ├───────────────┼──────────────────────┼───────────────────────────────────────────────────────────────────────────────┤
    │ Brooklyn      │ Bensonhurst          │ POLYGON ((582771.4257198056 4495167.427365481, 584651.2943549604 4497541.64…  │
    │ Manhattan     │ East Village         │ POLYGON ((585508.7534890148 4509691.267208001, 586826.3570590394 4508984.18…  │
    │ Manhattan     │ West Village         │ POLYGON ((583263.2776595836 4509242.626023987, 583276.8199068634 4509378.82…  │
    │ The Bronx     │ Throggs Neck         │ POLYGON ((597640.0090688139 4520272.719938631, 597647.7457808304 4520617.82…  │
    │ The Bronx     │ Wakefield-Williams…  │ POLYGON ((595285.2053417757 4525938.79838847, 595348.5452399419 4526158.776…  │
    │ Queens        │ Auburndale           │ POLYGON ((600973.0089608189 4510338.8574458575, 601002.162100491 4510743.04…  │
    │ Manhattan     │ Battery Park         │ POLYGON ((583408.1010054761 4508093.1107926965, 583356.0476085746 4507665.1…  │
    │ Manhattan     │ Carnegie Hill        │ POLYGON ((588501.208387079 4515525.879866604, 588125.0295366715 4514806.769…  │
    │ Staten Island │ Mariners Harbor      │ POLYGON ((570300.1080794979 4497031.156416456, 570393.8363765916 4497227.42…  │
    │ Staten Island │ Rossville            │ POLYGON ((564664.9567825551 4489358.426550738, 564771.4570512844 4489415.86…  │
    │     ·         │     ·                │                                       ·                                       │
    │     ·         │     ·                │                                       ·                                       │
    │     ·         │     ·                │                                       ·                                       │
    │ Manhattan     │ Yorkville            │ POLYGON ((589221.3042189671 4515275.918404947, 589137.4128152014 4515181.03…  │
    │ Manhattan     │ Chinatown            │ POLYGON ((584212.0263544542 4507715.994306428, 584418.5960554194 4508039.30…  │
    │ Queens        │ Bayside              │ MULTIPOLYGON (((602775.3139520675 4516699.252888854, 602773.8062540573 4516…  │
    │ Brooklyn      │ Coney Island         │ MULTIPOLYGON (((586797.2142308942 4491782.743200868, 586797.6993674854 4491…  │
    │ Queens        │ Corona               │ POLYGON ((595483.2191909261 4513816.729664179, 595482.9622063976 4513818.69…  │
    │ Brooklyn      │ Red Hook             │ MULTIPOLYGON (((584212.898102789 4502321.474446659, 584306.8204918649 45023…  │
    │ Queens        │ Douglastown-Little…  │ POLYGON ((605082.2876993549 4513540.148431633, 605091.565767156 4513533.146…  │
    │ Queens        │ Whitestone           │ MULTIPOLYGON (((600138.4932375996 4516909.499424748, 600138.4432112409 4516…  │
    │ Queens        │ Steinway             │ MULTIPOLYGON (((593231.5525230656 4515088.539060092, 593306.6081032692 4515…  │
    │ Staten Island │ Rosebank             │ POLYGON ((579051.0303273386 4495284.647006003, 579062.1215490546 4495347.57…  │
    ├───────────────┴──────────────────────┴───────────────────────────────────────────────────────────────────────────────┤
    │ 129 rows (20 shown)                                                                                        3 columns │
    └──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘




```python
# What neighborhood intersects that subway station?
con.sql("SELECT name, boroname  from nyc_neighborhoods WHERE ST_Intersects(geom, ST_GeomFromHEXWKB('01010000000EEBD4CF27CF2141BC17D69516315141'));")

```




    ┌────────────────────┬───────────┐
    │        NAME        │ BORONAME  │
    │      varchar       │  varchar  │
    ├────────────────────┼───────────┤
    │ Financial District │ Manhattan │
    └────────────────────┴───────────┘




```python
# What streets are within 10 meters of Broad Street subway station?

con.sql("SELECT name from nyc_streets WHERE ST_DWithin(geom, ST_GeomFromHEXWKB('01010000000EEBD4CF27CF2141BC17D69516315141'), 10);")

```




    ┌───────────┐
    │   NAME    │
    │  varchar  │
    ├───────────┤
    │ Wall St   │
    │ Broad St  │
    │ Nassau St │
    └───────────┘




```python
# What streets are within 10 meters of Broad Street subway station? (second method)
con.sql("SELECT name from nyc_streets WHERE ST_DWithin(geom, ST_GeomFromText('POINT(583571.9059213118 4506714.341192182)'), 10);")
```




    ┌───────────┐
    │   NAME    │
    │  varchar  │
    ├───────────┤
    │ Wall St   │
    │ Broad St  │
    │ Nassau St │
    └───────────┘




```python
# What is the well-known text for the street 'Atlantic Commons'?
con.sql("SELECT ST_AsText(geom) from nyc_streets WHERE name = 'Atlantic Commons';")
```




    ┌───────────────────────────────────────────────────────────────────────────────────────┐
    │                                    st_astext(geom)                                    │
    │                                        varchar                                        │
    ├───────────────────────────────────────────────────────────────────────────────────────┤
    │ LINESTRING (586781.7015777241 4504202.153143394, 586863.5196448397 4504215.988170098) │
    └───────────────────────────────────────────────────────────────────────────────────────┘




```python
# What neighborhood and borough is 'POINT(586782 4504202)' in?
con.sql("SELECT BORONAME, NAME FROM nyc_neighborhoods WHERE ST_Intersects(ST_GeomFromText('POINT(583571.9059213118 4506714.341192182)'), geom);")
```




    ┌───────────┬────────────────────┐
    │ BORONAME  │        NAME        │
    │  varchar  │      varchar       │
    ├───────────┼────────────────────┤
    │ Manhattan │ Financial District │
    └───────────┴────────────────────┘




```python
# How many people live within 50 meters of 'POINT(586782 4504202)'?

con.sql("SELECT Sum(POPN_TOTAL) FROM nyc_census_blocks WHERE ST_DWithin(ST_GeomFromText('POINT(586782 4504202)'), geom, 50);")
```




    ┌─────────────────┐
    │ sum(POPN_TOTAL) │
    │     int128      │
    ├─────────────────┤
    │             896 │
    └─────────────────┘




```python
con.sql("SELECT * FROM nyc_streets WHERE name = 'Columbus Cir';")
```




    ┌───────┬──────────────┬─────────┬─────────┬───────────────────────────────────────────────────────────────────────────┐
    │  ID   │     NAME     │ ONEWAY  │  TYPE   │                                   geom                                    │
    │ int64 │   varchar    │ varchar │ varchar │                                 geometry                                  │
    ├───────┼──────────────┼─────────┼─────────┼───────────────────────────────────────────────────────────────────────────┤
    │ 18686 │ Columbus Cir │ yes     │ primary │ LINESTRING (585880.4300287147 4513529.824197933, 585883.3962299527 4513…  │
    └───────┴──────────────┴─────────┴─────────┴───────────────────────────────────────────────────────────────────────────┘




```python
# How far apart are 'Columbus Cir' and 'Fulton Ave'?
con.sql("SELECT ST_Distance(a.geom, b.geom) FROM nyc_streets a , nyc_streets b WHERE a.name = 'Fulton Ave' AND b.name = 'Columbus Cir';")
```




    ┌─────────────────────────────┐
    │ st_distance(a.geom, b.geom) │
    │           double            │
    ├─────────────────────────────┤
    │           9184.786200076896 │
    └─────────────────────────────┘



## Spatial Joins


```python

```


```python

```


```python

```


```python

```


```python

```

## Resources:

* [Presentation Deck of PostGIS workshop](https://docs.google.com/presentation/d/1qYXdeCIymLl32uoAHvAPrp1r-hK-_4Z8InG7sHEo6vc/edit)

* [download and unzip in GColab](https://buomsoo-kim.github.io/colab/2020/05/04/Colab-downloading-files-from-web-2.md/)

* [Reprojection of geometry column in duckdb](https://tech.marksblogg.com/duckdb-gis-spatial-extension.html)


