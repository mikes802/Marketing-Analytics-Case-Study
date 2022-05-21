# Script Check
After completing my SQL script, I went back to Danny's tutorial to see how it compared to the final script. How bad could it be?
<p align="center">
  <img src="https://user-images.githubusercontent.com/99853599/169161084-a56f1d28-7105-40b0-8bb9-38a66f6dc729.gif">
</p>
Below are my takeaways from this little moment of truth.

## Wrong Answers
The first indication that something was awry was when I decided to take the section quiz and I was not consistently getting correct answers. Like, starting with question #1.
> Q1. Which film title was the most recommended for all customers?
```
WITH cte_1 AS (
  SELECT
    title
  FROM top_3_recs
  UNION ALL
  SELECT
    title
  FROM top_3_actor_recs
)
SELECT
  title,
  COUNT(*) AS movie_count
FROM cte_1
GROUP BY title
ORDER BY movie_count DESC;
```
| movie             | count_movie | latest_rental_date       |
|-------------------|-------------|--------------------------|
| JUGGLER HARDLY    | 124         | 2006-02-14T15:16:03.000Z |
| DOGMA FAMILY      | 124         | 2005-08-23T05:24:29.000Z |
| GOODFELLAS SALUTE | 120         | 2005-08-23T18:08:19.000Z |
| STORM HAPPINESS   | 116         | 2005-08-22T20:44:35.000Z |
| SATURDAY LAMBS    | 114         | 2005-08-23T19:42:04.000Z |

<img align="left"  src= "https://user-images.githubusercontent.com/99853599/169163820-cb5e194c-a3e1-4403-a024-eb879eb1d8de.png" width="250" height="250"/>
<br />
<br />
Um...

<br />
<br />
What is Juggler Dogma, Alex?
<br clear="left"/>
<br />

The right answer is *Dogma Family*, but that's not clear at all from my query. So something's up.

Not one to let this minor setback get me down, I forged ahead with questions 2 - 5. I had correct answers for #2, 4, & 5. My answer for question #3 was incorrect.
> Q3. Out of all the possible films - what percentage coverage do we have in our recommendations? (total unique films recommended divided by total available films)

I used my own method for this and my answer was 24%. It should be 25%. I used Danny's query, tweaked it so it would run with my script, and...
```
WITH cte_1 AS (
  SELECT
    title
  FROM top_3_recs
  UNION ALL
  SELECT
    title
  FROM top_3_actor_recs
),
distinct_recs AS (
  SELECT
    COUNT(DISTINCT title) AS rec_count
  FROM cte_1
),
all_movies AS (
  SELECT
    COUNT(DISTINCT title) AS all_avail
  FROM dvd_rentals.film
)
  SELECT
    t1.all_avail AS all,
    t2.rec_count AS recommended,
    ROUND(100 * t2.rec_count::NUMERIC/t1.all_avail::NUMERIC) AS coverage_percentage
  FROM all_movies t1 
  CROSS JOIN distinct_recs t2;
  ```
  | all  | recommended | coverage_percentage |
|------|-------------|---------------------|
| 1000 | 242         | 24                  |

Still get 24%. So I know it's the underlying data that's the issue. 

Since I'm getting some answers correct, hopefully it's a minor problem. My task now is to go through my code and Danny's, and try to figure out what the major differences are. Is the logic basically the same, just minor errors in the code on my part? Or is my logic completely off-base and all the code in the world can't help me? 

Let's start this.

## ROW_NUMBER, DENSE_RANK, RANK, FRANK?

<img align="left"  src= "https://user-images.githubusercontent.com/99853599/169186784-9b64f390-9688-4834-9e80-0ce56fb4d62f.jpg" width="350" height="250"/>
<br />
<br />
<br />
<br />
Yes?
<br clear="left"/>
<br />

Let's start with an easy one... Hahaha! Just kidding. All of these are freaking mind melters.

Near the beginning of his script, we were looking for the top-ranked and second-ranked movie categories for each customer. To do this, there are some window function options we can use to rank the categories, i.e. `ROW_NUMBER`, `DENSE_RANK`, and `RANK`. Danny used `DENSE_RANK`. I didn't.

