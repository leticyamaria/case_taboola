# Question 1
## For every publisher that has mobile traffic, find that publisher’s total number of auctions across all platforms.

```sql 
 select 
  b1.publisher_id,
  sum(b1.auctions) as total_auctions
from bidops_taboola b1
where b1.publisher_id in (
select 
  b2.publisher_id
from bidops_taboola b2
where platform = 'Mobile')
group by b1.publisher_id
```
| publisher_id | total_auctions |
| ------------ | -------------- |
| 7019         | 31887878       |
| 3406         | 75385835       |


**ANSWER**  
Total: **107.273.713**

---


## For publisher_id 3406, calculate the change in gross revenue as a percentage between the 2 variants. show the results broken down by platform.

```sql
WITH base AS (
  SELECT
    b.platform,
    SUM(CASE WHEN b.variant = 'test'    THEN b.gross_revenue ELSE 0 END) AS gr_test,
    SUM(CASE WHEN b.variant = 'control' THEN b.gross_revenue ELSE 0 END) AS gr_control
  FROM bidops_taboola b
  WHERE b.publisher_id = '3406'
  GROUP BY b.platform
)
SELECT
  platform,
  gr_test,
  gr_control,
  (gr_test - gr_control) AS difference_abs,
  round(100 * (gr_test - gr_control) / (gr_control),2) AS pct_change
FROM base
ORDER BY platform
```
| platform | gr_test     | gr_control  | difference_abs | pct_change |
| -------- | ----------- | ----------- | -------------- | ---------- |
| Desktop  | 1887.021300 | 1933.311200 | -46.289900     | -2.39      |
| Mobile   | 21.534600   | 20.110800   | 1.423800       | 7.08       |

**ANSWER**  
For test over control:  
**Mobile -2,39%**  
**Desktop + 7,08%**
