APPENDIX: SQL Queries Used
1. GLOBAL SITUATION
--To create view I used following queries
CREATE VIEW forestation
AS
(
SELECT fa.country_code,
fa.country_name,
fa.year,
fa.forest_area_sqkm,
la.total_area_sq_mi,
r.region,
r.income_group
FROM forest_area AS fa
JOIN land_area AS la
ON fa.country_code =la.country_code
AND fa.year = la.year
JOIN regions AS r
ON r.country_code = fa.country_code
);
--Forgot to add percentage_forest
DROP VIEW IF EXISTS forestation;
CREATE VIEW forestation
AS
(
SELECT fa.country_code,
fa.country_name, ROUND((forest_area_sqkm/(total_area_sq_mi*2.59)*100)::NUMERIC,2) as
fp,
fa.year,
fa.forest_area_sqkm,
la.total_area_sq_mi,
r.region,
r.income_group,
fa.forest_area_sqkm /(la.total_area_sq_mi * 2.59)*100 AS percentage_forest
FROM forest_area AS fa
JOIN land_area AS la
ON fa.country_code =la.country_code
AND fa.year = la.year
INNER JOIN regions AS r
ON r.country_code = fa.country_code
);
--Total area of the World in 1990
SELECT forest_area_sqkm AS fa_1990
FROM forestation
WHERE year= 1990 AND country_name = 'World'
ORDER BY country_name DESC;
--Total area of the World in 2016
SELECT forest_area_sqkm AS fa_2016
FROM forestation
WHERE year= 2016 AND country_name = 'World'
ORDER BY country_name DESC;
--I got the difference
SELECT
(SELECT forest_area_sqkm AS fa_2016
FROM forestation
WHERE year= 2016 AND country_name = 'World'
ORDER BY country_name DESC)
-
(SELECT forest_area_sqkm AS fa_1990
FROM forestation
WHERE year= 1990 AND country_name = 'World'
ORDER BY country_name DESC) AS difference;
--Difference in percentage
WITH areas_2016
AS (SELECT forest_area_sqkm AS a_2016,
year
FROM forestation
WHERE country_name = 'World'
AND year = 2016),
areas_1990
AS (SELECT forest_area_sqkm AS a_1990,
year
FROM forestation
WHERE country_name = 'World'
AND year = 1990),
difference
AS (SELECT a_2016,
a_1990,
a_2016 - a_1990 difference,
( a_2016 - a_1990 ) / a_1990 * 100 AS difference_percentage
FROM areas_2016,
areas_1990)
SELECT a_2016,
a_1990,
difference,
ROUND (difference_percentage:: NUMERIC,2) AS Difference_percentage
FROM difference
--Forest lost was more than the area of PERU
SELECT country_name,
ROUND ((total_area_sq_mi*2.59)::NUMERIC,2) AS ta_sqkm
FROM forestation
WHERE year= 2016 AND total_area_sq_mi <'1324449'
ORDER BY total_area_sq_mi DESC
LIMIT 13;
--2. REGIONAL OUTLOOK Selected percentage
SELECT SUM(forest_area_sqkm)/
(SUM(total_area_sq_mi)*2.59) *100 AS forest_percentage, region
FROM forestation
WHERE year = 2016 AND country_name ='World'
GROUP BY region
ORDER BY 1 DESC;
SELECT SUM(forest_area_sqkm)/
(SUM(total_area_sq_mi)*2.59)*100 AS forest_percentage, region
FROM forestation
WHERE year = 2016
GROUP BY region
ORDER BY 1 DESC;
SELECT SUM(forest_area_sqkm)/
(SUM(total_area_sq_mi)*2.59)*100 AS forest_percentage, region
FROM forestation
WHERE year = 1990 AND country_name ='World'
GROUP BY region
ORDER BY 1 DESC;
SELECT SUM(forest_area_sqkm)/
(SUM(total_area_sq_mi)*2.59)*100 AS forest_percentage, region
FROM forestation
WHERE year = 1990
GROUP BY region
ORDER BY 1 DESC;
-- COUNTRY-LEVEL DETAILTo get the Sq- Km difference between 1990 & 2016
WITH f1
AS (SELECT DISTINCT country_name,
region,
ROUND(forest_area_sqkm::NUMERIC, 0)fa_1990
FROM forestation
WHERE year ='1990'
AND country_name NOT LIKE 'World'
AND forest_area_sqkm IS NOT NULL
ORDER BY country_name),
f2
AS (SELECT DISTINCT country_name,
region,
ROUND(forest_area_sqkm::NUMERIC,0) fa_2016
FROM forestation
WHERE year ='2016'
AND country_name NOT LIKE 'World'
AND forest_area_sqkm IS NOT NULL
ORDER BY country_name)
SELECT f1.country_name, f1.region,
fa_1990,
fa_2016,
ROUND((f1.fa_1990 - f2.fa_2016):: NUMERIC,0) AS Difference,
ROUND(( (f1.fa_1990 - f2.fa_2016) / f1.fa_1990*100) :: NUMERIC, 2) AS Percentage
FROM f1
INNER JOIN f2
ON f1.country_name = f2.country_name
ORDER BY Difference DESC;
--To get the Percentage difference between 1990 & 2016
WITH f1
AS (SELECT DISTINCT country_name,
region,
ROUND(forest_area_sqkm::NUMERIC, 2)fa_1990
FROM forestation
WHERE year ='1990'
AND country_name NOT LIKE 'World'
AND forest_area_sqkm IS NOT NULL
ORDER BY country_name),
f2
AS (SELECT DISTINCT country_name,
region,
ROUND(forest_area_sqkm::NUMERIC,2) fa_2016
FROM forestation
WHERE year ='2016'
AND country_name NOT LIKE 'World'
AND forest_area_sqkm IS NOT NULL
ORDER BY country_name)
SELECT f1.country_name, f1.region,
fa_1990,
fa_2016,
ROUND((f1.fa_1990-f2.fa_2016):: NUMERIC,2) AS Difference,
ROUND(( (f1.fa_1990-f2.fa_2016) / f1.fa_1990*100) :: NUMERIC, 2) AS Percentage
FROM f1
INNER JOIN f2
ON f1.country_name = f2.country_name
ORDER BY Percentage DESC;
--To get the output difference for para 2 for Iceland.
WITH f1
AS (SELECT DISTINCT country_name,
region,
ROUND(forest_area_sqkm::NUMERIC, 0)fa_1990
FROM forestation
WHERE year ='1990'
AND country_name NOT LIKE 'World'
AND forest_area_sqkm IS NOT NULL
ORDER BY country_name),
f2
AS (SELECT DISTINCT country_name,
region,
ROUND(forest_area_sqkm::NUMERIC,0) fa_2016
FROM forestation
WHERE year ='2016'
AND country_name NOT LIKE 'World'
AND forest_area_sqkm IS NOT NULL
ORDER BY country_name)
SELECT f1.country_name, f1.region,
fa_1990,
fa_2016,
ROUND((f2.fa_2016 - f1.fa_1990):: NUMERIC,0) AS Difference,
ROUND(( (f2.fa_2016 - f1.fa_1990) / f1.fa_1990*100) :: NUMERIC, 2) AS Percentage
FROM f1
INNER JOIN f2
ON f1.country_name = f2.country_name
ORDER BY percentage DESC;
--QUARTILES
WITH qr
AS (SELECT region,
country_name, ROUND((forest_area_sqkm/(total_area_sq_mi*2.59)*100)::NUMERIC,2) AS
fp,
CASE
WHEN fp <= 25 THEN '1st Quartile'
WHEN fp >25 and fp <= 50 THEN '2nd Quartile'
WHEN fp >50 and fp <= 75 then '3rd Quartile'
ELSE '4th Quartile'
END AS Percentile
FROM forestation
Where fp IS NOT NULL
AND country_name NOT LIKE 'World'
AND year =2016
Order by fp )
SELECT Percentile,
count(*) AS COUNT
FROM qr
GROUP BY Percentile
ORDER BY Percentile desc ;
--TOP NINE COUNTRIES
SELECT country_name,
region,ROUND((forest_area_sqkm/(total_area_sq_mi*2.59)*100)::NUMERIC,2) AS fp
FROM forestation
WHERE percentage_forest > 75
AND country_name NOT LIKE 'World'
AND year =2016
Order by fp DESC;
The END.
