
## Marketing Analytics on E-commerce dataset using SQL

### **Analyzing Website Traffic Sources**

**Objective**  
Traffic source analysis is about understanding where your customers are coming from and which channels are driving the highest quality traffic

1)The leadership team would like to know where the bulk of the website sessions are coming by UTM source , campaign and referring domain 

```sql
SELECT utm_source,
       utm_campaign,
       http_referer,
       Count(DISTINCT website_session_id) AS 'Sessions'
FROM   website_sessions
WHERE  created_at <= '2012-04-12'
GROUP  BY utm_source,
          utm_campaign,
          http_referer
ORDER  BY sessions DESC;
```
**Output**  
![Screenshot_10](https://user-images.githubusercontent.com/113862057/192175252-531acdb8-e749-497c-9ed1-8b9cfa6c76db.png)

**Insights & Actions**
- This shows that 97.4% of the traffic source is from gsearch nonbrand.  

2)The leadership team would like to analyze if those sessions are driving sales(Conversion Rate from session to order).

```sql
SELECT Count(DISTINCT ws.website_session_id) AS 'Sessions',
       Count(DISTINCT o.website_session_id)  AS 'Orders',
       Count(DISTINCT o.website_session_id) / Count(DISTINCT ws.website_session_id) AS 'CVR'
FROM   website_sessions ws
       LEFT JOIN orders o
              ON ws.website_session_id = o.website_session_id
WHERE  utm_source = 'gsearch'
       AND utm_campaign = 'nonbrand'
       AND ws.created_at < '2012-04-14'; 

```

**Output**  
![Screenshot_1](https://user-images.githubusercontent.com/113862057/192175590-3b5b22db-8803-46b2-8ec4-8710c2d96915.png)

**Insights & Actions**
- The expected CVR is atleast 4% to make numbers work, based on what the company is paying for clicks. 
- The leadership team has decided to dial down the search bids a bit

3)Post the bid down, the marketing director would like to see the gsearch nonbrand trended session volume, by week, to see if the bid changes have caused volume to drop

```sql
SELECT Date(Min(created_at)),
       Count(DISTINCT website_session_id)
FROM   website_sessions ws
WHERE  utm_source = 'gsearch'
       AND utm_campaign = 'nonbrand'
       AND ws.created_at < '2012-05-10'
GROUP  BY Week(created_at); 
```

**Output**  
![Screenshot_2](https://user-images.githubusercontent.com/113862057/192181958-96a264ee-7879-491f-b3ad-2854c4e1f6f3.png)

**Insights & Actions**
- The gsearch nonbrand is sensitive to bid changes. 
- The executive team wants maximum volume, and want to drill into device level performance for gsearch nonbrand

4)The marketing director would like a conversion rate analysis, from session to order by device type. 

```sql
SELECT device_type,
       Count(DISTINCT ws.website_session_id) AS 'Sessions',
       Count(DISTINCT o.website_session_id)  AS 'Orders',
       Count(DISTINCT o.website_session_id) / Count(DISTINCT ws.website_session_id) AS 'CVR'
FROM   website_sessions ws
       LEFT JOIN orders o
              ON ws.website_session_id = o.website_session_id
WHERE  utm_source = 'gsearch'
       AND utm_campaign = 'nonbrand'
       AND ws.created_at < '2012-05-11'
GROUP  BY device_type; 
```

**Output**  
![Screenshot_3](https://user-images.githubusercontent.com/113862057/192182668-38e19a1d-ad3e-4608-8df6-e291d8600bec.png)

**Insights & Actions**
- The desktop performance for gsearch nonbrand is better than on mobile. 
- The company opts to increase the bids on desktop

5)The marketing team wants to do a post analysis to gauge the results of the bid up 

```sql
SELECT Date(Min(created_at)) AS 'Week_Start_date',
       Count(DISTINCT CASE
                        WHEN device_type = 'desktop' THEN website_session_id
                      end)   AS dtop_sessions,
       Count(DISTINCT CASE
                        WHEN device_type = 'mobile' THEN website_session_id
                      end)   AS mob_sessions
FROM   website_sessions ws
WHERE  utm_source = 'gsearch'
       AND utm_campaign = 'nonbrand'
       AND ws.created_at >= '2012-04-15'
       AND ws.created_at < '2012-06-09'
GROUP  BY Week(created_at); 
```

**Output**  
![Screenshot_4](https://user-images.githubusercontent.com/113862057/192183278-23feac7f-5c8a-4951-ac3f-eb1405f25953.png)


**Insights & Actions**
Desktop performance is looking strong thanks to the bid changes the company made based on the previous conversion analysis.


### **Analyzing Website Traffic Sources**

**Objective**  
Website content analysis is about understanding which pages are seen the most by users, to identify where to focus on improving the business

1)The website manager has a data request to pull the most viewed website pages ranked by session volume