There are explanations all over the internet on how these functions differ. Basically, `ROW_NUMBER` will give you, well, row numbers. Numbers that increase incrementally irregardless of the value in that row. `DENSE_RANK` does this, too, but it will give the same "row number" for tied values. `RANK` says f* that, I'll give you the same number for ties, but numbers aren't free you know, so I'll skip a bunch after the tied values. `RANK` is confusing.

I started out by using `RANK`.

This, surprisingly, led to an error that I didn't find until I finished nearly all of my script. There are 599 customers. However, I noticed I had 600 top-ranked movie results and 598 second-ranked movie results. I did some digging and found that this was because one customer had a tie for top-ranked movie (`rank_number` = 1). I went back to one of my first tables in the script, `top_2_ranking`, and added `category_name` as a parameter to the ORDER BY clause in the window function (see below). This eliminated the tie.
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
-- Including category_name to account for any ties in latest_rental_date
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

I used `ROW_NUMBER` from then on, thinking that there should be no difference between this and `RANK`, or even `DENSE_RANK` for that matter. Looking back, I now think that using `RANK` helped me notice a problem that I would have overlooked with `ROW_NUMBER`. Even though the end result would perhaps be the same, that's not necessarily the case. If the business task was to list top-ranked movies first by rental count, then by latest rental date, then alphabetically by category name, and I left out category name as a parameter, then `ROW_NUMBER` would not have helped me troubleshoot this issue, since it would have given me a rank number of both 1 and 2 no matter what.

In my original table, where I left out `category_name` in the window function's `ORDER BY` clause, one customer had two movies with `rank_number` = 1. If I had used `DENSE_RANK` instead of `RANK`, then the table should have had at least three different movies returned for that customer. `DENSE_RANK` would have given me the same movies with `rank_number` = 1, as well as any movies that are `rank_number` = 2. However, since I used `RANK`, it skipped "row number 2" and went straight on to 3, which do not get returned with my WHERE filter.

Should I try it out? I should, shouldn't I? Ok, I will.

First, to find out which customer it is, I'll leave out category_name and use `RANK`, which is what I did at first. I'm going to add `latest_rental_date` into the query after the CTE, as this will make it obvious why there are two categories ranked at #1.
```
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
-- Including category_name to account for any ties in latest_rental_date
      ORDER BY rental_count DESC, latest_rental_date DESC
    ) AS rank_number
  FROM category_rental_counts
)
SELECT
  customer_id,
  category_name,
  rental_count,
-- Add in latest_rental_date to show how this affects the rank_number
  latest_rental_date,
  rank_number
FROM cte_1
WHERE rank_number IN (1,2)
ORDER BY customer_id;

WITH cte AS (
SELECT DISTINCT(customer_id), COUNT(customer_id) AS count
FROM top_2_ranking
WHERE rank_number = 1
GROUP BY customer_id
)
SELECT * 
FROM cte 
WHERE count > 1;
```
| customer_id | count |
|-------------|-------|
| 284         | 2     |

Now I'll look for this customer in the `top_2_ranking` table:
```
SELECT *
FROM top_2_ranking
WHERE customer_id = 284;
```
| customer_id | category_name | rental_count | latest_rental_date       | rank_number |
|-------------|---------------|--------------|--------------------------|-------------|
| 284         | Foreign       | 4            | 2006-02-14T15:16:03.000Z | 1           |
| 284         | Action        | 4            | 2006-02-14T15:16:03.000Z | 1           |

Here I can clearly see that customer 284 has a tie for top-ranked category because not only does this customer have the same number of rentals from each category, but apparently 284 also rented one movie from each of these categories at the same time. Without anything else to distinguish between the categories in the window function `ORDER BY` clause, they are both ranked #1. Well durn.

