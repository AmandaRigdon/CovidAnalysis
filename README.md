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
Findings:

* Sadly as time went on, 5% of covid cases were resulting in death in some countries like Iraq.
* Even with vaccines getting introduced in Dec. of 2020, the death percentage vs. cases still continued to rise, probably from the new strains that were surfacing as time went on.

What about covid cases in the United States in comparison to its population?

```sql
SELECT Location, date, total_cases, Population, (total_cases/Population)*100 as Percentage_of_Population
FROM coviddeaths2
WHERE Location = "United States"
ORDER BY 1,2;
```

Findings:

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

Findings:

* Not so surpising that the United States is at the top of this list with a max of 103,436,829 cases.
* China is very close behind with 99,373,219 cases but because its population is so much bigger than the United States, it only reprsents around 7% of their population. In the United States, it's closer to 30% which was also found above.
* The data also shows the struggle that I remember Italy facing during the Covid outbreak: 45% of their population was infected and their highest case count was 59,037,472.

Which countries had the highest death count per population?

```sql
SELECT Location, MAX(Total_deaths) as Total_Death_Count
FROM coviddeaths2
WHERE continent is not NULL
GROUP BY Location
ORDER BY Total_Death_Count DESC;
```

Findings:

* Like above, the United States stays #1 with a total death count of 1,193,165 people.
* While China had a lot of cases as well as shown above, their death count is signifcantly lower at 122,304. This is probably due to their very strict quarantine measures.

I also ran a query to see what the death count looked like by continent:

```sql
SELECT location, MAX(Total_deaths) as Total_Death_Count
FROM coviddeaths2
WHERE continent is NULL
GROUP BY location
ORDER BY Total_Death_Count DESC;
```
Findings:

* Europe had the highest death count at 2,102,483 with North America second in line with a death count of 1,671,178.
* The lowest death count is in the continent of Oceania at 32,918. This continent includes Australia and many other island nations!

Which country had the highest vaccinations?

```sql
SELECT location, MAX(total_vaccinations) as Max_Vax
from vaccinations2
WHERE continent is not NULL
GROUP BY location
ORDER BY Max_Vax DESC;
```

Findings:

* China and India have the highest number of vaccinations, which makes sense due to their population size. The United States is third.
* Some countries have no vaccine info at all, like North Korea and Puerto Rico.
* Locations such as Greenland, Barbados, or Maldives have relatively low vaccinations.

I wanted to get an overview of total cases, deaths, and death percentage so I ran the following query:

```sql
SELECT SUM(new_cases) as Total_Cases, SUM(new_deaths) as Total_Deaths, SUM(new_deaths)/SUM(new_cases)*100 as Death_Percentage
FROM coviddeaths2
WHERE continent is not NULL
ORDER BY 1,2;
```

Findings:

* There have been 775,935,057 covid cases and 7,060,988 deaths worldwide. That means we have lost .91% of our population due to Covid.

In the next query, I joined both the coviddeaths and vaccination tables to see the number of vaccines administered side by side with a country's population.

```sql
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
FROM coviddeaths2 AS dea
JOIN vaccinations2 AS vac
	ON dea.location = vac.location 
    AND dea.date = vac.date
WHERE dea.continent is not NULL
ORDER BY 1,2,3;
```

Findings:

* In the United States starting in May of 2023, there is no new vaccine information at all. Maybe this information stopped being tracked? This seems to be a trend among other countries as well, like China and France.
* This information may not be obtained anymore possibly because Covid has died down a lot compared to when the outbreak started

I also made a query to see a rolling count of vaccinations by date:

```sql
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(vac.new_vaccinations) OVER (Partition by dea.location ORDER BY dea.location, dea.date) as Rolling_People_Vaccinated
FROM coviddeaths2 AS dea
JOIN vaccinations2 AS vac
	ON dea.location = vac.location 
    AND dea.date = vac.date
WHERE dea.continent is not NULL
ORDER BY 2,3;
```

To query off the new Rolling_People_Vaccinated column, I created a CTE as well as a temporary table to show an alternative way to get the same information.

CTE:

```sql
WITH Pop_Vs_Vac (Continent, Location, Date, Population, New_Vaccinations, Rolling_People_Vaccinated)
AS 
(
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(vac.new_vaccinations) OVER (Partition by dea.location ORDER BY dea.location, dea.date) as Rolling_People_Vaccinated
FROM coviddeaths2 AS dea
JOIN vaccinations2 AS vac
	ON dea.location = vac.location 
    AND dea.date = vac.date
WHERE dea.continent is not NULL
-- ORDER BY 2,3
)
SELECT *, (Rolling_People_Vaccinated/population)*100 as Ratio_Vaccinated
from Pop_Vs_Vac;
```

Temporary Table:

```sql
DROP TABLE IF EXISTS Percent_Population_Vaccinated;
CREATE TEMPORARY TABLE Percent_Population_Vaccinated 
(
Continent nvarchar(255),
Location nvarchar(255),
Date datetime,
Population bigint,
New_vaccinations int,
Rolling_People_Vaccinated bigint
);
```

```sql
INSERT INTO Percent_Population_Vaccinated 
(
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(vac.new_vaccinations) OVER (Partition by dea.location ORDER BY dea.location, dea.date) as Rolling_People_Vaccinated
FROM coviddeaths2 AS dea
JOIN vaccinations2 AS vac
	ON dea.location = vac.location 
    AND dea.date = vac.date
WHERE dea.continent is not NULL
-- ORDER BY 2,3
);
SELECT *, (Rolling_People_Vaccinated/Population)*100
from Percent_Population_Vaccinated;
```

Findings:

* Because we are at a point in time where people have gotten vaccines multiple times, the Ratio_Vaccinated column in some areas shows over 100%. Instead of percent of people vaccinated, I interpreted it more as a ratio of doses per population. For example, a value such as 156 would represent 156 vaccines per 100 people.

Looking to see if median age and total deaths have any relationship:

```sql
SELECT dea.continent, dea.location, MAX(dea.total_deaths), vac.median_age
FROM coviddeaths2 AS dea
JOIN vaccinations2 AS vac
	ON dea.location = vac.location
    AND dea.date = vac.date
WHERE dea.continent is not NULL
GROUP BY dea.location
ORDER BY 3 DESC;
```

Findings:

* Median age doesn't necessarily seem to mean a higher death count. UK had a median age of 40.8, but the US had a much higher death count with a median age of 38.3.
* In many cases, such as in European countries, the median age is higher but has a lower death count. This could also be more due to the total population of European countries being smaller than countries such as the United States.
* Population density (influencing the likelihood of being exposed) is also something to consider. India has 1.429 billion people with many covid deaths, but a low median age of 28.2.

