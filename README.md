# **CoffeeKing_Yelp_Academic_Dataset**

## Background
CoffeeKing, a startup coffee company, aims to offer a unique and inclusive experience. They seek insights from Yelp reviews and user data to inform location selection, operating hours, business enhancements, and loyalty-focused marketing strategies.

I'm choosing this project over the other ones because I'm highly intrigued by the potential insights I can found through an in-depth analysis of Yelp data. My extensive experience in the merchant cash advance industry has given me a deep appreciation for understanding the intricacies of how businesses operate.

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
- I'm primarily using SQLite for most of the analysis, as it aligns with the SQL course's focus.

#### Stores by City and AVG Reviews and Ratings

``` SQL
SELECT     b.State
           ,b.city
           ,COUNT(b.name) AS Stores
           ,ROUND(AVG(num_reviews), 2) AS AVG_Reviews
           ,ROUND(AVG(r.stars), 2) AS AVG_Stars

FROM       yelp_business b

LEFT JOIN  (

	SELECT business_id
	       ,COUNT(review_id) AS num_reviews
		   ,stars
        FROM   yelp_review
	GROUP BY
			business_id
	) r ON b.business_id = r.business_id

        WHERE categories  LIKE '%Coffee & Tea%'

GROUP BY    b.city
HAVING
           AVG_Stars BETWEEN 3.0 AND 3.5
ORDER BY Stores DESC;
LIMIT 5;
```

| State | City         | Stores     | AVG_Reviews | AVG_Stars |
|-------|--------------|------------|-------------|-----------|
| MO    | St. Louis    | 95         | 54.51       | 3.48      |
| FL    | Clearwater   | 80         | 44.91       | 3.49      |
| LA    | Metairie     | 65         | 55.42       | 3.37      |
| FL    | Largo        | 39         | 37.72       | 3.28      |
| NJ    | Cherry Hill  | 35         | 30.4        | 3.49      |


#### Percentage of Closed Businesses By State and Filtered by Category Coffee & Tea

``` SQL
SELECT 
    SUM(CASE WHEN is_open = 0 THEN 1 ELSE 0 END) AS Closed_businesses,
    COUNT(*) AS Total_count,
    ROUND((SUM(CASE WHEN is_open = 0 THEN 1 ELSE 0 END) * 100.0 / COUNT(*)), 2) AS Percentage_closed,
    state
FROM yelp_business
WHERE categories LIKE '%Coffee & Tea%'
GROUP BY state
ORDER BY Total_count DESC
LIMIT 5;
```

| Closed Businesses | Total Count | Percentage Closed | State |
|-------------------|-------------|--------------------|-------|
| 501               | 1725        | 29.04%             | PA    |
| 239               | 1106        | 21.61%             | FL    |
| 135               | 493         | 27.38%             | TN    |
| 141               | 472         | 29.87%             | LA    |
| 131               | 467         | 28.05%             | IN    |
