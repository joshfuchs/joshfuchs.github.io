# Analyzing CIA Factbook Data Using SQL

You can find the full Jupter notebook on [GitHub here](https://github.com/joshfuchs/DataScience_projects/blob/master/cia_factbook_sql.ipynb).

In this short project, we'll work with data from the CIA World Factbook, a compendium of statistics about all of the countries on Earth. The Factbook contains demographic information like the following:

- ```population``` — the global population.
- ```population_growth``` — the annual population growth rate, as a percentage.
- ```area``` — the total land and water area.

We will use SQL in Jupyter to analyze the data. 

To load sql and the database in Jupyter we start with

```
%%capture
%load_ext sql
%sql sqlite:///factbook.db
```

## Overview of the Data

Let's start with a simple query to return basic information about the database.

```
%%sql
SELECT *
    FROM facts
    LIMIT 5;
```

This let's us put together our data dictionary for the ```facts``` database

- ```name``` — the name of the country.
- ```area```— the country's total area (both land and water).
- ```area_land``` — the country's land area in square kilometers.
- ```area_water``` — the country's waterarea in square kilometers.
- ```population``` — the country's population.
- ```population_growth```— the country's population growth as a percentage.
- ```birth_rate``` — the country's birth rate, or the number of births per year per 1,000 people.
- ```death_rate``` — the country's death rate, or the number of death per year per 1,000 people.


## Summary Statistics
```
%%sql
SELECT MIN(population), MAX(population), MIN(population_growth), MAX(population_growth)
    FROM facts; 
```

A minimum population of 0 must be an error in the data, so let's look into that. Similarly, a population growth of 0 also is suspicious. 

The maximum population of 7.2 billion is about the population of the world. Let's look into these outliers.

We see there is are entries for Antarctica and the World, which account for our smallest and largest population. Let's take care to not include these in future analysis. 

There are a few countries with no population growth, mostly ones with very small populations. The exception is Greenland. For all of these, the reported population growth could be a result of rounding.


## Finding Densely Populated Countries
If we want to find countries that are densely populated, we will look for countries where the population is above average and the area is below average for a rough start. This gives us the opportunity to write a few subqueries:

```
%%sql
SELECT name, population, area
    FROM facts
    WHERE population > (SELECT AVG(population)
                       FROM facts
                       WHERE name <> 'Antarctica'
                       AND name <> 'World')
    AND area < (SELECT AVG(area)
               FROM facts
               WHERE name <> 'Antarctica'
               AND name <> 'World');
```

You can find a few more simple queries in the Jupyter notebook. While a simple project, this helps underscore some basic SQL usage. 