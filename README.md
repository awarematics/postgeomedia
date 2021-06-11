
# MGeometry Java Library(MJL)


## Supported MGemoetry Types

	MPeriod :  MPERIOD ((1556911345 1556911346), (1556911346 1556911347), ...)
	
	MDuration :  MDURATION (1000, 1000, 1000, ...)
	
 	MInstant : MINSTANT (15569113450, 15569114450, 15569115450, ...)
	
	MInt :  MINT (2 1556911346, 3 1556911347, ...)
	
 	MBool :  MBOOL (ture 1000, false 1000, true ...)   
	
 	MDouble : MDOUBLE (1743.6106216698727 1556811344, 1587.846969956488 1556911345 ...)
	
	MMultiPoint :  MMUltiPoint (((0 0) 1589302899, (1 1) 1589305899, ...) ...)

 	MString :  MSTRING (disjoint 1481480632123, meet 1481480637123 ...)

	MPoint :  MPOINT ((0.0 0.0) 1481480632123, (2.0 5.0) 1481480637123 ...)
	 
 	MLineString :  MLINESTRING ((-1 0, 0 0, 0 0.5, 5 5) 1481480632123, (0 0, -1 0) 1481480637123 ...)
	
 	MPolygon : MPOLYGON ((0 0, 1 1, 1 0, 0 0) 1000, (0 0, 1 1, 1 0, 0 0) 2000 ...)
	
	MVideo :  MVIDEO ('localhost:///tmp/drone/test1.jpg', MPOINT ((0.0 0.0) 1481480632123, (2.0 5.0) 1481480637123 ...), FRAME ((60 0 0.1 30 0 0), (60 0 0.1 30 0 0)...))
 	
	MPhoto :  MPHOTO (('localhost:///tmp/drone/test1.jpg' 200 200 60 0 0.1 30 0 0 'annotation' 'exif' 100 100) 1481480632123 ...)


## MGeometry SQL Real Examples

### Create TABLE examples with MVideo types

```

 create table car(
	taxi_id integer primary key,
	taxi_number varchar,
	taxi_model varchar,
	taxi_driver varchar
);

 

CREATE TABLE mgeometry_columns
(
	f_table_catalog character varying(256) NOT NULL,
	f_table_schema character varying(256) NOT NULL,
	f_table_name character varying(256) NOT NULL,
	f_mgeometry_column character varying(256) NOT NULL,
	f_mgeometry_segtable_name character varying(256) NOT NULL,
	mgeometry_compress character varying(256),
	coord_dimension integer,
	srid integer,
	"type" character varying(30),
	f_segtableoid character varying(256) NOT NULL,
	f_sequence_name character varying(256) NOT NULL,
	tpseg_size	integer
);


```
### Insert Examples 
```

insert into car values(1, '57NU2001', 'Optima', 'hongkd7');
insert into car values(2, '57NU2002', 'SonataYF', 'hongkd7');


select addmgeometrycolumn('public', 'car', 'mpoint', 4326, 'mpoint', 2, 50);
select addmgeometrycolumn('public', 'car', 'mvideo', 4326, 'mvideo', 2, 50);

select * from car;


``` 

