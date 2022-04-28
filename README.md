# Marketing-Analytics-Case-Study
Solving the marketing analytics case study from [Danny Ma's Serious SQL course](https://www.datawithdanny.com/ "Data With Danny").
## Danny Ma's DVD Rental Co.
During Danny's live training videos for Serious SQL, he challenged us to complete this case study on our own before looking at his solution. Using the tables created throughout the tutorial until section 5.1 as a base, I completed the case study with the knowledge gained so far in the program. As I continue with the tutorial, I imagine I will see all of the easier ways I could have done this, but this was fun nonetheless.
# Table of Contents
1. [Key Business Requirements](#key-business-requirements)
2. [Solution Plan](#solution-plan-as-provided-by-danny)
3. [SQL Script as Provided by Danny](#sql-script)
4. [SQL Script (No Training Wheels)](#sql-script-no-training-wheels)
5. [Text file with SQL Script](https://github.com/mikes802/Marketing-Analytics-Case-Study/blob/main/SQL%20Code%202022.04.27)
## Key Business Requirements
The goal is to create a table from the database that will give the data points for the marketing email below. The data we will need includes the following for each customer:
1. The top two rental categories
2. The three most popular films for each top-two category (cannot rent a film the customer has already viewed)
3. For the first category:
   1. How many total films have they watched?
   2. How many more films has the customer watched compared to the average customer?
   3. How does the customer rank in terms of the top X% compared to all other customers in this film category?
4. For the second category:
   1. How many total films have they watched?
   2. Proportion of the total films watched (percentages should be rounded to 0 decimal places)
5. Top actor and count of films with said actor, along with three more recommendations starring this actor.

The email generated will look like this template:

![Capture 01](https://user-images.githubusercontent.com/99853599/165205218-eccc4612-4b54-41d8-a69e-346601f2a6ec.PNG)

Here is the Entity-Relationship Diagram (ERD) for the `dvd_rentals` database:

![Capture 02](https://user-images.githubusercontent.com/99853599/165311429-8d54f9a7-9e94-447f-9afb-dd6a7ae5c077.PNG)

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
For each table below I will show the first ten rows of that table. Here are the first ten rows of the `complete_joint_dataset` table:
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
> 7. Generate the 2nd category insights
>     - [X] second_category_insights
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

A note about #6 above. I originally used `PERCENT_RANK` for the window function. I also had to study up a bit on exactly what this function was doing and what the difference is between `PERCENT_RANK` and `CUME_DIST` (cumulative distribution). I discovered that `PERCENT_RANK` gives you the percent of values that fall before the value in the current row. It seems that it does not calculate the current row in the calculation, as if it doesn't exist. Not surprisingly, that means you get a percentile of 0% for the first value, since there are no other values before that value. Well, that looks strange to tell a customer that they are in the 0 percentile. I decided to go with `CUME_DIST`, since that gives you the percent of values that are equal to or come before the value in the current row. That made a lot more sense to me, so I went with that.

The above was completed with a lot of hand-holding by Danny's tutorials. I could do my work first and then check back at his solutions to see if I was on the right track. Not everything matched up perfectly, but it worked (as far as I know). After this, however, I had to take off the training wheels. I stopped referring to the tutorials and just attempted it on my own.
## SQL Script (No Training Wheels)
The last table generated has a lot of the information I need, but it's missing a very important part: movie recommendations. I struggled with this one for quite some time. After a lot of thinking, I landed on using an anti-join to generate a table of movies that customers had not seen, ranking those, and then taking the top three for the top-three recommendations. This was going to take a few steps.
### Generate the base table
I know I'll need a base table of all the movies possible by category and ranked by popularity (`times_rented` and `latest_rental_date`). By anti-joining with a list of all the movies the customers have seen in their top-ranked categories, I will be eliminating those titles from the base table and leaving a list of movies, ranked by popularity, that have not been seen.

For the base table, I took it slow and did it in a couple of steps. First, I generated a table of all movies that have been rented and ordered them by `category_name` and popularity (`times_rented` and `latest_rental_date`).
```
-- A better code for ordering categories by most popular movies with latest_rental_date

DROP TABLE IF EXISTS recommendations_table;
CREATE TEMP TABLE recommendations_table AS 
SELECT 
  film_id,
  category_name,
  title,
  MAX(rental_date) AS latest_rental_date,
  COUNT(film_id) AS times_rented
FROM complete_joint_dataset
-- WHERE category_name = 'Classics'
GROUP BY film_id, category_name, title
ORDER BY category_name, times_rented DESC, latest_rental_date DESC; 
```
| film_id | category_name | title                 | latest_rental_date       | times_rented |
|---------|---------------|-----------------------|--------------------------|--------------|
| 869     | Action        | SUSPECTS QUILLS       | 2006-02-14T15:16:03.000Z | 30           |
| 748     | Action        | RUGRATS SHAKESPEARE   | 2005-08-23T20:47:28.000Z | 30           |
| 850     | Action        | STORY SIDE            | 2005-08-23T17:48:30.000Z | 28           |
| 911     | Action        | TRIP NEWTON           | 2005-08-23T11:37:32.000Z | 28           |
| 395     | Action        | HANDICAP BOONDOCK     | 2005-08-22T18:04:22.000Z | 28           |
| 697     | Action        | PRIMARY GLASS         | 2005-08-21T08:58:38.000Z | 27           |
| 838     | Action        | STAGECOACH ARMAGEDDON | 2005-08-23T16:19:02.000Z | 26           |
| 303     | Action        | FANTASY TROOPERS      | 2005-08-22T12:16:46.000Z | 26           |
| 162     | Action        | CLUELESS BUCKET       | 2005-08-23T10:25:45.000Z | 25           |
| 417     | Action        | HILLS NEIGHBORS       | 2005-08-23T04:17:56.000Z | 25           |

Then, I used my `top_2_ranking` table from above, which gave a list of each customer's top-two categories, and I joined this with the table above to get a very large list of all films, ranked by popularity and grouped by category, for every single customer's top-two categories. This is the base table. It's big.
```
-- Every cust_id's top_2_rank recommended films without filtering out films already seen 

DROP TABLE IF EXISTS per_cust_recommendations_table_full;
CREATE TEMP TABLE per_cust_recommendations_table_full AS 
SELECT
  t1.customer_id,
  t1.category_name,
  t2.title,
  t2.latest_rental_date,
  t2.times_rented
FROM top_2_ranking t1 
LEFT JOIN recommendations_table t2 
  ON t1.category_name = t2.category_name
-- WHERE t1.customer_id = 1 AND t1.category_name = 'Classics'
ORDER BY t1.customer_id, t1.category_name, t2.times_rented DESC;
```
| customer_id | category_name | title               | latest_rental_date       | times_rented |
|-------------|---------------|---------------------|--------------------------|--------------|
| 1           | Classics      | TIMBERLAND SKY      | 2005-08-23T00:53:57.000Z | 31           |
| 1           | Classics      | FROST HEAD          | 2006-02-14T15:16:03.000Z | 30           |
| 1           | Classics      | GILMORE BOILED      | 2005-08-22T17:18:32.000Z | 28           |
| 1           | Classics      | VOYAGE LEGALLY      | 2005-08-23T22:26:47.000Z | 28           |
| 1           | Classics      | DETECTIVE VISION    | 2006-02-14T15:16:03.000Z | 27           |
| 1           | Classics      | LOATHING LEGALLY    | 2006-02-14T15:16:03.000Z | 27           |
| 1           | Classics      | WESTWARD SEABISCUIT | 2005-08-23T08:34:10.000Z | 26           |
| 1           | Classics      | ISLAND EXORCIST     | 2005-08-23T11:28:49.000Z | 26           |
| 1           | Classics      | HYDE DOCTOR         | 2006-02-14T15:16:03.000Z | 26           |
| 1           | Classics      | MALKOVICH PET       | 2005-08-23T14:23:23.000Z | 26           |

### Generate the target table
Now I could create the target table. In other words, this is the table of all the movies each customer had seen in their top-two categories.
```
-- Every cust_id's top_2_rank film names with category_name

DROP TABLE IF EXISTS movies_seen_top_categories;
CREATE TABLE movies_seen_top_categories AS (
WITH cte_1 AS (
  SELECT
    customer_id,
    film_id,
    title,
    category_name,
    COUNT(film_id) AS count
  FROM complete_joint_dataset
  -- WHERE category_name = 'Classics' AND customer_id = 1
  GROUP BY customer_id, category_name, film_id, title
  ORDER BY customer_id, category_name, count DESC
)
SELECT
  t1.customer_id,
  t2.film_id,
  t2.title,
  t2.category_name
FROM top_2_ranking t1 
  LEFT JOIN cte_1 t2 
    ON t1.customer_id = t2.customer_id
    AND t1.category_name = t2.category_name
ORDER BY customer_id, category_name
);
```
| customer_id | film_id | title                 | category_name |
|-------------|---------|-----------------------|---------------|
| 1           | 228     | DETECTIVE VISION      | Classics      |
| 1           | 341     | FROST HEAD            | Classics      |
| 1           | 663     | PATIENT SISTER        | Classics      |
| 1           | 480     | JEEPERS WEDDING       | Classics      |
| 1           | 611     | MUSKETEERS WAIT       | Classics      |
| 1           | 308     | FERRIS MOTHER         | Comedy        |
| 1           | 814     | SNATCH SLIPPER        | Comedy        |
| 1           | 317     | FIREBALL PHILADELPHIA | Comedy        |
| 1           | 159     | CLOSER BANG           | Comedy        |
| 2           | 891     | TIMBERLAND SKY        | Classics      |

### Commence the anti-join!
The commented-out lines in some of these queries is my checking the tables to make sure I'm getting the information I want. For example, I shouldn't see the first five movies in the commented-out line below come back in the table for `customer_id` = 1 (prior to the final `WHERE` clause), but I should see the last movie. 
```
-- This table is the final table that has the top three recommended movies for each customer split by category

DROP TABLE IF EXISTS top_3_recs;
CREATE TEMP TABLE top_3_recs AS (
WITH cte_1 AS (
  SELECT *,
    ROW_NUMBER() OVER (
      PARTITION BY customer_id, category_name
      ORDER BY times_rented DESC, latest_rental_date DESC
    ) AS ranking
  FROM per_cust_recommendations_table_full t1
  WHERE NOT EXISTS (
    SELECT 1
    FROM movies_seen_top_categories t2 
    WHERE
      t1.customer_id = t2.customer_id AND 
      t1.category_name = t2.category_name AND 
      t1.title = t2.title
  )
  -- AND customer_id = 1 AND category_name = 'Classics' AND title IN ('FROST HEAD', 'JEEPERS WEDDING', 'DETECTIVE VISION', 'MUSKETEERS WAIT', 'PATIENT SISTER', 'ISLAND EXORCIST')
  ORDER BY customer_id, category_name, times_rented DESC
)
SELECT *
FROM cte_1
WHERE ranking IN (1,2,3)
);
```
| customer_id | category_name | title               | latest_rental_date       | times_rented | ranking |
|-------------|---------------|---------------------|--------------------------|--------------|---------|
| 1           | Classics      | TIMBERLAND SKY      | 2005-08-23T00:53:57.000Z | 31           | 1       |
| 1           | Classics      | VOYAGE LEGALLY      | 2005-08-23T22:26:47.000Z | 28           | 2       |
| 1           | Classics      | GILMORE BOILED      | 2005-08-22T17:18:32.000Z | 28           | 3       |
| 1           | Comedy        | ZORRO ARK           | 2005-08-23T17:56:01.000Z | 31           | 1       |
| 1           | Comedy        | CAT CONEHEADS       | 2006-02-14T15:16:03.000Z | 30           | 2       |
| 1           | Comedy        | OPERATION OPERATION | 2006-02-14T15:16:03.000Z | 27           | 3       |
| 2           | Classics      | FROST HEAD          | 2006-02-14T15:16:03.000Z | 30           | 1       |
| 2           | Classics      | VOYAGE LEGALLY      | 2005-08-23T22:26:47.000Z | 28           | 2       |
| 2           | Classics      | GILMORE BOILED      | 2005-08-22T17:18:32.000Z | 28           | 3       |
| 2           | Sports        | GLEAMING JAWBREAKER | 2006-02-14T15:16:03.000Z | 29           | 1       |

This `top_3_recs` table gives "the top three recommended movies" for each customer's top-two categories. This means they have never seen these movies before and they are ranked by popularity.
### Create the final output table for insights 1 & 2
It took me a little time to figure this one out. I wanted to join `output_table` with `top_3_recs`, but I needed to get the movie titles to pivot so they were no longer listed under `title` but split up by separate columns labelled `rec_movie_1`, `rec_movie_2`, and `rec_movie_3`. This is how I did it:
```
-- Mega table that can now be used to populate the final insight results (for 1 and 2)

DROP TABLE IF EXISTS insights_inputs;
CREATE TEMP TABLE insights_inputs AS (
WITH rec_movie_1 AS (
  SELECT
    customer_id,
    category_name,
    title
  FROM top_3_recs
  WHERE ranking = 1
),
rec_movie_2 AS (
   SELECT
    customer_id,
    category_name,
    title
  FROM top_3_recs
  WHERE ranking = 2
),
rec_movie_3 AS (
 SELECT
    customer_id,
    category_name,
    title
  FROM top_3_recs
  WHERE ranking = 3
)
SELECT
    t1.customer_id,
    t1.category_ranking,
    t1.category_name,
    t1.rental_count,
    t1.average_comparison,
    t1.percentile,
    t1.category_percentage,
    t2.title AS rec_movie_1,
    t3.title AS rec_movie_2,
    t4.title AS rec_movie_3
FROM output_table t1 
  LEFT JOIN rec_movie_1 t2 
    ON t1.customer_id = t2.customer_id
    AND t1.category_name = t2.category_name
  LEFT JOIN rec_movie_2 t3 
    ON t1.customer_id = t3.customer_id
    AND t1.category_name = t3.category_name
  LEFT JOIN rec_movie_3 t4 
    ON t1.customer_id = t4.customer_id
    AND t1.category_name = t4.category_name
);
```
| customer_id | category_ranking | category_name | rental_count | average_comparison | percentile | category_percentage | rec_movie_1         | rec_movie_2         | rec_movie_3         |
|-------------|------------------|---------------|--------------|--------------------|------------|---------------------|---------------------|---------------------|---------------------|
| 1           | 1                | Classics      | 6            | 4                  | 1          | 19                  | TIMBERLAND SKY      | VOYAGE LEGALLY      | GILMORE BOILED      |
| 1           | 2                | Comedy        | 5            | 4                  | 2          | 16                  | ZORRO ARK           | CAT CONEHEADS       | OPERATION OPERATION |
| 2           | 1                | Sports        | 5            | 3                  | 7          | 19                  | GLEAMING JAWBREAKER | TALENTED HOMICIDE   | SATURDAY LAMBS      |
| 2           | 2                | Classics      | 4            | 2                  | 5          | 15                  | FROST HEAD          | VOYAGE LEGALLY      | GILMORE BOILED      |
| 3           | 1                | Action        | 4            | 2                  | 13         | 15                  | SUSPECTS QUILLS     | RUGRATS SHAKESPEARE | STORY SIDE          |
| 3           | 2                | Sci-Fi        | 3            | 1                  | 18         | 12                  | GOODFELLAS SALUTE   | ENGLISH BULWORTH    | GRAFFITI LOVE       |
| 4           | 1                | Horror        | 3            | 2                  | 14         | 14                  | PULP BEVERLY        | FAMILY SWEET        | SWARM GOLD          |
| 4           | 2                | Drama         | 2            | 0                  | 35         | 9                   | HOBBIT ALIEN        | HARRY IDAHO         | WITCHES PANIC       |
| 5           | 1                | Classics      | 7            | 5                  | 1          | 18                  | TIMBERLAND SKY      | FROST HEAD          | GILMORE BOILED      |
| 5           | 2                | Animation     | 6            | 4                  | 2          | 16                  | JUGGLER HARDLY      | DOGMA FAMILY        | STORM HAPPINESS     |

Now I have a nice table that contains all of the information needed for insights one and two! There is the small problem that there are still two rows stacked for each customer, one row for their top-ranked category and one for their second-ranked category. Ideally, each customer would have just one row. But I'll tackle that later. For now, on to the actor insight!

> 9. Generate the actor insight section

Doing that last anti-join was helpful, as it occurred to me that I would probably have to do that again for this section. I developed the following checklist to help me strategize a plan of attack:
- [ ] 9.1 Generate table showing the top-watched actor per customer_id.
- [ ] 9.2 Use that table and join it to the other relevant tables to get a list of all movies these actors acted in, ordered by popularity (`rental_count`, `latest_rental_date`); this will be the base table for an anti-join.
- [ ] 9.3 Use the first list again to get a list of movies starring the top-watched actor that each customer has already seen; this is the target table for an anti-join
- [ ] 9.4 Anti-join these tables to get a table with customer_id, actor's first and last name, the movies this actor has acted in that the customer has NOT seen, in order of popularity.
- [ ] 9.5 Add rank/row to the last table and just get the top 3 movies per customer.
- [ ] 9.6 Do three CTEs making tables with recommended movie 1, 2, 3 respectively and join together so that each customer_id has its own row with the movies listed out horizontally.
- [ ] 9.7 Use this table to do the script table (with text).

To start, I need a dataset similar to `complete_joint_dataset` but with actor names pulled in. I need to know all of the actors for all of the movies that each customer has seen.
```
-- Join the original joined dataset with actor-related information now to start these explorations.

DROP TABLE IF EXISTS actor_dataset;
CREATE TEMP TABLE actor_dataset AS 
SELECT 
  complete_joint_dataset.*,
  dvd_rentals.actor.first_name,
  dvd_rentals.actor.last_name
FROM complete_joint_dataset
  LEFT JOIN dvd_rentals.film_actor
    ON complete_joint_dataset.film_id = film_actor.film_id
  LEFT JOIN dvd_rentals.actor 
    ON dvd_rentals.film_actor.actor_id = dvd_rentals.actor.actor_id;
```
| customer_id | film_id | title           | rental_date              | category_name | first_name | last_name |
|-------------|---------|-----------------|--------------------------|---------------|------------|-----------|
| 130         | 80      | BLANKET BEVERLY | 2005-05-24T22:53:30.000Z | Family        | FRED       | COSTNER   |
| 130         | 80      | BLANKET BEVERLY | 2005-05-24T22:53:30.000Z | Family        | ALAN       | DREYFUSS  |
| 130         | 80      | BLANKET BEVERLY | 2005-05-24T22:53:30.000Z | Family        | BURT       | TEMPLE    |
| 130         | 80      | BLANKET BEVERLY | 2005-05-24T22:53:30.000Z | Family        | THORA      | TEMPLE    |
| 459         | 333     | FREAKY POCUS    | 2005-05-24T22:54:33.000Z | Music         | TOM        | MIRANDA   |
| 459         | 333     | FREAKY POCUS    | 2005-05-24T22:54:33.000Z | Music         | MATTHEW    | LEIGH     |
| 459         | 333     | FREAKY POCUS    | 2005-05-24T22:54:33.000Z | Music         | SIDNEY     | CROWE     |
| 459         | 333     | FREAKY POCUS    | 2005-05-24T22:54:33.000Z | Music         | KEVIN      | GARLAND   |
| 459         | 333     | FREAKY POCUS    | 2005-05-24T22:54:33.000Z | Music         | FAY        | WINSLET   |
| 408         | 373     | GRADUATE LORD   | 2005-05-24T23:03:39.000Z | Children      | EWAN       | GOODING   |

- [X] 9.1 Generate table showing the top-watched actor per customer_id.

I thought about this one. If I rent nine movies over time, five of them are "Die Hard", but the other four star Bradley Cooper in four different movies, who do I like to watch more? Bruce Willis or Bradley Cooper? I think it must be Bradley Cooper. Obviously I like "Die Hard", but if that's the only Bruce Willis movie I'm renting, then I think it must be the action-packed "now I know what a TV dinner feels like" story that I like. On the other hand, even though I'm only watching each Bradley Cooper movie once, I clearly like this actor enough that I'm trying out a bunch of different movies with him.

All that is to say: I figure I need to eliminate repeat viewings in order to get an accurate idea of each customer's favorite actor. If I was analyzing my own information, I should only count "Die Hard" once.
```
-- This gets you the actors each customer has seen the most after filtering out repeat viewings of movies.
-- According to the business requirements, ties should be handled by taking the first actor listed by alphabetical order (`first_name`).

DROP TABLE IF EXISTS customer_actors;
CREATE TEMP TABLE customer_actors AS (
WITH cte_1 AS (
  SELECT
      customer_id,
      film_id,
      title,
      first_name,
      last_name,
      COUNT(*) AS actor_count_same_movie,
      MAX(rental_date) AS latest_rental_date
  FROM actor_dataset
  GROUP BY
     customer_id,
     film_id,
     title,
     first_name,
     last_name
  ORDER BY actor_count_same_movie DESC
)
SELECT
  customer_id,
  first_name,
  last_name,
  COUNT(*) AS actor_count,
  MAX(latest_rental_date) AS latest_rental_date
FROM cte_1
GROUP BY
  customer_id,
  first_name,
  last_name
ORDER BY customer_id, actor_count DESC, first_name
);
```
| customer_id | first_name | last_name | actor_count | latest_rental_date       |
|-------------|------------|-----------|-------------|--------------------------|
| 1           | SCARLETT   | BENING    | 4           | 2005-07-29T03:58:49.000Z |
| 1           | VAL        | BOLGER    | 4           | 2005-08-22T19:41:37.000Z |
| 1           | ED         | CHASE     | 3           | 2005-07-31T02:42:18.000Z |
| 1           | NICK       | STALLONE  | 3           | 2005-08-19T13:56:54.000Z |
| 1           | ANNE       | CRONYN    | 2           | 2005-08-17T12:37:54.000Z |
| 1           | BOB        | FAWCETT   | 2           | 2005-08-19T13:56:54.000Z |
| 1           | BURT       | POSEY     | 2           | 2005-08-17T12:37:54.000Z |
| 1           | BURT       | TEMPLE    | 2           | 2005-08-02T15:36:52.000Z |
| 1           | CHRISTIAN  | GABLE     | 2           | 2005-07-31T02:42:18.000Z |
| 1           | CHRISTIAN  | AKROYD    | 2           | 2005-07-28T09:04:45.000Z |

Now I can get the top-watched actor per `customer_id`.
```
-- This table has the top-watched actor per customer.

DROP TABLE IF EXISTS cust_top_actors;
CREATE TEMP TABLE cust_top_actors AS (
WITH cte_1 AS (
  SELECT *,
    ROW_NUMBER() OVER (
      PARTITION BY customer_id
      ORDER BY actor_count DESC, first_name
    ) AS row_num
  FROM customer_actors
)
SELECT
  customer_id,
  first_name,
  last_name
FROM cte_1
WHERE row_num = 1
);
```
| customer_id | first_name | last_name |
|-------------|------------|-----------|
| 1           | SCARLETT   | BENING    |
| 2           | GINA       | DEGENERES |
| 3           | JAYNE      | NOLTE     |
| 4           | KIRK       | JOVOVICH  |
| 5           | SUSAN      | DAVIS     |
| 6           | GREGORY    | GOODING   |
| 7           | ANGELA     | HUDSON    |
| 8           | LAURENCE   | BULLOCK   |
| 9           | HENRY      | BERRY     |
| 10          | KARL       | BERRY     |

- [X] 9.2 Use that table and join it to the other relevant tables to get a list of all movies these actors acted in, ordered by popularity (`rental_count`, `latest_rental_date`); this will be the base table for an anti-join.

For this one, I felt like I needed to take this one step at a time. I needed to join quite a few tables. So I mapped this out first, going from each actor's name back to the inventory table that tells us which movies were rented. This is how I was thinking in terms of tables and foreign keys:

`actor.actor_id`▶️`film_actor.actor_id`▶️`film_actor.film_id`▶️`film.film_id`▶️`inventory.film_id`▶️`inventory.inventory_id`▶️`rental.inventory_id` 

I knew the columns I wanted and the path to take, so this helped me write this query:
```
-- This is the raw table of all movies customer-favorite actors acted in

DROP TABLE IF EXISTS fav_actor_movies_no_filter;
CREATE TEMP TABLE fav_actor_movies_no_filter AS 
SELECT
  t1.customer_id,
  t1.first_name,
  t1.last_name,
  t4.title,
  t6.inventory_id,
  t6.rental_date
FROM cust_top_actors t1
  LEFT JOIN dvd_rentals.actor t2 
    ON t1.first_name = t2.first_name
    AND t1.last_name = t2.last_name
  LEFT JOIN dvd_rentals.film_actor t3 
    ON t2.actor_id = t3.actor_id
  LEFT JOIN dvd_rentals.film t4 
    ON t3.film_id = t4.film_id
  LEFT JOIN dvd_rentals.inventory t5 
    ON t4.film_id = t5.film_id
  LEFT JOIN dvd_rentals.rental t6 
    ON t5.inventory_id = t6.inventory_id
WHERE t6.inventory_id IS NOT NULL AND 
  t6.rental_date IS NOT NULL
ORDER BY t1.customer_id, t4.title, t6.rental_date DESC;
```
The reason for the `IS NOT NULL` part of the query above is because, without it, I discovered there was stock of movies that favorite actors had acted in that had never been rented out.
| customer_id | first_name | last_name | title             | inventory_id | rental_date              |
|-------------|------------|-----------|-------------------|--------------|--------------------------|
| 1           | SCARLETT   | BENING    | AMISTAD MIDSUMMER | 112          | 2006-02-14T15:16:03.000Z |
| 1           | SCARLETT   | BENING    | AMISTAD MIDSUMMER | 110          | 2005-08-22T17:12:29.000Z |
| 1           | SCARLETT   | BENING    | AMISTAD MIDSUMMER | 109          | 2005-08-21T08:42:26.000Z |
| 1           | SCARLETT   | BENING    | AMISTAD MIDSUMMER | 108          | 2005-08-20T18:24:26.000Z |
| 1           | SCARLETT   | BENING    | AMISTAD MIDSUMMER | 113          | 2005-08-19T18:07:47.000Z |
| 1           | SCARLETT   | BENING    | AMISTAD MIDSUMMER | 111          | 2005-08-19T14:56:05.000Z |
| 1           | SCARLETT   | BENING    | AMISTAD MIDSUMMER | 114          | 2005-08-18T13:57:58.000Z |
| 1           | SCARLETT   | BENING    | AMISTAD MIDSUMMER | 114          | 2005-08-01T05:27:13.000Z |
| 1           | SCARLETT   | BENING    | AMISTAD MIDSUMMER | 110          | 2005-08-01T04:35:34.000Z |
| 1           | SCARLETT   | BENING    | AMISTAD MIDSUMMER | 112          | 2005-07-31T20:54:20.000Z |

I can now manipulate this table to rank the actors each customer likes in order of popularity. This is the base table for the anti-join.
```
-- This table is a list of all movies customer-favorite actors acted in ordered by popularity (rental_count, latest_rental_date); this will be the base table for an anti-join

DROP TABLE IF EXISTS fav_actor_movies_by_popularity;
CREATE TEMP TABLE fav_actor_movies_by_popularity AS
SELECT
  customer_id,
  first_name,
  last_name,
  title,
  COUNT(*) AS total_rented,
  MAX(rental_date) AS latest_rental_date
FROM fav_actor_movies_no_filter
GROUP BY
  customer_id,
  first_name,
  last_name,
  title
ORDER BY customer_id, total_rented DESC, latest_rental_date DESC;
```
| customer_id | first_name | last_name | title               | total_rented | latest_rental_date       |
|-------------|------------|-----------|---------------------|--------------|--------------------------|
| 1           | SCARLETT   | BENING    | INVASION CYCLONE    | 27           | 2005-08-23T17:00:12.000Z |
| 1           | SCARLETT   | BENING    | DURHAM PANKY        | 26           | 2005-08-22T01:12:44.000Z |
| 1           | SCARLETT   | BENING    | SEATTLE EXPECATIONS | 24           | 2006-02-14T15:16:03.000Z |
| 1           | SCARLETT   | BENING    | SNATCH SLIPPER      | 22           | 2005-08-23T15:34:46.000Z |
| 1           | SCARLETT   | BENING    | FLATLINERS KILLER   | 22           | 2005-08-22T16:30:43.000Z |
| 1           | SCARLETT   | BENING    | AMISTAD MIDSUMMER   | 21           | 2006-02-14T15:16:03.000Z |
| 1           | SCARLETT   | BENING    | SHAWSHANK BUBBLE    | 19           | 2006-02-14T15:16:03.000Z |
| 1           | SCARLETT   | BENING    | DUDE BLINDNESS      | 16           | 2005-08-23T07:10:22.000Z |
| 1           | SCARLETT   | BENING    | WORDS HUNTER        | 16           | 2005-08-22T18:59:01.000Z |
| 1           | SCARLETT   | BENING    | MOULIN WAKE         | 16           | 2005-08-22T10:24:32.000Z |

- [X] 9.3 Use the first list again to get a list of movies starring the top-watched actor that each customer has already seen; this is the target table for an anti-join
```
-- This table is a list of movies starring the top-watched actor that each customer has already seen; this is the target table for an anti-join

DROP TABLE IF EXISTS fav_actor_movies_seen;
CREATE TEMP TABLE fav_actor_movies_seen AS 
SELECT 
  t1.customer_id,
  t1.first_name,
  t1.last_name,
  t2.title
FROM cust_top_actors t1 
LEFT JOIN actor_dataset t2 
  ON t1.customer_id = t2.customer_id
  AND t1.first_name = t2.first_name
  AND t1.last_name = t2.last_name
ORDER BY t1.customer_id, t1.last_name, t1.first_name;
```
| customer_id | first_name | last_name | title              |
|-------------|------------|-----------|--------------------|
| 1           | SCARLETT   | BENING    | AMISTAD MIDSUMMER  |
| 1           | SCARLETT   | BENING    | YOUTH KICK         |
| 1           | SCARLETT   | BENING    | LUCK OPUS          |
| 1           | SCARLETT   | BENING    | SNATCH SLIPPER     |
| 2           | GINA       | DEGENERES | MUMMY CREATURES    |
| 2           | GINA       | DEGENERES | TELEGRAPH VOYAGE   |
| 2           | GINA       | DEGENERES | CLUELESS BUCKET    |
| 2           | GINA       | DEGENERES | CHAPLIN LICENSE    |
| 2           | GINA       | DEGENERES | OPEN AFRICAN       |
| 3           | JAYNE      | NOLTE     | STRANGERS GRAFFITI |

- [X] 9.4 Anti-join these tables to get a table with customer_id, actor's first and last name, the movies this actor has acted in that the customer has NOT seen, in order of popularity.
```
-- This is a table with customer_id, actor's first and last name, the movies this actor has acted in that the customer has NOT seen, in order of popularity (rental_count, latest_rental_date).

DROP TABLE IF EXISTS fav_actor_movies_not_seen;
CREATE TEMP TABLE fav_actor_movies_not_seen AS 
SELECT *
FROM fav_actor_movies_by_popularity t1 
WHERE NOT EXISTS (
  SELECT 1
  FROM fav_actor_movies_seen t2
  WHERE
    t1.customer_id = t2.customer_id AND 
    t1.title = t2.title
)
-- AND customer_id = 1
-- AND title IN ('AMISTAD MIDSUMMER', 'YOUTH KICK', 'LUCK OPUS', 'SNATCH SLIPPER', 'INVASION CYCLONE')
ORDER BY customer_id, total_rented DESC, latest_rental_date DESC;
```
| customer_id | first_name | last_name | title               | total_rented | latest_rental_date       |
|-------------|------------|-----------|---------------------|--------------|--------------------------|
| 1           | SCARLETT   | BENING    | INVASION CYCLONE    | 27           | 2005-08-23T17:00:12.000Z |
| 1           | SCARLETT   | BENING    | DURHAM PANKY        | 26           | 2005-08-22T01:12:44.000Z |
| 1           | SCARLETT   | BENING    | SEATTLE EXPECATIONS | 24           | 2006-02-14T15:16:03.000Z |
| 1           | SCARLETT   | BENING    | FLATLINERS KILLER   | 22           | 2005-08-22T16:30:43.000Z |
| 1           | SCARLETT   | BENING    | SHAWSHANK BUBBLE    | 19           | 2006-02-14T15:16:03.000Z |
| 1           | SCARLETT   | BENING    | DUDE BLINDNESS      | 16           | 2005-08-23T07:10:22.000Z |
| 1           | SCARLETT   | BENING    | WORDS HUNTER        | 16           | 2005-08-22T18:59:01.000Z |
| 1           | SCARLETT   | BENING    | MOULIN WAKE         | 16           | 2005-08-22T10:24:32.000Z |
| 1           | SCARLETT   | BENING    | SUBMARINE BED       | 15           | 2005-08-21T10:22:51.000Z |
| 1           | SCARLETT   | BENING    | CREEPERS KANE       | 14           | 2005-08-23T01:15:07.000Z |

I'm not done yet. The business requirements state that the movies in this list cannot repeat previously recommended movies for the top-two categories. To accomplish this, I will need to do another anti-join using the above table as the base and `top_3_recs` as the target table. My table names are getting...unwieldy.
```
DROP TABLE IF EXISTS fav_actor_movies_not_seen_not_rec_prev;
CREATE TEMP TABLE fav_actor_movies_not_seen_not_rec_prev AS
SELECT *
FROM fav_actor_movies_not_seen t1 
WHERE NOT EXISTS (
  SELECT 1
  FROM top_3_recs t2
  WHERE 
    t1.customer_id = t2.customer_id AND 
    t1.title = t2.title
)
ORDER BY customer_id, total_rented DESC, latest_rental_date DESC;
```
The movie recommendations for `customer_id` = 1 did not change. However, a quick `COUNT(*)` for the two tables will show us that quite a number of movies were extracted:
```
SELECT COUNT(*)
FROM fav_actor_movies_not_seen;
```
| count |
|-------|
| 15367 |
```
SELECT COUNT(*)
FROM fav_actor_movies_not_seen_not_rec_prev;
```
| count |
|-------|
| 15262 |

- [X] 9.5 Add rank/row to the last table and just get the top 3 movies per customer.
```
-- This table gives the three recommendations for movies starring each customer's favorite actor.

DROP TABLE IF EXISTS top_3_actor_recs;
CREATE TEMP TABLE top_3_actor_recs AS (
WITH cte_1 AS (
  SELECT *,
    ROW_NUMBER() OVER (
      PARTITION BY
        customer_id
      ORDER BY
        customer_id,
        total_rented DESC,
        latest_rental_date DESC
    ) AS rank_num
  FROM fav_actor_movies_not_seen_not_rec_prev
  ORDER BY customer_id, rank_num
)
SELECT *
FROM cte_1
WHERE rank_num IN (1,2,3)
);
```
| customer_id | first_name | last_name | title                | total_rented | latest_rental_date       | rank_num |
|-------------|------------|-----------|----------------------|--------------|--------------------------|----------|
| 1           | SCARLETT   | BENING    | INVASION CYCLONE     | 27           | 2005-08-23T17:00:12.000Z | 1        |
| 1           | SCARLETT   | BENING    | DURHAM PANKY         | 26           | 2005-08-22T01:12:44.000Z | 2        |
| 1           | SCARLETT   | BENING    | SEATTLE EXPECATIONS  | 24           | 2006-02-14T15:16:03.000Z | 3        |
| 2           | GINA       | DEGENERES | GOODFELLAS SALUTE    | 31           | 2005-08-23T18:08:19.000Z | 1        |
| 2           | GINA       | DEGENERES | WIFE TURN            | 31           | 2005-08-23T14:47:26.000Z | 2        |
| 2           | GINA       | DEGENERES | DOGMA FAMILY         | 30           | 2005-08-23T05:24:29.000Z | 3        |
| 3           | JAYNE      | NOLTE     | SWEETHEARTS SUSPECTS | 29           | 2006-02-14T15:16:03.000Z | 1        |
| 3           | JAYNE      | NOLTE     | INVASION CYCLONE     | 27           | 2005-08-23T17:00:12.000Z | 2        |
| 3           | JAYNE      | NOLTE     | DANCING FEVER        | 27           | 2005-08-22T11:11:51.000Z | 3        |
| 4           | KIRK       | JOVOVICH  | NETWORK PEAK         | 31           | 2005-08-23T09:07:11.000Z | 1        |

- [X] 9.6 Do three CTEs making tables with recommended movie 1, 2, 3 respectively and join together so that each customer_id has its own row with the movies listed out horizontally.

Before completing this, I know I also need a table giving the number of movies each customer has seen that stars their favorite actor.
```
DROP TABLE IF EXISTS num_movies_w_fav_actor_seen;
CREATE TEMP TABLE num_movies_w_fav_actor_seen AS (
WITH cte_1 AS (
  SELECT
    customer_id,
    first_name,
    last_name,
    title
  FROM fav_actor_movies_seen
  GROUP BY
    customer_id,
    first_name,
    last_name,
    title
)
SELECT
  customer_id,
  COUNT(*) AS num_movies_w_fav_actor_seen
FROM cte_1
GROUP BY customer_id
ORDER BY customer_id
);
```
| customer_id | num_movies_w_fav_actor_seen |
|-------------|-----------------------------|
| 1           | 4                           |
| 2           | 5                           |
| 3           | 4                           |
| 4           | 4                           |
| 5           | 5                           |
| 6           | 4                           |
| 7           | 5                           |
| 8           | 4                           |
| 9           | 3                           |
| 10          | 4                           |

Now I can do my CTEs and join these tables.
```
-- This table gives each customer_id its own row with their favorite actor and three movie recs, as well as number of movies with this actor each customer has already seen.

DROP TABLE IF EXISTS final_actor_recs;
CREATE TEMP TABLE final_actor_recs AS (
WITH actor_rec_movie_1 AS (
SELECT
  customer_id,
  first_name,
  last_name,
  title
FROM top_3_actor_recs
WHERE rank_num = 1
),
actor_rec_movie_2 AS (
SELECT  
  customer_id,
  first_name,
  last_name,
  title
FROM top_3_actor_recs
WHERE rank_num = 2
),
actor_rec_movie_3 AS (
SELECT
  customer_id,
  first_name,
  last_name,
  title
FROM top_3_actor_recs
WHERE rank_num = 3
)
SELECT
  t1.customer_id,
  t1.first_name,
  t1.last_name,
  t1.title AS actor_movie_rec_1,
  t2.title AS actor_movie_rec_2,
  t3.title AS actor_movie_rec_3,
  t4.num_movies_w_fav_actor_seen AS number_movies_seen
FROM actor_rec_movie_1 t1 
  LEFT JOIN actor_rec_movie_2 t2 
  ON t1.customer_id = t2.customer_id
  LEFT JOIN actor_rec_movie_3 t3
  ON t1.customer_id = t3.customer_id
  LEFT JOIN num_movies_w_fav_actor_seen t4 
  ON t1.customer_id = t4.customer_id
);
```
| customer_id | first_name | last_name | actor_movie_rec_1      | actor_movie_rec_2    | actor_movie_rec_3     | number_movies_seen |
|-------------|------------|-----------|------------------------|----------------------|-----------------------|--------------------|
| 1           | SCARLETT   | BENING    | INVASION CYCLONE       | DURHAM PANKY         | SEATTLE EXPECATIONS   | 4                  |
| 2           | GINA       | DEGENERES | GOODFELLAS SALUTE      | WIFE TURN            | DOGMA FAMILY          | 5                  |
| 3           | JAYNE      | NOLTE     | SWEETHEARTS SUSPECTS   | INVASION CYCLONE     | DANCING FEVER         | 4                  |
| 4           | KIRK       | JOVOVICH  | NETWORK PEAK           | RUSH GOODFELLAS      | FORRESTER COMANCHEROS | 4                  |
| 5           | SUSAN      | DAVIS     | GOODFELLAS SALUTE      | PULP BEVERLY         | NONE SPIKING          | 5                  |
| 6           | GREGORY    | GOODING   | GREATEST NORTH         | OPERATION OPERATION  | WARDROBE PHANTOM      | 4                  |
| 7           | ANGELA     | HUDSON    | ROBBERS JOON           | VOYAGE LEGALLY       | VELVET TERMINATOR     | 5                  |
| 8           | LAURENCE   | BULLOCK   | FISH OPUS              | STREETCAR INTENTIONS | CROOKED FROGMEN       | 4                  |
| 9           | HENRY      | BERRY     | APACHE DIVINE          | DOGMA FAMILY         | SPY MILE              | 3                  |
| 10          | KARL       | BERRY     | TELEMARK HEARTBREAKERS | ARIZONA BANG         | HIGHBALL POTTER       | 4                  |

- [X] 9.7 Use this table to do the script table (with text).
```
-- Here is the text table for actor_insights

DROP TABLE IF EXISTS actor_insights;
CREATE TEMP TABLE actor_insights AS 
SELECT
  customer_id,
  first_name || ' ' || last_name AS actor_name,
  'You''ve watched ' || number_movies_seen || ' films featuring ' || first_name || ' ' || last_name || '!' || ' Here are some other films ' || first_name || ' stars in that might interest you!' AS actor_insight,
  actor_movie_rec_1,
  actor_movie_rec_2,
  actor_movie_rec_3
FROM final_actor_recs;
```
| customer_id | actor_name       | actor_insight                                                                                                             | actor_movie_rec_1      | actor_movie_rec_2    | actor_movie_rec_3     |
|-------------|------------------|---------------------------------------------------------------------------------------------------------------------------|------------------------|----------------------|-----------------------|
| 1           | SCARLETT BENING  | You've watched 4 films featuring SCARLETT BENING! Here are some other   films SCARLETT stars in that might interest you!  | INVASION CYCLONE       | DURHAM PANKY         | SEATTLE EXPECATIONS   |
| 2           | GINA DEGENERES   | You've watched 5 films featuring GINA DEGENERES! Here are some other   films GINA stars in that might interest you!       | GOODFELLAS SALUTE      | WIFE TURN            | DOGMA FAMILY          |
| 3           | JAYNE NOLTE      | You've watched 4 films featuring JAYNE NOLTE! Here are some other films   JAYNE stars in that might interest you!         | SWEETHEARTS SUSPECTS   | INVASION CYCLONE     | DANCING FEVER         |
| 4           | KIRK JOVOVICH    | You've watched 4 films featuring KIRK JOVOVICH! Here are some other films   KIRK stars in that might interest you!        | NETWORK PEAK           | RUSH GOODFELLAS      | FORRESTER COMANCHEROS |
| 5           | SUSAN DAVIS      | You've watched 5 films featuring SUSAN DAVIS! Here are some other films   SUSAN stars in that might interest you!         | GOODFELLAS SALUTE      | PULP BEVERLY         | NONE SPIKING          |
| 6           | GREGORY GOODING  | You've watched 4 films featuring GREGORY GOODING! Here are some other   films GREGORY stars in that might interest you!   | GREATEST NORTH         | OPERATION OPERATION  | WARDROBE PHANTOM      |
| 7           | ANGELA HUDSON    | You've watched 5 films featuring ANGELA HUDSON! Here are some other films   ANGELA stars in that might interest you!      | ROBBERS JOON           | VOYAGE LEGALLY       | VELVET TERMINATOR     |
| 8           | LAURENCE BULLOCK | You've watched 4 films featuring LAURENCE BULLOCK! Here are some other   films LAURENCE stars in that might interest you! | FISH OPUS              | STREETCAR INTENTIONS | CROOKED FROGMEN       |
| 9           | HENRY BERRY      | You've watched 3 films featuring HENRY BERRY! Here are some other films   HENRY stars in that might interest you!         | APACHE DIVINE          | DOGMA FAMILY         | SPY MILE              |
| 10          | KARL BERRY       | You've watched 4 films featuring KARL BERRY! Here are some other films   KARL stars in that might interest you!           | TELEMARK HEARTBREAKERS | ARIZONA BANG         | HIGHBALL POTTER       |

Now it's time to put this all together!
> 10. Generate the final master insight table

I need to create the text table with the data from the first two insights and then join it with the last table I made. About the query below, I have what seems to be a useless `CASE WHEN` clause in it that gives an output that makes sense if the `average_comparison` is 0. This is a possible scenario and I thought it was the case for one movie in particular. What I didn't realize at the time was that the movie I thought would need this was actually a second-ranked movie. Second-ranked movies don't use the `average_comparison` value in their insight text. However, I learned a lot about `CASE WHEN` while doing this and thought it was worth it even though I didn't need it in the `END` 😜.
```
-- This is the text table from the original top ranked insights. However, this is collapsed. I need to change it so that each customer only has one row. Then I can join with the actor_insights table for the final insights table.

DROP TABLE IF EXISTS top_movie_insights_collapsed;
CREATE TEMP TABLE top_movie_insights_collapsed AS 
SELECT
  customer_id,
  category_ranking,
  category_name,
  CASE WHEN category_ranking = 1 THEN
  
      CASE WHEN average_comparison > 0 THEN
      'You''ve watched ' || rental_count || ' ' || category_name || ' films. That''s ' || average_comparison || ' more than the DVD Rental Co average and puts you in the top ' || percentile || '% of ' || category_name || ' Gurus!'
      ELSE 
      'You''ve watched ' || rental_count || ' ' || category_name || ' films. That is the DVD Rental Co average and puts you in the top ' || percentile || '% of ' || category_name || ' Gurus!'
      END
    
  ELSE 
  'You''ve watched ' || rental_count || ' ' || category_name || ' films, making up ' || category_percentage || '%' || ' of your entire viewing history!' 
  END AS insight,
 
  CASE WHEN category_ranking = 1 THEN
  'Your expertly chosen recommendations: '
  ELSE 
   'Your hand-picked recommendations: '
  END AS movie_recs,
  rec_movie_1,
  rec_movie_2,
  rec_movie_3

FROM insights_inputs;
```
I altered some of the column names so the table would look nicer:
| cust_id | cat_rank | cat_name | insight_in_text_form                                                                                                                      | movie_recs                             | rec_movie_1         | rec_movie_2         | rec_movie_3         |
|-------------|------------------|---------------|-------------------------------------------------------------------------------------------------------------------------------|----------------------------------------|---------------------|---------------------|---------------------|
| 1           | 1                | Classics      | You've watched 6 Classics films. That's 4 more than the DVD Rental Co   average and puts you in the top 1% of Classics Gurus! | Your expertly chosen recommendations:  | TIMBERLAND SKY      | VOYAGE LEGALLY      | GILMORE BOILED      |
| 1           | 2                | Comedy        | You've watched 5 Comedy films, making up 16% of your entire viewing   history!                                                | Your hand-picked recommendations:      | ZORRO ARK           | CAT CONEHEADS       | OPERATION OPERATION |
| 2           | 1                | Sports        | You've watched 5 Sports films. That's 3 more than the DVD Rental Co   average and puts you in the top 7% of Sports Gurus!     | Your expertly chosen recommendations:  | GLEAMING JAWBREAKER | TALENTED HOMICIDE   | SATURDAY LAMBS      |
| 2           | 2                | Classics      | You've watched 4 Classics films, making up 15% of your entire viewing   history!                                              | Your hand-picked recommendations:      | FROST HEAD          | VOYAGE LEGALLY      | GILMORE BOILED      |
| 3           | 1                | Action        | You've watched 4 Action films. That's 2 more than the DVD Rental Co   average and puts you in the top 13% of Action Gurus!    | Your expertly chosen recommendations:  | SUSPECTS QUILLS     | RUGRATS SHAKESPEARE | STORY SIDE          |
| 3           | 2                | Sci-Fi        | You've watched 3 Sci-Fi films, making up 12% of your entire viewing   history!                                                | Your hand-picked recommendations:      | GOODFELLAS SALUTE   | ENGLISH BULWORTH    | GRAFFITI LOVE       |
| 4           | 1                | Horror        | You've watched 3 Horror films. That's 2 more than the DVD Rental Co   average and puts you in the top 14% of Horror Gurus!    | Your expertly chosen recommendations:  | PULP BEVERLY        | FAMILY SWEET        | SWARM GOLD          |
| 4           | 2                | Drama         | You've watched 2 Drama films, making up 9% of your entire viewing   history!                                                  | Your hand-picked recommendations:      | HOBBIT ALIEN        | HARRY IDAHO         | WITCHES PANIC       |
| 5           | 1                | Classics      | You've watched 7 Classics films. That's 5 more than the DVD Rental Co   average and puts you in the top 1% of Classics Gurus! | Your expertly chosen recommendations:  | TIMBERLAND SKY      | FROST HEAD          | GILMORE BOILED      |
| 5           | 2                | Animation     | You've watched 6 Animation films, making up 16% of your entire viewing   history!                                             | Your hand-picked recommendations:      | JUGGLER HARDLY      | DOGMA FAMILY        | STORM HAPPINESS     |

Great table, but now I need to fix the stacked rows problem. Each customer needs all of their data on one row. Then I can join with `actor_insights` and all will be right with the world.
```
-- The following table now has each customer's info on only one row.

DROP TABLE IF EXISTS top_movie_insights_de_collapsed;
CREATE TEMP TABLE top_movie_insights_de_collapsed AS (
WITH cat_1 AS (
  SELECT
    customer_id,
    category_ranking,
    category_name,
    insight,
    movie_recs,
    rec_movie_1,
    rec_movie_2,
    rec_movie_3
  FROM top_movie_insights_collapsed
  WHERE category_ranking = 1
), cat_2 AS (
  SELECT
    customer_id,
    category_ranking,
    category_name,
    insight,
    movie_recs,
    rec_movie_1,
    rec_movie_2,
    rec_movie_3
  FROM top_movie_insights_collapsed
  WHERE category_ranking = 2
)
SELECT
  t1.customer_id,
  t1.category_name AS first_category,
  t1.insight AS first_cat_insight,
  t1.movie_recs AS first_cat_movie_rec_line,
  t1.rec_movie_1 AS first_cat_movie_1,
  t1.rec_movie_2 AS first_cat_movie_2,
  t1.rec_movie_3 AS first_cat_movie_3,
  t2.category_name AS second_category,
  t2.insight AS second_cat_insight,
  t2.movie_recs AS second_cat_movie_rec_line,
  t2.rec_movie_1 AS second_cat_movie_1,
  t2.rec_movie_2 AS second_cat_movie_2,
  t2.rec_movie_3 AS second_cat_movie_3
FROM cat_1 t1 
  LEFT JOIN cat_2 t2 
  ON t1.customer_id = t2.customer_id
);
```
I altered the columns so the table would look nicer:
| cust_id | first_category | first_cat_insight                                                                                                                    | first_cat_movie_rec_line               | ... | second_category | … | second_cat_movie_3  |
|-------------|----------------|--------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------|-----|-----------------|---|---------------------|
| 1           | Classics       | You've watched 6 Classics films. That's 4 more than the DVD Rental Co   average and puts you in the top 1% of Classics Gurus!        | Your expertly chosen recommendations:  | ... | Comedy          | … | OPERATION OPERATION |
| 2           | Sports         | You've watched 5 Sports films. That's 3 more than the DVD Rental Co   average and puts you in the top 7% of Sports Gurus!            | Your expertly chosen recommendations:  | ... | Classics        | … | GILMORE BOILED      |
| 3           | Action         | You've watched 4 Action films. That's 2 more than the DVD Rental Co   average and puts you in the top 13% of Action Gurus!           | Your expertly chosen recommendations:  | ... | Sci-Fi          | … | GRAFFITI LOVE       |
| 4           | Horror         | You've watched 3 Horror films. That's 2 more than the DVD Rental Co   average and puts you in the top 14% of Horror Gurus!           | Your expertly chosen recommendations:  | ... | Drama           | … | WITCHES PANIC       |
| 5           | Classics       | You've watched 7 Classics films. That's 5 more than the DVD Rental Co   average and puts you in the top 1% of Classics Gurus!        | Your expertly chosen recommendations:  | ... | Animation       | … | STORM HAPPINESS     |
| 6           | Drama          | You've watched 4 Drama films. That's 2 more than the DVD Rental Co   average and puts you in the top 8% of Drama Gurus!              | Your expertly chosen recommendations:  | ... | Sci-Fi          | … | MARRIED GO          |
| 7           | Sports         | You've watched 5 Sports films. That's 3 more than the DVD Rental Co   average and puts you in the top 6% of Sports Gurus!            | Your expertly chosen recommendations:  | ... | Animation       | … | STORM HAPPINESS     |
| 8           | Classics       | You've watched 4 Classics films. That's 2 more than the DVD Rental Co   average and puts you in the top 9% of Classics Gurus!        | Your expertly chosen recommendations:  | ... | Drama           | … | WITCHES PANIC       |
| 9           | Foreign        | You've watched 4 Foreign films. That's 2 more than the DVD Rental Co   average and puts you in the top 10% of Foreign Gurus!         | Your expertly chosen recommendations:  | ... | Travel          | … | COMA HEAD           |
| 10          | Documentary    | You've watched 4 Documentary films. That's 2 more than the DVD Rental Co   average and puts you in the top 11% of Documentary Gurus! | Your expertly chosen recommendations:  | ... | Games           | … | VIDEOTAPE ARSENIC   |

Finally, now to join them.
```
-- This will combine the two insights tables

SELECT t1.*,
  t2.actor_name,
  t2.actor_insight,
  t2.actor_movie_rec_1,
  t2.actor_movie_rec_2,
  t2.actor_movie_rec_3
FROM top_movie_insights_de_collapsed t1 
  LEFT JOIN actor_insights t2 
  ON t1.customer_id = t2.customer_id;
```
I altered the columns so the table would look nicer. An image of the full table is below, too.
| cust_id | first_cat_insight_text                                                                                                                 | … | second_cat_insight_text                                                           | ... | actor_name       | actor_insight_text                                                                                                        | ... | actor_rec_3           |
|--------|----------------------------------------------------------------------------------------------------------------------------------------|---|-----------------------------------------------------------------------------------|-----|------------------|---------------------------------------------------------------------------------------------------------------------------|-----|-----------------------|
| 1      | You've watched 6 Classics films. That's 4 more than   the DVD Rental Co average and puts you in the top 1% of Classics Gurus!          | … | You've watched 5 Comedy films, making up 16% of   your entire viewing history!    | …   | SCARLETT BENING  | You've watched 4 films featuring SCARLETT BENING!   Here are some other films SCARLETT stars in that might interest you!  | …   | SEATTLE EXPECATIONS   |
| 2      | You've watched 5 Sports films. That's 3 more than   the DVD Rental Co average and puts you in the top 7% of Sports Gurus!              | … | You've watched 4 Classics films, making up 15% of   your entire viewing history!  | …   | GINA DEGENERES   | You've watched 5 films featuring GINA DEGENERES!   Here are some other films GINA stars in that might interest you!       | …   | DOGMA FAMILY          |
| 3      | You've watched 4 Action films. That's 2 more than   the DVD Rental Co average and puts you in the top 13% of Action Gurus!             | … | You've watched 3 Sci-Fi films, making up 12% of   your entire viewing history!    | …   | JAYNE NOLTE      | You've watched 4 films featuring JAYNE NOLTE! Here   are some other films JAYNE stars in that might interest you!         | …   | DANCING FEVER         |
| 4      | You've watched 3 Horror films. That's 2 more than   the DVD Rental Co average and puts you in the top 14% of Horror Gurus!             | … | You've watched 2 Drama films, making up 9% of your   entire viewing history!      | …   | KIRK JOVOVICH    | You've watched 4 films featuring KIRK JOVOVICH!   Here are some other films KIRK stars in that might interest you!        | …   | FORRESTER COMANCHEROS |
| 5      | You've watched 7 Classics films. That's 5 more than   the DVD Rental Co average and puts you in the top 1% of Classics Gurus!          | … | You've watched 6 Animation films, making up 16% of   your entire viewing history! | …   | SUSAN DAVIS      | You've watched 5 films featuring SUSAN DAVIS! Here   are some other films SUSAN stars in that might interest you!         | …   | NONE SPIKING          |
| 6      | You've watched 4 Drama films. That's 2 more than   the DVD Rental Co average and puts you in the top 8% of Drama Gurus!                | … | You've watched 3 Sci-Fi films, making up 11% of   your entire viewing history!    | …   | GREGORY GOODING  | You've watched 4 films featuring GREGORY GOODING!   Here are some other films GREGORY stars in that might interest you!   | …   | WARDROBE PHANTOM      |
| 7      | You've watched 5 Sports films. That's 3 more than   the DVD Rental Co average and puts you in the top 6% of Sports Gurus!              | … | You've watched 5 Animation films, making up 15% of   your entire viewing history! | …   | ANGELA HUDSON    | You've watched 5 films featuring ANGELA HUDSON!   Here are some other films ANGELA stars in that might interest you!      | …   | VELVET TERMINATOR     |
| 8      | You've watched 4 Classics films. That's 2 more than   the DVD Rental Co average and puts you in the top 9% of Classics Gurus!          | … | You've watched 4 Drama films, making up 17% of your   entire viewing history!     | …   | LAURENCE BULLOCK | You've watched 4 films featuring LAURENCE BULLOCK!   Here are some other films LAURENCE stars in that might interest you! | …   | CROOKED FROGMEN       |
| 9      | You've watched 4 Foreign films. That's 2 more than   the DVD Rental Co average and puts you in the top 10% of Foreign Gurus!           | … | You've watched 4 Travel films, making up 17% of   your entire viewing history!    | …   | HENRY BERRY      | You've watched 3 films featuring HENRY BERRY! Here   are some other films HENRY stars in that might interest you!         | …   | SPY MILE              |
| 10     | You've watched 4 Documentary films. That's   2 more than the DVD Rental Co average and puts you in the top 11% of   Documentary Gurus! | … | You've watched 4 Games films, making up 16%   of your entire viewing history!     | …   | KARL BERRY       | You've watched 4 films featuring KARL   BERRY! Here are some other films KARL stars in that might interest you!           | …   | HIGHBALL POTTER       |

![image](https://user-images.githubusercontent.com/99853599/165386314-b6e59bbb-5091-4bd3-bcc6-406b81b7ac44.png)

### Last but not least
The business requirements also state that any customers in the table who don't generate any recommended movies must be flagged. I will drop the last query into a table and then create this flag. From the output, all of the customers seem to have recommendations.
```
DROP TABLE IF EXISTS final_table;
CREATE TEMP TABLE final_table AS 
SELECT t1.*,
  t2.actor_name,
  t2.actor_insight,
  t2.actor_movie_rec_1,
  t2.actor_movie_rec_2,
  t2.actor_movie_rec_3
FROM top_movie_insights_de_collapsed t1 
  LEFT JOIN actor_insights t2 
  ON t1.customer_id = t2.customer_id;
  
WITH cte_1 AS (
  SELECT *,
    CASE WHEN first_cat_movie_1 IS NULL AND
      first_cat_movie_2 IS NULL AND 
      first_cat_movie_3 IS NULL
      THEN 1
      ELSE 0 
    END AS no_top_cat_recs,
    CASE WHEN second_cat_movie_1 IS NULL AND
      second_cat_movie_2 IS NULL AND 
      second_cat_movie_3 IS NULL
      THEN 1
      ELSE 0 
    END AS no_second_cat_recs,
    CASE WHEN actor_movie_rec_1 IS NULL AND
      actor_movie_rec_2 IS NULL AND 
      actor_movie_rec_3 IS NULL
      THEN 1
      ELSE 0 
    END AS no_actor_recs
  FROM final_table
)
SELECT *
FROM cte_1
WHERE 
  no_top_cat_recs = 1 OR 
  no_second_cat_recs = 1 OR 
  no_actor_recs = 1;
```
This query does not return any rows, indicating that every customer has a recommended movie in all the relevant columns. Here's what the table looks like with the columns altered so it fits here.
| cust_id | first_category | … | second_category | … | actor_name       | … | no_top_cat_recs | no_second_cat_recs | no_actor_recs |
|---------|----------------|---|-----------------|---|------------------|---|-----------------|--------------------|---------------|
| 1       | Classics       | … | Comedy          | … | SCARLETT BENING  | … | 0               | 0                  | 0             |
| 2       | Sports         | … | Classics        | … | GINA DEGENERES   | … | 0               | 0                  | 0             |
| 3       | Action         | … | Sci-Fi          | … | JAYNE NOLTE      | … | 0               | 0                  | 0             |
| 4       | Horror         | … | Drama           | … | KIRK JOVOVICH    | … | 0               | 0                  | 0             |
| 5       | Classics       | … | Animation       | … | SUSAN DAVIS      | … | 0               | 0                  | 0             |
| 6       | Drama          | … | Sci-Fi          | … | GREGORY GOODING  | … | 0               | 0                  | 0             |
| 7       | Sports         | … | Animation       | … | ANGELA HUDSON    | … | 0               | 0                  | 0             |
| 8       | Classics       | … | Drama           | … | LAURENCE BULLOCK | … | 0               | 0                  | 0             |
| 9       | Foreign        | … | Travel          | … | HENRY BERRY      | … | 0               | 0                  | 0             |
| 10      | Documentary    | … | Games           | … | KARL BERRY       | … | 0               | 0                  | 0             |

![image](https://user-images.githubusercontent.com/99853599/165314990-782608a5-4668-487c-a55f-94e90e9cf01a.png)
