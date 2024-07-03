# SQL_Queries_Collection

This repository contains a collection of SQL queries that demonstrate various techniques and solutions for common data-related problems that I created during my Data Analytics course. It includes SQL queries for different scenarios, such as data retrieval, manipulation, and analysis. Each query is accompanied by a brief description of its purpose and usage. The queries can be used as references or templates for other projects, if needed.

## Project: Rockbuster Stealth Analysis

### Introduction

Rockbuster Stealth Analysis is a movie rental company that previously had physical stores, but is now planning to launch an online video rental service. The SQL queries listed below were created to inform Rockbuster's business intelligence department for the launch strategy.

### Queries

**1. Movies that contributed the most to revenue gain (overall)**

*This query identifies the movie that generated the highest total revenue.*

```
SELECT 
    f.film_id,
    f.title,
    SUM(p.amount) AS total_revenue
FROM 
    film f
JOIN 
   inventory i ON f.film_id = i.film_id
JOIN 
   rental r ON i.inventory_id = r.inventory_id   
JOIN 
    payment p ON r.rental_id = p.rental_id
GROUP BY 
   f.film_id, f.title
ORDER BY 
  total_revenue DESC
LIMIT 1;
```

**2. Movies that contributed the least to revenue gain (overall)**

*This query identifies the movie that generated the lowest total revenue.*

```
SELECT 
    f.film_id,
    f.title,
    SUM(p.amount) AS total_revenue
FROM 
    film f
JOIN 
    inventory i ON f.film_id = i.film_id
JOIN 
    rental r ON i.inventory_id = r.inventory_id
JOIN 
    payment p ON r.rental_id = p.rental_id
GROUP BY 
    f.film_id, f.title
ORDER BY 
    total_revenue ASC
LIMIT 1;
```

**3. What was the average rental duration for all videos?**

*This query calculates the average duration that videos were rented out.*

```
SELECT 
    AVG(DATE_PART('day', r.return_date - r.rental_date)) AS average_rental_duration
FROM 
    rental r
WHERE 
    r.return_date IS NOT NULL;
```

**4. Which countries are Rockbuster customers based in?**

*This query lists all the distinct countries where Rockbuster's customers are located.*

```
SELECT 
    DISTINCT co.country
FROM 
    customer cu
JOIN 
    address a ON cu.address_id = a.address_id
JOIN 
    city ci ON a.city_id = ci.city_id
JOIN 
    country co ON ci.country_id = co.country_id;
```

**5. Where are customers with a high lifetime value based?**

*This query identifies customers with a high lifetime value based on their total spending and lists the countries they are from.*

```
SELECT 
    cu.customer_id,
    co.country,
    SUM(p.amount) AS lifetime_value
FROM 
    customer cu
JOIN 
    payment p ON cu.customer_id = p.customer_id
JOIN 
    address a ON cu.address_id = a.address_id
JOIN 
    city ci ON a.city_id = ci.city_id
JOIN 
    country co ON ci.country_id = co.country_id
GROUP BY 
    cu.customer_id, co.country
HAVING 
    SUM(p.amount) > (SELECT AVG(total_spent) FROM (SELECT SUM(amount) as total_spent FROM payment GROUP BY customer_id) sub) -- High lifetime value based on average spending
ORDER BY 
    lifetime_value DESC;
```

**6. Do sales figures vary between geographic regions?**

*This query calculates the total sales for each country to determine if there are regional differences in sales figures.*

```
SELECT 
    co.country,
    SUM(p.amount) AS total_sales
FROM 
    payment p
JOIN 
    customer cu ON p.customer_id = cu.customer_id
JOIN 
    address a ON cu.address_id = a.address_id
JOIN 
    city ci ON a.city_id = ci.city_id
JOIN 
    country co ON ci.country_id = co.country_id
GROUP BY 
    co.country
ORDER BY 
    total_sales DESC;
```

**7. List of top customers from the top countries and cities**

*This query lists the top customers from the top 10 revenue-generating countries and cities.*

```
WITH top_countries AS (
    SELECT 
        co.country_id,
        co.country,
        SUM(p.amount) AS total_revenue
    FROM 
        payment p
    JOIN 
        rental r ON p.rental_id = r.rental_id
    JOIN 
        customer c ON r.customer_id = c.customer_id
    JOIN 
        address a ON c.address_id = a.address_id
    JOIN 
        city ci ON a.city_id = ci.city_id
    JOIN 
        country co ON ci.country_id = co.country_id
    GROUP BY 
        co.country_id, co.country
    ORDER BY 
        total_revenue DESC
    LIMIT 10
),
top_cities AS (
    SELECT 
        ci.city_id,
        ci.city,
        co.country,
        SUM(p.amount) AS total_revenue
    FROM 
        payment p
    JOIN 
        rental r ON p.rental_id = r.rental_id
    JOIN 
        customer c ON r.customer_id = c.customer_id
    JOIN 
        address a ON c.address_id = a.address_id
    JOIN 
        city ci ON a.city_id = ci.city_id
    JOIN 
        country co ON ci.country_id = co.country_id
    JOIN 
        top_countries tc ON co.country_id = tc.country_id
    GROUP BY 
        ci.city_id, ci.city, co.country
    ORDER BY 
        total_revenue DESC
    LIMIT 10
)
SELECT 
    c.customer_id,
    c.first_name,
    c.last_name,
    ci.city,
    co.country,
    SUM(p.amount) AS total_revenue
FROM 
    payment p
JOIN 
    rental r ON p.rental_id = r.rental_id
JOIN 
    customer c ON r.customer_id = c.customer_id
JOIN 
    address a ON c.address_id = a.address_id
JOIN 
    city ci ON a.city_id = ci.city_id
JOIN 
    country co ON ci.country_id = co.country_id
JOIN 
    top_cities tc ON ci.city_id = tc.city_id
GROUP BY 
    c.customer_id, c.first_name, c.last_name, ci.city, co.country
ORDER BY 
    total_revenue DESC
LIMIT 5;
```
