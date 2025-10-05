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

---
## Analyze the test based on the above KPI - Which variant would you suggest? What is the uplift we are expecting? Show calculations or graphs that support your conclusions. Write your analysis as if you are presenting it to the product team

We have two main options here depending on the goal.   
 a) *Best efficiency:* I'd consider using `CTR` and `RPC`  
 This option shows the test performed worse than control. Test was less efficient, and brought less revenue per click. (CTR -13% | RCP -2%)
<img width="900" height="250" alt="Suporting graphs" src="https://github.com/user-attachments/assets/65545089-9d60-48db-80c6-cb748a3b48d7" />  

 b) *Best income:* I'd use profit 
 This  options shows the control variable performed better considering it brings more money. (+2,44% )
 <img width="450" height="250" alt="Captura de Tela 2025-10-04 às 13 53 46" src="https://github.com/user-attachments/assets/d8a8c4b2-212b-4fd5-bc8e-2774b0eda919" /> 
  
Both options are valid, depending on what we are looking for: Test brings scalability and spends more money. Control has better quality of bidding, is more effective but brings less money overall.   
 
---
## If Taboola decides we should optimize for gross revenue, what KPIs we should be using and which variant would you suggest? Show calculations or graphs that support your conclusions.
Key KPIs:
- `gross_revenue` - target metric
- `spend` - input
- `marginal_roi` - `a/spend` by variant through log curve
- `daily_roi` - `gross_revenue/spend` as guard rail
- headroom to efective cap - `a*ln(spend_cap/spend)`

Through the calculations, control variant has more space to growth, it means we cant spend more money before it saturates and brings less revenue. 

Step by step on google sheets:

1) data prep:
  - set a table by date and variable with:
    `date`. `variant`, `sum(spend)` and `sum(gross_revenue`
  - calculate `daily_roi` as `gross+revenue/spend` 
    |date|variant|spend|gross_revenue|daily_roi|
    |---|---|---|---|---|
    |2022-05-28|control|509,47131|889,8992|1,746711115|813,5766772|
    |2022-05-28|test|596,37034|981,0296|1,645000655|983,3490742|
    |2022-05-29|control|627,35335|960,4248|1,530915233|1037,947714|
    |2022-05-29|test|663,20208|1011,5877|1,525308395|1097,85196|
    |2022-05-27|control|669,29348|1227,5993|1,834171909|1107,708015|
    |2022-05-30|control|783,43418|1205,4067|1,538618981|1277,454641|
    |2022-05-26|control|809,14477|1346,277|1,663827105|1312,264123|
    |2022-05-27|test|836,24687|1358,2186|1,624183777|1347,77993|
    |2022-05-31|control|879,34533|1428,4054|1,624396413|1401,953547|
    |2022-05-26|test|972,94054|1533,5142|1,576164356|1510,988162|
    |2022-05-30|test|973,88712|1393,4838|1,430847345|1512,036445|
    |2022-05-25|control|992,9329|1517,3265|1,528125919|1532,91481|
    |2022-05-31|test|1207,50362|1738,352|1,43962467|1743,822593|
    |2022-05-25|test|1254,57629|1873,5242|1,49335215|1785,048325|
    
  - plot a scatter graph `spend` vs `gross_revenue` in order to get the general log curve `-5906+1078ln(x)`  
  <img width="600" height="300" alt="scatter plot graph" src="https://github.com/user-attachments/assets/a9c7bc59-20b9-44a0-8647-bfa383990339" />

2) Curve adjustment for each variable  
  - define **slope** and **intercept** using `ln(spend)` and `gross_revenue` for each variant  
  - define `gross_revenue_pred` by `intercept+slope*ln(spend)` for each line  
  - define `marginal_roi_pred` by `slope/spend`for each line  
3) Mesure `max_spend` (maximum value we can have in spend without saturation) for each variant  
  - assuming we dont have variable cost, i'm using 1. Meaning each dolar spent on this model, has to bring at least 1 dolar in revenue. Above this value the model starts to saturate.   
    - control = **984,330545**  
    - test = **1175,719815**  
4) Mesure headroom (space we have to spend without saturation)  
  - using the last day of spend and comparing with our slope, we have:
    
|last date|variable|spend|max spend|headroom|
|---|---|---|---|---|
| 2022-05-31 | control	| 879,34533| 984,330545 |104,985215|
| 2022-05-31 | test	| 1207,50362| 1175,719815 | -31,78380517|

>[!note]
>Here we can already see test variant is already saturated and we have more space on control

**In conclusion, i'd escalate spend on control**

