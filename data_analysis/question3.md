# Question 3
## Each auction we handle has an additional serving costs of 0.00001$ (or 0.01$ per 1k auctions).  
Based on this data, what would you suggest to change\add to the test variant? What additional data do you need?  

Based on this new data, we can calculate:
- `net_profit`: profit after all costs considered  - `gross_revenue - spend - (auctions * 0,00001)`  
- `net_margin`: margin of profit on gross_revenue -  `net_profit / gross_revenue`  
- `CPAuc`: how much each auction actually costs - `spend / auctions + 0.00001`
- `RPAuc`: how much GR each auction gerenates - `gross_revenue / auctions`

>[!NOTE]  
> If `RPAuc` < `CPAuc` = loss   

```sql
WITH base AS (
  SELECT
    b.variant,
    sum(b.gross_revenue) as gross_revenue,
    sum(b.spend) as spend,
    sum(b.auctions) as auctions,
    (sum(b.auctions)*0.00001)::numeric as auction_value
  FROM bidops_taboola b
  GROUP BY b.variant
)
SELECT
  base.variant as variant,
  sum(auction_value) as auction_value,
  (sum(gross_revenue)-(sum(spend))-(sum(auction_value))) as net_profit,
  round(((sum(gross_revenue)-(sum(spend))-(sum(auction_value)))/(sum(gross_revenue))),5) as net_margin,
  round((sum(spend)/sum(auctions)+0.00001),5) as CPAuc,
  round((sum(gross_revenue)/sum(auctions)),5) as RPAuc
FROM base
group by base.variant
```   
 
| variant | auction_value | net_profit  | net_margin | cpauc   | rpauc   |
| ------- | ------------- | ----------- | ---------- | ------- | ------- |
| control | 1040.70679    | 2263.656790 | 0.26397    | 0.00006 | 0.00008 |
| test    | 1040.64206    | 2344.341180 | 0.23705    | 0.00007 | 0.00010 |

**what change/add on the test variant**  
- Keep scaling
- Monitor `RPAuc` and `CPAuc` by subset, rpauc must be higher than cpauc
- Use `net_margin` as guard rail, net margin should be >0
