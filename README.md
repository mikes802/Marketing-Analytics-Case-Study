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
For each table below I will show the first ten rows of that table. Here are the first ten rows of the complete_joint_dataset table:
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

The above was completed with a lot of hand-holding by Danny's tutorials. I could do my work first and then check back at his solutions to see if I was on the right track. Not everything matched up perfectly, but it worked (as far as I know). After this, however, I had to take off the training wheels. I stopped referring to the tutorials and just attempted it on my own.
## SQL Script (No Training Wheels)
The last table generated has a lot of the information I need, but it's missing a very important part: movie recommendations. I struggled with this one for quite some time. After a lot of thinking, I landed on using an anti-join to generate a table of movies that customers had not seen, ranking those, and then taking the top three for the top-three recommendations. This was going to take a few steps.
### Generate the base table
I know I'll need a base table of all the movies possible by category and ranked by popularity (`times_rented` and `latest_rental_date`). By anti-joining with a list of all the movies the customers have seen in their top-ranked categories, I will be eliminated those titles from the base table and leaving a list of movies, ranked by popularity, that have not been seen.

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
The commented-out lines in some of these queries is my checking the tables to make sure I'm getting the information I want. For example, I shouldn't see the first five movies in the commented line below come out in the table for `customer_id` = 1, but I should see the last movie. 
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

> 9. Generate the actor insight section
I developed the following checklist to help me strategize a plan of attack for this part:
- [ ] Generate table showing the top-watched actor per customer_id.
- [ ] Use that table and join it to the other relevant tables to get a list of all movies these actors acted in, ordered by popularity (rental_count, latest_rental_date); this will be the base table for an anti-join.
- [ ] Use the first list again to get a list of movies starring the top-watched actor that each customer has already seen; this is the target table for an anti-join
- [ ] Anti-join these tables to get a table with customer_id, actor's first and last name, the movies this actor has acted in that the customer has NOT seen, in order of popularity.
- [ ] Add rank/row to the last table and just get the top 3 movies per customer.
- [ ] Do three CTEs making tables with recommended movie 1, 2, 3 respectively and join together so that each customer_id has its own row with the movies listed out horizontally.
- [ ] Use this table to do the script table (with text).