```sql
SELECT pageview_url,
       Count(DISTINCT website_session_id) AS 'sessions'
FROM   website_pageviews
WHERE  created_at < '2012-06-09'
GROUP  BY pageview_url
ORDER  BY Count(DISTINCT website_session_id) DESC; 
```

**Output**  
![Screenshot_5](https://user-images.githubusercontent.com/113862057/192184728-af8e9ca4-cbb3-4531-b00e-5f531880e6c2.png)

**Insights & Actions**
The homepage, the products page,and the Mr. Fuzzy page get the bulk of the traffic.

2)The website manager wants to confirm where the users are hitting the site. She wants to rank all entry pages based on entry volume.

```sql
WITH top_entry_ids
     AS (SELECT website_session_id,
                Min(website_pageview_id) AS 'min_pageview_id'
         FROM   website_pageviews wp
         WHERE  created_at < '2012-06-12'
         GROUP  BY website_session_id),
     pageview_url_name
     AS (SELECT t.website_session_id,
                pageview_url
         FROM   top_entry_ids t
                JOIN website_pageviews wp
                  ON t.min_pageview_id = wp.website_pageview_id)
SELECT pageview_url,
       Count(DISTINCT website_session_id)
FROM   pageview_url_name; 
```

**Output**   
![Screenshot_6](https://user-images.githubusercontent.com/113862057/192185837-e68e34ac-24f5-499c-9f3a-888f1fd3d9a0.png)

**Insights & Actions**
All  traffic all comes in through the homepage 
There has to be improvements made to attract traffic sources from alternate entry pages


3)The website manager wants to see bounce rates for traffic landing on the homepage- Sessions ,Bounced Sessions , and Bounce Rate.

```sql
WITH total_sessions
     AS (SELECT website_session_id
         FROM   website_pageviews
         WHERE  created_at < '2012-06-14'),
     bounced_sessions
     AS (SELECT website_session_id,
                Count(DISTINCT website_pageview_id)
         FROM   website_pageviews
         WHERE  created_at < '2012-06-14'
         GROUP  BY website_session_id
         HAVING Count(DISTINCT website_pageview_id) = 1)
SELECT Count(DISTINCT ts.website_session_id) AS 'Sessions',
       Count(DISTINCT bs.website_session_id) AS 'Bounced Sessions',
       Count(DISTINCT bs.website_session_id) / Count(DISTINCT ts.website_session_id) AS 'Bounce_Rate'
FROM   total_sessions ts
       LEFT JOIN bounced_sessions bs
              ON ts.website_session_id = bs.website_session_id; 
```

