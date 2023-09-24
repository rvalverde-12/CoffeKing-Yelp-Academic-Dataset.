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
 ```ruby
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
colnames(hours_df) <- gsub("^hours\\.", "", colnames(hours_df))
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
### Exporting into CSV files
```ruby
write.csv(business,file= 'C:/Users/Lipsky/yelp_business.csv')
write.csv(checkin,file= 'C:/Users/Lipsky/yelp_checkin.csv')
write.csv(review,file= 'C:/Users/Lipsky/yelp_review.csv')
write.csv(tip,file= 'C:/Users/Lipsky/yelp_tip.csv')
write.csv(user,file= 'C:/Users/Lipsky/yelp_user.csv')
write.csv(attributes_df,file= 'C:/Users/Lipsky/yelp_attributes.csv')
write.csv(hours_df,file= 'C:/Users/Lipsky/yelp_hours.csv')
```

### Entity Relationship Diagram (ERD)
<br>
        <p align="left">
          <img src="images/ERD Cows Foot Notation.jpeg" width="800"/>
        </p>
      </li>
    </ul>
  </li>

