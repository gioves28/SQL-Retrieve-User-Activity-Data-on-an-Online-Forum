# SQL-Retrieve-User-Activity-Data-on-an-Online-Forum

This project and the data used was part of a case study . It focuses on examining ChatData, a popular network of question-and-answer (Q&A) websites in the fields of data analytics, data science and artificial intelligence. They help budding data analysts stay up-to-date with current innovations, find answers to burning questions, and stay active in the data community!

### Background

Oliver, our social media manager. He is putting together the roadmap for next year and has some specific questions about user activity (interaction) on our sites. 


### Problem Statement

Olivers wants to learn how ChatData sites are being used in the real world to understand which features are useful to the users and what additional features might be worth introducing.

### Entity Relationship Diagram

![Escalona_Giovanni_1_ERD_022023](https://github.com/gioves28/SQL-Retrieve-User-Activity-Data-on-an-Online-Forum/assets/131261225/6966f1ef-7ed2-4b1e-8715-f3b5c8005ee5)


### Skills Applied
- Manage to clean and format the data into three separate CSV files attached: posts, comments and users.
- Prepare an entity relationship diagram (ERD).
- Database SQLite
- Window Functions
- CTEs
- Aggregations
- JOINs
- Write scripts to generate basic reports that can be run every period

### Questions Explored

1. How many posts have 0 comments?
2. How many posts have 1 comments?
3. How many posts have 2 comments or more?
4. Find the 5 posts with the highest viewcount
5. Find the top 5 posts with the highest scores
6. What are the 5 most frequent scores on posts?
7. How many posts have the keyword "data" in their tags?
8. What are the 5 most frequent commentcount for posts?
9. How many posts have an accepted answer?
10. What is the average reputation of table users?
11. What are the min and max reputation of users?
12. What is the length of the body of 5 most viewed posts?
13. How many different locations are there in the users table?
14. What are the top 5 locations of users?
15. Rank the days of the week from highest to lowest in terms of the volume of ViewCount as a percentage.
16. How many posts have been created by a user that has a filled out the "AboutMe" section?
17. Considering only the users with an "AboutMe," how many posts are there per user?
18. Not taking into account the commentcount field in the table posts, what are the Top 10 posts in terms of number of comments?
19. What are the Top 10 posts which have the highest cummulative (post score + comment score) score? 
20. Who are the top 10 users who comment the most? 
21. Who are the top 10 users who post the most? 

### Some interesting queries

**Q15 - Rank the days of the week from highest to lowest in terms of the volume of ViewCount as a percentage**

```sql
 select case cast (strftime('%w', CreationDate) as integer)
  when 0 then 'Sunday'
  when 1 then 'Monday'
  when 2 then 'Tuesday'
  when 3 then 'Wednesday'
  when 4 then 'Thursday'
  when 5 then 'Friday'
  else 'Saturday' end as Day, ROUND(SUM(ViewCount)*100.0/(select SUM(ViewCount) from posts),2) as SumViewCountPercent
    FROM posts  
    GROUP BY Day
    ORDER BY 2 DESC
```


**Q10 - In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

```sql
SELECT s.customer_id, SUM(
    CASE 
        WHEN s.order_date BETWEEN mb.join_date AND DATEADD(day, 7, mb.join_date) THEN m.price*20
        WHEN m.product_name = 'sushi' THEN m.price*20 
        ELSE m.price*10 
    END) AS total_points
FROM dbo.sales s
JOIN dbo.menu m ON s.product_id = m.product_id
LEFT JOIN dbo.members mb ON s.customer_id = mb.customer_id
WHERE s.customer_id IN ('A', 'B') AND s.order_date <= '2021-01-31'
--WHERE s.customer_id = mb.customer_id AND s.order_date <= '2021-01-31'
GROUP BY s.customer_id;
```


**Bonus Q2 - Danny also requires further information about the ranking of products. he purposely does not need the ranking of non member purchases so he expects NULL ranking values for customers who are not yet part of the loyalty program.**

```sql
WITH customers_data AS (
  SELECT 
    s.customer_id, s.order_date,  m.product_name, m.price,
    CASE
      WHEN s.order_date < mb.join_date THEN 'N'
      WHEN s.order_date >= mb.join_date THEN 'Y'
      ELSE 'N' END AS member
  FROM sales s
  LEFT JOIN members mb
    ON s.customer_id = mb.customer_id
  JOIN menu m
    ON s.product_id = m.product_id
)
SELECT 
  *, 
  CASE
    WHEN member = 'N' THEN NULL
    ELSE RANK () OVER(
      PARTITION BY customer_id, member
      ORDER BY order_date) END AS ranking
FROM customers_data
ORDER BY customer_id, order_date;
```
   
### Insights

- Customer B is the most frequent visitor with 6 visits in Jan 2021.
- Danny’s Diner’s most popular item is ramen, followed by curry and sushi.
- Customer A loves ramen, Customer C loves only ramen whereas Customer B seems to enjoy sushi, curry and ramen equally.
- The last item ordered by Customers A and B before they became members are sushi and curry. Does it mean both of these items are the deciding factor?
