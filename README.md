---
title: "Data Assignment #3: Data Wrangling"
author: "Harlee Scordo"
date: "`r Sys.Date()`"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```
# Background and Instructions
**Goal**: In this data analysis report, you will estimate which region in the United States had the highest rate of people (over age 12) who consume alcohol in 2018. You will also answer a couple extra questions using the same dataset. To do this, you will go through all the steps of data analysis from data analysis, cleaning, preprocessing, modeling/description, and creating a graph to communicate your findings. Each step builds on previous steps. **However**, if you get stuck on a step, you are not completely at a loss! At multiple points in this document, you can choose to load a correct version of what your data frame should look like. If you are stuck on an earlier section, then you can choose to load the new, correct data frame and go from that point. To complete this assignment, simply work through the rest of this document and fill in code or text as requested when you see an Exercise by filling in the ...s in the code chunks. Not text answers are required on this assignments.

# Alcohol Use Data Analysis

## Setup Libraries
Set up the libraries required for visualizations.
```{r}
#If you get an error saying dplyr or ggplot2 have not been found, then you have not installed them yet! To install them, "uncomment" the next two lines by deleting the hash/pound symbol and try again! Warnings are OK!
#install.packages("dyplr")
#install.packages("ggplot2")
library(dplyr)
library(ggplot2)
```

## Data acquisition
This code gets both of the necessary datasets (it required Internet access!).
```{r}
id <- '11UTI52VxABvWDqqdLB0axJfm9_vQuDUm'
NSDUH <- read.csv(sprintf("https://docs.google.com/uc?id=%s&export=download", id))
id <- '1PUpU17nBGw7-qfkqVH5O5zrcHB0x9vfp'
CDC_Geographic <- read.csv(sprintf("https://docs.google.com/uc?id=%s&export=download", id))
```

### Exercise 1 Combine data
The NSDUH data frame contains information about drug use across US States and DC. The CDC_Geography data frame contains a guide to the geographic "divisions" that the CDC uses to categorize different US states and territories.

```{r}
NSDUH <- merge(NSDUH, CDC_Geographic, by = "State")

```

## Data Exploration
Have a look at the data!
```{r}
glimpse(NSDUH)
```

```{r, include=FALSE}
## Data Checkpoint 1
# If you would like to make sure you have the correct version of the dataframe from this point forward, then delete the # on the next two lines and run this code chunk.
id <- '14aWcteTT2SwqYM2OjCEvd6_5fLAR_uVq'
NSDUH <- read.csv(sprintf("https://docs.google.com/uc?id=%s&export=download", id))
```

## Data Cleaning
### Exercise 2: Look only at 2018
For this analysis, we only want data from the year 2018. The following code should use a data wrangling tool to make sure NSDUH only has 2018 data.
```{r}
NSDUH <- NSDUH %>% 
  filter (Year == "2018")

glimpse(NSDUH)
```

## Preprocessing
We ultimately want to look at the *overall* rate of alcohol consumption in states and regions. Our dataset only includes rates for specific age ranges. We'll need to do some math and transformations to find the overall rate! This'll take a couple preprocessing steps to create a variable called AlcoholRate.

### Exercise 3: Calculate Total Population
Create a new variable in NSDUH called "PopTotal" that includes the total population in each state, *combining* or adding together the 12-17 year-olds, the 18-25 year-olds, and the 26+ year-olds.
```{r}
NSDUH <- NSDUH %>%
  mutate(PopTotal = `Pop12_17` + `Pop18_25` + `Pop26Plus`)

glimpse(NSDUH)
```

### Exercise 4: Calculate Total Alcohol Users
Add a new variable to NSDUH called "AlcoholTotal". This should include an estimate for the *total* number of people over the age of 12 (from all three age groups) that consumed alcohol last month. The data wrangling function may look similar to Exercise 3, but with different math. As a hint, to calculate an estimate for the number of 12-17 year-olds that drink alcohol, you can multiply the total number of 12-17 year-olds by the rate of alcohol use for 12-17 year olds. Some numbers might include decimals -- it doesn't really make sense to have .5 or .3 people, but for these estimates it's OK that the number of people isn't always whole numbers! They're imperfect estimates!
```{r}
NSDUH <- NSDUH %>%
  mutate(AlcoholTotal = (`Pop12_17` * (`Alc12_17` / 100)) +
                   (`Pop18_25` * (`Alc18_25` / 100)) +
                   (`Pop26Plus` * (`Alc26Plus` / 100)))

