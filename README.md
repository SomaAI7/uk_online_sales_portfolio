# UK Online Sales Analyis
## Project Background
In this project, we analyzed a UK-based online retail dataset to uncover meaningful business insights. The company primarily sells home goods, gifts, and lifestyle products. While the majority of its transactions occur within the UK, it also serves international customers on a smaller scale. Key business metrics include revenue, quantity sold and customer location. We acquired one year of data spanning from December 2010 to December 2011.
### We aim to answer the following business questions:
-	Which individual products and product categories generate the highest revenue and quantity sold across the entire dataset?-
-	Are there noticeable monthly or seasonal patterns in customer purchasing behavior throughout the year?
-	Which countries or regions contribute the most to total revenue, and how does revenue distribution vary across locations?
### Insights and recommendations are organized around the following key areas:
-	Top-performing categories and products
-	Monthly and seasonal sales trends
-	Regional sales performance
The SQL queries used to inspect and clean the data for this analysis can be found here.
Targeted SQL queries addressing key business questions are available here.
An interactive Tableau dashboard to explore sales trends can be found here.
## Data Structure & Initial Checks
The database for this analysis consists of three tables, online_sales, product_category and continents with more than 300,000 rows in total.

These tables were joined and cleaned using SQL to ensure accurate analysis.
# Executive Summary

During this one-year period, our company generated a total revenue of £8,519,675. While we operate globally, our primary market remains the UK, contributing 86.77% of the total revenue. While we operate globally, our data highlights significant opportunities for targeted expansion, particularly within Europe.
The most promising region is Europe, where we made £1,273,943, accounting for 85.11% of our revenue outside of the UK. Other regions generate significantly less revenue, such as Oceania (£139,071), Asia (£71,173), and the Americas (£8,839).
Four European countries account for 73% of the total European revenue. These countries are:

Netherlands: £284,675
Ireland: £257,654
Germany: £205,389
France: £183,802

The next highest contributor, Spain, generated only £55,000, indicating a steep drop-off beyond these core markets. These four European countries already show a big demand in our products and also share  market similarities with the UK, making them ideal targets for strategic expansion
An analysis of regional sales by product revealed that in the Netherlands, there is a particularly high demand for bag and box category products. This indicates  that a  targeted marketing campaign focusing on these categories significantly boost performance in that market.

