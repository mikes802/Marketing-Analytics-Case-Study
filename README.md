# Marketing-Analytics-Case-Study
Solving the marketing analytics case study from Danny Ma's Serious SQL course.
## Danny Ma's DVD Rental Co.
During Danny's live training videos for Serious SQL, he challenged us to complete this case study on our own before looking at his solution. Starting with some of the tables created throughout the tutorial until section 5.1 as a base, I completed the case study with the knowledge gained so far in the program. As I continue with the tutorial, I imagine I will see all of the easier ways I could have done this, but this was fun nonetheless. 
## Solution Plan as Provided by Danny
1. Create a base dataset and join all relevant tables
	- [ ] complete_joint_dataset
2. Calculate customer rental counts for each category
	- [ ] category_counts
3. Aggregate all customer total films watched
	- [ ] total_counts
4. Identify the top 2 categories for each customer
	- [ ] top_categories
5. Calculate each category’s aggregated average rental count
	- [ ] average_category_count
6. Calculate the percentile metric for each customer’s top category film count
	- [ ] top_category_percentile
7. Generate our first top category insights table using all previously generated tables
	- [ ] top_category_insights
8. Generate the 2nd category insights
	- [ ] second_category_insights

*I added the following to this list to finish off the case study*

9. Generate the actor insight section
10. Generate the final master insight table
## SQL Script
> 1. Create a base dataset and join all relevant tables
>     - [X] complete_joint_dataset