**Output**  
![Screenshot_7](https://user-images.githubusercontent.com/113862057/192186393-638263a1-3482-410c-8923-b97e63271c8a.png)


**Insights & Actions**
There is a 60% bounce rate which is very high for a paid search 
The website team could work on a custom landing page for search, and set up an experiment to see if the new page does better. 

4)Based on the bounce rate analysis, the website team has created a new custom landing page ( (/lander 1 ) in a 50/50 test against the homepage ((/home for our gsearch nonbrand traffic.The team would like to view bounce rates for the two groups to evaluate the new page baseb on a common time frame.

```sql
WITH total_sessions
     AS (SELECT pageview_url,
                Count(DISTINCT ws.website_session_id) AS total_sessions
         FROM   website_sessions ws
                LEFT JOIN website_pageviews wp
                       ON ws.website_session_id = wp.website_session_id
         WHERE  ( ws.created_at >= '2012-06-19'
                  AND ws.created_at < '2012-07-28' )
                AND utm_source = 'gsearch'
                AND utm_campaign = 'nonbrand'
                AND ( pageview_url = '/home'
                       OR pageview_url = '/lander-1' )
         GROUP  BY pageview_url),
     bounced_sessions_tbl
     AS (SELECT wp.website_session_id,
                Count(DISTINCT website_pageview_id) AS 'pageview_id'
         FROM   website_pageviews wp
                JOIN website_sessions ws
                  ON ws.website_session_id = wp.website_session_id
         WHERE  ( ws.created_at >= '2012-06-19'
                  AND ws.created_at < '2012-07-28' )
                AND utm_source = 'gsearch'
                AND utm_campaign = 'nonbrand'
         GROUP  BY ws.website_session_id
         HAVING Count(DISTINCT website_pageview_id) = 1),
     bounced_sessions
     AS (SELECT pageview_url                         AS 'landing_page',
                Count(DISTINCT s.website_session_id) AS 'bounced_sessions'
         FROM   bounced_sessions_tbl s
                JOIN website_pageviews p
                  ON p.website_session_id = s.website_session_id
         GROUP  BY pageview_url)
SELECT bs.landing_page,
       ts.total_sessions,
       bs.bounced_sessions,
       bs.bounced_sessions / ts.total_sessions AS 'Bounce_rate'
FROM   bounced_sessions bs
       JOIN total_sessions ts
         ON bs.landing_page = ts.pageview_url; 
```

**Output**  
![Screenshot_8](https://user-images.githubusercontent.com/113862057/192187062-acb7c7b4-fd27-4543-8ccf-e38ee9423801.png)


**Insights & Actions**
- The custom lander has a lower bounce rate success
- As a next step, the business can get the campaigns updated so that all nonbrand paid traffic is pointing to the new page. 


5)The website manager has a request to pull the volume of paid search nonbrand traffic landing on /home and /lander 1, trended weekly since June 1st and also pull the  overall paid search bounce rate trended weekly ? 

```sql
WITH sessions_count
     AS (SELECT Min(Date(wp.created_at)) AS week_start_date,
                Count(CASE
                        WHEN pageview_url = '/home' THEN wp.website_session_id
                      END)               AS 'home_sessions',
                Count(CASE
                        WHEN pageview_url = '/lander-1' THEN
                        wp.website_session_id
                      END)               AS 'lander_sessions'
         FROM   website_sessions ws
                LEFT JOIN website_pageviews wp
                       ON ws.website_session_id = wp.website_session_id
         WHERE  utm_source = 'gsearch'
                AND utm_campaign = 'nonbrand'
                AND ( wp.created_at >= '2012-06-01'
                      AND wp.created_at < '2012-08-31' )
         GROUP  BY Week(wp.created_at)),
     total_sessions
     AS (SELECT Min(Date(wp.created_at))              AS week_start_date,
                Count(DISTINCT wp.website_session_id) AS 'total_sessions'
         FROM   website_sessions ws
                LEFT JOIN website_pageviews wp
                       ON ws.website_session_id = wp.website_session_id
         WHERE  utm_source = 'gsearch'
                AND utm_campaign = 'nonbrand'
                AND ( wp.created_at >= '2012-06-01'
                      AND wp.created_at < '2012-08-31' )
         GROUP  BY Week(wp.created_at)),
     bounced_sessions_tbl
     AS (SELECT Min(Date(wp.created_at)) AS week_start_date,
                wp.website_session_id
         FROM   website_sessions ws
                LEFT JOIN website_pageviews wp
                       ON ws.website_session_id = wp.website_session_id
         WHERE  utm_source = 'gsearch'
                AND utm_campaign = 'nonbrand'
                AND ( wp.created_at >= '2012-06-01'
                      AND wp.created_at < '2012-08-31' )
         GROUP  BY Date(wp.created_at),
                   wp.website_session_id
         HAVING Count(DISTINCT website_pageview_id) = 1),
     bounced_sessions
     AS (SELECT Min(week_start_date)               AS week_start_date,
                Count(DISTINCT website_session_id) AS 'Bounced_sessions'
         FROM   bounced_sessions_tbl
         GROUP  BY Week(week_start_date)),
     bounce_rate
     AS (SELECT bs.week_start_date,
                bounced_sessions / total_sessions AS bounce_rate
         FROM   bounced_sessions bs
                JOIN total_sessions ts
                  ON ts.week_start_date = bs.week_start_date)
SELECT br.week_start_date,
       bounce_rate,
       home_sessions,
       lander_sessions
FROM   bounce_rate br
       JOIN sessions_count sc
         ON br. week_start_date = sc.week_start_date; 
```


**Output**  
![Screenshot_9](https://user-images.githubusercontent.com/113862057/192188098-90fe0901-27cf-4c5b-b264-41126cb52ce1.png)


**Insights & Actions**
The overall bounce rate has come down over time


### **Conversion Funnel Analysis**

**Objective**  
Conversion funnel analysis is about understanding and optimizing each step of your user’s experience on their journey toward purchasing your products

1)The website manager would like to understand where the gsearch visitors are lost between the new /lander 1 page and placing an order and would like to see a full conversion funnel, analyzing how many customers make it to each step,starting with /lander 1 and  all the way to the thank you page,since August 5th

```sql
SELECT Count(DISTINCT website_session_id) AS 'sessions',
       Count(CASE WHEN to_products=1 THEN website_session_id ELSE NULL end ) to_products,
       Count(CASE WHEN to_mrfuzzy=1 THEN website_session_id ELSE NULL end ) to_mrfuzzy,
       Count(CASE WHEN to_cart=1 THEN website_session_id ELSE NULL end ) to_cart,
       Count(CASE WHEN to_shipping=1 THEN website_session_id ELSE NULL end ) to_shipping,
       Count(CASE WHEN to_billing=1 THEN website_session_id ELSE NULL end ) to_billing,
       Count(CASE WHEN to_thankyou=1 THEN website_session_id ELSE NULL end ) to_thankyou
FROM   (
                SELECT   website_session_id,
                         pageview_url,
                         CASE WHEN pageview_url='/products' THEN 1 ELSE 0 end AS to_products,
                         CASE WHEN pageview_url='/the-original-mr-fuzzy' THEN 1 ELSE 0 end AS to_mrfuzzy,
                         CASE WHEN pageview_url='/cart' THEN 1 ELSE 0 end AS to_cart,
                         CASE WHEN pageview_url='/shipping' THEN 1 ELSE 0 end AS to_shipping,
                         CASE WHEN pageview_url='/billing' THEN 1  ELSE 0 end AS to_billing,
                         CASE WHEN pageview_url='/thank-you-for-your-order' THEN 1 ELSE 0 end AS to_thankyou
                FROM     (
                                SELECT website_session_id,
                                       pageview_url
                                FROM   website_pageviews
                                WHERE  website_session_id IN
                                                              (
                                                              SELECT DISTINCT wp.website_session_id
                                                              FROM            website_sessions ws
                                                              LEFT JOIN       website_pageviews wp
                                                              ON              ws.website_session_id=wp.website_session_id
                                                              WHERE           utm_source='gsearch'
                                                              AND             utm_campaign='nonbrand'
                                                              AND             (
                                                                                              wp.created_at>='2012-08-05'
                                                                              AND             wp.created_at<'2012-09-05')
                                                              AND             pageview_url='/lander-1' ))a
                GROUP BY website_session_id,
                         pageview_url)b;
```

**Click Rates**  

```sql
WITH final_output
     AS (SELECT Count(DISTINCT website_session_id) AS 'sessions',
                Count(CASE WHEN to_products = 1 THEN website_session_id ELSE NULL END) to_products,
                Count(CASE WHEN to_mrfuzzy = 1 THEN website_session_id  ELSE NULL END) to_mrfuzzy,
                Count(CASE WHEN to_cart = 1 THEN website_session_id ELSE NULL END) to_cart,
                Count(CASE WHEN to_shipping = 1 THEN website_session_id ELSE NULL END) to_shipping,
                Count(CASE WHEN to_billing = 1 THEN website_session_id ELSE NULL END) to_billing,
                Count(CASE WHEN to_thankyou = 1 THEN website_session_id ELSE NULL END) to_thankyou
         FROM   (SELECT website_session_id,
                        pageview_url,
                        CASE WHEN pageview_url = '/products' THEN 1 ELSE 0 END AS to_products,
                        CASE WHEN pageview_url = '/the-original-mr-fuzzy' THEN 1 ELSE 0 END AS to_mrfuzzy,
                        CASE WHEN pageview_url = '/cart' THEN 1 ELSE 0 END AS to_cart,
                        CASE WHEN pageview_url = '/shipping' THEN 1 ELSE 0 END AS to_shipping,
                        CASE WHEN pageview_url = '/billing' THEN 1  ELSE 0 END AS to_billing,
                        CASE WHEN pageview_url = '/thank-you-for-your-order' THEN 1 ELSE 0 END AS to_thankyou
                 FROM   (SELECT website_session_id,
                                pageview_url
                         FROM   website_pageviews
                         WHERE  website_session_id IN
                                (SELECT DISTINCT wp.website_session_id
                                 FROM   website_sessions ws
                                        LEFT JOIN website_pageviews wp
                                               ON
                                ws.website_session_id = wp.website_session_id
                                                       WHERE
                                utm_source = 'gsearch'
                                AND
                                utm_campaign = 'nonbrand'
                                                              AND (
                                wp.created_at >= '2012-08-05'
                                AND
                                wp.created_at < '2012-09-05' )
                                    AND
                                pageview_url = '/lander-1'))a
                 GROUP  BY website_session_id,
                           pageview_url)b)
SELECT to_products / sessions   AS lander_click_rt,
       to_mrfuzzy / to_products AS products_click_rt,
       to_cart / to_mrfuzzy     AS mrfuzzy_click_rt,
       to_shipping / to_cart    AS cart_click_rt,
       to_billing / to_shipping AS shipping_click_rt,
       to_thankyou / to_billing AS billing_click_rt
FROM   final_output; 
```

 **Output**                         
![Screenshot_11](https://user-images.githubusercontent.com/113862057/192189183-760753f6-d918-4e1f-8886-75c9cdad9ef3.png)
![Screenshot_13](https://user-images.githubusercontent.com/113862057/192190442-e7add925-c100-4861-b104-4e596fe80dc2.png)

**Insights & Actions**  
The lander, Mr. Fuzzy page ,and the billing page have the lowest click rates.