glimpse(NSDUH)
```

### Exercise 5: Calculate Alcohol Use Rate for All Residents Over 12
Create a new variable in NSDUH called "AlcoholRate". This variable should have the rate of alcohol consumption for all people over the year of 12 in each state. Recall that the overall rate of consumption should be the total number of people who drink alcohol divided by the total population.
```{r}
NSDUH <- NSDUH %>%
  mutate(AlcoholRate = (AlcoholTotal / PopTotal) * 100)

glimpse(NSDUH)
```
```{r, include=FALSE}
## Data Checkpoint 2
# If you would like to make sure you have the correct version of the dataframe from this point forward, then delete the # on the next two lines and run this code chunk.
id <- '1BuoWWQMQRI5NZaJCfDsLn-PjvF-PINQM'
NSDUH <- read.csv(sprintf("https://docs.google.com/uc?id=%s&export=download", id))
```

## Modeling and Describing Data
Now, let's find out which *regions* have the highest and lowest average rates of alcohol consumption. To do this, we can look at the average rates of alcohol consumption for all states in each of the 4 CDC regions of the US (West, Northeast, South, Midwest).

### Exercise 6: Find averages per Region
Create a new data frame called NSDUH_Regions. This data frame should have 4 rows (one for each Region). It should have 2 variables: "Region" should be the region name and "AverageAlcoholRate" should be the mean alcohol rate for all states within the particular region.
```{r}
NSDUH_Regions <- NSDUH %>%
  group_by(Region) %>%
  summarise(AverageAlcoholRate = mean(AlcoholRate, na.rm = TRUE)) %>%
  ungroup()

glimpse(NSDUH_Regions)
```

### Exercise 7: Visualize averages per Region
Create a *bar plot* that visualizes the NSDUH_Regions data in a sensible fashion.
```{r}
NSDUH_Regions %>%
  ggplot(aes(x = Region, y = AverageAlcoholRate)) +
  geom_bar(stat = "identity", fill = "darkgreen") +
  labs(title = "Average Alcohol Rates per Region",
       x = "Region",
       y = "Average Alcohol Rate")
```
# Additional Questions
Excellent! Based on the graph above, you should be able to figure out which regions had the highest and lowest rates of alcohol consumption (at least in 2018)! The document from the top until this point is basically what you would expect to see in a real data analysis. For the next 3 exercises, you're going to answer a couple extra questions that are less directly tied to the original question.

### Exercise 8: Find the State with the Lowest Rate of Alcohol Use
Rearrange the NSDUH data frame so that all the states are ordered so that the state with the *lowest* overall alcohol consumption rate is at the top of the data frame and the state with the *highest* overall alcohol consumption rate is at the bottom.
```{r}
NSDUH <- NSDUH %>% 
  arrange(AlcoholRate)

glimpse(NSDUH)
```

# Additional Questions
Excellent! Based on the graph above, you should be able to figure out which regions had the highest and lowest rates of alcohol consumption (at least in 2018)! The document from the top until this point is basically what you would expect to see in a real data analysis. For the next 3 exercises, you're going to answer a couple extra questions that are less directly tied to the original question.

### Exercise 8: Find the State with the Lowest Rate of Alcohol Use
Rearrange the NSDUH data frame so that all the states are ordered so that the state with the *lowest* overall alcohol consumption rate is at the top of the data frame and the state with the *highest* overall alcohol consumption rate is at the bottom.
```{r}
NSDUH <- NSDUH %>% 
  arrange(AlcoholRate)

glimpse(NSDUH)
```

### Exercise 9: Estimate of Total Minor Alcohol Users in all US States
What is the best estimate you can calculate of the *total* number of 12-17 year-olds who consume alcohol in the United States (minus the territories and DC)? Use datawrangling tools to create a new data frame called TotalAlcYouth. TotalAlcYouth should have 1 row and 1 variable. That variable should be called "TotalYouthDrinkers". TotalYouthDrinkers should be an estimate of the total, combined number of youth who drink alcohol in all the states in the United States. To create this dataframe, you will need to use multiple data wrangling steps. Make sure to remove the District of Columbia from the calculation, as it is not a US State.
```{r}

TotalAlcYouth <- NSDUH %>%
  filter(State != "District of Columbia") %>%
  mutate(EstimatedYouthDrinkers = `Pop12_17` * (`Alc12_17` / 100)) %>%
  summarise(TotalYouthDrinkers = sum(EstimatedYouthDrinkers)) 