### Append Examples 
```

UPDATE car 
SET    mpoint = append(mpoint, ('POINT (200 200)'::geometry)::point,'1180389003000'::bigint) 
WHERE  taxi_id = 1;

UPDATE car 
SET    mvideo = append(mvideo, ('POINT (200 200)'::geometry)::point, '1180389003000'::bigint, 1.0, 2.0, 3.0, 5.0, 6.0, 'http://u-gist/1.mp4') 
WHERE  taxi_id = 1;


### UDF Function Examples 
```
select M_AsText( M_Time( mpoint ) )
SELECT M_AsText(M_Spatial( 'MPOINT ((0.0 0.0) 1481480632123, (2.0 5.0) 1481480637123 ...)' );


### SELECT Examples 
--- 
SELECT id, mpoint
FROM bdd10k;

---
SELECT id, M_GEO2JSON( mpoint )
FROM bdd10k;

---
SELECT id, M_AsText( m_time( mpoint ) )
FROM bdd10k;

---
SELECT id, ST_AsText(m_spatial( mpoint ))
FROM bdd10k;

---
SELECT id, m_sp( mpoint )
FROM bdd10k;

SELECT id, m_spatial( mpoint )
FROM bdd10k;


``` 

### Range Queries

### Temporal Range Queries
```
---temporal range query

--- basic temporal query with index
---Execution Time: 7788.456 ms

SELECT  *
FROM car a 
WHERE M_tIntersects(a.mpoint, 'Period (1000 2000)')

SELECT  *
FROM car a, queryperiod b
WHERE M_tIntersects(a.mpoint, b.times)

--- temporal range query with optimization index
--- Execution Time: 6876.625 ms

SELECT *
FROM car a
WHERE M_tIntersects_index(a.mpoint, 'Period (1000 2000)') 	

SELECT *
FROM car a, queryperiod b
WHERE M_tIntersects_index(a.mpoint, b.times) 	

```

### Spatial Range Queries
```
--- basic spatial query with index
--- Execution Time: 31070.263 ms

SELECT *
FROM car a
WHERE M_sIntersects(a.mpoint, 'LINESTRING (-1 0, 0 0, 0 0.5, 5 5)') 

SELECT *
FROM car a, querylinestring b
WHERE M_sIntersects(a.mpoint, b.geo) 

--- spatial range query with optimization index
--- Execution Time: 7328.937 ms

SELECT *
FROM car a
WHERE M_sIntersects_index(a.mpoint, 'LINESTRING (-1 0, 0 0, 0 0.5, 5 5)') 

SELECT *
FROM car a, querylinestring b
WHERE M_sIntersects_index(a.mpoint, b.geo) 

```
### Spatial-temporal Range Queries
```

--- basic spatial-temporal query with index
--- Execution Time: 8344.807 ms
--- nest loop 1620348493.90 rows=2053055795

SELECT *
FROM car a, queryperiod b, querylinestring c
WHERE M_sIntersects(a.mpoint, 'LINESTRING (-1 0, 0 0, 0 0.5, 5 5)') 
AND M_tIntersects(a.mpoint, 'Period (1000 2000)')

SELECT *
FROM car a, queryperiod b, querylinestring c
WHERE M_sIntersects(a.mpoint, c.geo) 
AND M_tIntersects(a.mpoint, b.times)

--- spatial-temporal range query with optimization index
--- Execution Time: 7547.573 ms
--nest loop 3566866.90 rows=13518468 


SELECT *
FROM car a
WHERE M_tIntersects_id(a.mpoint, 'Period (1000 2000)') IS NOT NULL 
AND M_sIntersects_id(a.mpoint, 'LINESTRING (-1 0, 0 0, 0 0.5, 5 5)') IS NOT NULL
AND M_Intersects_index(M_tIntersects_id(a.mpoint, 'Period (1000 2000)'), M_sIntersects_id(a.mpoint, 'LINESTRING (-1 0, 0 0, 0 0.5, 5 5)'))


SELECT *
FROM car a, queryperiod b, querylinestring c
WHERE M_tIntersects_id(a.mpoint, b.times) IS NOT NULL 
AND M_sIntersects_id(a.mpoint, c.geo) IS NOT NULL
AND M_Intersects_index(M_tIntersects_id(a.mpoint, b.times), M_sIntersects_id(a.mpoint, c.geo))



```

### BerlinMOD Queries

###  1. What are the models of the vehicles with license plate numbers from Licenses?
```

explain analyze SELECT L.Licence, C.Model AS Model
FROM Cars C, Licences L
WHERE C.Licence = L.Licence;
```
### 2. How many vehicles exist that are "passenger" cars?
```
explain analyze SELECT COUNT (Licence)
FROM Cars C
WHERE Type = 'passenger';
```
### 3. Where have the vehicles with licenses from Licences been at each of the instants from QueryInstants?
```
	
explain analyze 
WITH CarList AS (
SELECT C.Licence As Licence, C.mt AS mpoint
FROM Cars C, Licences L
WHERE C.licence = L.licence
)
SELECT DISTINCT I.Instant AS Instant, C.Licence, m_snapshots(C.mpoint, I.instant) As Positions 
FROM QueryInstants I, CarList C;


```
### 4. Which vehicles have passed the points from QueryPoints?
```
	
explain analyze 
WITH CarList AS (
SELECT  P.PointId, P.geom, m_sintersects(C.mt, P.geom) AS Intersects
FROM Cars C, QueryPoints P
)
SELECT C.PointId, C.geom
FROM CarList C
where C.Intersects;

```
### 5. What is the minimum distance between places, where a vehicle with a license from Licences and a vehicle with a license from Licences have been?
```

explain analyze 
With CarList AS(
SELECT Distinct C1.Licence AS Licence1, C2.Licence AS Licence2, C1.mt AS mt1, C2.mt AS mt2
FROM Cars C1, Licences L1, Cars C2, Licences L2
WHERE C1.CarId < C2.CarId 
AND L1.LicenceId = C1.CarId AND C2.CarId = L2.LicenceId
)
SELECT m_mindistance(C.mt1, C.mt2) AS MinDist
FROM CarList C;
``` 
###  6. What are the pairs of trips from Licences that have ever been as close as 10m or less to each other?
```

explain analyze 
WITH CarList AS (
SELECT  C1.Licence AS Licence1, C2.Licence AS Licence2,  m_dwithin(C1.mt, C2.mt, 10) AS Dwithin
FROM Cars C1, Cars C2
WHERE C1.CarId < C2.CarId 
)
SELECT C.Dwithin, C.Licence1 , C.Licence2 
FROM CarList C
WHERE C.Dwithin


```
###  7. What are the licence plate numbers of the "passenger" cars that have reached the points from QueryPoints first of all "passenger" cars during the complete observation period?
```

explain analyze 
WITH CarList AS (
SELECT C.Licence AS Licence, P.Geom AS Geom, C.mt AS mpoint, m_sintersects(C.mt, P.Geom) As Intersects
FROM Cars C, QueryPoints P
WHERE C.Type = 'passenger'
)
SELECT DISTINCT CL1.Licence, CL1.Geom, m_eventtime(CL1.mpoint, CL1.Geom) AS Instant
FROM CarList CL1 
WHERE CL1.Intersects

```
### 8. What are the overall travelled distances of the vehicles with licence plate numbers from Licences during the periods from QueryPeriods?
```

explain analyze 
WITH CarList AS (
SELECT C.Licence, (m_slice(C.mt, P.Period)) AS Mpoint, P.PeriodId 
FROM  Cars C, Licences L, QueryPeriods P
WHERE C.Licence = L.licence
)
SELECT C.Licence, C.PeriodId, m_timeAtCummulative(C.Mpoint) AS Distance
FROM CarList C 
GROUP BY C.Licence, C.PeriodId, C.Mpoint

```
### 9. What is the longest distance that was travelled by a vehicle during each of the periods from QueryPeriods?
```

explain analyze 
WITH CarList AS (
SELECT m_timeAtCummulative(m_slice(C.mt, P.Period)) AS Distance, P.PeriodId 
FROM  Cars C, QueryPeriods P
)
SELECT MAX(CT.Distance)
FROM CarList CT

```
### 10. When and where did the vehicles with licence plate numbers from Licences meet other vehicles (distance < 3m) and what are the latters' licences?
```
SELECT C1.Licence AS QueryLicence , C2.Licence AS OtherLicence,
m_eventposition(C1.mt, C2.mt, 3) AS meetPos, m_eventtime(C1.mt, C2.mt, 3) AS meetTime
FROM Cars C1, Cars C2, Licences L1
WHERE C1.Licence = L1.Licence AND C2.Licence <> C1.Licence
AND st_dwithin(m_spatial(C1.mt), m_spatial(C2.mt), 3.0)
AND m_tintersects(C1.mt, m_time(C2.mp));
```
### 11. Which vehicles passed a point from QueryPoints at one of the instants from QueryInstants?
 ```
SELECT C.licence AS Licence, P.geom AS QueryPoint, I.instant As Instant
FROM Cars C, QueryPoints P, QueryInstants I
WHERE m_passes(C.mt,P.geom)
AND m_tintersects(C.mt, I.instant)
```
### 12. Which vehicles met at a point from QueryPoints at an instant from QueryInstants?
```
SELECT C1.licence AS Licence1, P.geom AS QueryPoint, I.instant As Instant
FROM Cars C1, QueryPoints P, QueryInstants I
WHERE m_meets(C1.mt, P.geom)
AND m_tintersects(C1.mt, I.instant)
```
### 13. Which vehicles travelled within one of the regions from QueryRegions during the periods from QueryPeriods?
```
SELECT C1.CarId AS CarId, P.period AS Period, R.geom AS QueryRegions
FROM Cars C1, QueryPeriods P, QueryRegions R
WHERE m_tintersects(C1.mt, P.period)
AND m_mstayIn(C1.mt, R.geom)
```
### 14. Which vehicles travelled within one of the regions from QueryRegions at one of the instants from QueryInstants?
```
SELECT C1.CarId AS CarId, I.Instant AS Instant, R.geom AS QueryRegions
FROM Cars C1, QueryInstants I, QueryRegions R
WHERE ST_WITHIN(m_spatial(C1.mt), R.geom) 
AND m_tintersects(C1.mt, I.instant)
```
### 15. Which vehicles passed a point from QueryPoints during a period from QueryPeriods?
```
SELECT C1.CarId AS CarId, Pr.period AS Period, P.geom AS QueryPoint
FROM Cars C1, QueryPeriods Pr, QueryPoints P
WHERE m_passes(C1.mt, P.geom)
AND m_tintersects(C1.mt, Pr.period)
```
### 16. List the pairs of licences for vehicles from Licences where the corresponding vehicles are both present within a region from Regions1 during a period from QueryPeriod, but do not meet each other there and then.
```
SELECT C1.Licence, C2.Licence, P.period, R.geom 
FROM Cars C1, Cars C2, QueryPeriods P, QueryRegions R
WHERE m_insides(C1.mt, R.geom) 
AND m_insides(C2.mt, R.geom)
And m_disjoints(C1.mt,C2.mt,P.period)
```
### 17. Which points from Points have been visited by a maximum number of different vehicles?
```

WITH PointCount AS (
SELECT P.PointId, COUNT(C.CarId) AS Hits
FROM Cars C, QueryPoints P
WHERE m_sintersects(C.mt, P.geom )
GROUP BY P.PointId 
)
SELECT PointId, max(P.Hits) 
FROM PointCount P

```

	
