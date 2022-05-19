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

Not one to let this minor setback get me down, I forged ahead with questions 2 - 5. I received correct answers for #2, 4, & 5. My answer for question #3 was incorrect.
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

Still get 24% So I know it's the underlying data that's the issue. Since I was getting some answers correct, I figured it was hopefully a minor problem. My task now was to go through my code and Danny's, and try to figure out what the major differences were. Was the logic basically the same, just minor errors in the code on my part? Or was my logic completely off-base and all the code in the world couldn't help me? Only one way to find out.

## ROW_NUMBER, DENSE_RANK, RANK, FRANK?

<img align="left"  src= "https://user-images.githubusercontent.com/99853599/169186784-9b64f390-9688-4834-9e80-0ce56fb4d62f.jpg" width="350" height="250"/>
<br />
<br />
<br />
<br />
Yes?
<br clear="left"/>
<br />
