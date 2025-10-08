# Taboola - Publisher PS Bid Ops Home Exam  

This repository contains the resolution of the **Home Exam** for **Taboola** (Bid Ops).  
It includes SQL data analysis, KPI evaluation, business insights, and the implementation of a Taboola Widget in HTML/CSS/JS.  

---

##  Project Structure
- [`data_analysis/`](case_taboola/data_analisys)
  - [`question1`](case_taboola/data_analisys/question1.md)
  - [`question2`](case_taboola/data_analisys/question2.md)
  - [`question3`](case_taboola/data_analisys/question3.md)
- [`tech_evaluation/`](case_taboola/tech_evaluation)
  - [`question1`](case_taboola/tech_evaluation/question1.md)

---

# **Part 1 — Data Analysis** 

## 1.a) For each publisher with mobile traffic, find the total number of auctions across all platforms
Answer details: [`question1`](case_taboola/data_analisys/question1.md)

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

## 1.b) For publisher_id 3406, calculate the change in gross revenue as a percentage between the 2 variants. show the results broken down by platform.    
Answer details: [`question1`](case_taboola/data_analisys/question1.md)

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
**Desktop -2,39%**  
**Mobile + 7,08%**

---

## 2.a) Suggest KPI’s that can measure which bidding strategy is better, explain what it measures.
Answer details: [`question2`](case_taboola/data_analisys/question2.md)   

- **Win Rate (WR)** – efficiency of converting auctions into impressions `wins / auctions`  
- **CTR** – user engagement with ads `clicks / wins`    
- **CPC** – average cost per click `spend / clicks`    
- **RPC** – monetization of each click   `gross_revenue / clicks`  
- **Profit** – generated liquid value  `gross_revenue − spend`

---

## 2.b) Analyze the test based on the above KPI - Which variant would you suggest? What is the uplift we are expecting? Show calculations or graphs that support your conclusions. Write your analysis as if you are presenting it to the product team.
Answer details: [`question2`](case_taboola/data_analisys/question2.md)  

We have two main options here depending on the goal.   
 a) *Best efficiency:* I'd consider using `CTR` and `RPC`  
 This option shows the test performed worse than control. Test was less efficient, and brought less revenue per click. (CTR -13% | RCP -2%)
<img width="900" height="250" alt="Suporting graphs" src="https://github.com/user-attachments/assets/65545089-9d60-48db-80c6-cb748a3b48d7" />  

 b) *Best income:* I'd use profit 
 This  options shows the control variable performed better considering it brings more money. (+2,44% )
 <img width="450" height="250" alt="Captura de Tela 2025-10-04 às 13 53 46" src="https://github.com/user-attachments/assets/d8a8c4b2-212b-4fd5-bc8e-2774b0eda919" /> 
  
Both options are valid, depending on what we are looking for: Test brings scalability and spends more money. Control has better quality of bidding, is more effective but brings less money overall.   
 
---
## 2.c) If Taboola decides we should optimize for gross revenue, what KPIs we should be using and which variant would you suggest? Show calculations or graphs that support your conclusions.  
Answer details: [`question2`](case_taboola/data_analisys/question2.md)   

- `gross_revenue` - target metric  
- `spend` - input  
- `marginal_roi` - `a/spend` by variant through log curve  
- `daily_roi` - `gross_revenue/spend` as guard rail  
- headroom to efective cap - `a*ln(spend_cap/spend)`  

Through the calculations, control variant has more space to growth, it means we cant spend more money before it saturates and brings less revenue.  

---

## 3.a) Each auction we handle has an additional serving costs of 0.00001$ (or 0.01$ per 1k auctions). Based on this data, what would you suggest to change\add to the test variant? What additional data do you need?  
Answer details: [`question3`](case_taboola/data_analisys/question3.md) 

Calculated metrics:  
- `net_profit`: profit after all costs considered  - `gross_revenue - spend - (auctions * 0,00001)`  
- `net_margin`: margin of profit on gross_revenue -  `net_profit / gross_revenue`  
- `CPAuc`: how much each auction actually costs - `spend / auctions + 0.00001`
- `RPAuc`: how much GR each auction gerenates - `gross_revenue / auctions`


**Recommendation:** keep scaling *test* but monitor `RPAuc > CPAuc` and ensure positive net margin.  

---

# Part 2 — Tech Evaluation

## Using the JSON response from the below Recommendations API request, create a responsive Taboola widget with the following requirements:

**For desktop and tablet:**   
- 3 columns and 2 rows (3x2).  
- The item thumbnail image aspect ratio should be 6:5.  
- Include the item title and branding below each image.  
- Add a header to the top of the widget with your choice of text.
- Clickable thumbnail image, item title, and branding text.  

**For smartphone**
- 1 column with six rows (1x6).
- The item thumbnail image aspect ratio should be 3:2.
- Include the item title and branding below each image.
- Add a header to the top of the widget with your choice of text.
- Clickable thumbnail image, item title, and branding text.

Answer details: [`question1`](case_taboola/tech_evaluation/question3.md)  
Live code on jsbin: [Taboola Widget](https://jsbin.com/mogeyisuyo/edit?html,css,js) 