```

### Exercise 10: Comparing Minor Alcohol and Marijuana Rates 
Is it typical for the rate of alcohol use to be greater than the rate of marijuana use in Western states? Figure this out by creating a data frame called "RegionalDifferences". RegionalDifferences should include 4 rows and 2 variables: "Region" include the name of the 4 CDC US regions; "AverageAlcoholOverMarijuana" should list the *median* amount that the rate of youth (12-17) alcohol consumption is *greater than* the rate of youth marijuana consumption. To do this you will need to use multiple data wrangling steps. Note that to compare rates of consumption for marijuana and alcohol, you will need to subtract rates of consumption for those two drugs at some point.
```{r}
RegionalDifferences <- NSDUH %>%
  mutate(AlcoholMinusMarijuana = `Alc12_17` - `Mar1_17`) %>%
  group_by(Region) %>%
  summarise(AverageAlcoholOverMarijuana = median(AlcoholMinusMarijuana))

colnames(RegionalDifferences)[1] <- "Region"


```




---
title: "NEU 290 Data Assignment 2: Visualizing Birth Data"
author: "Harlee Scordo"
output:
  html_document: default
---

```{r, include=FALSE}
# Do not edit this code block/chunk!
knitr::opts_chunk$set(
  echo = TRUE, message=FALSE, warning = FALSE, fig.width = 16/2, 
  fig.height = 9/2
)
```

# Visualization and Interpretation Exercises
This document explores several questions about maternal and newborn health by visualizing data in the 2004 North Carolina births registry. Make sure to replace all questions with your answers and to fill in code in each code chunk with a ... in it.

##Setup Libraries
Set up the libraries required for visualizations.
```{r}
#If you get an error saying dplyr or ggplot2 have not been found, then you have not installed them yet! To install them, "uncomment" the next two lines by deleting the hash/pound symbol and try again!
#install.packages("dyplr")
#install.packages("ggplot2")
library(dplyr)
library(ggplot2)
library(tidyverse)
```
## Load Data
The following loads a dataset that was released in 2004 from the state of North Carolina and includes information about about the maternal and newborn health from 800 randomly chosen live births in that state.

```{r}
# Run this to load the data! It's OK if it doesn't make sense yet!
id <- '19vUudLrACnxKpP-X0M_6wGdf2Suid1_a'
births <- read.csv(sprintf("https://docs.google.com/uc?id=%s&export=download", id))
```
## Explore the Data
Have a look at the variables recorded in this dataset. Each row is one birth. 

```{r}
glimpse(births)
```
Most of the variables have transparent names, but some are a little less apparent:

- fage - father's age at time of birth
- mage - mother's age at time of birth
- weeks - duration of pregnancy in weeks
- visits - the number of prenatal care visits to a clinician that the mother attended during pregnancy
- gained - the maximum amount of weight gained by the mother during pregnancy (in pounds)
- weight - the weight of the newborn infant (in pounds)
- gender - the gender identified on NC birth registry for the newborn infant

## Exercise 0: Example
This "Exercise" is an example to show how you could format exercises 1-5. Your exercises should look similar to this, with one or two code chunks, followed by your answer to the provided question (having deleted the original question text).
```{r}
births %>% 
  ggplot(aes(x = weeks, y = visits)) + 
  geom_point() +
    labs(x = "Length of pregnancy (in weeks)",
         y = "Number of Prenatal Visits",
         title = "Length of Pregnancy and Prenatal Care")
```         

As seen in the above figure, there is perhaps a week relationship between length of pregnancy and number of visits to prenatal care appointments. Mother's with shorter pregnancies rarely attended that many visits, but this is not surprising as there was simply less time to attend or schedule such visits. Some mothers who had longer pregnancies also attended few or even zero appointments.

## Exercise 1: Pregnancy Duration
```{r}

births %>% 
  ggplot(aes(x = weeks, y = visits, color = premie)) + 
  geom_point() +
    labs(x = "Length of pregnancy (in weeks)",
         title = "Pregnancy Duration")

# What is a typical length of pregnancy in the North Carolina birth dataset? What is the earliest and latest ages that you would consider "typical" based on the graph and why? Replace this text with your answer (around 2 sentences)

```

I would consider anywhere from 35 to 43 weeks to be a typical length of pregnancy given the density in that range, even given the label of "premie" 41-42 seems to be the most births. . 
The earliest is 20 and the latest is 45, but the earliest typical would be 35, latest 43. 



## Exercise 2: Birthweight and Recorded Gender

```{r}
births %>% 
  ggplot(aes(x = weight, y = gender)) + 
  geom_point(alpha = 0.2) +
    labs(x = "Birthweight",
         y = "Gender",
         title = "Birthweight and Recorded Gender")
