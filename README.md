# Ad-Hoc-Request
Atliq_promotion_analysis_resume_challenge

-- Question 1 :- List of products which have base price greater than 500 & have promo feature of BOGOF.
select distinct f.product_code,product_name, base_price, promo_type="BOGOF"
from fact_events f join dim_products p
on f.product_code = p.product_code 
where (base_price > 500) and (promo_type='BOGOF');

-- Question 2 :- generate a report that provides the number of stores in each city. Allowing us to identify the number of stores presence in each city. The order wil be in descending order as per the store counts.
Select city, count(*) as total_stores 
from dim_stores
group by city
order by Total_stores DESC;

-- Question 3:- Generate a report with campaign , its total revenue before and after the promo.
  SELECT c.campaign_name,
           concat(round(SUM(base_price * fe.quantity_sold_before_promo) / 1000000, 2), 'M') AS total_revenue_before_promo,
           concat(round(SUM(
               CASE 
                   WHEN promo_type = '50% OFF' THEN base_price * 0.5 * fe.quantity_sold_after_promo
                   WHEN promo_type = '33% OFF' THEN base_price * 0.67 * fe.quantity_sold_after_promo
                   WHEN promo_type = '25% OFF' THEN base_price * 0.75 * fe.quantity_sold_after_promo
                   WHEN promo_type = 'BOGOF' THEN base_price * fe.quantity_sold_after_promo
                   WHEN promo_type = '500 Cashback' THEN (base_price - 500) * fe.quantity_sold_after_promo
                   ELSE base_price * fe.quantity_sold_after_promo
               END) / 1000000, 2), 'M') AS total_revenue_after_promo
    FROM fact_events AS fe
    LEFT JOIN dim_campaigns AS c ON fe.campaign_id = c.campaign_id
    GROUP BY c.campaign_name;

    -- Question 4:-Generate a report to calculate incremental sold quantity percentage (ISU%) per each category during diwali campaign.
    with category_revenue as (
    SELECT 
    d.category,
    SUM(f.quantity_sold_before_promo) AS quantity_sold_before_promo,
    SUM(f.quantity_sold_after_promo) AS quantity_sold_after_promo,
    ((SUM(f.quantity_sold_after_promo) - SUM(f.quantity_sold_before_promo)) / SUM(f.quantity_sold_before_promo)) * 100 AS isu_percentage,
    DENSE_RANK() OVER (ORDER BY ((SUM(f.quantity_sold_after_promo) - SUM(f.quantity_sold_before_promo)) / SUM(f.quantity_sold_before_promo)) DESC) AS rank_order
FROM 
    fact_events f
JOIN 
    dim_products d ON f.product_code = d.product_code
JOIN 
    dim_campaigns c ON f.campaign_id = c.campaign_id
WHERE 
    c.campaign_name = 'Diwali'
GROUP BY 
    d.category
ORDER BY 
    isu_percentage DESC)
select category, isu_percentage, rank_order
from category_revenue;

-- Question 5:- Generate a report with top 5 products, ranked by incremental revenue %. Report should have product name ,category & IR%.
SELECT 
    p.product_name,
    p.category,
    ROUND(((SUM(fe.quantity_sold_after_promo * fe.base_price) - SUM(fe.quantity_sold_before_promo * fe.base_price)) / SUM(fe.quantity_sold_before_promo * fe.base_price) * 100), 2) AS IR_percentage
FROM 
    fact_events fe
JOIN 
    dim_products p ON fe.product_code = p.product_code
GROUP BY 
    p.product_name, 
    p.category
ORDER BY 
    ir_percentage DESC
LIMIT 5;
    
