# Question 2
## Suggest KPI’s that can measure which bidding strategy is better, explain what it measures.

**Win Rate (WR)** `wins / auctions`  
Efficiency of transforming auctions in served impressions  
Positive: more bidding, more reach  
Negative: could indicate weaker bids or bad segmentation  
```sql
  SELECT
    variant,
    sum(wins) as wins,
    sum(auctions) as auctions,
    sum(wins)/sum(auctions)::decimal(18,6) as win_rate
  FROM bidops_taboola b
  group by variant
```
| variant | wins    | auctions  | win_rate               |
| ------- | ------- | --------- | ---------------------- |
| test    | 8100836 | 104064206 | 0.07784459528764386094 |
| control | 5949252 | 104070679 | 0.05716549615285973103 |

**test over control: +36%**

---
**Click-Through Rate (CTR)** `clicks / wins`  
User engagement with ads  
Positive: relevant creative, qualified traffic  
Negative: impressions not converting to clicks  
```sql
 SELECT
    variant,
    sum(clicks) as clicks,
    sum(wins) as wins,
    sum(clicks)/sum(wins)::decimal(18,6) as ctr
  FROM bidops_taboola b
  group by variant
```
| variant | clicks | wins    | ctr                    |
| ------- | ------ | ------- | ---------------------- |
| test    | 26254  | 8100836 | 0.00324090007500460446 |
| control | 22176  | 5949252 | 0.00372752742697737463 |

**test over control: -13%** - worst traffic quality

---
**Cost per click (CPC)** `spend / clicks`  
Average cost per click  
Positive: cost efficient    
Negative: could indicate bidding inflation
```sql
 SELECT
    variant,
    sum(spend) as spend,
    sum(clicks) as clicks,
    sum(spend)/sum(clicks)::decimal(18,6) as cpc
  FROM bidops_taboola b
  group by variant
```
| variant | spend       | clicks | cpc                    |
| ------- | ----------- | ------ | ---------------------- |
| test    | 6504.726860 | 26254  | 0.24776136436352555801 |
| control | 5270.975320 | 22176  | 0.23768828102453102453 |

**test over control: +4%** - each click costs more

---

**Revenue per Click (RPC)** `gross_revenue / clicks`  
Monetization of each click  
Positive: each click brings more money  
Negative: could indicate problems with conversion or traffic quality  
```sql
 SELECT
    variant,
    sum(gross_revenue) as gross_revenue,
    sum(clicks) as clicks,
    sum(gross_revenue)/sum(clicks)::decimal(18,6) as rcp
  FROM bidops_taboola b
  group by variant
```
| variant | gross_revenue | clicks | rcp                    |
| ------- | ------------- | ------ | ---------------------- |
| test    | 9889.710100   | 26254  | 0.37669346004418374343 |
| control | 8575.338900   | 22176  | 0.38669457521645021645 |

**test over control: -2,6%** - each click brought less revenue

---

**Profit** `gross_revenue − spend`  
Generated liquid value  
Positive: more income  
Negative: best used followed by margin  
```sql
 SELECT
    variant,
    sum(gross_revenue) as gross_revenue,
    sum(spend) as spend,
    sum(gross_revenue)-sum(spend)::decimal(18,6) as profit
  FROM bidops_taboola b
  group by variant
```
| variant | gross_revenue | spend       | profit      |
| ------- | ------------- | ----------- | ----------- |
| test    | 9889.710100   | 6504.726860 | 3384.983240 |
| control | 8575.338900   | 5270.975320 | 3304.363580 |  

**test over control: +2,4%** - test was more profitable
