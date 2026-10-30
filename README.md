**Formula 1 SQL Practice**

**Overview**

This repository contains a collection of SQL queries I wrote to practice and refine my database querying skills. The exercises utilize a Formula 1 dataset (til_playground.f1) to answer specific analytical questions, progressing from basic data retrieval to advanced analytical functions.

**Dataset Context**

The queries interact with a relational database schema containing various F1-related tables, including:

races, drivers, seasons, circuits

results, lap_times, pit_stops, sprint_results, status, qualifying

**Skills Demonstrated**

**Fundamentals:** SELECT, WHERE, ORDER BY, LIMIT, Basic Aggregations (COUNT, SUM, AVG).

**Intermediate:** Relational Joins (LEFT JOIN, JOIN), Grouping (GROUP BY), Conditional Filtering.

**Advanced:** Common Table Expressions (CTEs), Subqueries, and Window Functions (LAG(), LEAD(), ROW_NUMBER(), OVER(PARTITION BY ...)).

**🏆 Set 1: Core SQL Fundamentals**
**🟢 Easy Questions**

1. Grand Prix Calendar: Select the NAME, DATE, and URL of all races held in the year 2022, ordered by the ROUND number.

SELECT 
    name,
    round,
    date,
    url
FROM til_playground.f1.races
WHERE year = 2022
ORDER BY round ASC;


2. Driver Nationalities: Find the total number of unique NATIONALITY values present in the DRIVERS table.

SELECT  
    nationality,
    COUNT(nationality) AS count
FROM til_playground.f1.drivers 
GROUP BY nationality;


3. Active Seasons: List all years from the SEASONS table in descending order (newest to oldest).

SELECT
    year
FROM til_playground.f1.seasons
ORDER BY year DESC;

**🟡 Medium Questions**

1. Circuit Locations: Join the RACES and CIRCUITS tables to list every race NAME from the 2022 season along with its corresponding circuit LOCATION and COUNTRY.

SELECT
    ra.name,
    ra.circuitid,
    ci.location,
    ci.country
FROM til_playground.f1.races AS ra 
LEFT JOIN til_playground.f1.circuits AS ci
    ON ra.circuitid = ci.circuitid
WHERE year = 2022;


2. Point Scorers: Find the top 10 drivers who have earned the most career POINTS.

SELECT 
    dr.forename,
    dr.surname,
    SUM(re.points) AS total_points
FROM til_playground.f1.results AS re
LEFT JOIN til_playground.f1.drivers AS dr 
    ON re.driverid = dr.driverid 
GROUP BY dr.forename, dr.surname
ORDER BY total_points DESC
LIMIT 10;


3. DNFs by Constructor: Using the RESULTS and STATUS tables, count how many times each CONSTRUCTORID has finished a race with the status 'Engine' or 'Gearbox'.

SELECT
    re.constructorid,
    COUNT(re.resultid) AS dnf_count
FROM til_playground.f1.results AS re
LEFT JOIN til_playground.f1.status AS st
    ON re.statusid = st.statusid
WHERE st.status IN ('Engine', 'Gearbox')
GROUP BY re.constructorid
ORDER BY dnf_count DESC;


**🔴 Hard Questions**

1. The Lap Time Delta: For RACEID = 1094, use a Window Function to show each driver's lap time alongside the time of the previous lap they completed. Calculate the difference.

WITH q1 AS (
    SELECT 
        driverid,
        lap,
        milliseconds,
        LAG(milliseconds) OVER (
            PARTITION BY driverid
            ORDER BY lap ASC
        ) AS prev_time
    FROM til_playground.f1.lap_times
    WHERE raceid = 1094
)
SELECT 
    driverid,
    lap,
    milliseconds,
    prev_time,
    (prev_time - milliseconds) AS difference 
FROM q1;


2. Consistency Ranking: Rank the drivers from 1 to 10 based on who has the fastest average pit stop durations.

