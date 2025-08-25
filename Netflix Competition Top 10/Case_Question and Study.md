# Problem 1 Statement

Demonstrate the total watching hours to the top_10 shows on each category globally.

### Schema Setup

```sql
WITH cte AS (
    SELECT
        CONCAT(
            EXTRACT(YEAR FROM TO_TIMESTAMP(week, 'YYYY-MM-DD"T"HH24:MI:SS.FF3')),
            'Q',
            EXTRACT(QUARTER FROM TO_TIMESTAMP(week, 'YYYY-MM-DD"T"HH24:MI:SS.FF3'))
        ) AS quarter,
	ROW_NUMBER() OVER (PARTITION BY show_title ORDER BY weekly_hours_viewed DESC) as rn,
        *
    FROM all_weeks_global
)

--- Quarterly Performance in General

SELECT 
    quarter,
    SUM(weekly_hours_viewed) AS total_weekly_hours_viewed
FROM cte
GROUP BY quarter, category
ORDER BY total_weekly_hours_viewed DESC;

--- Quarterly Performance by Top10 shows Category
  
SELECT 
    quarter,
    category, 
    SUM(weekly_hours_viewed) AS total_weekly_hours_viewed
FROM cte
GROUP BY quarter, category
ORDER BY quarter;

--- Which 10 shows have the highest viewed hours among that period (highest viewed hours: 2021Q4)

SELECT show_title, quarter, weekly_hours_viewed
FROM cte
WHERE TO_TIMESTAMP(week, 'YYYY-MM-DD"T"HH24:MI:SS.FF3') BETWEEN '2021-10-01' AND '2021-12-31' ##result that 2021Q4 has the most viewed hours from above data code
	AND rn = 1 
ORDER BY weekly_hours_viewed DESC
LIMIT 10;
```

### Insights
2021Q4 reached the highest viewed hours globally among the top10 Netflix shows and TV (non-english) obtains the highest watching hours among the 4 categories. The trend can be explained by the launches of a Netflix series called Squid Game. It premiered in late September 2021 and hit numerous records and become one of the most successful k-drama series in the world. To highlight, My Name (ranked in 10th) also is thriller action korean content. 

<img width="1149" height="243" alt="Image" src="https://github.com/user-attachments/assets/8c5a5699-a077-4da3-b114-e5b47e66aaca" />

<img width="896" height="317" alt="Image" src="https://github.com/user-attachments/assets/2ea852bc-bf72-4331-9f24-0cb82e4c2919" />

Among the Category, English TV shows is always the top preference among the netflix audience and the viewed hours is doubled compared with English Film. The practical and creative factors on launching the English TV shows in series over the media industry over decades allows to extend storytelling and audiences connections that encourage certain amount of fans to return and watch the shows.

Non-English show is not the dominance in global TV & Movie industry because of the language barrier and well-established network in the English market, watching hours is lowered than the English shows in general. However in 2021Q4, Top 10 ranked non-English TV shows transcend 4 billions total watching hours. One of the reasons behind could be the rapid industry development of Korean and Japanese Drama and Animations with high production quality. Parallelly, culture of these two countries influences and stimulates people's curiosity and loyalty to the content.

<img width="942" height="631" alt="Image" src="https://github.com/user-attachments/assets/6289d612-4f4a-4f76-8507-8bc94bf46322" />

<img width="980" height="631" alt="Image" src="https://github.com/user-attachments/assets/68fb5b05-4229-46fc-b1ad-8fcbfa2a3663" />


# Problem 2 Statement

In general, how Netflix content performed (by category) globally?

### Schema Setup

```sql
SELECT
    category,
    SUM(weekly_hours_viewed) AS total_weekly_hours_viewed,
    AVG(cumulative_weeks_in_top_10) AS total_cumulative_weeks_in_top_10
FROM all_weeks_global
GROUP BY category, type
ORDER BY total_weekly_hours_viewed DESC;
```

### Insights

<img width="930" height="619" alt="Image" src="https://github.com/user-attachments/assets/95f9f5bb-a755-41fd-8343-5bf9ac47115c" />

Both English and Non-English TV shows had a significantly higher total watched hours and longer average cumulative weeks in top 10 compared to the films counterpart, which indicated that TV series has greater viewership and sustained popularity.

While English TV Top 10 shows content was held the highest hours viewed among all categories, the Non-English TV shows content outstood and stayed on top 10 for 5 weeks on average, meaning that it had a more enduring presence to the audiences.

The viewed hours of English vs. Non-English films was doubled and it was a bigger gap compared with the average culmulative weeks in top 10 chart. The suggested that English films had a high viewership peak yet did not necessarily have the same long-term chart staying power.