Now why don't I get a #2 rank in the result? Because it doesn't exist. Since there are already two results ranked #1, `RANK` will skip the number 2 and go straight to 3 for the next value. Let's include 3 in my WHERE clause and see if that proves to be true.
```
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
-- Including category_name to account for any ties in latest_rental_date
      ORDER BY rental_count DESC, latest_rental_date DESC
    ) AS rank_number
  FROM category_rental_counts
)
SELECT
  customer_id,
  category_name,
  rental_count,
-- Add in latest_rental_date to show how this affects the rank_number
  latest_rental_date,
  rank_number
FROM cte_1
-- Change this WHERE clause to include rank_number = 3
WHERE rank_number <= 3
ORDER BY customer_id;

SELECT *
FROM top_2_ranking
WHERE customer_id = 284;
```
| customer_id | category_name | rental_count | latest_rental_date       | rank_number |
|-------------|---------------|--------------|--------------------------|-------------|
| 284         | Foreign       | 4            | 2006-02-14T15:16:03.000Z | 1           |
| 284         | Action        | 4            | 2006-02-14T15:16:03.000Z | 1           |
| 284         | Horror        | 4            | 2005-08-21T22:54:02.000Z | 3           |

As expected, #2 is skipped, but we get #3.

Now if I had not added `category_name` to the window function `ORDER BY` clause and had used `DENSE_RANK` instead? I would have gotten the above three results, but they would be ranked #1, #1, and #2, respectively. Let's check that out:
```
DROP TABLE IF EXISTS top_2_ranking;
CREATE TEMP TABLE top_2_ranking AS 
WITH cte_1 AS (  
  SELECT 
    customer_id,
    category_name,
    rental_count,
    latest_rental_date,
-- Change to DENSE_RANK to see difference in results
    DENSE_RANK() OVER (
      PARTITION BY customer_id
-- Including category_name to account for any ties in latest_rental_date
      ORDER BY rental_count DESC, latest_rental_date DESC
    ) AS rank_number
  FROM category_rental_counts
)
SELECT
  customer_id,
  category_name,
  rental_count,
-- Add in latest_rental_date to show how this affects the rank_number
  latest_rental_date,
  rank_number
FROM cte_1
WHERE rank_number in (1,2)
ORDER BY customer_id;

SELECT *
FROM top_2_ranking
WHERE customer_id = 284;
```
| customer_id | category_name | rental_count | latest_rental_date       | rank_number |
|-------------|---------------|--------------|--------------------------|-------------|
| 284         | Foreign       | 4            | 2006-02-14T15:16:03.000Z | 1           |
| 284         | Action        | 4            | 2006-02-14T15:16:03.000Z | 1           |
| 284         | Horror        | 4            | 2005-08-21T22:54:02.000Z | 2           |

Same categories, but now Horror is ranked #2. We still have two #1 categories.

All this shows that I need all of the proper parameters in that `ORDER BY` clause to get two results. But what about `ROW_NUMBER`? That will get me two results, right? `ROW_NUMBER` doesn't care about all that parameter stuff. You want two results ranked #1 and #2? Ok, you got it:
```
DROP TABLE IF EXISTS top_2_ranking;
CREATE TEMP TABLE top_2_ranking AS 
WITH cte_1 AS (  
  SELECT 
    customer_id,
    category_name,
    rental_count,
    latest_rental_date,
-- Change to ROW_NUMBER to see difference in results
    ROW_NUMBER() OVER (
      PARTITION BY customer_id
-- Including category_name to account for any ties in latest_rental_date
      ORDER BY rental_count DESC, latest_rental_date DESC
    ) AS rank_number
  FROM category_rental_counts
)
SELECT
  customer_id,
  category_name,
  rental_count,
-- Add in latest_rental_date to show how this affects the rank_number
  latest_rental_date,
  rank_number
FROM cte_1
WHERE rank_number in (1,2)
ORDER BY customer_id;

SELECT *
FROM top_2_ranking
WHERE customer_id = 284;
```
| customer_id | category_name | rental_count | latest_rental_date       | rank_number |
|-------------|---------------|--------------|--------------------------|-------------|
| 284         | Foreign       | 4            | 2006-02-14T15:16:03.000Z | 1           |
| 284         | Action        | 4            | 2006-02-14T15:16:03.000Z | 2           |

![image](https://user-images.githubusercontent.com/99853599/169631530-9ec3e68b-7f42-4282-a798-5527d9fceda7.png)

