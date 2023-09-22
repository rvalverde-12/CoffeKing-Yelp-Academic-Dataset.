# **CoffeKing_Yelp_Academic_Dataset**

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

library(jsonlite)
library(dplyr)
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
I've created separate tables for "hours" and "attributes" in adherence to normalization principles. This approach optimizes storage, simplifies querying for specific data, and supports scalability.
```ruby
attributes <- business[c("business_id", "attributes" )]
hours <- business[c("business_id", "hours" )]
business <- business[, !(colnames(business) %in% c("attributes", "hours"))
```
### Exporting into CSV files
```ruby
write.csv(business,file= 'C:/Users/Lipsky/yelp_business.csv')
write.csv(checkin,file= 'C:/Users/Lipsky/yelp_checkin.csv')
write.csv(review,file= 'C:/Users/Lipsky/yelp_review.csv')
write.csv(tip,file= 'C:/Users/Lipsky/yelp_tip.csv')
write.csv(user,file= 'C:/Users/Lipsky/yelp_user.csv')
write.csv(attributes,file= 'C:/Users/Lipsky/yelp_attributes.csv')
write.csv(hours,file= 'C:/Users/Lipsky/yelp_hours.csv')
```

