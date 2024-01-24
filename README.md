![image](https://github.com/sndaba/RPackagesForDataCleaning/assets/53818579/ff8e16cb-6e5e-411b-8afd-38fcb6882809)
[![Twitter Follow](https://img.shields.io/twitter/follow/rladiesgaborone.svg?style=social)](https://twitter.com/rladiesgaborone)

🗓 25-01-2022

 [🎥 Recording available here](https://www.youtube.com/watch?v=gZ517exPzX0&t=24s)

[R-Ladies Brisbane GitHub](https://github.com/rladies/meetup-presentations_brisbane) hosts an event for R-Ladies Gaborone on the introduction to R packages for data cleaning, namely Naniar, Janitor, Amelia and Datawizard. 

The presentation used the [2022 NYC housing dataset](https://github.com/sndaba/RPackagesForDataCleaning/blob/main/NYC_2022.csv) to demonstrate the R package for data cleaning.

## 1. Data wrangling and Exploration
[Step 1](https://github.com/sndaba/RPackagesForDataCleaning/blob/main/Step_1.R) installs and loads the packages and this is where data wrangling is done with datawizard and janitor for exploration.
```
library(Amelia)
library(naniar)
library(data.table)
library(datawizard)
library(janitor)
library(readr)
library(ggplot2)
library(dplyr)

HP <- read_csv("https://raw.githubusercontent.com/sndaba/RPackagesForDataCleaning/main/NYC_2022.csv")
View(HP)

HP <- datawizard::data_remove(HP,"latitude")        #remove data.frame,column
HP <- datawizard::data_remove(HP,"longitude")       #remove data.frame,column
HP <- datawizard::data_remove(HP,"id")              #remove data.frame,column
HP <- datawizard::data_reorder(HP,c("host_id","name")) #add the names of the cols in the new order
HP <- datawizard::data_reorder(HP,c("host_name","name")) #add the names of the cols in the new order
HP <- datawizard::data_reorder(HP,c("host_id","host_name")) #add the names of the cols in the new order
HP <- janitor::clean_names(HP) #changes to lower case
HP <- datawizard::data_rename(HP,"price","house_price") #changes col name

janitor::get_dupes(HP,colnames(HP)) #checks whether there are any duplicates
janitor::tabyl(HP,host_name) %>% adorn_pct_formatting(digit=0,affix_sign=TRUE)  #col tabulation
janitor::top_levels(as.factor(HP$house_price),5) %>%    #shows the lowest, middle and highest numeric range
  adorn_pct_formatting(digits = 0, affix_sign=TRUE)
```

## 2. Missing values and visualisation
The next [Step 2](https://github.com/sndaba/RPackagesForDataCleaning/blob/main/Step_2.R) looks for missing values and visualise the findings using naniar and ggplot.

```
naniar::any_miss(HP)           #check for NA
naniar::miss_var_summary(HP)   #NA frequency 
naniar::gg_miss_var(HP)        #NA visualization
naniar::gg_miss_upset(HP,order.by="freq")  #variable NA values relationship

ggplot2::ggplot(HP,aes(x=year_built,y=year_remod_add))+ #categorical variable
        geom_miss_point()+
        facet_wrap(~calculated_host_listings_count)+
        theme_dark()
```

## 3. Data transformation and Multiple imputation
In [Step 3](https://github.com/sndaba/RPackagesForDataCleaning/blob/main/Step_3.R), the data frame is changed to a data table using data.table and Multiple imputation is used fo missing values with Amelia.
```
drop_dt <- data.table::as.data.table(HP) #set data.frame to data.table
drop_col <- c('name',                    #drop column 
              'host_name',        
              'neighbourhood',
              'neighbourhood_group',
              'room_type')
col <- drop_dt[,!drop_col,with=FALSE]   #create new table
res.amelia <- Amelia::amelia(col,m=5)  #5 imputed data sets 
Amelia::compare.density(res.amelia,var="house_price")  #density plot to analysis
HP <- naniar::impute_mean_if(HP,.predicate = is.numeric)
naniar::any_miss(HP)         #check if there are any NA
```

## 4. Filtering, group using by and Binary search using keys.
Finally, [Step 4](https://github.com/sndaba/RPackagesForDataCleaning/blob/main/Step_4.R) sets data frame to data table, iltering rows based on conditions and data.table for the key concept for binary search to sort the data table using the key.
```
View(HP_dt <- data.table::as.data.table(HP))   #set data frame to data table
class(HP_dt)
head(HP_dt[room_type=="Private room" & house_price>181500],4) #filtering rows based on conditions
head(HP_dt[,.(host_id,host_name,name)],4)   #select given columns                                                       
head(HP_dt[neighbourhood=="Harlem",.(neighbourhood,     #select given cols by row selection
                                     number_of_reviews,
                                     availability_365
                                     )],4)
head(average <- HP_dt[,.(mean_price=mean(house_price)), #grouping using by 
                      by=neighbourhood],4)
head(chain_gang <- HP_dt[,.(.N,maximum=max(house_price),  #chaining statement
              minimum=min(house_price)  
         %>% round(2)),by=neighbourhood],4)   

#key concept for binary search. Sorts the data table by the key
data.table::setkey(HP_dt,neighbourhood)  #setting key for the data table
data.table::key(HP_dt)   #check data table key
head(HP_dt[.("Harlem")],3)     #select rows using key
room_by_neighbourhood <- HP_dt[.("Bedford-Stuyvesant"),
                               .(neighbourhood_group,neighbourhood,
                                room_type)]
head(room_by_neighbourhood)

#group using keyby
head(ans <- HP_dt["Hell's Kitchen",.(neighbourhood,
                                    max_review=number_of_reviews),
                                    keyby=availability_365],4)
```

**Useful URLs Shared during the presentation**

[CODATA webinar, importance of data cleaning](https://codata.org/initiatives/data-skills/codata-connect/webinar-series-research-skills-enhancement/webinar-4-importance-of-data-cleaning/)

CRAN: Simple Tools for Examining and Cleaning Dirty Data [Janitor](https://cran.r-project.org/web/packages/janitor/vignettes/janitor.html)

Data Structures, Summaries, and Visualisations for Missing Data [Naniar](https://naniar.njtierney.com/articles/getting-started-w-naniar.html) 

A Program for Missing Data [Amelia](https://cran.r-project.org/web/packages/Amelia/index.html) 

The validity of multiple-imputation-based analyses 
[Multiple imputation](https://ete-online.biomedcentral.com/articles/10.1186/s12982-017-0062-6#Sec5:The%20validity%20of%20multiple-imputation-based%20analyses)

Package to easily manipulate, clean, transform, and prepare your data for analysis. [DataWizard](https://easystats.github.io/datawizard/)

Provides a high-performance version of base R’s data.frame with syntax and feature enhancements for ease of use, convenience and programming speed. [Data.table](https://cran.r-project.org/web/packages/data.table/vignettes/datatable-intro.html)