This dataset was created step-by-step throughout the tutorial up to this point.
```
-- Joined Base Dataset

DROP TABLE IF EXISTS complete_joint_dataset;
CREATE TEMP TABLE complete_joint_dataset AS
SELECT
  rental.customer_id,
  inventory.film_id,
  film.title,
  rental.rental_date,
  category.name AS category_name
FROM dvd_rentals.rental
INNER JOIN dvd_rentals.inventory
  ON rental.inventory_id = inventory.inventory_id
INNER JOIN dvd_rentals.film
  ON inventory.film_id = film.film_id
INNER JOIN dvd_rentals.film_category
  ON film.film_id = film_category.film_id
INNER JOIN dvd_rentals.category
  ON film_category.category_id = category.category_id;
```
Here are the first 10 rows of the complete_joint_dataset table:
| customer_id | film_id | title            | rental_date              | category_name |
|-------------|---------|------------------|--------------------------|---------------|
| 431         | 1       | ACADEMY DINOSAUR | 2005-07-08T19:03:15.000Z | Documentary   |
| 518         | 1       | ACADEMY DINOSAUR | 2005-08-02T20:13:10.000Z | Documentary   |
| 279         | 1       | ACADEMY DINOSAUR | 2005-08-21T21:27:43.000Z | Documentary   |
| 411         | 1       | ACADEMY DINOSAUR | 2005-05-30T20:21:07.000Z | Documentary   |
| 170         | 1       | ACADEMY DINOSAUR | 2005-06-17T20:24:00.000Z | Documentary   |
| 161         | 1       | ACADEMY DINOSAUR | 2005-07-07T10:41:31.000Z | Documentary   |
| 581         | 1       | ACADEMY DINOSAUR | 2005-07-30T22:02:34.000Z | Documentary   |
| 359         | 1       | ACADEMY DINOSAUR | 2005-08-23T01:01:01.000Z | Documentary   |
| 39          | 1       | ACADEMY DINOSAUR | 2005-07-31T21:36:07.000Z | Documentary   |
| 541         | 1       | ACADEMY DINOSAUR | 2005-08-22T23:56:37.000Z | Documentary   |
> 2. Calculate customer rental counts for each category
>     - [X] category_counts
```
-- CATEGORY RENTAL COUNTS: Customer rental count by category

DROP TABLE IF EXISTS category_rental_counts;
CREATE TEMP TABLE category_rental_counts AS
SELECT
  customer_id,
  category_name,
  COUNT(*) AS rental_count,
  MAX(rental_date) AS latest_rental_date
FROM complete_joint_dataset
GROUP BY
  customer_id,
  category_name;
```
| customer_id | category_name | rental_count | latest_rental_date       |
|-------------|---------------|--------------|--------------------------|
| 538         | Comedy        | 1            | 2005-08-22T21:01:25.000Z |
| 314         | Comedy        | 3            | 2005-08-23T21:37:59.000Z |
| 286         | Travel        | 1            | 2005-08-18T18:58:35.000Z |
| 409         | Sci-Fi        | 2            | 2005-08-19T22:03:22.000Z |
| 166         | Documentary   | 2            | 2005-08-23T14:31:50.000Z |
| 316         | Documentary   | 1            | 2005-07-27T23:19:29.000Z |
| 15          | Drama         | 2            | 2005-08-22T15:36:04.000Z |
| 91          | Games         | 1            | 2005-05-26T09:17:43.000Z |
| 100         | Drama         | 1            | 2005-08-18T19:02:16.000Z |
| 31          | Drama         | 3            | 2005-07-30T04:53:56.000Z |
> 3. Aggregate all customer total films watched
>     - [X] total_counts
```
-- CUSTOMER TOTAL RENTALS  

DROP TABLE IF EXISTS customer_total_rentals;
CREATE TEMP TABLE customer_total_rentals AS
SELECT
  customer_id,
  SUM(rental_count) AS total_rental_count
FROM category_rental_counts
GROUP BY customer_id;
```
| customer_id | total_rental_count |
|-------------|--------------------|
| 184         | 23                 |
| 87          | 30                 |
| 477         | 22                 |
| 273         | 35                 |
| 550         | 32                 |
| 51          | 33                 |
| 394         | 22                 |
| 272         | 20                 |
| 70          | 18                 |
| 190         | 27                 |
> 4. Identify the top 2 categories for each customer
>     - [X] top_categories
```
-- TOP 2 RANKING: Per customer

DROP TABLE IF EXISTS top_2_ranking;
CREATE TEMP TABLE top_2_ranking AS 
WITH cte_1 AS (  
  SELECT 
    customer_id,
    category_name,
    rental_count,
    latest_rental_date,
    RANK() OVER (
      PARTITION BY customer_id
      ORDER BY rental_count DESC, latest_rental_date DESC, category_name
    ) AS rank_number
  FROM category_rental_counts
)
SELECT
  customer_id,
  category_name,
  rental_count,
  rank_number
FROM cte_1
WHERE rank_number IN (1,2)
ORDER BY customer_id;
```
| customer_id | category_name | rental_count | rank_number |
|-------------|---------------|--------------|-------------|
| 1           | Classics      | 6            | 1           |
| 1           | Comedy        | 5            | 2           |
| 2           | Sports        | 5            | 1           |
| 2           | Classics      | 4            | 2           |
| 3           | Action        | 4            | 1           |
| 3           | Sci-Fi        | 3            | 2           |
| 4           | Horror        | 3            | 1           |
| 4           | Drama         | 2            | 2           |
| 5           | Classics      | 7            | 1           |
| 5           | Animation     | 6            | 2           |
> 5. Calculate each category’s aggregated average rental count
>     - [X] average_category_count
```
-- AVERAGE CATEGORY RENTAL COUNTS: Per category

DROP TABLE IF EXISTS average_category_rental_counts;
CREATE TEMP TABLE average_category_rental_counts AS 
SELECT
  category_name,
  FLOOR(AVG(rental_count)) AS avg_rental_per_category
FROM category_rental_counts
GROUP BY category_name
ORDER BY category_name;
```
| category_name | avg_rental_per_category |
|---------------|-------------------------|
| Action        | 2                       |
| Animation     | 2                       |
| Children      | 1                       |
| Classics      | 2                       |
| Comedy        | 1                       |
| Documentary   | 2                       |
| Drama         | 2                       |
| Family        | 2                       |
| Foreign       | 2                       |
| Games         | 2                       |
| Horror        | 1                       |
| Music         | 1                       |
| New           | 2                       |
| Sci-Fi        | 2                       |
| Sports        | 2                       |
| Travel        | 1                       |
> 6. Calculate the percentile metric for each customer’s top category film count
>     - [X] top_category_percentile
```
-- PERCENTILE RANK: The top x% 

DROP TABLE IF EXISTS percentile_rank;
CREATE TEMP TABLE percentile_rank AS
SELECT
  customer_id,
  category_name,
  rental_count,
  latest_rental_date,
  CEILING(
    100 * CUME_DIST() OVER (
    PARTITION BY category_name
    ORDER BY rental_count DESC, latest_rental_date DESC
    )
  ) AS percentile
FROM category_rental_counts;
```
| customer_id | category_name | rental_count | latest_rental_date       | percentile |
|-------------|---------------|--------------|--------------------------|------------|
| 506         | Action        | 7            | 2005-08-22T08:55:43.000Z | 1          |
| 323         | Action        | 7            | 2005-08-21T04:53:08.000Z | 1          |
| 147         | Action        | 6            | 2005-08-23T11:32:35.000Z | 1          |
| 209         | Action        | 6            | 2005-08-22T18:13:07.000Z | 1          |
| 410         | Action        | 6            | 2005-08-22T16:40:21.000Z | 1          |
| 51          | Action        | 6            | 2005-08-21T14:47:09.000Z | 2          |
| 560         | Action        | 6            | 2005-08-21T08:38:24.000Z | 2          |
| 363         | Action        | 6            | 2005-08-20T09:32:56.000Z | 2          |
| 487         | Action        | 6            | 2005-08-20T08:02:22.000Z | 2          |
| 126         | Action        | 6            | 2005-08-19T13:56:58.000Z | 2          |
> 7. Generate our first top category insights table using all previously generated tables
>     - [X] top_category_insights
```
-- Join the above tables to create an output table where insights will be drawn from.

DROP TABLE IF EXISTS output_table;
CREATE TEMP TABLE output_table AS 
SELECT
  t1.customer_id,
  t1.rank_number AS category_ranking,
  t1.category_name,
  t1.rental_count,
  t1.rental_count - t2.avg_rental_per_category AS average_comparison,
  t3.percentile,
  ROUND(100 * t1.rental_count / t4.total_rental_count) AS category_percentage
FROM top_2_ranking t1 
  INNER JOIN average_category_rental_counts t2 
    ON t1.category_name = t2.category_name
  INNER JOIN percentile_rank t3 
    ON t1.customer_id = t3.customer_id AND 
       t1.category_name = t3.category_name
  INNER JOIN customer_total_rentals t4 
    ON t1.customer_id = t4.customer_id
ORDER BY customer_id, category_ranking;
```
| customer_id | category_ranking | category_name | rental_count | average_comparison | percentile | category_percentage |
|-------------|------------------|---------------|--------------|--------------------|------------|---------------------|
| 1           | 1                | Classics      | 6            | 4                  | 1          | 19                  |
| 1           | 2                | Comedy        | 5            | 4                  | 2          | 16                  |
| 2           | 1                | Sports        | 5            | 3                  | 7          | 19                  |
| 2           | 2                | Classics      | 4            | 2                  | 5          | 15                  |
| 3           | 1                | Action        | 4            | 2                  | 13         | 15                  |
| 3           | 2                | Sci-Fi        | 3            | 1                  | 18         | 12                  |
| 4           | 1                | Horror        | 3            | 2                  | 14         | 14                  |
| 4           | 2                | Drama         | 2            | 0                  | 35         | 9                   |
| 5           | 1                | Classics      | 7            | 5                  | 1          | 18                  |
| 5           | 2                | Animation     | 6            | 4                  | 2          | 16                  |