```


```{r}
births %>% 
   ggplot(aes(x = weight, fill = lowbirthweight)) +
    geom_histogram() +
    facet_wrap(~ gender) +
     labs (x = "Birthweight",
           title = "Birthweight and Recorded Gender")

#Does the typical birthweight for male newborns or female newborns seem higher in this dataset? Does the difference seem large or small? How can you tell from the graph (write 2-3 sentences)?

```

The typical birthweight of males seems to be higher than females. The females tend to be in the 5.5-7.5 ranger, whereas the density of male weight is in the 7-9 range.The difference is noticeable, but not outrageous. 


## Exercise 3: Birthweight, Mother's Maturity, & Mother's Smoking Habit

```{r}
births %>% 
  ggplot(aes(x = weight, y = mature)) + 
  geom_point(alpha = 0.2) +
    labs(x = "Birthweight",
         y = "Mother's Maturity",
         title = "Birthweight, Mother's Maturity, & Mother's Smoking Habit")
```
```{r}
births %>% 
  ggplot(aes(x = weight, y = habit)) + 
  geom_point(alpha = 0.2) +
    labs(x = "Birthweight",
         y = "Mother's Smoking Habit",
         title = "Birthweight, Mother's Maturity, & Mother's Smoking Habit")
```


```{r}

births %>% 
  ggplot(aes(x = weight)) + 
  geom_histogram() +
  facet_wrap(~ habit) +
  labs (x = "Birthweight",
        title = "Birthweight & Mother's Smoking Habit")


#Smoking has been associated with lower birthweights. Additionally, studies have found that mature mother (older mothers) have newborns with lower birthweights than younger mothers. In the North Carolina births data, does it seem like smoking status or the maturity of the mother is a more reliable predictor of low birthweight? In other words, is the difference between birthweights greater between smokers and non-smokers or between younger and mature mothers? How can you tell from these two graphs? Replace this text with your answer (1-2 sentences)
```
```{r}
births %>% 
  ggplot(aes(x = weight)) + 
  geom_histogram() +
  facet_wrap(~ mature) +
  labs (x = "Birthweight",
        title = "Birthweight & Mother's Maturity")
```


I feel like the maturity is a better indicator of birthweight, just due to the volume of data collected. There doesn't seem to be a lot of data on smoker's versus non smokers. However, the small amount of data we do have does indicate a smaller bithweight average. However, age seems to keep a pretty consistent weight of above 6 pounds. 


## Exercise 4: Birthweight and length of pregnancy
```{r}

births %>% 
  ggplot(mapping = aes(x = weeks, y = weight, color = premie)) + 
  geom_point() +
    labs(x = "Weeks of pregnancy",
         y = "Birthweight",
         title = "Birthweight and Length of pregnancy")


#In general, the longer a pregnancy, the longer we would higher we would expect the birthweight of a newborn. Does this rule of thumb apply for both full-term and premie pregnancies, or just one or the other? How can you tell? Replace this text with your answer (1-2 sentences).
```

It does seems like the longer the pregnancy, the higher the birthweight. That goes for both premie and full term. If you wee to draw a line between the data sets, it would consistently climb up. 


## Exercise 5: Birthweight and length of pregnancy
```{r}
births %>% 
  ggplot(aes(x = weight, y = weeks, color = premie)) + 
  geom_point(alpha = 0.2) +
    labs(x = "Birthweight",
         y = "Length of prengancy",
         title = "Birthweight and length of pregnancy")
```

```{r}

births %>% 
  ggplot(aes(x = premie, y = weight, color = lowbirthweight)) + 
  geom_boxplot() +
  geom_point() +
    labs(x = "Premie or Full term",
         y = "Birthweight",
         title = "Birthweight and Full term vs Premie")

#Approximately what proportion of full-term babies in NC were identified as having low birthweight? What about premie babies? As a proportion of the whole population of live births, would it be unusual to have a premie newborn that was born with a typical (not low) birth weight? Replace this text with your answers
```

A very small amount of babies were classified as having low birthweight when it comesto full-term babies. Whereas the majority of premie babies were classified as being of low birthweight. However, given the large amount of premie babies that were not classified as having low birthweight, it would not be outrageous to think of premie babies being born at a normal weight. 

