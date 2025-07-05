# Netflix Movies & TvShows (SQL+Powerbi)

## Overview

This project contains advanced SQL analysis and PowerBi Report Analysis on the **Netflix dataset**, which includes:
- show_id
- type
- title 
- director
- casts  
- country 
- date_added
- release_year
- rating
- duration 
- listed_in (genres)
- description


## Objectives
- Identify content trends and shifts in genres, release patterns, and director popularity across years.
- Detect key actors, genres, and combinations that consistently co-occur or dominate viewership.
- Analyze country-level performance and uncover geographic content patterns and lags in availability.
- Measure content upload behavior over time (monthly & yearly) and find outlier months with spike uploads.
- Determine genre diversity, director impact, and the delay between release and Netflix availability.


## Schema

```sql
CREATE TABLE netflix (
    show_id      VARCHAR(5),
    type         VARCHAR(10),
    title        VARCHAR(250),
    director     VARCHAR(550),
    casts        VARCHAR(1050),
    country      VARCHAR(550),
    date_added   VARCHAR(55),
    release_year INT,
    rating       VARCHAR(15),
    duration     VARCHAR(15),
    listed_in    VARCHAR(250),
    description  VARCHAR(550)
);
```

<details> <summary><strong>Click to expand SQL Code</strong></summary>