# Problem 3 Statement

Which 10 Shows are most popular on Netflix of all time?

### Schema Setup

```sql
WITH global_all_weeks AS (
	SELECT TO_TIMESTAMP(week, 'YYYY-MM-DD"T"HH24:MI:SS.FF3') AS time, 
	*
	FROM all_weeks_global
)

SELECT
	RANK() OVER(ORDER BY COUNT(*) DESC) AS rank,
	CASE 
		WHEN season_title = 'N/A' THEN show_title
		ELSE season_title
		END AS show_title,
	SUM(weekly_hours_viewed) AS total_weekly_hours_viewed,
	TO_CHAR(MIN(time), 'YYYYMM') AS first_record_month,
	COUNT(*) AS total_weeks_in_top_10
FROM global_all_weeks
GROUP BY show_title, season_title
ORDER BY total_weeks_in_top_10 DESC
LIMIT 10;
```

### Insights

<img width="1236" height="604" alt="Image" src="https://github.com/user-attachments/assets/f0227763-a53d-4b1f-923e-cf55d75afeac" />

As the data demonstrated above, Squid Game undoubtedly ranked number 1 in the total hours viewed as of its unique theme, the series reflected social phenomenons (like economic inequality and social anxiety and desperation) through childhood games, which resonating with the audiences. In Problem Statement 1, English TV contents reached the highest viewed hours among other 3 categories except 2021Q4, one of the reasons is the release of first season of Squid Game. The social echo and discussion was unprecedented to Korea TV series. It is also the only show categorized as Non-english TV shows among the top 10 most popular shows on Netflix.

However, total hours viewed is not a must to be correlated with the longevity on top chart. While Squid Game has the highest viewing hours, the coloured bar shows a lighter red compared to 'Yo soy Betty, la fea' and 'Café con aroma de mujer'. It implies that the 2 shows stayed longer weeks in the top 10 even they had fewer total hours viewed. 


# Problem 4 Statement

How do the cumulative weeks in the top 10 correlate with the total hours viewed in the first 28 days for the most popular TV shows?

Note: Only TV shows category can be analyzed due to no data record provided for other categories.

### Schema Setup

```sql
SELECT a.show_title, 
	MAX(a.cumulative_weeks_in_top_10) AS cumulative_weeks_in_top_10,
	b.hours_viewed_first_28_days
FROM all_weeks_global AS a
INNER JOIN most_popular AS b 
	ON a.show_title = b.show_title
--- only most_popular.csv (34 records) include data of viewed hours in first 28 days, and show title is the only matching column in both tables
WHERE a.cumulative_weeks_in_top_10 IS NOT NULL 
	AND b.hours_viewed_first_28_days IS NOT NULL
GROUP BY a.show_title, b.hours_viewed_first_28_days;
```

# Insights

<img width="871" height="609" alt="Image" src="https://github.com/user-attachments/assets/e21a7f50-e15a-4558-b7a4-b11487702ac6" />

In the dataset, 34 sample shows selected and resulted that there is moderate relationship between cumulative weeks in top 10 vs hours viewed in first 28 days. For example, Wednesday, a famous comedy horror series, had more than 700 million viewed hours on first 28 days, however the series stayed on chart only for 2 weeks. On the other hand, Café con aroma de mujer (2021 version, new adaption to same series first broadcasted in 1994), a Spanish romance series, had comparatively lower audience engagement yet entered top 10 chart for 28 weeks consecutively. 

The above result incidated that other factors influenced to let the shows stick in top ranking. For Wednesday, it's well-known and has built a huge fan communities around the globe, curiosity and emotional connections can be created in a short period of time. However, Non-english TV show like Café con aroma de mujer, even language barrier limit amount of viewers due to audiences access restriction and misunderstanding to plot, canonical story with high-quality reproduction and attractive casting can catch eyes from local audiences. TV series can hit the top ranking and stay for an extended period.

# Recommendation for Netflix
The massive success of non-English content like Squid Game demonstrates the potential to dominate the global market. Also, Spanish drama Café con aroma de mujer illustrates high-quality re-production of classical series can attract audience and have a longer staying power on top 10 chart copmared with the English titles. Netflix should prioritize investing in compelling and brand-new non-English content to capitalize on this growing trend.

Although some shows had a lower viewership hours, like 'Yo soy Betty, la fea', they stayed on top 10 charts for significant number of weeks. This 'long-tail' content represents a valuable asset for audience retention. To endure show's presence to both new and existing viewers, implementing robust and localized themed events are needed for viewers engagement and subscriber value. 
