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
