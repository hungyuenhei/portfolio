### Problem Statement

What are the top 10 shows globally over the past year, and how to do their viewing hours compare?

### Schema Setup

```sql
SELECT show_title, SUM(weekly_hours_viewed) AS
total_hours_viewed
FROM all_weeks_global
WHERE week >= DATE_TRUNC('year',CURRENT_DATE) - INTERVAL '1 year'::interval
GROUP BY show_title
ORDER BY total_hours_viewed DESC
LIMIT 10;
```
