# **CoffeeKing_Yelp_Academic_Dataset**

## Background
CoffeeKing, a startup coffee company, aims to offer a unique and inclusive experience. They seek insights from Yelp reviews and user data to inform location selection, operating hours, business enhancements, and loyalty-focused marketing strategies.

For my final project in the Coursera Course, "Learn SQL Basics for Data Science Specialization," I've decided to focus on Yelp data analysis. I find this project particularly compelling due to the potential insights it can offer. My background in the merchant cash advance industry has instilled in me a profound understanding of business operations and their complexities, which I hope to leverage in this analysis.

### **Data Overview**

- The [Yelp Academic Dataset](https://www.yelp.com/dataset/download) comprises a RAR file containing 5 JSON files, with data including:
  - 6,990,280 reviews
  - 150,346 businesses
  - 11 metropolitan areas
  - 908,915 tips by 1,987,897 users
  - Over 1.2 million business attributes like hours, parking, availability, and ambience
 
 ### **Data Limitation**
- Absence of demographic details such as age or gender. 
- Lack of population data for geographical context.
- No information on rent costs, which can be crucial for business planning.
- Absence of review dates for studying seasonality.
   
## **Prepare**
- I'm using R to convert JSON data to CSV format, facilitating the creation of a SQLite database.

### Installing and loading necessary packages
 ``` ruby
install.packages("jsonlite")
install.packages("dplyr")
install.packages("tidyverse")

library(jsonlite)
library(dplyr)
library(tidyverse)
```
### Importing the JSON files
 ```ruby
setwd("C:\\Users\\Lipsky\\Documents\\yelp_dataset")
tip <- stream_in(file("yelp_academic_dataset_tip.json"))
checkin <- stream_in(file("yelp_academic_dataset_checkin.json"))
user <- stream_in(file("yelp_academic_dataset_user.json"))
review <- stream_in(file("yelp_academic_dataset_review.json"))
business <- stream_in(file("yelp_academic_dataset_business.json"))

```
### Extract business_id, attributes and hours into a new data frame
```ruby
attributes_df <- data.frame(business_id = business$business_id, attributes = business$attributes)
hours_df <- data.frame(business_id = business$business_id, hours = business$hours)
```
### Remove the attributes and hours dataframes from the business data frame
```ruby
business1$attributes <- NULL
business1$hours <- NULL
```

## Process - Cleaning

### Remove "attributes" from column names in attributes_df
```ruby
colnames(attributes_df) <- gsub("^attributes\\.", "", colnames(attributes_df))

```
### Cleaning attributes Wifi
```ruby
attributes_df$WiFi <- ifelse(attributes_df$WiFi == "u'no'", "No",
                             ifelse(attributes_df$WiFi == "u'free'", "Free",
                                    ifelse(attributes_df$WiFi == "'free'", "Free",
                                           ifelse(attributes_df$WiFi == "'no'", "No",
                                                  ifelse(is.na(attributes_df$WiFi), "No", "Paid")))))
```
### Cleaning attributes Alcohol
```ruby
attributes_df$Alcohol <- ifelse(attributes_df$Alcohol == "u'none'", "N/A",
                                ifelse(attributes_df$Alcohol == "u'full_bar'", "Full_Bar",
                                       ifelse(attributes_df$Alcohol == "'full_bar'", "Full_Bar",
                                              ifelse(attributes_df$Alcohol == "u'beer_and_wine'", "Beer_and_Wine",
                                                     ifelse(attributes_df$Alcohol == "'beer_and_wine'", "Beer_and_Wine",
                                                            ifelse(attributes_df$Alcohol == "'none'", "N/A",
                                                                   ifelse(attributes_df$Alcohol == "NA", "N/A", attributes_df$Alcohol)))))))
```
## Hours table

### Remove "hours" from column names in hours_df
```ruby
colnames(hours_df) <- gsub("^hours\\.", "", colnames(hours_df))
```

### Split opening and closing hours into separate columns
```ruby
 hours_df <- hours_df %>%
   separate(Monday, into = c("Monday_Open", "Monday_Close"), sep = "-") %>%
   separate(Tuesday, into = c("Tuesday_Open", "Tuesday_Close"), sep = "-") %>%
   separate(Wednesday, into = c("Wednesday_Open", "Wednesday_Close"), sep = "-") %>%
   separate(Thursday, into = c("Thursday_Open", "Thursday_Close"), sep = "-") %>%
   separate(Friday, into = c("Friday_Open", "Friday_Close"), sep = "-") %>%
   separate(Saturday, into = c("Saturday_Open", "Saturday_Close"), sep = "-") %>%
   separate(Sunday, into = c("Sunday_Open", "Sunday_Close"), sep = "-")
```
### Define a function to convert time format and handle null values
```ruby
convert_time <- function(time_str) {
   ifelse(
     is.na(time_str) | time_str == "",
     NA,
     vapply(strsplit(time_str, "-"), function(x) {
       parts <- strsplit(x, ":")
       hours <- as.numeric(parts[[1]][1])
       minutes <- as.numeric(parts[[1]][2])
       am_pm <- ifelse(hours < 12, "AM", "PM")
       hours <- ifelse(hours == 0, 12, ifelse(hours > 12, hours - 12, hours))
       sprintf("%02d:%02d %s", hours, minutes, am_pm)
     }, character(1))
   )
 }
```

### Apply the function to all columns with opening and closing hours
```ruby
hour_cols <- grep("_Open$|_Close$", names(hours_df), value = TRUE)
 hours_df[hour_cols] <- lapply(hours_df[hour_cols], convert_time)
```

### Create a data frame for opening hours
```ruby
 yelp_opening_hours <- hours_df %>%
   select(business_id, ends_with("_Open"))
```

### Create a data frame for closing hours
```ruby
 yelp_closing_hours <- hours_df %>%
   select(business_id, ends_with("_Close"))
```
### Exporting into CSV files
```ruby
write.csv(business,file= 'C:/Users/Lipsky/yelp_business.csv')
write.csv(checkin,file= 'C:/Users/Lipsky/yelp_checkin.csv')
write.csv(review,file= 'C:/Users/Lipsky/yelp_review.csv')
write.csv(tip,file= 'C:/Users/Lipsky/yelp_tip.csv')
write.csv(user,file= 'C:/Users/Lipsky/yelp_user.csv')
write.csv(attributes_df,file= 'C:/Users/Lipsky/yelp_attributes.csv')
write.csv(yelp_opening_hours,file= 'C:/Users/Lipsky/yelp_opening_hours.csv')
write.csv(yelp_closing_hours,file= 'C:/Users/Lipsky/yelp_closing_hours')
```

### Entity Relationship Diagram (ERD)
<br>
        <p align="left">
          <img src="images/Entity Relationship Diagram.jpeg" width="800"/>
        </p>
      </li>
    </ul>
  </li>

## Analysis

I'm primarily using SQLite for most of the analysis, as it aligns with the SQL course's focus.

 * #### Stores by City and AVG Reviews and Ratings

The goal of this query is to identify prime store locations by analyzing cities based on store volume, reviews, and star ratings, focusing on coffee shop interest, engagement, and competition.

``` SQL
SELECT       state
             ,city
             ,COUNT(name) AS Stores
             ,ROUND(AVG(review_count),1) AS AvgReviews
             ,ROUND(AVG(stars),1) AS AvgStarRating
FROM  
		yelp_business
		
WHERE stars BETWEEN 3 AND 4 AND categories  LIKE '%Coffee & Tea%' 

GROUP BY city
ORDER BY Stores DESC
LIMIT 5;

```

| State | City          | Stores | AvgReviews | AvgStarRating |
|-------|---------------|--------|------------|---------------|
| PA    | Philadelphia  | 467    | 96.8       | 3.6           |
| AB    | Edmonton      | 211    | 24.3       | 3.6           |
| FL    | Tampa         | 181    | 74.1       | 3.7           |
| LA    | New Orleans   | 163    | 184.7      | 3.7           |
| AZ    | Tucson        | 162    | 74.3       | 3.6           |


* #### Percentage of Closed Businesses by State and Filtered by Category Coffee & Tea

With this new query, I aimed to determine the closure rate of businesses by state, assessing  the market's level of challenge.

``` SQL
SELECT SUM(CASE WHEN is_open = 0 THEN 1 ELSE 0 END) AS ClosedBusinesse
		,COUNT(*) AS TotalCount 
		,ROUND((SUM(CASE WHEN is_open = 0 THEN 1 ELSE 0 END) * 100.00/ COUNT(*)),2)  AS PercentageClosed
		,state
FROM yelp_business
WHERE categories  LIKE '%Coffee & Tea%'
GROUP BY state
ORDER BY Total_count DESC
LIMIT 5;
```
| ClosedBusinesses  | TotalCount  | PercentageClosed   | State |
|-------------------|-------------|--------------------|-------|
| 501               | 1725        | 29.04%             | PA    |
| 239               | 1106        | 21.61%             | FL    |
| 135               | 493         | 27.38%             | TN    |
| 141               | 472         | 29.87%             | LA    |
| 131               | 467         | 28.05%             | IN    |

* #### Percentage of closed businesses by City

 With this  query, I aimed to determine the closure rate of businesses by city, assessing  the market's level of challenge

``` SQL
SELECT SUM(CASE WHEN is_open = 0 THEN 1 ELSE 0 END) AS Closed_businesses
		,COUNT(*) AS Total_count
		,ROUND((SUM(CASE WHEN is_open = 0 THEN 1 ELSE 0 END) * 100.00/ COUNT(*)),2)  AS Percentage_closed
		,state
		,city
FROM yelp_business

WHERE state = 'FL' AND categories  LIKE '%Coffee & Tea%'
GROUP BY city
ORDER BY Total_count DESC
LIMIT 5;
```

| Closed Businesses | Total Count | Percentage Closed  | State  | City             |
|-------------------|-------------|--------------------|-------|-------------------|
| 99                | 395         | 25.06%             | FL    | Tampa             |
| 23                | 80          | 28.75%             | FL    | Clearwater        |
| 17                | 76          | 22.37%             | FL    | Saint Petersburg  |
| 11                | 57          | 19.3%              | FL    | St. Petersburg    |
| 7                 | 39          | 17.95%             | FL    | Largo             |

### Key Takeaways

We've chosen Tampa, FL for our new coffee shop location for the following reasons:

* Attracted by the city's average star rating of 3.7, indicating market differentiation opportunities and room for improvement.
* CoffeeKing is confident in offering a unique product that can stand out among competitors.
* City has an average of 74 reviews per coffee and tea business, showing active customer engagement. We'll also consider population and the overall business environment.

### Strategic Analysis for Competitive Distinction

#### Attributes vs. Star Ratings

The purpose of this query is to analyze the correlation between various attributes (outdoor seating, dogs allowed, bike parking, alcohol service, restaurant delivery, and table service) and the star ratings of coffee and tea shops in Saint Petersburg, Florida, with star ratings of 3.5 or higher..

``` SQL
SELECT
     b.stars  AS StarRating
	,COUNT(*) AS NumOfStores
	,(ROUND(SUM(CASE WHEN a.OutdoorSeating = 'True' THEN 1 ELSE 0 END) * 100.0 / COUNT(*),1)) AS PercentageOutdoorSeating
	,(ROUND(SUM(CASE WHEN a.DogsAllowed = 'True' THEN 1 ELSE 0 END) * 100.0 / COUNT(*),1)) AS PercentageAllowedDogs
	,(ROUND(SUM(CASE WHEN a.BikeParking = 'True' THEN 1 ELSE 0 END) * 100.0 / COUNT(*),1)) AS PercentageBikeParking
	,(ROUND(SUM(CASE WHEN a.Alcohol = 'Full_Bar' OR 'Beer_and_Wine' THEN 1 ELSE 0 END) * 100.0 / COUNT(*),1)) AS PercentageAlcohol
	,(ROUND(SUM(CASE WHEN a.RestaurantsDelivery = 'True' THEN 1 ELSE 0 END) * 100.0 / COUNT(*),1)) AS PercentageRestaurantDelivery
	,(ROUND(SUM(CASE WHEN a.RestaurantsTableService = 'True' THEN 1 ELSE 0 END) * 100.0 / COUNT(*),1)) AS PercentageTableService
FROM
    yelp_attributes a
INNER JOIN
    yelp_business b ON a.business_id = b.business_id
WHERE
     state = 'FL' AND city = 'Tampa' AND categories LIKE '%Coffee & Tea%' AND StarRating >= '3.5'
GROUP BY
     StarRating
ORDER BY
     StarRating DESC;

```

| Star Rating | Num of Stores | Percentage Outdoor Seating | Percentage Allowed Dogs | Percentage Bike Parking | Percentage Alcohol | Percentage Restaurant Delivery | Percentage Table Service |
|-------------|---------------|-----------------------------|-------------------------|------------------------|--------------------|-----------------------------|--------------------------|
| 5.0         | 28            | 50.0                      | 21.4                  | 50.0                   | 0.0                | 32.1                        | 7.1                      |
| 4.5         | 97            | 54.6                      | 27.8                  | 64.9                   | 3.1                | 46.4                        | 11.3                     |
| 4.0         | 97            | 53.6                      | 19.6                  | 69.1                   | 10.3               | 45.4                        | 12.4                     |
| 3.5         | 47            | 51.1                      | 8.5                   | 59.6                   | 0.0                | 40.4                        | 10.6                     |


#### Opening Hours Range ##

The purpose of this query is to calculate the average opening hours based on the star rating and the number of reviews for coffee and tea businesses in Saint Petersburg, Florida

``` SQL
SELECT  
		 b.stars AS StarRating
		,ROUND(AVG(o.Monday_Open + o.Tuesday_Open + o.Wednesday_Open + o.Thursday_Open + o.Friday_Open) / 5, 2) AS AvgOpeningHour
		,SUM(b.review_count)as Num_Reviews
		
FROM yelp_opening_hours o
	
INNER JOIN yelp_business b

ON o.business_id = b.business_id

WHERE 

	 state = 'FL' AND city = 'Tampa' AND categories LIKE '%Coffee & Tea%'
	 
GROUP BY b.stars
ORDER BY StarRating DESC
LIMIT 5;

```

| Star Rating | Avg Opening Hour | Num Reviews |
|-------------|------------------|------------|
| 5.0         | 7.84             | 1608       |
| 4.5         | 7.33             | 10787      |
| 4.0         | 7.67             | 10270      |
| 3.5         | 7.05             | 1722       |
| 3.0         | 6.48             | 1422       |


#### Closing Hours Range ##

The purpose of this query is to calculate the average closing hours based on the star rating and the number of reviews for coffee and tea businesses in Saint Petersburg, Florida

``` SQL
SELECT  
		 b.stars AS StarRating
		,ROUND(AVG(c.Monday_Close + c.Tuesday_Close + c.Wednesday_Close + c.Thursday_Close + c.Friday_Close) / 5, 2) AS AvgClosingHour
		,SUM(b.review_count)as Num_Reviews
		
FROM yelp_closing_hours c
	
INNER JOIN yelp_business b

ON c.business_id = b.business_id

WHERE 

	 state = 'FL' AND city = 'Tampa' AND categories LIKE '%Coffee & Tea%'
	 
GROUP BY b.stars
ORDER BY StarRating DESC
LIMIT 5;
		
```

| Star Rating | Avg Closing Hour | Num Reviews |
|-------------|------------------|------------|
| 5.0         | 5.58             | 1608       |
| 4.5         | 6.22             | 10787      |
| 4.0         | 7.11             | 10270      |
| 3.5         | 7.51             | 1722       |
| 3.0         | 7.39             | 1422       |



#### Price Range vs Rating ##

The query below reflects the relationship between price range and star rating:

``` SQL
SELECT b.stars AS StarRating
	   ,ROUND(AVG(a.RestaurantsPriceRange2),1) AS PriceRange
	   
	
FROM yelp_business b

INNER JOIN yelp_attributes a

ON b.business_id = a.business_id

WHERE state = 'FL' AND city = 'Tampa' AND categories LIKE '%Coffee & Tea%'

GROUP BY b.stars
		
ORDER BY b.stars DESC;

```
| Star Rating | Price Range |
|-------------|-------------|
| 5.0         | 0.9         |
| 4.5         | 1.2         |
| 4.0         | 1.3         |
| 3.5         | 1.3         |
| 3.0         | 1.5         |
| 2.5         | 1.1         |
| 2.0         | 1.0         |
| 1.5         | 1.0         |
| 1.0         | 1.0         |


