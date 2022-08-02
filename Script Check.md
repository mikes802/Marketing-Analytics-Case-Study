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
After completing my SQL script, I went back to Danny's tutorial to see how it compared to the final script. How bad could it be?
<br />
<br />
<p align="center">
  <img src="https://user-images.githubusercontent.com/99853599/169161084-a56f1d28-7105-40b0-8bb9-38a66f6dc729.gif">
</p>
Below are my takeaways from this little moment of truth.

## Table of Contents
- [Background: Wrong Answers](#wrong-answers)
- [ROW_NUMBER, DENSE_RANK, RANK, FRANK](#row_number-dense_rank-rank-frank)
  - [Takeaway:](#rank-takeaway) Window functions and the importance of ORDER BY 
- [PERCENTILE RANK AND CUMULATIVE DISTRIBUTION](#percentile-rank-and-cumulative-distribution)
  - [Takeaway](#percentile-takeaway) 
- [The JOINs](#the-joins)
  - [Takeaway](#joins-takeaway)
- [Dealing with Duplicates](#whats-in-a-name-dealing-with-duplicates)
  - [Takeaway](#duplicates-takeaway)
- [Summary](#summary)
  - [New Code](#new-code) 
- [Leftover Questions](#leftover-questions)
  - [Group Aggregate vs Window Function](#group-aggregate-vs-window-function)

## Wrong Answers
This is a follow-up to solving the marketing analytics case study from [Danny Ma's Serious SQL course](https://www.datawithdanny.com/ "Data With Danny"). My original work can be found here: [Marketing Analytics Case Study](/README.md#marketing-analytics-case-study).

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

Let's start with an easy one... Hahaha! Just kidding. All of these are freaking mind melters.

Near the beginning of the script, we were looking for the top-ranked and second-ranked movie categories for each customer. To do this, there are some window function options we can use to rank the categories, i.e. `ROW_NUMBER`, `DENSE_RANK`, and `RANK`. Danny used `DENSE_RANK`. I didn't.

There are explanations all over the internet on how these functions differ. Basically, `ROW_NUMBER` will give you, well, row numbers. Numbers that increase incrementally irregardless of the value in that row. `DENSE_RANK` does this, too, but it will give the same "row number" for tied values. `RANK` says f* that, I'll give you the same number for ties, but numbers aren't free you know, so I'll skip a bunch after the tied values. `RANK` is confusing.

I used `RANK`.

This, surprisingly, led to an error that I didn't find until I finished nearly all of my script. There are 599 customers. However, I noticed I had 600 top-ranked movie results and 598 second-ranked movie results. I did some digging and found that this was because one customer had a tie for top-ranked movie (`rank_number` = 1). I went back to one of my first tables in the script, `top_2_ranking`, and added `category_name` as a parameter to the ORDER BY clause in the window function (see below). This eliminated the tie.
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

I used `ROW_NUMBER` from then on, thinking that the results should be the same for this and `RANK`, or even `DENSE_RANK` for that matter. Looking back, I now think that using `RANK` helped me notice a problem that I would have overlooked with `ROW_NUMBER`. If the business task was to list top-ranked movies first by rental count, then by latest rental date, then alphabetically by category name, using `ROW_NUMBER` would not have helped me discover I made a mistake, since it would have given me a rank number of both 1 and 2 no matter what, irregardless of ties. Whereas by using `RANK`, leaving out `category_name` in the window function's `ORDER BY` clause resulted in one customer linked to two catgories with `rank_number` = 1. This was an error that made itself known and I could trace back to where I made the mistake. Similarly, if I had used `DENSE_RANK` instead of `RANK`, then the table should have had at least three different movies returned for that customer, also increasing the odds that I would find this error. `DENSE_RANK` would have given me the same movies with `rank_number` = 1 as in the `RANK` result, as well as any movies that are `rank_number` = 2. `RANK` didn't do this because it skipped "row number 2" and went straight on to 3, which do not get returned with my WHERE filter.

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

Now why don't I get a #2 rank in the result? Because it doesn't exist. Since there are already two results ranked #1, `RANK` will skip the number 2 and go straight to 3 for the next value, but my WHERE clause doesn't ask for that. Let's include 3 in my WHERE clause and see if that proves to be true.
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

All this shows that I need all of the proper parameters in that `ORDER BY` clause to get two results. 

But what about `ROW_NUMBER`? That will get me two results, right? `ROW_NUMBER` doesn't care about all that parameter stuff. You want two results ranked #1 and #2? Ok, you got it:
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

Again, though, if the business task required that ties are handled by alphabatizing, then I would have messed this one up and not even realized it. Instead of getting recommendations for action movies, customer 284 would be getting recommendations for foreign movies. Furthermore, this definitely affects my calculations downstream for questions like question #3 in the quiz, regarding coverage percentage. Bad business. Let's fix it by putting `category_name` back in and moving on. When I do this, I will get the same results for customer 284 regardless of my ranking window function, `ROW_NUMBER`, `DENSE_RANK`, or `RANK`.
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

### Rank Takeaway
This was a big lesson for me. I learned a lot about these different window functions and the importance of the `ORDER BY` clause within them.

## [PERCENTILE RANK AND CUMULATIVE DISTRIBUTION](#table-of-contents)

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

I used this to get just the percentile results for the top-ranked category per customer to compare with Danny's results:
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

Let's compare my answers with Danny's:

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

This is exactly what I was expecting. Now, let's change one of the scores to match another score. I'm going to use a trick Danny used to make a duplicate of this table and then change that new table:
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
```
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
```
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
```
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

I hit upon the solution after some more tinkering. What it came down to was, once again, the `ORDER BY` clause in the window function. After I got my head wrapped around it, it made sense. I was ordering by `rental_count` and `latest_rental_date` thinking that both mattered to account for ties in `rental_count`. Upon reflection, I realized that `latest_rental_date` is not a parameter. The "percentile" is calculated by looking at the `rental_count` in decreasing order. If the value is 4, then I want to know the percent of rows with values that come before that value, i.e. all the rows with values of 5, 6, 7 and higher. However, if I add an extra parameter of `latest_rental_date`, then it will also take into consideration all of the other rows with values of 4 that come prior to the date of the current row. That is going to drive the "percentile" value up. 

By eliminating `latest_rental_date` from the window function's `ORDER BY` clause, the percentiles should now be correct.
```
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
```

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

### Percentile Takeaway

Through a lot of self-study, trial and error, and help from Danny, I was able to learn a lot about the `PERCENT_RANK` and `CUME_DIST` functions from this part of the case study.

## [The JOINs](#table-of-contents)
One of the biggest tutorials in Danny's course, besides the one on windows functions, is the tutorial regarding `JOINS`. In this case study, the `LEFT JOIN` and the `INNER JOIN` are used extensively. Picking the wrong one can lead to mistakes. 

Since I used to use Excel a lot before learning SQL, I see both `LEFT JOIN` and `INNER JOIN` as siblings of VLOOKUP. `LEFT JOIN` is basically the twin sibling of VLOOKUP. It will search the values that exist in a column on the base table (primary keys) and look for them in the target table. If it finds that key (now called a foreign key in the target table), it will pull out corresponding values in the same row of other columns in the target table and slap that on to your base table, effectively combining the two tables. If it doesn't find that key in the target table, you will get a NULL.

`INNER JOIN` is like the above, but your final table will only give you rows if the primary key was found in the base table. If it wasn't, the whole row is gone.

In the section of the script where we start looking for the most-watched actor per customer, an actor dataset is created by joining multiple tables. Danny runs some `DISTINCT` queries against his dataset and gets 955 for `unique_film_id`. I ran the same queries against my dataset and got 958. Eventually, I hit upon the idea that maybe my joins were to blame. I had used the `LEFT JOIN`, whereas Danny had used the `INNER JOIN`. I then used a trick Danny showed us in the joins tutorial to see if the rows differ when using a `LEFT JOIN` versus using an `INNER JOIN` to create the dataset:
```
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
```
| join_type  | record_count | unique_film_id |
|------------|--------------|----------------|
| inner join | 87980        | 955            |
| left join  | 88020        | 958            |

This makes it very clear that the join I used gave a different result. There is a difference of 40 in `record_count` and a difference of 3 in `unique_film_id`. But why? I thought I wanted to keep all of the rows from the base table. To dig further, I used the following to pull out any rows that had a NULL value in it after the `LEFT JOIN`:
```
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
```
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
```
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
```
| film_id | title            |
|---------|------------------|
| 257     | DRUMLINE CYCLONE |
| 323     | FLIGHT LIES      |
| 803     | SLACKER LIAISONS |

The second way:
```
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
```
| film_id |
|---------|
| 803     |
| 257     |
| 323     |

```
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

### Joins Takeaway

## [What's in a Name? Dealing with Duplicates](#table-of-contents)
There were many interesting discoveries I made when comparing my code to Danny's and tweaking it to see how it changes my results. One thing I discovered is that you can't trust that people with the same name are...the same person. Go figure.

My result for the most-watched actor for `customer_id` = 5 was Susan Davis. Danny had a different actor.

Let me cut to the chase:
```
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
```
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

This is why the Google course discussed at length cleaning the data, as does Danny. Checking for duplicates, or doppelgangers, is helpful. In my original script, I used the actors' first and last names to join tables and aggregate information, when I should have been using `actor_id`. The actors with `actor_id` 101 and 110, while they have the same name, represent different actors who are connected to different movies. This is most likely the cause of some of my wrong answers. 

### Duplicates Takeaway
The lesson I learned here is to think carefully about how I retrieve information, and to use primary/foreign keys whenever possible.

## [Summary](#table-of-contents)
### New Code
Click below to see the new, improved SQL code.
<details>
<summary>New SQL code</summary>
  
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
</pre>
</details>

## [Leftover Questions](#table-of-contents)

### Group Aggregate vs Window Function
Early on in the script, it is clear that a list of movies will be needed that gives a rental count for each one. I approached this task with the following code:
```
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
```
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
