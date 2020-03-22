# Homework

## Instructions

Clone this repo and make a PR with your answers to the questions below, and tag your reviewer in the PR.

Keep good notes of the queries you run.

## Part 1: Missing Station Data

As we've discussed in class, some of the station_id columns are NULL because of missing data. We also have inconsistent names for some of the station IDs.

Your database contains a `stations` schema:

```sql
CREATE TABLE stations
(
  id INT NOT NULL PRIMARY KEY,
  name VARCHAR(100) NOT NULL
);
```

1. Write a query that fills out this table. Try your best to pick the correct station name for each ID. You may have to make some manual choices or editing based on the inconsistencies we've found. Do try to pick the correct name for each station ID based on how popular it is in the trip data.

Hint: your query will look something like

```sql
INSERT INTO stations (SELECT ... FROM tripe);
```

Hint 2: You don't have to do it all in one query

Answer:

```sql
-- First create a temporary table that will store station ID, station name and the number of times it's shown up in the trips table.
CREATE TABLE stations_temp(id integer NOT NULL, name varchar(50) NOT NULL, total integer NOT NULL);

-- Insert query into stations_temp table.
INSERT INTO stations_temp
SELECT from_station_id as id,
from_station_name as name,
COUNT(*) as total
  FROM trips
  WHERE from_station_id IS NOT null
  GROUP BY from_station_id, from_station_name
  ORDER BY from_station_id ASC;

-- Finally, insert only the row that has the largest count for each station ID into the stations table.
INSERT INTO stations
SELECT id, max(name)
from stations_temp
  GROUP BY id
  ORDER BY id ASC;


-- Delete the temporary table.
DROP TABLE stations_temp;
```

2. Should we add any indexes to the stations table, why or why not?

Answer:

```I don't believe we need to. Everything has a unique key and value. I tried adding an index for the id, the indexes doubled in size but the execution time took longer.

```

3. Fill in the missing data in the `trips` table based on the work you did above

## Part 2: Missing Date Data

You may have noticed that we have the dates as strings. This is because the dataset we have uses an inconsistent format for dates 😔😔😔

Note that the `original_filename` column is broken down into quarters, knowing this is helpful here!

1. What's the inconsistency in date formats? You can assume that each quarter's trips are numbered sequentially, starting with the first day of the first month of that quarter.

Answer:

```Some dates are formatted as DD/MM/YYYY and others as MM/DD/YYYY.
DD/MM/YYYY: Bikeshare Ridership (2017 Q1).csv, Bikeshare Ridership (2017 Q2).csv,
MM/DD/YYYY: Bikeshare Ridership (2017 Q3).csv, Bikeshare Ridership (2017 Q4).csv (MM/DD/YY specifically), and all 2018 data.
```

2. Take a look at Postgres's [date functions](https://www.postgresql.org/docs/12/functions-datetime.html), and fill in the missing date data using proper timestamps. You may have to write several queries to do this.

Hint: your queries will look something like

```sql
UPDATE trips
SET start_time = ..., end_time = ...
WHERE ...;
```

Answer:

```sql
-- First, update the inconsistencies which are found in 2017 Q1 and 2017 Q2.
UPDATE trips
SET start_time_str = to_char(to_timestamp(start_time_str, 'DD/MM/YYYY HH24:MI:SS'), 'MM/DD/YYYY HH24:MI:SS'),
    end_time_str = to_char(to_timestamp(end_time_str, 'DD/MM/YYYY HH24:MI'), 'MM/DD/YYYY HH24:MI:SS')
WHERE original_filename = 'Bikeshare Ridership (2017 Q1).csv';

UPDATE trips
SET start_time_str = to_char(to_timestamp(start_time_str, 'DD/MM/YYYY HH24:MI:SS'), 'MM/DD/YYYY HH24:MI:SS'),
    end_time_str = to_char(to_timestamp(end_time_str, 'DD/MM/YYYY HH24:MI'), 'MM/DD/YYYY HH24:MI:SS')
WHERE original_filename = 'Bikeshare Ridership (2017 Q2).csv';

-- The second inconsistency is that in 2017 Q4 there is a row with a NULL value for end_time_str. We should remove this.
DELETE FROM trips WHERE end_time_str = 'NULLNULL';

-- Also, update the start_time_str and end_time_str with the correct year format.
UPDATE trips
SET start_time_str = to_char(to_timestamp(start_time_str, 'MM/DD/YY HH24:MI:SS'), 'MM/DD/YYYY HH24:MI:SS'),
    end_time_str = to_char(to_timestamp(end_time_str, 'MM/DD/YY HH24:MI'), 'MM/DD/YYYY HH24:MI:SS')
WHERE original_filename = 'Bikeshare Ridership (2017 Q4).csv';

-- Finally, set the start_time and end_time with the timestamp generated from start_time_str and end_time_str respectively in all
UPDATE trips
SET start_time = to_timestamp(start_time_str, 'MM/DD/YYYY HH24:MI:SS'),
end_time = to_timestamp(end_time_str, 'MM/DD/YYYY HH24:MI:SS');
```

3. Other than the index in class, would we benefit from any other indexes on this table? Why or why not?

Answer:
`I'm not sure how we would do it, but perhaps being able to split the time stamps into monthly indexes or yearly indexes could be useful.`

## Part 3: Data-driven insights

Using the table you made in part 1 and the dates you added in part 2, let's answer some questions about the bike share data

1. Build a mini-report that does a breakdown of number of trips by month
2. Build a mini-report that does a breakdown of number trips by time of day of their start and end times
3. What are the most popular stations to bike to in the summer?
4. What are the most popular stations to bike from in the winter?
5. Come up with a question that's interesting to you about this data that hasn't been asked and answer it.
