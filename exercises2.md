# Questions

## DATA SETS 

- **Playback Table (__table name - “streams”__):** streaming data of customers. It contains information at a stream level, with data such as minutes streamed, buffers encountered while streaming, location, device used, and the customer ID.

    - **Columns**: playback_date, stream_id, device_id, device_category, customer_id, minutes_streamed, buffer_count, customer_country

    - **Primary key** - Stream_id

    |playback_date|stream_id|device_id|device_category|customer_id|minutes_streamed|buffer_count|customer_country|
    |-------------|---------|---------|---------------|-----------|----------------|------------|----------------|
    |2017-01-01   |112344     |108    | Television |        YYTGBCSDFG      |    12                 |     0              |       US|
    |2017-01-04   |927612192  |112    | Television    |     HIGDSGKJAF      |      34                |    10              |     UK|
    |2017-04-03   |812618726  |179    | Web Browser   |HJFGJLFFHJK    |       80                |     95              |     IN|
    |2017-06-01   |921629712  |401    | Web Browser   |TEEWRJGHKJKG    |  238                |    1              |       UK|
    |2017-12-05   |4982472376 |105    | Mobile         |     KHGHJFDKLJHFF  |   10                  |     5              |       US|


- **Purchases table (__table name - "transactions"__):** contains information on transactions conducted by customers.

    - **Primary Key** - Purchase_id

    |purchase_id  |purchase_date |customer_id|
    |-------------|--------------|-----------|
    |716219621    |2017-09-26    |ASKGASKAJSA|
    |918261982    |2017-03-30    |UKHJDLKGSDG|
    |984673876    |2017-12-16    |UPFASFJSHDV|
    |836453323    |2018-09-24    |AWETUYOIUBV|

---
###QUESTIONS 

#####Easy
1. List the total streams for each device category in the last 7 months?
2. List the device category with at least 100,000 streams in the last 7 months?
3. How many customers streamed without buffers in the last 1 month?
4. How many customers have streamed for at least 30 hours in the last 30 days?
5. **What country has the highest number of streams in the last 7 days?**
6. **Find the number of customers who have streamed and purchased during the last 30 days?**
#####Medium
7. What is the monthly average minutes streamed of streams from US in 2017?
8. Which device_category has the highest coverage in terms of device_ids in US?
9. Find the number of customers who have streamed and purchased during the last 30 days?
10. **Find the number of customers who have streamed but not purchased anything in last 7 days?**
11. What device_category results in most number of purchases? (assuming customers stream and purchase on the same device_id)
12. **Find the number of customers who have used more than 1 device in the last 7 days for streaming.**
13. What is the device_category with most number of streams?
14. What are the top 3 countries in terms of buffers?
15. Are there higher number of streams on the weekdays or weekend?
16. Do customers stream longer on weekdays or weekends?
17. Which month of 2017 had the most number of hours streamed? Why do you think so?
18. What are the top 3 countries in terms of buffers?
19. How does the hours per customer metric vary between holiday season (October - December) and non-holiday season (January-September) in 2017?
20. Define a new column called stream quality based on buffer count (high stream quality if buffer count for a given stream is <3 and low stream quality otherwise). Find the number of streams by stream quality in last 7 days in US.
21. Write a query for identifying YoY change in number of streamers for US for the month of January between 2017-2018?
22. Identify the top 3 countries with most number of hours per customer?
#####Hard
23. How would you define month over month customer retention from Feb 2017? Write a query for the same.
24. How would you define binge watching (do not worry about titles) based on number of hours watched by a customer? 
    Identify the number of binge watchers in India (IN) in 2017.
25. How would you define a stream quality metric for each country? 
    Write a query for stream quality for US in the last 30 days.
26. **How would you define a stream quality metric for each country? 
    Write a query for stream quality for US in the last 30 days.**
27. **What is the top device category in the top 3 countries with most minutes streamed?**

####Answers
1. List the total streams for each device category in the last 7 months? 

