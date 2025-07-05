# Netflix Movies & TvShows (SQL+Powerbi)

## 📝 Overview

This project contains advanced SQL analysis and Powerbi Report Analysis on the **Netflix dataset**, which includes:
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

---

## 🗂️ Schema

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