WITH q1 AS (
    SELECT 
        driverid,
        AVG(duration) AS avg_ps
    FROM til_playground.f1.pit_stops
    GROUP BY driverid
) 
SELECT 
    driverid,
    avg_ps
FROM q1 
ORDER BY avg_ps ASC
LIMIT 10;


**🚀 Set 2: Advanced Joins, CTEs, and Window Functions**

**🟢 Easy Questions**

1. Status Lookup: Count how many times each STATUS appears in the RESULTS table. Sort by the highest count.

SELECT 
    status,
    COUNT(status) AS count_result
FROM til_playground.f1.results AS re
LEFT JOIN til_playground.f1.status AS st
    ON re.statusid = st.statusid
GROUP BY status
ORDER BY count_result DESC;


2. Recent Races (Work in Progress): List the NAME and DATE of all races held in the year 2023, ordered by DATE descending.

-- TODO: Write SELECT statement targeting races table where year = 2023


3. Specific Nationality: Find the FORENAME and SURNAME of all drivers who are 'Italian'.

SELECT 
    forename,
    surname,
    nationality 
FROM til_playground.f1.drivers
WHERE nationality = 'Italian';


**🟡 Medium Questions**

1. The "Top 10" Club: Find the total number of times each driver has finished in the top 10.

SELECT 
    dr.forename,
    dr.surname,
    re.position,
    COUNT(position) AS top_ten_times
FROM til_playground.f1.results AS re
LEFT JOIN til_playground.f1.drivers AS dr
    ON re.driverid = dr.driverid
WHERE re.position <= 10
GROUP BY 
    dr.forename,
    dr.surname,
    re.position
ORDER BY top_ten_times DESC;


2. Circuit Popularity: Identify which COUNTRY has hosted the most races.

SELECT 
    country,
    COUNT(country) AS country_count
FROM til_playground.f1.circuits AS ci
LEFT JOIN til_playground.f1.races AS ra 
    ON ci.circuitid = ra.circuitid
GROUP BY country
ORDER BY country_count DESC;


3. Sprint Points vs. Race Points: For RACEID = 1077, list each driver's points from the SPRINT_RESULTS and RESULTS tables.

SELECT 
    sr.driverid,
    sr.raceid,
    re.points AS race_points,
    sr.points AS sprint_points
FROM til_playground.f1.sprint_results AS sr
LEFT JOIN til_playground.f1.results AS re
    ON sr.driverid = re.driverid
WHERE sr.raceid = 1077;


**🔴 Hard Questions**

1. The "Bounced Back" Driver (Work in Progress): Identify drivers who finished outside the top 10 in one race but finished on the podium in the next race.

WITH q1 AS (
    SELECT 
        driverid,
        raceid,
        position 
    FROM til_playground.f1.results
    WHERE position <= 3
) 
SELECT
-- TODO: Complete logic using LAG() to check previous race position
FROM q1 
LEFT JOIN til_playground.f1.results
-- ON ...


2. Pit Stop Delta: Calculate the difference in time between a driver's first pit stop and their second pit stop for every driver in RACEID = 1077.

SELECT 
    driverid,
    stop,
    milliseconds,
    LEAD(milliseconds) OVER (
        PARTITION BY driverid
        ORDER BY stop ASC 
    ) AS next_ps,
    (milliseconds - LEAD(milliseconds) OVER (
        PARTITION BY driverid
        ORDER BY stop ASC 
    )) AS time_difference
FROM til_playground.f1.pit_stops
WHERE raceid = 1077
  AND (stop = 1 OR stop = 2);


3. Season Point Progression: Use a window function to show a running total of Constructor points.

WITH q1 AS (
    SELECT
        re.constructorid,
        re.raceid,
        SUM(points) AS total_points
    FROM til_playground.f1.results AS re
    GROUP BY re.constructorid, re.raceid
)
SELECT 
    constructorid,
    raceid,
    total_points,
    SUM(total_points) OVER (
        PARTITION BY constructorid
        ORDER BY raceid ASC
    ) AS running_total_points
FROM q1 
ORDER BY constructorid, raceid;