```SQL
SELECT device_category, count(*) AS streams FROM streams
    WHERE playback_date BETWEEN '2017-01-01' AND '2017-07-01'
GROUP BY device_category;
```

3. How many customers streamed without buffers in the last 1 month?

```SQL
SELECT count(stream_id) AS streams FROM streams
    WHERE buffer_count = 0;
```

5. **What country has the highest number of streams in the last 7 days?**
```SQL
with tb as (
    select customer_country, count(stream_id) as stream_count,
    dense_rank() over ( order by count(stream_id) desc) as rank 
from streams where playback_date between dateadd(d ,-7, '2018-01-04') and '2018-01-04' group by customer_country
)
select customer_country from tb where rank = 1;
```

6. **Find the number of customers who have streamed and purchased during the last 30 days?**

```SQL
with 
    tb1 as (select a.customer_id from streams as a where playback_date between dateadd( d , -30 , '2017-01-04') and '2017-01-04'),
    tb2 as (select a.customer_id from transactions as a where purchase_date between dateadd( d , -30 , '2017-01-04') and '2017-01-04')
select tb1.customer_id from tb1 inner join tb2 on tb1.customer_id = tb2.customer_id;
```

7. What is the monthly average minutes streamed of streams from US in 2017?
```SQL
SELECT EXTRACT(MONTH FROM playback_date) AS month, avg(minutes_streamed) AS avgstreams FROM streams
    WHERE customer_country = 'US' AND 
          EXTRACT(YEAR FROM playback_date) = '2017'
GROUP BY month;
```

10. **Find the number of customers who have streamed but not purchased anything in last 7 days?**
```SQL
with 
    tb1 as (select a.customer_id from streams as a where playback_date between dateadd( d , -7 , '2017-01-04') and '2017-01-04'),
    tb2 as (select a.customer_id from transactions as a where purchase_date between dateadd( d , -7 , '2017-01-04') and '2017-01-04')
select tb1.customer_id from tb1 left join tb2 on tb1.customer_id = tb2.customer_id where tb2.customer_id is null;
```

12. **Find the number of customers who have used more than 1 device in the last 7 days for streaming.**

```SQL
with devices as (select customer_id, count(distinct device_id) as no_devices 
        from streams 
    group by customer_id having count(distinct device_id) > 1)
select count(*) as total_customers from devices;
```

26. **How would you define a stream quality metric for each country? 
    Write a query for stream quality for US in the last 30 days.**
    
```SQL
with country_avg as (
    select  customer_country, 
            avg(buffer_count) as buffer_count_avg
    from streams
    group by customer_country 
), 
country_quality as(
    select  b.customer_country,
            b.playback_date,
            case when b.buffer_count >= a.buffer_count_avg then 'LOW'ELSE 'HIGH' END as stream_quality_cd
    from country_avg as a inner join streams as b on a.customer_country = b.customer_country
)
select  customer_country, 
        stream_quality_cd,
        count(*) as quality_quantity_no
from country_quality
where customer_country = 'US' and playback_date between (SELECT CURRENT_DATE - INTERVAL '30 day') and CURRENT_DATE
group by customer_country, 
        stream_quality_cd;
```

27. **What is the top device category in the top 3 countries with most minutes streamed?**
```SQL
with country_vw as (
    select  customer_country,
            sum(minutes_streamed) as sm_minutes_streamed, 
            dense_rank() over( order by sum(minutes_streamed) desc) as rank_country
    from streams
    group by customer_country
),
device_vw as (
    select  customer_country,
            device_category,
            sum(minutes_streamed) as sm_minutes_streamed, 
            dense_rank() over(partition by customer_country order by sum(minutes_streamed) desc) as rank_device
    from streams
    group by customer_country , device_category
)
select a.customer_country, a.rank_country, b.device_category
from country_vw as a inner join device_vw as b on a.customer_country = b.customer_country
where   b.rank_device = 1
    and a.rank_country <=3
order by a.rank_country;
```
