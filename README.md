## Overview

While there are a plethora of data analysis on Covid, I wanted to give it a shot on my own and parse through the data to see if anything surprised me.

Questions I wanted to answer:

1. How many covid cases were there in relation to deaths? Worldwide and in the US?
2. How many covid cases are there in relation to the population in the US?
3. What countries have the highest infection rates?
4. How many deaths were there by country?
5. What country had the highest vaccinations?
6. Is there a relationship between median age and death count?
7. Is there a relationship between GDP and vaccinations/death count?
8. As covid rose, did cardiovascular death rate rise?

### The Datasets 

The datasets I used are here from Our World in Data:

[Covid Datasets](https://ourworldindata.org/covid-deaths)

These datasets are very large, encompassing covid information from when covid started until 8/4/2024.

Because of this, I'll be using MySQL to do all the data analysis and Tableau to visualize some of the data. 

### Data Cleaning 

Because MySQL will not import tables with integer columns that had blanks, I had to use Python to quickly enter "NULL" for any blank values.

```python 
import pandas as pd
df = pd.read_csv("/Users/amandarigdon/Downloads/CovidDeaths.csv") 
df
df2 = df.fillna("Null")
df2.to_csv("Desktop/CovidVaccinations.csv")

df = pd.read_csv("/Users/amandarigdon/Downloads/Vaccinations.csv") 
df
df2 = df.fillna("Null")
df2.to_csv("Desktop/Vaccinations2.csv")
```

Once that was done, I imported the data into MySQL, paying close attention to change the data types of the columns to "int" and "double" when needed.

### EDA

I started out selecting the data that I wanted to initially work with:

```sql
SELECT Location, date, total_cases, new_cases, total_deaths
FROM coviddeaths2
ORDER BY 1,2;
```
Now to answer one of my first questions: how many covid cases are there in relation to covid deaths?

```sql
SELECT Location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 as death_percentage
FROM coviddeaths2
ORDER BY 1,2;
```
Some insights:

* Sadly as time went on, 5% of covid cases were resulting in death in some countries like Iraq.
* Even with vaccines getting introduced in Dec. of 2020, the death percentage vs. cases still continued to rise, probably from the new strains that were surfacing as time went on.

What about covid cases in the United States in comparison to its population?

```sql
SELECT Location, date, total_cases, Population, (total_cases/Population)*100 as Percentage_of_Population
FROM coviddeaths2
WHERE Location = "United States"
ORDER BY 1,2;
```

Some insights:

* By 8/4/2024 (the most current this data is), a staggering 30% of the US population has been infected with Covid at least once.
* Even with vaccines, the percentage was steadily climbing throughout the years.

What countries have the highest infection rates?

```sql
SELECT Location, Population, MAX(total_cases) as Highest_Infection_Count, MAX((total_cases/Population))*100 as Percent_Population_Infected
FROM coviddeaths2
WHERE continent is not NULL
GROUP BY Location, Population
ORDER BY Highest_Infection_Count DESC;
```

