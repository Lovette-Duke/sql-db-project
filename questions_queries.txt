
/*Question 1
We want to understand more about the movies that families are watching. 
The following categories are considered family movies: Animation, Children, Classics, 
Comedy, Family and Music.

Create a query that lists each movie, the film category it is classified in, 
and the number of times it has been rented out.*/

WITH sub AS (SELECT film.film_id, 
             film.title AS film_title,
	count(*) AS rental_count
    FROM film
    JOIN inventory 
          ON inventory.film_id = film.film_id
    LEFT JOIN rental ON rental.inventory_id = 
        inventory.inventory_id
    GROUP BY 1,2
    ORDER BY 2),

sub1 AS (SELECT film.title AS film_title,
	category.name AS category_name
    FROM film
    JOIN film_category
        ON film_category.film_id = film.film_id
    JOIN category
        ON category.category_id = 	
        film_category.category_id)
   
SELECT sub.film_title AS film_title, 
	sub1.category_name AS category_name, 
    sub.rental_count AS rental_count
from sub
JOIN sub1 ON sub1.film_title = sub.film_title
WHERE category_name = 'Animation' OR 
	category_name = 'Children' OR
    category_name = 'Classic' OR
    category_name = 'Comedy' OR
    category_name = 'Family' OR
    category_name = 'Music'
ORDER BY 2, 1



/*Question 2:
We want to find out how the two stores compare in their count of rental orders 
during every month for all the years we have data for. 

Write a query that returns the store ID for the store, the year and month and the 
number of rental orders each store has fulfilled for that month. Your table should 
include a column for each of the following: year, month, store ID and count of rental 
orders fulfilled during that month.*/

SELECT 
	DATE_PART('month', rental.rental_date) 
    AS rental_month,
    DATE_PART('year', rental.rental_date) 
    AS rental_year,
    store.store_id,
    COUNT(*) AS count_rentals
FROM rental
JOIN staff
  ON rental.staff_id = staff.staff_id
JOIN store
ON store.store_id = staff.store_id
GROUP BY 1,2,3
ORDER BY count_rentals DESC




/*Question 3
We would like to know who were our top 10 paying customers, 
how many payments they made on a monthly basis during 2007, 
and what was the amount of the monthly payments. 
Can you write a query to capture the customer name, month and year of payment, 
and total payment amount for each month by these top 10 paying customers?*/

WITH sub AS(SELECT customer_id, 
    SUM(amount) AS pay_amount,
    COUNT(*) AS pay_count
    FROM payment
    GROUP BY 1
    ORDER BY 3 DESC
    LIMIT 10),

	sub1 AS (SELECT
             payment.customer_id AS customer_id,
    DATE_TRUNC('month', payment.payment_date) 
    AS payment_month,
    CONCAT(customer.first_name, 
    ' ', customer.last_name) AS full_name,
    SUM(amount) AS pay_amount,
    COUNT(*) AS pay_count_permonth
    FROM payment
    JOIN customer
    ON customer.customer_id = payment.customer_id 
    GROUP BY 1, 2, 3)
 

SELECT sub1.payment_month AS payment_month, sub1.full_name AS full_name,
	sub1.pay_amount AS pay_amount, sub1.pay_count_permonth AS pay_count_permonth
FROM sub1
JOIN sub
ON sub.customer_id = sub1.customer_id
ORDER BY 2, 1




/*Question 4
Finally, for each of these top 10 paying customers, I would like to find out the 
difference across their monthly payments during 2007. 
Please go ahead and write a query to compare the payment amounts in each successive month. 
Repeat this for each of these 10 paying customers. 
Also, it will be tremendously helpful if you can identify the customer name who 
paid the most difference in terms of payments.*/

WITH sub AS(SELECT customer_id, 
    SUM(amount) AS pay_amount,
    COUNT(*) AS pay_count
    FROM payment
    GROUP BY 1
    ORDER BY 3 DESC
    LIMIT 10),

	sub1 AS (SELECT
             payment.customer_id AS customer_id,
    DATE_TRUNC('month', payment.payment_date) 
    AS payment_month,
    CONCAT(customer.first_name, 
    ' ', customer.last_name) AS full_name,
    SUM(amount) AS pay_amount,
    COUNT(*) AS pay_count_permonth
    FROM payment
    JOIN customer
    ON customer.customer_id = payment.customer_id 
    GROUP BY 1, 2, 3),

sub2 AS	(SELECT sub1.payment_month 
             AS payment_month, 
        sub1.full_name AS full_name,
        sub1.pay_amount AS pay_amount, 
        sub1.pay_count_permonth 
         AS pay_count_permonth
    FROM sub1
    JOIN sub
    ON sub.customer_id = sub1.customer_id
    ORDER BY 2, 1)


SELECT payment_month, full_name, pay_amount,
	(LEAD(pay_amount) OVER (ORDER BY full_name)
     - pay_amount) AS monthly_pay_diff
    
FROM sub2
ORDER BY 1, 4 DESC