**🏎️ Set 3: Gap Analysis, Consistency, and Validation**

**🟢 Easy Questions**

1. Podium Count: Find the total number of podium finishes for each CONSTRUCTORID.

SELECT 
    constructorid,
    COUNT(position) AS top_3_finishes
FROM til_playground.f1.results 
WHERE position <= 3
GROUP BY constructorid
ORDER BY top_3_finishes DESC;


2. Circuit Altitudes: List all circuits located at an altitude higher than 500 meters.

SELECT 
    name,
    location,
    alt
FROM til_playground.f1.circuits
WHERE alt > 500;


3. Driver Contact Info: List all drivers who have a URL containing the word 'Wikipedia'.

SELECT 
    forename,
    surname,
    url
FROM til_playground.f1.drivers
WHERE CONTAINS(url, 'wikipedia');


🟡 Medium Questions

1. Qualifying Performance (Work in Progress): Find the average qualifying position for each constructor in the year 2022.

WITH q1 AS (
    SELECT 
        qu.constructorid,
        AVG(qu.position) AS avg_position,
        AVG(qu.position) OVER (
            PARTITION BY qu.constructorid
        ) AS season_avg_position  
    FROM til_playground.f1.qualifying AS qu
    GROUP BY qu.constructorid
) 
SELECT * 
FROM q1 
-- TODO: Complete LEFT JOIN to til_playground.f1.races


2. Home Race Heroes (Work in Progress): Find all drivers who have won a race in their own country.

SELECT
-- TODO: Join RESULTS, RACES, CIRCUITS, and DRIVERS where CIRCUITS.COUNTRY = DRIVERS.NATIONALITY
FROM til_playground.f1.races


3. Pit Stop Counts (Work in Progress): List each driver's name and the total number of pit stops they made. Include drivers with zero stops.

-- TODO: Write query

**🔴 Hard Questions**
1. The "Overtaker" Tracker: Calculate the position change for every lap for a single driver (Driver 840, Race 1094).

SELECT 
    lt.driverid,
    lt.lap,
    lt.position,
    LAG(lt.position) OVER (ORDER BY lt.lap ASC) AS prev_position,
    (lt.position - LAG(lt.position) OVER (ORDER BY lt.lap ASC)) AS pos_diff
FROM til_playground.f1.lap_times AS lt 
WHERE lt.raceid = 1094
  AND lt.driverid = 840;


2. Gap to the Leader: Calculate the time difference between each driver and the leader on Lap 5 of Race 1094.

WITH q1 AS (
    SELECT
        driverid AS leaderid,
        position AS leaderposition,
        milliseconds AS leader_ms
    FROM til_playground.f1.lap_times AS lt 
    WHERE lt.raceid = 1094 
      AND lt.lap = 5
      AND lt.position = 1 
)
SELECT
    lt.driverid,
    lt.position,
    lt.milliseconds,
    (q1.leader_ms - lt.milliseconds) AS time_diff
FROM q1 
JOIN til_playground.f1.lap_times AS lt
    ON q1.leaderid = lt.driverid
WHERE lt.raceid = 1094
  AND lt.lap = 5;


3. The Fastest Lap Leaderboard (Window Function Solution): Identify the driver who set the fastest lap for each race in the 2023 season.

WITH FastestLapsPerRace AS (
    SELECT
        resultid,
        raceid,
        driverid,
        fastestlaptime,
        ROW_NUMBER() OVER (
            PARTITION BY raceid 
            ORDER BY fastestlaptime ASC
        ) as rn
    FROM til_playground.f1.results
    WHERE fastestlaptime IS NOT NULL
)
SELECT
    r.name AS RaceName,
    d.forename,
    d.surname,
    f.fastestlaptime AS FastestLapTime
FROM FastestLapsPerRace f
JOIN til_playground.f1.races r 
    ON f.raceid = r.raceid
JOIN til_playground.f1.drivers d 
    ON f.driverid = d.driverid
WHERE f.rn = 1 
  AND r.year = 2023;
