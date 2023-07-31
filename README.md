![image](https://github.com/Dorothywimberly/Unicorn_Companies/assets/131917095/1b420e9f-04ed-4973-99c6-da3a45b9144e)
![image](https://github.com/Dorothywimberly/Unicorn_Companies/assets/131917095/d570ce37-8e1b-4245-8bc5-32aa8fd93c9c)
![image](https://github.com/Dorothywimberly/Unicorn_Companies/assets/131917095/971a62bc-9ef1-4e47-9efb-3913ab241eeb)

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
Unicorn Companies -  Would be identified up under the Company column in the Unicorn_Companies table.            
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
Full Query Results found above or in link bellow:                        
https://github.com/Dorothywimberly/Unicorn_Companies/blob/main/Top_ROI_Unicorn.csv


2.	How long does it usually take for a company to become a unicorn? Has it always been this way?                     
Year the company was founded- Would be found under the Year_Founded column.                                      
When the Company became a unicorn- Would be found under the Date_Joined.                                          
Find how long it takes for a company to become a Unicorn.                                                            
* Date_Joined - Year_Founded = Years to join
* Remove any year joined that come out to be a negative value.
* List by year Founded.
```
/* Extract year from Date Joined and subtract date joined from year founded, Remove any year joined that come out to be a negative value. */
SELECT Year_Founded, Company, Date_Joined, EXTRACT(YEAR FROM Date_Joined) - Year_Founded AS Years_to_Join
FROM `Unicorn_Companies_Dataset.Unicorn_Companies`
WHERE EXTRACT(YEAR FROM Date_Joined) >= Year_Founded
/* Order by year founded */
ORDER BY Year_Founded;
```
* Find the Average number of years it takes to become a Unicorn company.                                      
* List by the Year Founded.                                                                  
```
/* Find the Average number of years it takes to become a Unicorn company and round the number*/
SELECT
  Year_Founded,
  ROUND(AVG(EXTRACT(YEAR FROM Date_Joined) - Year_Founded), 2) AS Avg_Years_to_Join
FROM `Unicorn_Companies_Dataset.Unicorn_Companies`
/*Group the number by year and order it by year*/
GROUP BY Year_Founded
ORDER BY Year_Founded;
```
*Query Results*
Full Query Results found above or in link bellow:                                        
https://github.com/Dorothywimberly/Unicorn_Companies/blob/main/Years_to_Join.csv                                 
https://github.com/Dorothywimberly/Unicorn_Companies/blob/main/Years_to_Join.csv

3. Which countries have the most unicorns? Are there any cities that appear to be industry hubs?                  
Countries- Would be found under the Country column.
Unicorns- Would be under the Company column.                        
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
Full Query Results found above or in link bellow:                                
https://github.com/Dorothywimberly/Unicorn_Companies/blob/main/Top_Unicorn_Country.csv


Unicorns- Would be under the Company column.                                        
Cities- Would be found under the city column.                                                  
* Count the number of Companies in each City and remove Null cities.
*	List the cities from highest to lowest and only show top 20 cities.

                               
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
Full Query Results found above or in link bellow:     
https://github.com/Dorothywimberly/Unicorn_Companies/blob/main/Top_Unicorn_Cities.csv

4. Which investors have funded the most unicorns?                                      
Investors- Would be under the Select_Invesstors column.                                                
Funding- Would be under the Funding column.                                              
*	Find the total number of funding invested from each investor.                                              
*	List the top 5 investors.                                                                           

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
Full Query Results found above or in link bellow:     
https://github.com/Dorothywimberly/Unicorn_Companies/blob/main/Top_5_Investors.csv

# Visualize Results:

# Interpret and Communicate Findings:

# Draw Conclusions and Limitations:



