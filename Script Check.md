# Script Check
It's time for reflection...

<img align="left"  src= "https://user-images.githubusercontent.com/99853599/182252679-378f1916-c866-4d24-a7b5-ff790b45b8ec.png" width="350" height="250"/>
<br />
<br />
<br />
<br />
Not me
<br clear="left"/>
<br />
It's time for introspection.
<br />
<br />

After completing my SQL code for the Marketing Analytics Case Study from [Danny Ma's Serious SQL course](https://www.datawithdanny.com/ "Data With Danny"), I went back to Danny's tutorial to see how it compared to his official, final script. How bad could it be?
<br />
<br />
<p align="center">
  <img src="https://user-images.githubusercontent.com/99853599/169161084-a56f1d28-7105-40b0-8bb9-38a66f6dc729.gif">
</p>
Below are my takeaways from this little moment of truth.

## Table of Contents
- [Background: Wrong Answers](#wrong-answers)
- [ROW_NUMBER, DENSE_RANK, RANK, FRANK](#row_number-dense_rank-rank-frank)
- [PERCENT_RANK and CUME_DIST](#percent_rank-and-cume_dist)
- [The JOINs](#the-joins)
- [Dealing with Duplicates](#whats-in-a-name-dealing-with-duplicates)
- [Final Takeaways](#final-takeaways)
- [New Code](#new-code) 
- [Leftover Question: Group Aggregate vs Window Function](#leftover-question)

## [Wrong Answers](#table-of-contents)
Again, this is a follow-up to solving the Marketing Analytics Case Study from [Danny Ma's Serious SQL course](https://www.datawithdanny.com/ "Data With Danny"). My original work can be found here: [Marketing Analytics Case Study](/README.md#marketing-analytics-case-study).

The first indication that something was awry was when I decided to take the section quiz and I was not consistently getting correct answers. Like, starting with question #1.
> Q1. Which film title was the most recommended for all customers?
<details><summary> 🔴 SQL code</summary>
  
<pre>
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
</pre>
</details>

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
<details>
<summary> 🔴 SQL code</summary>
  
<pre>
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
</pre>
</details>

| all  | recommended | coverage_percentage |
|------|-------------|---------------------|
| 1000 | 242         | 24                  |

Still get 24%. So I know it's the underlying data that's the issue. 

Since I'm getting some answers correct, hopefully it's a minor problem. My task now is to go through my code and Danny's, and try to figure out what the major differences are. Is the logic basically the same, just minor errors in the code on my part? Or is my logic completely off-base and all the code in the world can't help me? 

Let's start this.

## [ROW_NUMBER, DENSE_RANK, RANK, FRANK?](#table-of-contents)

<img align="left"  src= "https://user-images.githubusercontent.com/99853599/169186784-9b64f390-9688-4834-9e80-0ce56fb4d62f.jpg" width="350" height="250"/>
<br />
<br />
<br />
<br />
Yes?
<br clear="left"/>
<br />

---
### *Summary*
I tested out the differences in query returns when using `ROW_NUMBER`, `RANK`, and `DENSE_RANK`. I also got my first lesson (of many) in how important it is to be careful with the ORDER BY clause in a window function, and how to align my queries to match the business requirements.
***

Near the beginning of the script, we were looking for the top-ranked and second-ranked movie categories for each customer. To do this, there are some window function options we can use to rank the categories, i.e. `ROW_NUMBER`, `DENSE_RANK`, and `RANK`. Danny used `DENSE_RANK`. I didn't.

There are explanations all over the internet on how these functions differ. Basically, `ROW_NUMBER` will give you, well, row numbers. Numbers that increase incrementally irregardless of the value in that row. `DENSE_RANK` does this, too, but it will give the same "row number" for tied values. `RANK` says, I'll give you the same number for ties, but numbers aren't free you know, so I'll skip a bunch after the tied values. `RANK` is confusing.

I used `RANK`.

Later, when I was nearly finished writing my code, I discovered an error. There are 599 customers. However, I noticed I had 600 top-ranked movie results and 598 second-ranked movie results. I did some digging and found that this was because one customer had a tie for top-ranked movie (`rank_number` = 1). I went back to one of my first tables in the script, `top_2_ranking`, and added `category_name` as a parameter to the ORDER BY clause in the window function (see below). This eliminated the tie.
<details>
<summary> 🔴 SQL code</summary>
  
<pre>
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
</pre>
</details>

Later on in the code, I used `ROW_NUMBER` for ranking, thinking that the results should be the same as `RANK` or even `DENSE_RANK` for that matter. Looking back, I now think that using `RANK` helped me notice a problem that I would have overlooked with `ROW_NUMBER`. If the business task was to just list top-ranked movies first by rental count and then by latest rental date, using `ROW_NUMBER` would not have alerted me to any problem, since it would have given me a rank number of both 1 and 2 no matter what, irregardless of ties. However, by using `RANK`, leaving out `category_name` in the window function's `ORDER BY` clause resulted in one customer linked to two categories with `rank_number` = 1. This was a red flag that I caught later on, and I could trace the mistake back to where the ranking took place. Similarly, if I had used `DENSE_RANK`, then the query should have returned at least three different categories for that customer, also increasing the odds that I would find this error. `DENSE_RANK` would have given me the same categories with `rank_number` = 1 as in the `RANK` result, as well as any categories that are `rank_number` = 2. `RANK` didn't do this because it skipped "row number 2" and went straight on to 3, which do not get returned with my `WHERE` filter.

Should I try it out? I should, shouldn't I? Ok, I will.

First, to find out which customer it is, I'll leave out category_name and use `RANK`, which is what I did at first. I'm going to add `latest_rental_date` into the query after the CTE, as this will make it obvious why there are two categories ranked at #1.
<details>
<summary> 🔴 SQL code</summary>
  
<pre>
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
</pre>
</details>

| customer_id | count |
|-------------|-------|
| 284         | 2     |

Now I'll look for this customer in the `top_2_ranking` table:
```sql
SELECT *
FROM top_2_ranking
WHERE customer_id = 284;
```
| customer_id | category_name | rental_count | latest_rental_date       | rank_number |
|-------------|---------------|--------------|--------------------------|-------------|
| 284         | Foreign       | 4            | 2006-02-14T15:16:03.000Z | 1           |
| 284         | Action        | 4            | 2006-02-14T15:16:03.000Z | 1           |

Here I can clearly see that customer 284 has a tie for top-ranked category because not only does this customer have the same number of rentals from each category, but apparently 284 also rented one movie from each of these categories at the same time. Without anything else to distinguish between the categories in the window function `ORDER BY` clause, they are both ranked #1. Well durn.

Now why don't I get a #2 rank in the result? Because it doesn't exist. Since there are already two results ranked #1, `RANK` will skip the number 2 and go straight to 3 for the next value, but my `WHERE` clause doesn't ask for that. Let's include `rank_number` = 3 in my `WHERE` clause and see if that proves to be true.
<details>
<summary> 🔴 SQL code</summary>

<pre>
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
</pre>
</details>

| customer_id | category_name | rental_count | latest_rental_date       | rank_number |
|-------------|---------------|--------------|--------------------------|-------------|
| 284         | Foreign       | 4            | 2006-02-14T15:16:03.000Z | 1           |
| 284         | Action        | 4            | 2006-02-14T15:16:03.000Z | 1           |
| 284         | Horror        | 4            | 2005-08-21T22:54:02.000Z | 3           |

As expected, #2 is skipped, but we get #3.

Now let's look at `DENSE_RANK`. If I had not added `category_name` to the window function `ORDER BY` clause, I would have gotten the above three results, but they should be ranked #1, #1, and #2, respectively. Let's check that out:
<details>
<summary> 🔴 SQL code</summary>
  
<pre>
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
</pre>
</details>

| customer_id | category_name | rental_count | latest_rental_date       | rank_number |
|-------------|---------------|--------------|--------------------------|-------------|
| 284         | Foreign       | 4            | 2006-02-14T15:16:03.000Z | 1           |
| 284         | Action        | 4            | 2006-02-14T15:16:03.000Z | 1           |
| 284         | Horror        | 4            | 2005-08-21T22:54:02.000Z | 2           |

Same categories, but now Horror is ranked #2. We still have two #1 categories.

All this shows that I need all of the proper parameters in that `ORDER BY` clause to get the correct results. 

But what about `ROW_NUMBER`? That will get me two ranks, right? `ROW_NUMBER` doesn't care about all that parameter stuff. You want two results ranked #1 and #2? Ok, you got it:
<details>
<summary> 🔴 SQL code</summary>

<pre>
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
</pre>
</details>

| customer_id | category_name | rental_count | latest_rental_date       | rank_number |
|-------------|---------------|--------------|--------------------------|-------------|
| 284         | Foreign       | 4            | 2006-02-14T15:16:03.000Z | 1           |
| 284         | Action        | 4            | 2006-02-14T15:16:03.000Z | 2           |

Is that what you wanted?

<img src= "https://user-images.githubusercontent.com/99853599/169631606-d1bd5de1-fd56-44e1-a3bb-a07836692e86.gif" width="400" height="225"/>

However, if the business task required that ties are handled by alphabatizing, then I would have messed this one up and not even realized it. Instead of getting recommendations for action movies as their top category, customer 284 would be getting recommendations for foreign movies. Furthermore, this definitely affects my calculations downstream for questions like question #3 in the quiz, regarding coverage percentage. Bad business. Let's fix it by putting `category_name` back in and moving on. When I do this, I will get the same results for customer 284 regardless of my ranking window function, `ROW_NUMBER`, `DENSE_RANK`, or `RANK`.
<details>
<summary> 🔴 SQL code</summary>

<pre>
DROP TABLE IF EXISTS top_2_ranking;
CREATE TEMP TABLE top_2_ranking AS 
WITH cte_1 AS (  
  SELECT 
    customer_id,
    category_name,
    rental_count,
    latest_rental_date,
    DENSE_RANK() OVER (
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
-- Add in latest_rental_date to show how this affects the rank_number
  latest_rental_date,
  rank_number
FROM cte_1
WHERE rank_number in (1,2)
ORDER BY customer_id;

SELECT *
FROM top_2_ranking
WHERE customer_id = 284;
</pre>
</details>

| customer_id | category_name | rental_count | latest_rental_date       | rank_number |
|-------------|---------------|--------------|--------------------------|-------------|
| 284         | Action        | 4            | 2006-02-14T15:16:03.000Z | 1           |
| 284         | Foreign       | 4            | 2006-02-14T15:16:03.000Z | 2           |

## [PERCENT_RANK and CUME_DIST](#table-of-contents)
---
### *Summary*
Through a lot of self-study, trial and error, and help from Danny, I was able to discern when it is more appropriate to use the `CUME_DIST` versus the `PERCENT_RANK` function. I was also forced to think critically about how an `ORDER BY` clause works in a window function. Finally, I learned when `CASE WHEN` can come in handy.
***

Have you ever been studying something and you thought you were studying this one thing but then you slowly realize you're now studying something completely different to just understand the first thing you were studying...?

![400](https://user-images.githubusercontent.com/99853599/171045188-63feb116-33fc-49cd-a209-b0a9ed0fc732.gif)

Danny asks us to find the percentile rank of each customer's top-rated category to show that they are in the top X% of customers in that particular category. This sent me down a rabbit hole of trying to figure out the difference between "percentile rank" and "cumulative distribution", and their SQL function equivalents: `PERCENT_RANK` and `CUME_DIST`.

I spent quite some time on this and thought I had it figured out. The issue with percentile rank, I figured, is that I would get some results of 0%. This is because percentile rank gives you the percentage of values prior to the value in the current row. What that means is that, for the very first value at row #1, the percentile rank will be 0%, since there are no other rows before it. Well we can't have that! How do you tell a customer they are in the top 0%? Insanity.

I knew the answer. Cumulative distribution will give you the percentage of values that not only come before the value of the current row, but it will also include the current row. So if you have 100 rows going from 1 to 100, `CUME_DIST` will give you a result of 1% for row #1. I am a genius. 

Here was my code:
<details>
<summary> 🔴 SQL code</summary>
  
<pre>
-- PERCENTILE RANK: The top x% 

DROP TABLE IF EXISTS percentile_rank;
CREATE TEMP TABLE percentile_rank AS
SELECT
  customer_id,
  category_name,
  rental_count,
  latest_rental_date,
  CEILING(
-- Using CUME_DIST since PERCENT_RANK will bring back values of 0%, which looks odd in the email text.
    100 * CUME_DIST() OVER (
    PARTITION BY category_name
    ORDER BY rental_count DESC, latest_rental_date DESC
    )
  ) AS percentile
FROM category_rental_counts;
</pre>
</details>

I used the following code to get the percentile results for just the top-ranked category per customer to compare with Danny's results:
<details>
<summary> 🔴 SQL code</summary>

<pre>
-- To get only top-category percentages

SELECT
  t1.customer_id,
  t1.category_name,
  t1.rental_count,
  t1.rank_number,
  t2.percentile
FROM top_2_ranking t1
LEFT JOIN percentile_rank t2 
 ON t1.customer_id = t2.customer_id
WHERE 
  t1.rank_number = 1 AND 
  t1.category_name = t2.category_name
ORDER BY customer_id;
</pre>
</details>

Let's compare my answers with Danny's, paying attention to the `percentile` column at the far right:

Mine
| customer_id | category_name | rental_count | rank_number | percentile |
|-------------|---------------|--------------|-------------|------------|
| 1           | Classics      | 6            | 1           | 1          |
| 2           | Sports        | 5            | 1           | 7          |
| 3           | Action        | 4            | 1           | 13         |
| 4           | Horror        | 3            | 1           | 14         |
| 5           | Classics      | 7            | 1           | 1          |
| 6           | Drama         | 4            | 1           | 8          |
| 7           | Sports        | 5            | 1           | 6          |
| 8           | Classics      | 4            | 1           | 9          |
| 9           | Foreign       | 4            | 1           | 10         |
| 10          | Documentary   | 4            | 1           | 11         |

Danny's
| customer_id | category_name | rental_count | category_rank | percentile |
|-------------|---------------|--------------|---------------|------------|
| 1           | Classics      | 6            | 1             | 1          |
| 2           | Sports        | 5            | 1             | 2          |
| 3           | Action        | 4            | 1             | 4          |
| 4           | Horror        | 3            | 1             | 8          |
| 5           | Classics      | 7            | 1             | 1          |
| 6           | Drama         | 4            | 1             | 3          |
| 7           | Sports        | 5            | 1             | 2          |
| 8           | Classics      | 4            | 1             | 2          |
| 9           | Foreign       | 4            | 1             | 6          |
| 10          | Documentary   | 4            | 1             | 5          |

![image](https://user-images.githubusercontent.com/99853599/171050748-a41cb056-c37d-4182-904a-10a06815e743.png)

It took me a lot of tinkering to figure out what should have been obvious but didn't click for me until I had done a lot of...well, tinkering.

`CUME_DIST` does indeed include the row we are concerned with when calculating the result. What I failed to realize, however, is that it also includes all other rows with values that are equal to the value in the current row. In other words, that percentile result is going to go up if that row has a value equal to the values of other rows. I made two different, very simple tables so I could see if my thinking was correct.

As I said above, if you have 100 rows going from 1 to 100, `CUME_DIST` will give you a result of 1% for row #1. It should then go up in increments of 1% for each succeeding value. Let's look at the first ten rows of just such a table:
<details>
<summary> 🔴 SQL code</summary>
  
<pre>
CREATE TEMP TABLE my_table (
  score NUMERIC
);

INSERT into my_table (score)
SELECT i
FROM generate_series(1,100) i;

SELECT
  score,
  CUME_DIST() OVER(
  ORDER BY score
  ) AS cume_dist
FROM my_table;
</pre>
</details>

| score | cume_dist |
|-------|-----------|
| 1     | 0.01      |
| 2     | 0.02      |
| 3     | 0.03      |
| 4     | 0.04      |
| 5     | 0.05      |
| 6     | 0.06      |
| 7     | 0.07      |
| 8     | 0.08      |
| 9     | 0.09      |
| 10    | 0.1       |

This is exactly what I was expecting. Now, let's change one of the scores to match another score. I'm going to use a trick I saw Danny use in his course. I will make a duplicate of this table and then change that new table:
<details>
<summary> 🔴 SQL code</summary>

<pre>  
DROP TABLE IF EXISTS test_my_table;
CREATE TEMP TABLE test_my_table AS
  TABLE my_table;
  
UPDATE test_my_table
SET score = 1 
WHERE score = 2;

SELECT *
FROM test_my_table
ORDER BY score;
</pre>
</details>

| score |
|-------|
| 1     |
| 1     |
| 3     |
| 4     |
| 5     |
| 6     |
| 7     |
| 8     |
| 9     |
| 10    |

Now I have two scores equal to the value 1. Since `CUME_DIST` includes rows of equal value in its calculation, I think it will now return 2% for the first two values. Let's check that out:
```sql
SELECT
  score,
  CUME_DIST() OVER(
  ORDER BY score
  ) AS cume_dist
FROM test_my_table;
```
| score | cume_dist |
|-------|-----------|
| 1     | 0.02      |
| 1     | 0.02      |
| 3     | 0.03      |
| 4     | 0.04      |
| 5     | 0.05      |
| 6     | 0.06      |
| 7     | 0.07      |
| 8     | 0.08      |
| 9     | 0.09      |
| 10    | 0.1       |

As expected, these two rows, with a score of 1, now account for the first 2% of all rows in the table.

Just by saying this I now realize that is not what Danny was asking for. He wants to know the percentage of rows with values that come BEFORE the value in the current row. That is definitely the job of PERCENT_RANK.

There's another way to handle the 0% issue. Danny uses a `CASE WHEN` clause. So let's do that, change `CUME_DIST` to `PERCENT_RANK`, and then I'll double-check that my answers match Danny's, and I can move on.
<details>
<summary> 🔴 SQL code</summary>
  
<pre>
DROP TABLE IF EXISTS percentile_rank;
CREATE TEMP TABLE percentile_rank AS
WITH cte_1 AS (
  SELECT
    customer_id,
    category_name,
    rental_count,
    latest_rental_date,
    ROUND(100 * PERCENT_RANK() OVER (
      PARTITION BY category_name
      ORDER BY rental_count DESC, latest_rental_date DESC
      )
    ) AS percentile
  FROM category_rental_counts
)
SELECT
  customer_id,
  category_name,
  rental_count,
  CASE
    WHEN percentile = 0 THEN 1
    ELSE percentile
  END AS percentile
FROM cte_1;

SELECT
  t1.customer_id,
  t1.category_name,
  t1.rental_count,
  t1.rank_number,
  t2.percentile
FROM top_2_ranking t1
LEFT JOIN percentile_rank t2 
 ON t1.customer_id = t2.customer_id
WHERE 
  t1.rank_number = 1 AND 
  t1.category_name = t2.category_name
ORDER BY customer_id;
</pre>
</details>
  
Yep, just punch that in and we should be good... Wait... whuu?

| customer_id | category_name | rental_count | rank_number | percentile |
|-------------|---------------|--------------|-------------|------------|
| 1           | Classics      | 6            | 1           | 1          |
| 2           | Sports        | 5            | 1           | 6          |
| 3           | Action        | 4            | 1           | 13         |
| 4           | Horror        | 3            | 1           | 14         |
| 5           | Classics      | 7            | 1           | 1          |
| 6           | Drama         | 4            | 1           | 7          |
| 7           | Sports        | 5            | 1           | 5          |
| 8           | Classics      | 4            | 1           | 8          |
| 9           | Foreign       | 4            | 1           | 10         |
| 10          | Documentary   | 4            | 1           | 10         |

![image](https://user-images.githubusercontent.com/99853599/171066316-e59512fa-ccbb-45f7-9a1e-9be185046551.png)

Permission to cry freely now, Captain?

I hit upon the solution after some more thinking. What it came down to was, once again, the `ORDER BY` clause in the window function. After I got my head wrapped around it, it made sense. I was ordering by `rental_count` and `latest_rental_date` thinking that both mattered to account for ties in `rental_count`. Upon reflection, I realized that `latest_rental_date` is not a parameter. The "percentile" is calculated by looking at the `rental_count` in decreasing order. If the value is 4, then I want to know the percent of rows with values that come before that value, i.e. all the rows with values of 5, 6, 7 and higher. However, if I add an extra parameter of `latest_rental_date`, then it will also take into consideration all of the other rows with values of 4 that come prior to the date of the current row. That is going to drive the "percentile" value up. 

By eliminating `latest_rental_date` from the window function's `ORDER BY` clause, the percentiles should now be correct.
<details>
<summary> 🔴 SQL code</summary>

<pre>
DROP TABLE IF EXISTS percentile_rank;
CREATE TEMP TABLE percentile_rank AS
WITH cte_1 AS (
  SELECT
    customer_id,
    category_name,
    rental_count,
    latest_rental_date,
    ROUND(100 * PERCENT_RANK() OVER (
      PARTITION BY category_name
-- Delete latest_rental_date DESC from the ORDER BY here
      ORDER BY rental_count DESC
      )
    ) AS percentile
  FROM category_rental_counts
)
SELECT
  customer_id,
  category_name,
  rental_count,
  CASE
    WHEN percentile = 0 THEN 1
    ELSE percentile
  END AS percentile
FROM cte_1;

SELECT
  t1.customer_id,
  t1.category_name,
  t1.rental_count,
  t1.rank_number,
  t2.percentile
FROM top_2_ranking t1
LEFT JOIN percentile_rank t2 
 ON t1.customer_id = t2.customer_id
WHERE 
  t1.rank_number = 1 AND 
  t1.category_name = t2.category_name
ORDER BY customer_id;
</pre>
</details>

| customer_id | category_name | rental_count | rank_number | percentile |
|-------------|---------------|--------------|-------------|------------|
| 1           | Classics      | 6            | 1           | 1          |
| 2           | Sports        | 5            | 1           | 2          |
| 3           | Action        | 4            | 1           | 4          |
| 4           | Horror        | 3            | 1           | 8          |
| 5           | Classics      | 7            | 1           | 1          |
| 6           | Drama         | 4            | 1           | 3          |
| 7           | Sports        | 5            | 1           | 2          |
| 8           | Classics      | 4            | 1           | 2          |
| 9           | Foreign       | 4            | 1           | 6          |
| 10          | Documentary   | 4            | 1           | 5          |

![image](https://user-images.githubusercontent.com/99853599/171082063-a6b8df69-2209-4247-8eb3-254e257b7c5e.png)

<img align="left"  src= "https://user-images.githubusercontent.com/99853599/182891270-127b296c-b5b7-4b75-a383-50de4510228e.png" width="250" height="350"/>
<br />
<br />
<br />
<br />
<br />
<br />
Perfect match
<br clear="left"/>
<br />
	
## [The JOINs](#table-of-contents)
---
### *Summary*
Despite having been taught to think carefully about which JOIN function to use before implementing one, I had to make a mistake before the lesson hit home. I learned not to assume the information I wanted was in my target table (i.e., the data was not "clean") and that `INNER JOIN` is preferable in this case. 
***
  
One of the biggest tutorials in Danny's course, besides the one on windows functions, is the tutorial regarding `JOINS`. In this case study, the `LEFT JOIN` and the `INNER JOIN` are used extensively. Picking the wrong one can lead to mistakes. 

Since I used to use Excel a lot before learning SQL, I see both `LEFT JOIN` and `INNER JOIN` as siblings of VLOOKUP. `LEFT JOIN` will search the values that exist in a column on the base table (primary keys) and look for them in the target table. If it finds that key (now called a foreign key in the target table), it will pull out corresponding values in other columns of that same row in the target table and slap that on to your base table, effectively combining the two tables. If it doesn't find that key in the target table, you will get a NULL in the column(s) it brings over.

`INNER JOIN` is like the above, but your final table will only give you rows if the primary key was found in the target table. If it wasn't, the whole row is gone.

In the section of the script where we start looking for the most-watched actor per customer, an actor dataset is created by joining multiple tables. Danny runs some `DISTINCT` queries against his dataset and gets 955 for `unique_film_id`. I ran the same queries against my dataset and got 958. Eventually, I hit upon the idea that maybe my joins were to blame. I had used `LEFT JOIN`, whereas Danny had used `INNER JOIN`. I then used a trick Danny showed us in the joins tutorial to see if the rows differ when using a `LEFT JOIN` versus using an `INNER JOIN` to create the dataset:
<details>
<summary> 🔴 SQL code</summary>
  
<pre>
## Confirming any differences in foreign key values between `LEFT JOIN` and `INNER JOIN`

DROP TABLE IF EXISTS left_join_actor_joint_dataset;
CREATE TEMP TABLE left_join_actor_joint_dataset AS
SELECT
  rental.customer_id,
  rental.rental_id,
  rental.rental_date,
  film.film_id,
  film.title,
  actor.actor_id,
  actor.first_name,
  actor.last_name
FROM dvd_rentals.rental
LEFT JOIN dvd_rentals.inventory
  ON rental.inventory_id = inventory.inventory_id
LEFT JOIN dvd_rentals.film
  ON inventory.film_id = film.film_id
LEFT JOIN dvd_rentals.film_actor
  ON film.film_id = film_actor.film_id
LEFT JOIN dvd_rentals.actor
  ON film_actor.actor_id = actor.actor_id;
 
DROP TABLE IF EXISTS inner_join_actor_joint_dataset;
CREATE TEMP TABLE inner_join_actor_joint_dataset AS
SELECT
  rental.customer_id,
  rental.rental_id,
  rental.rental_date,
  film.film_id,
  film.title,
  actor.actor_id,
  actor.first_name,
  actor.last_name
FROM dvd_rentals.rental
INNER JOIN dvd_rentals.inventory
  ON rental.inventory_id = inventory.inventory_id
INNER JOIN dvd_rentals.film
  ON inventory.film_id = film.film_id
INNER JOIN dvd_rentals.film_actor
  ON film.film_id = film_actor.film_id
INNER JOIN dvd_rentals.actor
  ON film_actor.actor_id = actor.actor_id;

-- Output SQL
(
  SELECT
    'left join' AS join_type,
    COUNT(*) AS record_count,
    COUNT(DISTINCT film_id) AS unique_film_id
  FROM left_join_actor_joint_dataset
)
UNION
(
  SELECT
    'inner join' AS join_type,
    COUNT(*) AS record_count,
    COUNT(DISTINCT film_id) AS unique_film_id
  FROM inner_join_actor_joint_dataset
);
</pre>
</details>

| join_type  | record_count | unique_film_id |
|------------|--------------|----------------|
| inner join | 87980        | 955            |
| left join  | 88020        | 958            |

This makes it very clear that the type of join I use will give a different result. There is a difference of 40 in `record_count` and a difference of 3 in `unique_film_id`. But why? I thought I wanted to keep all of the rows from the base table. To dig further, I used the following to pull out any rows that had a NULL value in it after the `LEFT JOIN`:
<details>
<summary> 🔴 SQL code</summary>
  
<pre>
SELECT *
FROM left_join_actor_joint_dataset
WHERE 
  customer_id IS NULL OR 
  rental_id IS NULL OR 
  rental_date IS NULL OR 
  film_id IS NULL OR 
  title IS NULL OR 
  actor_id IS NULL OR 
  first_name IS NULL OR 
  last_name IS NULL;
</pre>
</details>

Forty rows came back, accounting for the difference in `record_count`. Here are the first ten:

| customer_id | rental_id | rental_date              | film_id | title            | actor_id | first_name | last_name |
|-------------|-----------|--------------------------|---------|------------------|----------|------------|-----------|
| 86          | 66        | 2005-05-25T09:35:12.000Z | 257     | DRUMLINE CYCLONE |          |            |           |
| 274         | 208       | 2005-05-26T08:10:22.000Z | 803     | SLACKER LIAISONS |          |            |           |
| 215         | 1599      | 2005-06-16T06:03:33.000Z | 323     | FLIGHT LIES      |          |            |           |
| 205         | 1767      | 2005-06-16T18:01:36.000Z | 257     | DRUMLINE CYCLONE |          |            |           |
| 208         | 1949      | 2005-06-17T08:19:22.000Z | 323     | FLIGHT LIES      |          |            |           |
| 256         | 1973      | 2005-06-17T09:26:15.000Z | 803     | SLACKER LIAISONS |          |            |           |
| 420         | 2672      | 2005-06-19T11:42:04.000Z | 257     | DRUMLINE CYCLONE |          |            |           |
| 332         | 4302      | 2005-07-07T16:47:53.000Z | 803     | SLACKER LIAISONS |          |            |           |
| 96          | 4961      | 2005-07-08T23:35:53.000Z | 803     | SLACKER LIAISONS |          |            |           |
| 198         | 5794      | 2005-07-10T14:34:53.000Z | 257     | DRUMLINE CYCLONE |          |            |           |

My suspicion was that all of these rows were simply three movies that didn't have actors names attached to them, hence the NULL returns. This would account for the difference of three in the `unique_film_id` results. I can check this from this current table I just created by putting the last query into a CTE, or I can check by doing an anti-join using both the `INNER JOIN` and `LEFT JOIN` datasets.

Here's the first way:
<details>
<summary> 🔴 SQL code</summary>
  
<pre>
WITH cte_1 AS (
  SELECT *
  FROM left_join_actor_joint_dataset
  WHERE 
    customer_id IS NULL OR 
    rental_id IS NULL OR 
    rental_date IS NULL OR 
    film_id IS NULL OR 
    title IS NULL OR 
    actor_id IS NULL OR 
    first_name IS NULL OR 
    last_name IS NULL
)
SELECT 
  film_id,
  title
FROM cte_1
GROUP BY
  film_id,
  title
ORDER BY title;
</pre>
</details>

| film_id | title            |
|---------|------------------|
| 257     | DRUMLINE CYCLONE |
| 323     | FLIGHT LIES      |
| 803     | SLACKER LIAISONS |

The second way:
<details>
<summary> 🔴 SQL code</summary>

<pre>
DROP TABLE IF EXISTS left_unique_film_id;
CREATE TEMP TABLE left_unique_film_id AS 
SELECT DISTINCT(film_id)
FROM left_join_actor_joint_dataset;

DROP TABLE IF EXISTS inner_unique_film_id;
CREATE TEMP TABLE inner_unique_film_id AS 
SELECT DISTINCT(film_id)
FROM inner_join_actor_joint_dataset;

SELECT
  t1.film_id
FROM left_unique_film_id t1
WHERE NOT EXISTS (
  SELECT 1
  FROM inner_unique_film_id t2 
  WHERE t1.film_id = t2.film_id
);
</pre>
</details>
  
| film_id |
|---------|
| 803     |
| 257     |
| 323     |

```sql
SELECT
  film_id,
  title
FROM dvd_rentals.film
WHERE film_id IN (803,257,323)
ORDER BY title;
```
| film_id | title            |
|---------|------------------|
| 257     | DRUMLINE CYCLONE |
| 323     | FLIGHT LIES      |
| 803     | SLACKER LIAISONS |

Since we are interested in the most-watched actor per customer, pulling out movies that are not attached to any actors is pointless. It can also lead to calculation errors downstream if we are including those movies in our later datasets. Had I thought about this possibility beforehand, I would have opted for the `INNER JOIN` instead of the `LEFT JOIN`, because the `INNER JOIN` does not return rows for keys it cannot find in the target table. My "key" here was `film_id`. Since these three `film_id` values do not exist in the tables containing `actor_id` and actor names, `INNER JOIN` will not return any information on these three films, leaving only films that have actors attached to them. 

## [What's in a Name? Dealing with Duplicates](#table-of-contents)
---
### *Summary*
The lesson I learned here is to think carefully about how I retrieve information, and to use primary/foreign keys whenever possible.
***
  
There were many interesting discoveries I made when comparing my code to Danny's and tweaking it to see how it changes my results. One thing I discovered is that you can't trust that people with the same name are...the same person. Go figure.

My result for the most-watched actor for `customer_id` = 5 was Susan Davis. Danny had a different actor.

Let me cut to the chase:
<details>
<summary> 🔴 SQL code</summary>

<pre>
WITH cte_1 AS (
  SELECT
    first_name,
    last_name,
    COUNT(*) AS count_actor
  FROM dvd_rentals.actor
  GROUP BY
    first_name,
    last_name
)
SELECT
  t1.*,
  t2.actor_id
FROM cte_1 t1 
  LEFT JOIN dvd_rentals.actor t2
  ON t1.first_name = t2.first_name
  AND t1.last_name = t2.last_name
ORDER BY
  count_actor DESC
LIMIT 10;
</pre>
</details>

| first_name | last_name    | count_actor | actor_id |
|------------|--------------|-------------|----------|
| SUSAN      | DAVIS        | 2           | 101      |
| SUSAN      | DAVIS        | 2           | 110      |
| ED         | CHASE        | 1           | 3        |
| JOHNNY     | LOLLOBRIGIDA | 1           | 5        |
| BETTE      | NICHOLSON    | 1           | 6        |
| GRACE      | MOSTEL       | 1           | 7        |
| JOE        | SWANK        | 1           | 9        |
| MATTHEW    | JOHANSSON    | 1           | 8        |
| CHRISTIAN  | GABLE        | 1           | 10       |
| JENNIFER   | DAVIS        | 1           | 4        |

This is why the [Google course](https://grow.google/certificates/data-analytics/#?modal_active=none) has lengthy discussions about cleaning data, as does Danny. Checking for duplicates, or doppelgangers, is helpful. In my original script, I used the actors' first and last names to join tables and aggregate information, when I should have been using `actor_id`. The actors with `actor_id` 101 and 110, while they have the same name, represent different actors who are connected to different movies. This is most likely the cause of some of my wrong answers. 

## [Final Takeaways](#table-of-contents)
Despite the time and effort it took to go through my code again, snippet after snippet, and compare it with Danny's, it was a great way to challenge myself and really make sure I understood the concepts I have been learning in his course. I found some of my weak points and now know what to watch out for in the future. For example, I know now to be very mindful of the parameters I set in window functions, the type of `JOIN` clause I use, to work with primary keys whenever possible, etc.
  
I also found out that there is more than one way to retrieve the correct output. There are still many places in my code that are very different from Danny's. Needless to say, Danny is much more efficient. I discovered that I had written some unnecessarily complex code to retrieve results that Danny did in a very straightforward way (I'm looking at you `movies_seen_top_categories` table). I suspect this happened because I neglected to step back, take a breath, and remember the big picture. It is so easy to get lost in the weeds and get stuck in having a very specific way to approach a problem.
  
On the other hand, there were areas where my code was longer, but I needed to do it that way because that's the way my brain thinks through the problem. Especially when doing the big anti-joins to get recommended movies, I needed to lay out my queries in a more straightforward, step-by-step path. I noticed that Danny's code did essentially the same thing, but it was more sophisticated, putting some snippets within CTEs to make the code shorter and neater, and most likely faster. I expect I will get better with this over time, once I get used to solving more problems that require this kind of thinking.
  
Another thing I quickly realized while checking this code was the need to keep accurate notes! Going through code, testing it, changing it, looking at the results, and comparing the results with those from other renditions of the code, all of this requires fastidious note-taking. I spent days on figuring out some of the problems I bring up here. Then life would happen and I would come back to it after a few more days or possibly a couple of weeks. Trying to get back into that mindset of problem-solving would have been extremely difficult if I hadn't taken very clear notes about what I had already tried and what I needed to work on. I think this was great practice to get into the habit of keeping organized notes for long projects.
  
Overall, this was a great experience and I'm grateful for the opportunity to increase my knowledge and strengthen my SQL skills. I look forward to the next challenge!
  
## [New Code](#table-of-contents)
Click below to see the new, improved SQL code. As I mention above, there are still areas that are very different from Danny's. For example, Danny's use of the `MAX` function with string values is a quick and efficient way to pivot the tables into a wide format. This is something I never even thought of. However, after a lot of testing on all of these areas that differ, I found that they return the same results when all is said and done. In some places, I had to slightly change the code so that the results matched. In one place in particular, I don't necessarily agree with the reasoning behind it. It seems that Danny does not take into consideration repeat viewings of the same movie when calculating how many movies a customer has seen of their favorite actor. However, I may have also misunderstood the requirements. If this were a real-world project I was doing for a company, I would have asked my manager for clarification. For the purposes of this project, I was happy that I was able to figure out why my results were not matching Danny's.

All this is to say that while this code is far from perfect, it returns the correct results and can be used to correctly answer the questions to the section quiz.
<details>
<summary> ❗New SQL code❗</summary>
  
<pre>-- SQL Code

/*
Joined Base Dataset
From complete_joint_dataset, I will generate multiple tables
that will contain the data I need for my insights.
I will join all of these into an output_table.
*/

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

-- CUSTOMER TOTAL RENTALS  

DROP TABLE IF EXISTS customer_total_rentals;
CREATE TEMP TABLE customer_total_rentals AS
SELECT
  customer_id,
  SUM(rental_count) AS total_rental_count
FROM category_rental_counts
GROUP BY customer_id;

-- TOP 2 RANKING: Per customer

DROP TABLE IF EXISTS top_2_ranking;
CREATE TEMP TABLE top_2_ranking AS 
WITH cte_1 AS (  
  SELECT 
    customer_id,
    category_name,
    rental_count,
    latest_rental_date,
    DENSE_RANK() OVER (
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
-- Add in latest_rental_date to show how this affects the rank_number
  latest_rental_date,
  rank_number
FROM cte_1
WHERE rank_number in (1,2)
ORDER BY customer_id;

-- AVERAGE CATEGORY RENTAL COUNTS: Per category

DROP TABLE IF EXISTS average_category_rental_counts;
CREATE TEMP TABLE average_category_rental_counts AS 
SELECT
  category_name,
-- Using FLOOR to get a whole number that makes sense when rendered 
-- on the marketing email
  FLOOR(AVG(rental_count)) AS avg_rental_per_category
FROM category_rental_counts
GROUP BY category_name
ORDER BY category_name;

-- PERCENTILE RANK: The top x% 

DROP TABLE IF EXISTS percentile_rank;
CREATE TEMP TABLE percentile_rank AS
WITH cte_1 AS (
  SELECT
    customer_id,
    category_name,
    rental_count,
    latest_rental_date,
    ROUND(100 * PERCENT_RANK() OVER (
      PARTITION BY category_name
      ORDER BY rental_count DESC
      )
    ) AS percentile
  FROM category_rental_counts
)
SELECT
  customer_id,
  category_name,
  rental_count,
  CASE
    WHEN percentile = 0 THEN 1
    ELSE percentile
  END AS percentile
FROM cte_1;

-- JOIN TABLES

DROP TABLE IF EXISTS output_table;
CREATE TEMP TABLE output_table AS 
SELECT
  t1.customer_id,
  t1.rank_number AS category_ranking,
  t1.category_name,
  t1.rental_count,
-- This will give me the "you've watched X more movies than the average" data point
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

/* Now I will do an anti-join to generate a list of recommended movies.
These are movies that (a) the customer has not rented before and
(b) listed in order of popularity (rental_count and latest_rental_date).
*/ 

-- Generate the base table for the anti-join in two steps.

-- Step 1: Order categories by most popular movies with latest_rental_date

DROP TABLE IF EXISTS recommendations_table;
CREATE TEMP TABLE recommendations_table AS 
SELECT 
  film_id,
  category_name,
  title,
  MAX(rental_date) AS latest_rental_date,
  COUNT(film_id) AS times_rented
FROM complete_joint_dataset
GROUP BY film_id, category_name, title
ORDER BY category_name, times_rented DESC, latest_rental_date DESC; 

-- Step 2: Every cust_id's top_2_rank recommended films without 
-- filtering out films already seen 

DROP TABLE IF EXISTS per_cust_recommendations_table_full;
CREATE TEMP TABLE per_cust_recommendations_table_full AS 
SELECT
  t1.customer_id,
  t1.category_name,
  t1.rank_number AS cat_ranking,
  t2.title,
  t2.latest_rental_date,
  t2.times_rented
FROM top_2_ranking t1
-- Customer id's are already tied to two categories in the top_2_ranking table.
-- A LEFT JOIN here will pull in all of the movies for those two categories.
LEFT JOIN recommendations_table t2 
  ON t1.category_name = t2.category_name
ORDER BY t1.customer_id, t1.category_name, t2.times_rented DESC;

-- Generate the target table.

-- Every cust_id's top_2_rank film names with category_name

DROP TABLE IF EXISTS movies_seen_top_categories;
CREATE TABLE movies_seen_top_categories AS (
-- Generating a list of distinct rows
WITH cte_1 AS (
  SELECT
    customer_id,
    film_id,
    title,
    category_name,
    COUNT(film_id) AS count
  FROM complete_joint_dataset
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

-- This table is the final table that has the top three recommended movies 
-- for each customer split by category

DROP TABLE IF EXISTS top_3_recs;
CREATE TEMP TABLE top_3_recs AS (
WITH cte_1 AS (
  SELECT *,
-- I decided to go with ROW_NUMBER here because RANK will return 
-- the same number for ties.
    ROW_NUMBER() OVER (
      PARTITION BY customer_id, category_name
-- In order to make the top_3_actor_recs match Danny's 100%, 
-- had to change latest_rental_date to title here.
      ORDER BY times_rented DESC, title
    ) AS ranking
  FROM per_cust_recommendations_table_full t1
-- An anti-join will eliminate rows from the base table that 
-- show up in the target table.
  WHERE NOT EXISTS (
    SELECT 1
    FROM movies_seen_top_categories t2 
    WHERE
      t1.customer_id = t2.customer_id AND 
      t1.category_name = t2.category_name AND 
      t1.title = t2.title
  )
-- Took out category_name, replaced with cat_ranking, and 
-- added title to order correctly and match Danny's table 100%
  ORDER BY customer_id, cat_ranking, times_rented DESC, title 
)
-- Grab the top three movies for each customer.
SELECT *
FROM cte_1
WHERE ranking IN (1,2,3)
);

-- Mega table that can now be used to populate the 
-- final insight results (for 1 and 2)

DROP TABLE IF EXISTS insights_inputs;
CREATE TEMP TABLE insights_inputs AS (
-- Create three separate tables split by movie recommendation ranking.
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
-- Join the above tables together along with output_table.
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

/*
Now begin analysis to find most-watched actor and the top-three
recommendations for that actor.
*/

-- Join the original joined dataset with actor names.

DROP TABLE IF EXISTS actor_dataset;
CREATE TEMP TABLE actor_dataset AS 
SELECT 
  complete_joint_dataset.*,
  dvd_rentals.film_actor.actor_id,
  dvd_rentals.actor.first_name,
  dvd_rentals.actor.last_name
FROM complete_joint_dataset
  INNER JOIN dvd_rentals.film_actor
    ON complete_joint_dataset.film_id = film_actor.film_id
  INNER JOIN dvd_rentals.actor 
    ON dvd_rentals.film_actor.actor_id = dvd_rentals.actor.actor_id;

DROP TABLE IF EXISTS customer_actors;
CREATE TEMP TABLE customer_actors AS 
-- I changed this code and it now returns number of times a movie 
-- was rented with top actor, regardless of repeat viewing. 
-- This seems to help it match Danny's answers.
SELECT
  customer_id,
  actor_id,
  first_name,
  last_name,
  COUNT(*) AS actor_count,
  MAX(rental_date) AS latest_rental_date
FROM actor_dataset
GROUP BY
  customer_id,
  actor_id,
  first_name,
  last_name
-- Business requirements state that ties should be handled by 
-- choosing the actor who comes first in alphabetical order.
-- But I changed this because it looks like Danny didn't do that.
ORDER BY customer_id, actor_count DESC, latest_rental_date DESC;

-- This table has the top-watched actor per customer.

DROP TABLE IF EXISTS cust_top_actors;
CREATE TEMP TABLE cust_top_actors AS (
WITH cte_1 AS (
  SELECT *,
-- I decided to go with ROW_NUMBER here because RANK will return 
-- the same number for ties.
    ROW_NUMBER() OVER (
      PARTITION BY customer_id
-- Business requirements state that ties should be handled by 
-- choosing the actor who comes first in alphabetical order.
-- But I changed this because it looks like Danny didn't do that.
      ORDER BY actor_count DESC, latest_rental_date DESC, first_name, last_name   
    ) AS row_num
  FROM customer_actors
)
SELECT
  customer_id,
  actor_id,
  first_name,
  last_name,
-- 7/21/2022 Added this to bring into final table later on.
  actor_count
FROM cte_1
WHERE row_num = 1
);

/*
I will need to do another anti-join to get a list of recommended movies 
starring the top-watched actor for each customer. This will takes a few steps,
requiring the creation of a base table and target table.
*/

-- Generate the base table for the anti-join in two steps.

-- STEP 1: This is the raw table of all movies customer-favorite actors acted in

DROP TABLE IF EXISTS fav_actor_movies_no_filter;
CREATE TEMP TABLE fav_actor_movies_no_filter AS 
SELECT
  t1.customer_id,
  t1.actor_id,
  t1.first_name,
  t1.last_name,
  t4.title,
  t6.inventory_id,
  t6.rental_date
FROM cust_top_actors t1
  LEFT JOIN dvd_rentals.actor t2 
    ON t1.actor_id = t2.actor_id
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

-- STEP 2: This table is a list of all movies customer-favorite actors acted in 
-- ordered by popularity (rental_count, latest_rental_date).

DROP TABLE IF EXISTS fav_actor_movies_by_popularity;
CREATE TEMP TABLE fav_actor_movies_by_popularity AS
SELECT
  customer_id,
  actor_id,
  first_name,
  last_name,
  title,
  COUNT(*) AS total_rented,
  MAX(rental_date) AS latest_rental_date
FROM fav_actor_movies_no_filter
GROUP BY
  customer_id,
  actor_id,
  first_name,
  last_name,
  title
ORDER BY customer_id, total_rented DESC, latest_rental_date DESC;

-- Generate the target table.
 
-- This table is a list of movies starring the top-watched actor 
-- that each customer has already seen.

DROP TABLE IF EXISTS fav_actor_movies_seen;
CREATE TEMP TABLE fav_actor_movies_seen AS 
SELECT 
  t1.customer_id,
  t1.actor_id,
  t1.first_name,
  t1.last_name,
  t2.title
FROM cust_top_actors t1 
LEFT JOIN actor_dataset t2 
  ON t1.customer_id = t2.customer_id
  AND t1.actor_id = t2.actor_id
ORDER BY t1.customer_id, t1.actor_id, t1.last_name, t1.first_name;

-- This is a table with customer_id, actor's first and last name, 
-- the movies this actor has acted in that the customer has NOT seen, 
-- in order of popularity (rental_count, latest_rental_date).

DROP TABLE IF EXISTS fav_actor_movies_not_seen;
CREATE TEMP TABLE fav_actor_movies_not_seen AS 
SELECT *
FROM fav_actor_movies_by_popularity t1
-- An anti-join will eliminate rows from the base table that show up in the target table. 
WHERE NOT EXISTS (
  SELECT 1
  FROM fav_actor_movies_seen t2
  WHERE
    t1.customer_id = t2.customer_id AND 
    t1.title = t2.title
)
ORDER BY customer_id, total_rented DESC, latest_rental_date DESC;

-- This anti-join customizes the table even further. The business requirements state 
-- that the movies in this list cannot repeat previously recommended movies 
-- for the top-two categories.

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

-- This table gives the three recommendations for movies starring 
-- each customer's favorite actor.

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
-- Changed this 6/27/22 from latest_rental_date to title and 
-- now matches Danny's 100%. 
-- Had to do the same with top_3_recs above.
        title
    ) AS rank_num
  FROM fav_actor_movies_not_seen_not_rec_prev
  ORDER BY customer_id, rank_num
)
SELECT *
FROM cte_1
WHERE rank_num IN (1,2,3)
);

SELECT *
FROM top_3_actor_recs;

-- This table gives each customer_id its own row with their favorite actor
-- and three movie recs, as well as number of movies with this actor each 
-- customer has already seen.

DROP TABLE IF EXISTS final_actor_recs;
CREATE TEMP TABLE final_actor_recs AS (
-- Create three separate tables split by movie recommendation ranking.
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
-- Join the above tables together along with the previous table.
SELECT
  t1.customer_id,
  t1.first_name,
  t1.last_name,
  t1.title AS actor_movie_rec_1,
  t2.title AS actor_movie_rec_2,
  t3.title AS actor_movie_rec_3,
-- 7/21/2022 Changed to pull directly from cust_top_actors
  t4.actor_count
FROM actor_rec_movie_1 t1 
  LEFT JOIN actor_rec_movie_2 t2 
  ON t1.customer_id = t2.customer_id
  LEFT JOIN actor_rec_movie_3 t3
  ON t1.customer_id = t3.customer_id
-- 7/21/2022 Changed to direct to this table instead of 
-- num_movies_w_fav_actor_seen
  LEFT JOIN cust_top_actors t4 
  ON t1.customer_id = t4.customer_id
);

-- Here is the text table for actor_insights

DROP TABLE IF EXISTS actor_insights;
CREATE TEMP TABLE actor_insights AS 
SELECT
  customer_id,
  INITCAP(first_name) || ' ' || INITCAP(last_name) AS actor_name,
-- 7/21/2022 changed number_of_movies_seen to actor_count since I changed this above
    'You''ve watched ' || actor_count || ' films featuring ' || 
	INITCAP(first_name) || ' ' || INITCAP(last_name) || '!' || ' Here are some other films ' || 
	INITCAP(first_name) || ' stars in that might interest you!' AS actor_insight,
  INITCAP(actor_movie_rec_1) AS actor_movie_rec_1,
  INITCAP(actor_movie_rec_2) AS actor_movie_rec_2,
  INITCAP(actor_movie_rec_3) AS actor_movie_rec_3
FROM final_actor_recs;

/*
This is the text table from the original top ranked insights. However, this is collapsed. 
I need to change it so that each customer only has one row. 
Then I can join with the actor_insights table for the final insights table.
*/

DROP TABLE IF EXISTS top_movie_insights_collapsed;
CREATE TEMP TABLE top_movie_insights_collapsed AS 
SELECT
  customer_id,
  category_ranking,
  category_name,
-- Text for category 1 movies.
  CASE WHEN category_ranking = 1 THEN
-- Another CASE WHEN for any movies that have an average_comparison of 0. 
-- In other words, the number of time the customer rented from this category 
-- equals the average rentals for that category.
      CASE WHEN average_comparison > 0 THEN
      'You''ve watched ' || rental_count || ' ' || category_name || ' films. That''s ' || 
	  average_comparison || ' more than the DVD Rental Co average and puts you in the top ' || 
	  percentile || '% of ' || category_name || ' Gurus!'
      ELSE 
      'You''ve watched ' || rental_count || ' ' || category_name || ' films. 
	  That is the DVD Rental Co average and puts you in the top ' || percentile || '% of ' || 
	  category_name || ' Gurus!'
      END
-- Text for category 2 movies.    
  ELSE 
  'You''ve watched ' || rental_count || ' ' || category_name || ' films, making up ' || 
  category_percentage || '%' || ' of your entire viewing history!' 
  END AS insight,
-- Text for category 1 movies.
  CASE WHEN category_ranking = 1 THEN
  'Your expertly chosen recommendations: '
-- Text for category 2 movies.  
  ELSE 
   'Your hand-picked recommendations: '
  END AS movie_recs,
  INITCAP(rec_movie_1) AS rec_movie_1,
  INITCAP(rec_movie_2) AS rec_movie_2,
  INITCAP(rec_movie_3) AS rec_movie_3

FROM insights_inputs;

-- The following table now has each customer's info on only one row.

DROP TABLE IF EXISTS top_movie_insights_de_collapsed;
CREATE TEMP TABLE top_movie_insights_de_collapsed AS (
-- As above, split by ranking and then join.
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

-- This will combine the two insights tables and complete the case study.

SELECT t1.*,
  t2.actor_name,
  t2.actor_insight,
  t2.actor_movie_rec_1,
  t2.actor_movie_rec_2,
  t2.actor_movie_rec_3
FROM top_movie_insights_de_collapsed t1 
  LEFT JOIN actor_insights t2 
  ON t1.customer_id = t2.customer_id;
</pre>
</details>

## [Leftover Question](#table-of-contents)

### Group Aggregate vs Window Function
Early on in the script, it is clear that a list of movies will be needed that gives a rental count for each one. I approached this task with the following code:
```sql
DROP TABLE IF EXISTS recommendations_table;
CREATE TEMP TABLE recommendations_table AS 
SELECT 
  film_id,
  title,
  category_name,
  COUNT(film_id) AS rental_count
FROM complete_joint_dataset
GROUP BY film_id, category_name, title; 

SELECT *
FROM recommendations_table
ORDER BY film_id
LIMIT 10;
```
| film_id | title            | category_name | rental_count |
|---------|------------------|---------------|--------------|
| 1       | ACADEMY DINOSAUR | Documentary   | 23           |
| 2       | ACE GOLDFINGER   | Horror        | 7            |
| 3       | ADAPTATION HOLES | Documentary   | 12           |
| 4       | AFFAIR PREJUDICE | Horror        | 23           |
| 5       | AFRICAN EGG      | Family        | 12           |
| 6       | AGENT TRUMAN     | Foreign       | 21           |
| 7       | AIRPLANE SIERRA  | Comedy        | 15           |
| 8       | AIRPORT POLLOCK  | Horror        | 18           |
| 9       | ALABAMA DEVIL    | Horror        | 12           |
| 10      | ALADDIN CALENDAR | Sports        | 23           |

Danny's method is completely different. He says that, since we need the `category_name` column, "we will need to use a window function instead of a group by to perform this step." He also mentions the importance of using `DISTINCT` here:
```sql
DROP TABLE IF EXISTS film_counts;
CREATE TEMP TABLE film_counts AS
SELECT DISTINCT
  film_id,
  title,
  category_name,
  COUNT(*) OVER (
    PARTITION BY film_id
  ) AS rental_count
FROM complete_joint_dataset;

SELECT *
FROM film_counts
ORDER BY film_id
LIMIT 10;
```
| film_id | title            | category_name | rental_count |
|---------|------------------|---------------|--------------|
| 1       | ACADEMY DINOSAUR | Documentary   | 23           |
| 2       | ACE GOLDFINGER   | Horror        | 7            |
| 3       | ADAPTATION HOLES | Documentary   | 12           |
| 4       | AFFAIR PREJUDICE | Horror        | 23           |
| 5       | AFRICAN EGG      | Family        | 12           |
| 6       | AGENT TRUMAN     | Foreign       | 21           |
| 7       | AIRPLANE SIERRA  | Comedy        | 15           |
| 8       | AIRPORT POLLOCK  | Horror        | 18           |
| 9       | ALABAMA DEVIL    | Horror        | 12           |
| 10      | ALADDIN CALENDAR | Sports        | 23           |

These look the same and they both have 958 rows. To be certain, I downloaded both my table and Danny's and compared them in Excel. They are 100% identical. (By the way, if someone knows how to do this in SQL, please tell me!)

![Screenshot 2022-05-31 122909](https://user-images.githubusercontent.com/99853599/171228497-58ab33cb-d08e-449b-b316-5486be6f631f.png)
  
My question is, are these just two different ways to get the same results, or am I missing some important concept here that explains why Danny's method is preferable?
