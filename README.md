**Formula 1 SQL Practice**

**Overview**

This repository contains a collection of SQL queries I wrote to practice and refine my database querying skills. The exercises utilize a Formula 1 dataset (til_playground.f1) to answer specific analytical questions, progressing from basic data retrieval to advanced analytical functions.

**Dataset Context**
The queries interact with a relational database schema containing various F1-related tables, including:

races

drivers

seasons

circuits

results

lap_times

pit_stops

**Skills Demonstrated**
Easy: Basic SELECT statements, Filtering (WHERE), Sorting (ORDER BY), and Simple Aggregations (COUNT).

Medium: Relational Joins (LEFT JOIN), Grouping (GROUP BY), and Aggregations (SUM).

Hard: Common Table Expressions (CTEs) and Window Functions (LAG() OVER()).

**🟢 Easy Questions**

Target: 30–60 seconds per question.

1. Grand Prix Calendar

Question: Select the NAME, DATE, and URL of all races held in the year 2023, ordered by the ROUND number. (Note: Query adapted to 2022 data)

    SELECT 
        name,
        round,
        date,
        url
    FROM til_playground.f1.races
    WHERE year = 2022
    ORDER BY round ASC;


2. Driver Nationalities

Question: Find the total number of unique NATIONALITY values present in the DRIVERS table.

    SELECT  
        nationality,
        COUNT(nationality) AS count
    FROM til_playground.f1.drivers 
    GROUP BY nationality;


3. Active Seasons

Question: List all years from the SEASONS table in descending order (newest to oldest).

    SELECT
        year
    FROM til_playground.f1.seasons
    ORDER BY year DESC;


**🟡 Medium Questions**

Target: 2–3 minutes per question. Focus: Joins and Grouping.

1. Circuit Locations

Question: Join the RACES and CIRCUITS tables to list every race NAME from the 2023 season along with its corresponding circuit LOCATION and COUNTRY.

    SELECT
        ra.name,
        ra.circuitid,
        ci.location,
        ci.country
    FROM til_playground.f1.races AS ra 
    LEFT JOIN til_playground.f1.circuits AS ci
        ON ra.circuitid = ci.circuitid
    WHERE year = 2022;


2. Point Scorers

Question: Find the top 10 drivers who have earned the most career POINTS (from the RESULTS table). Return their FORENAME, SURNAME, and the sum of their points.

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
    

3. DNFs by Constructor (Work in Progress)

Question: Using the RESULTS and STATUS tables, count how many times each CONSTRUCTORID has finished a race with the status 'Engine' or 'Gearbox'.

SELECT
-- TODO: Complete logic joining RESULTS and STATUS
FROM til_playground.f1.results


**🔴 Hard Questions**
Target: 3–4 minutes per question. Focus: Window Functions and CTEs.

1. The Lap Time Delta

Question: For a specific race (e.g., RACEID = 1100), use a Window Function to show each driver's lap time (MILLISECONDS) alongside the time of the previous lap they completed. Calculate the difference between the two.

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


2. Consistency Ranking

Question: Use a CTE to calculate the average pit stop DURATION for every driver in the 2023 season. In the main query, rank the drivers from 1 to 10 based on who has the fastest average stops.

    WITH q1 AS (
        SELECT 
            driverid,
            AVG(duration) AS avg_ps
        FROM til_playground.f1.pit_stops
        GROUP BY driverid -- Note: Added missing GROUP BY for aggregation
    ) 
    SELECT 
        driverid,
        avg_ps
    FROM q1 
    ORDER BY avg_ps DESC;



-- Performance vs. Teammate (Points Share)
-- For every race, calculate what percentage of the team's total points were contributed by each driver.
  
    WITH q1 AS (
        SELECT
            re.driverid,
            re.raceid,
            re.constructorid,
            re.points AS driver_points,
            -- Total points for the team in this race (No ORDER BY ensures a flat sum)
            SUM(re.points) OVER (PARTITION BY re.raceid, re.constructorid) AS total_team_points
        FROM til_playground.f1.results AS re
    ) 
    
    SELECT 
        dr.forename,
        dr.surname,
        c.name AS constructor_name,
        ra.name AS race_name,
        -- Multiply by 100.0 to force float math for accuracy
        (driver_points * 100.0) / NULLIF(total_team_points, 0) AS perc_teams_points
    FROM q1 
        LEFT JOIN til_playground.f1.races AS ra ON q1.raceid = ra.raceid
        LEFT JOIN til_playground.f1.drivers AS dr ON q1.driverid = dr.driverid
        LEFT JOIN til_playground.f1.constructors AS c ON q1.constructorid = c.constructorid
    WHERE ra.year = 2022
    ORDER BY ra.raceid, perc_teams_points DESC;
    with q1 as (
    select
    re.driverid
    ,re.raceid
    ,re.constructorid
    ,sum(re.points) over (
        partition by re.raceid, re.constructorid
        order by re.points) as total_team_points
    ,re.points as total_driver_points
    from til_playground.f1.results as re
    ) 
    
    select 
    dr.forename
    ,dr.surname
    ,constructorid
    ,ra.name
    ,(total_driver_points*100)/nullif(total_team_points,0) as perc_teams_points
    from q1 
        left join til_playground.f1.races as ra
        on q1.raceid=ra.raceid
        left join til_playground.f1.drivers as dr
        on q1.driverid=dr.driverid
    where ra.year =2022
    order by ra.raceid, constructorid
