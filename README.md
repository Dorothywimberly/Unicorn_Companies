![image](https://github.com/Dorothywimberly/Unicorn_Companies/assets/131917095/429e1c63-1228-4f7e-8ad8-5b9c035e60fa) 
![image](https://github.com/Dorothywimberly/Unicorn_Companies/assets/131917095/742f12b4-4ad5-456c-9136-92f64933ba7d)
![image](https://github.com/Dorothywimberly/Unicorn_Companies/assets/131917095/429e1c63-1228-4f7e-8ad8-5b9c035e60fa)

# Unicorn_Companies


Unicorn Companies Maven Analytics
Private companies with a valuation over $1 billion as of March 2022, including each company's current valuation, funding, country of origin, industry, select investors, and the years they were founded and became unicorns.

# Objective:

1. Which unicorn companies have had the biggest return on investment?

2. How long does it usually take for a company to become a unicorn? Has it always been this way?

3. Which countries have the most unicorns? Are there any cities that appear to be industry hubs?

4. Which investors have funded the most unicorns?

# Variables:

- Company: Company name.
- Valuation: Company's current valuation.                    	
 *Company valuation in billions (B) of dollars.
- Funding: Total amount raised across all funding rounds in billions (B) or millions (M) of dollars.
- Continent: Continent the company was founded in.
- Country: Country the company was founded in.
- City: City the company was founded in.
- Industry: Company industry.
- Select_Investors: Investors to the companies. *Top 4 investing firms or individual investors (some have less than 4).
- Date_Joined: Company's became unicorns.
  *The date in which the company reached $1 billion in valuation.
- Year_Founded: Year the company was founded.

# Identify Data Sources:

https://mavenanalytics.io/data-playground?search=unicorn

# Data Cleaning and Preprocessing:
Clean and preprocess the collected data to ensure accuracy and consistency. This step involves handling missing values, removing duplicates, and formatting data in a standardized manner.

```
/* Change Strings to Floats, remove M, B, and $ symbols and Letters, change strings to floats, remove unknown and na*/
WITH TransformedData AS (
  SELECT
    company,
    Country,
    CASE
      WHEN Valuation = 'unknown' THEN NULL
      WHEN RIGHT(Valuation, 1) = 'M' THEN CAST(SUBSTR(Valuation, 2, LENGTH(Valuation) - 2) AS FLOAT64) * 1000000
      WHEN RIGHT(Valuation, 1) = 'B' THEN CAST(SUBSTR(Valuation, 2, LENGTH(Valuation) - 2) AS FLOAT64) * 1000000000
      WHEN REPLACE(Valuation, '$', '') LIKE r'^[0-9]+$' THEN CAST(REPLACE(Valuation, '$', '') AS FLOAT64)
      ELSE 0
    END AS Valuation,
    CASE
      WHEN Funding = 'unknown' THEN NULL
      WHEN RIGHT(Funding, 1) = 'M' THEN CAST(SUBSTR(Funding, 2, LENGTH(Funding) - 2) AS FLOAT64) * 1000000
      WHEN RIGHT(Funding, 1) = 'B' THEN CAST(SUBSTR(Funding, 2, LENGTH(Funding) - 2) AS FLOAT64) * 1000000000
      WHEN REPLACE(Funding, '$', '') LIKE r'^[0-9]+$' THEN CAST(REPLACE(Funding, '$', '') AS FLOAT64)
      ELSE 0
    END AS Funding
  FROM `Unicorn_Companies_Dataset.Unicorn_Companies`
```

# Finding Solutions:

1. Which unicorn companies have had the biggest return on investment?                                                                 
Unicorn Companies -  Would be idenified up under the Company coulmn in the Unicorn_Companies table.            
Return on investment (ROI) - Use the investment data to calculate the ROI for each unicorn company.            
* ( Valuation - Funding) / Funding  * 100 = ROI                                             
Only list the top 10 companies. 
```
/* Find the ROI for companies list from highest to lowest*/
SELECT
  Valuation,
  Funding,
  Company,
  Country,
  CASE
    WHEN Valuation = 0 THEN NULL
    WHEN Funding = 0 THEN NULL
    ELSE ROUND((Valuation - Funding) / Funding) * 100
  END AS ROI
FROM TransformedData
ORDER BY ROI DESC
LIMIT 10;
```
*Query Results*
![image](https://github.com/Dorothywimberly/Unicorn_Companies/assets/131917095/db6e0972-e4cd-419d-9f9d-ea730451cf7b)



2. How long does it usually take for a company to become a unicorn? Has it always been this way?                   
Year the company was founded- Would be fould under the Year_Founded coulmn.                      
When the Company became a unicorn- Would be found under the Date_Joined.                                     
Find how long it takes for a company to become a Unicorn.                     
* Date_Joined - Year_Founded = Years to join
* List by year Founded.
```
/* Extract year from Date Joined and subtract date joined from year founded */
SELECT Year_Founded, Date_Joined, EXTRACT(YEAR FROM Date_Joined) - Year_Founded AS Years_to_Join
FROM `Unicorn_Companies_Dataset.Unicorn_Companies`
/* Order by year founded */
ORDER BY Year_Founded;
```
*Query Results*
Full Query Results found above or in link bellow:                                        
https://github.com/Dorothywimberly/Unicorn_Companies/blob/main/Years_to_Join.csv

3. Which countries have the most unicorns? Are there any cities that appear to be industry hubs?                  

Countries- Would be fould under the Country coulmn.                                      
Unicorns- Would be under the Company coulmn.                           
* Count the number of companies in each Country.
* Limit the number of Countries to 10.                    
```
/* Count number of Companies in each Country */
SELECT Country, COUNT(*) AS NumberOfCompanies
FROM `Unicorn_Companies_Dataset.Unicorn_Companies`
GROUP BY Country
/* List from hightest to lowest */
ORDER BY NumberOfCompanies DESC
LIMIT 10;
```
*Query Results*                                                                 
![image](https://github.com/Dorothywimberly/Unicorn_Companies/assets/131917095/ab5b472e-b5a9-42ec-925d-0624b2103345)

Unicorns- Would be under the Company coulmn.                                        
Cities- Would be fould inder the city coulmn.                          
* Count the number of Companies in each City and remove Null cities.                                
* List the cities from highest to lowest and only show top 20 cities.                                  
```
/* Count the number of Companies in each City and remove Null cities  */
SELECT City, COUNT(*) AS NumberOfCompanies
FROM `Unicorn_Companies_Dataset.Unicorn_Companies`
WHERE City IS NOT NULL
GROUP BY City
/* List the cities from highest to lowest and only show top 20 cities.*/
ORDER BY NumberOfCompanies DESC
LIMIT 20;
```
*Query Results*                                                                           
![image](https://github.com/Dorothywimberly/Unicorn_Companies/assets/131917095/5d4c9501-53f9-48d9-a4bf-39171fa60b1b)


4. Which investors have funded the most unicorns?                                                    
Investors- Would be under the Select_Invesstors coulmn.                      
Funding- Would be under the Funding coulmn.                                  
* Find the total number of funding invested from each inverstor.
* List the top 5 investors. 
```
/* Change Strings to Floats, remove M, B, and $ symbols and Letters, change strings to floats, remove unknown and na*/
WITH TransformedData AS (
  SELECT
    Select_Investors,
    CASE
      WHEN RIGHT(Funding, 1) = 'M' THEN CAST(SUBSTR(Funding, 2, LENGTH(Funding) - 2) AS FLOAT64) * 1000000
      WHEN RIGHT(Funding, 1) = 'B' THEN CAST(SUBSTR(Funding, 2, LENGTH(Funding) - 2) AS FLOAT64) * 1000000000
      WHEN REPLACE(Funding, '$', '') LIKE r'^[0-9]+$' THEN CAST(REPLACE(Funding, '$', '') AS FLOAT64)
      ELSE 0
    END AS Funding
  FROM `Unicorn_Companies_Dataset.Unicorn_Companies`
)
/* List the Investors from highest to lowest and only show top 5 Invesrors.*/
SELECT
  Select_Investors,
  SUM(Funding) AS TotalFunding
FROM TransformedData
GROUP BY Select_Investors
ORDER BY TotalFunding DESC
LIMIT 5;
```
*Query Results*   
![image](https://github.com/Dorothywimberly/Unicorn_Companies/assets/131917095/842faab8-9c16-44ff-aa0b-f6d1aec037fe)

# Visualize Results:

# Interpret and Communicate Findings:

# Draw Conclusions and Limitations:



