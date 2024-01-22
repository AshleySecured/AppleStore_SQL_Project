# AppleStore_SQL_Project
Analyzes three apple store tables to examine the number of apps and their user ratings to provide recommendations to increase customer downloads, engagement, and quality of apps. These suggestions represent a market opportunity to create a quality app that addresses user needs and a potential for higher user ratings and market penetration.
![image](https://github.com/AshleyMetaData/AppleStore_SQL_Project/assets/143841434/0c376685-6330-47ba-9d7e-05faf9dfb48c)

## Creating a Combined Table
Since this dataset comes in two CSV files. The first is appleStore_description and the other is AppleStore. The appleStore_description file is broken down into four smaller tables. I want to combine these smaller tables to make it easier to work with and have my data all in one place. For this project, I am using SQLite.
```
CREATE TABLE applestore_description_combined AS

SELECT * FROM appleStore_description1

UNION ALL

SELECT * FROM appleStore_description2

UNION ALL

SELECT * FROM appleStore_description3

UNION ALL

SELECT * FROM appleStore_description4
```

Now that my data is all in one table we can get cooking!

## Stakeholder Identification
First, I want to identify the stakeholders in working with. In this pseudo-case, our stakeholder is an app developer that needs data driven insights to decide what type of app to build. To begin, we need to ask questions like:

What app categories are most popular?

What price should we set?

How can we maximize user ratings?

Now that we have our questions in mind, we know how to move forward when exploring the data.

## Exploratory Data Analysis
Like any dataset, we need to explore the data to ensure there are not missing or inconsistent values, outliers, or errors that could prevent us from reaching a conclusive goal. Identifying these issues early can save us a lot of time and effort.

```
-- check the number of unique apps in both tables--
SELECT COUNT(DISTINCT id) as UniqueAppIDs
FROM AppleStore

SELECT COUNT(DISTINCT id) as UniqueAppIDs
FROM applestore_description_combined
```
By checking the number of unique apps, we can ensure that we are working with the same applications across all tables. All the same apps here!

Next we check for any missing values in some key columns:

```
-- check for any missing values in key fields-- 
SELECT COUNT(*) AS MissingValues
FROM AppleStore
WHERE track_name IS NULL OR user_rating IS NULL OR prime_genre IS NULL

SELECT COUNT(*) AS MissingValues
FROM applestore_description_combined
WHERE app_desc IS NULL
```

Now, let's find out the number of app per genre. This gives an overview of the types of app categories in the AppleStore and help us identify any dominating genres.

```
-- number of apps per genre--
SELECT prime_genre, COUNT(*) AS NumApps
FROM AppleStore
GROUP BY prime_genre
ORDER BY NumApps DESC
```
From this analysis, the Games and Entertainment genre is dominating in the AppleStore.

I also want to look at the apps ratings. Since the Games and Entertainment genre is so popular, that could mean the app ratings are significantly high.
```
--overview of apps ratings-- 
SELECT min(user_rating) AS MinRating,
	max(user_rating) AS MaxRating,
    avg(user_rating) AS AvgRating
FROM AppleStore
```
The minimun rating is 0 and the max is 5 and the average is around 3.5.

## Data Analysis
After we've finished our exploring and gotten a good overview of the apps, I want to determine whether paid apps have a higher rating than non-paid apps.
```
--determine whether paid apps have higher ratings than free apps-- 
SELECT CASE
			when PRICE > 0 then 'Paid'
            ELSE 'Free'
       END AS App_Type,
       avg(user_rating) AS Avg_Rating
FROM AppleStore
GROUP BY App_Type
```
On average, the rating of paid apps is slightly higher than that of free apps.

Now I want to check if apps that support diverse languages have a higher rating than those who don't.
```
-- check if apps with language support have higher ratings-- 
SELECT CASE
			WHEN lang_num <10 THEN '<10 languages'
            WHEN lang_num BETWEEN 10 AND 30 THEN '10-30 languages'
            ELSE '>10 languages'
       END AS language_bucket,
       avg(user_rating) AS Avg_Rating
FROM AppleStore
GROUP BY language_bucket
ORDER BY Avg_Rating DESC
```
The middle market of about 10-30 languages have a higher rating than apps that have 30+. Therefore, we don't need to worry about having too many languages in our apps.

Let's also check the genres with low ratings. 
```
--genres with low ratings-- 
SELECT prime_genre,
	avg(user_rating) AS Avg_Rating
    FROM AppleStore
    GROUP BY prime_genre
    ORDER BY Avg_Rating ASC
```
The Catalogs, Finance, and Book categories rank significantly lower than the rest. Leading to believe that the users are not satisfied with these categories and that there might be a good opportunity to create an app in this space regarding the users concerns.

Now I want to see if there is a direct correlation between the app description length and the user rating. For this were using a CASE statement and a JOIN statement.
```
--see if correlation between app desc and user rating-- 
SELECT CASE
			WHEN length(b.app_desc) <500 THEN 'Short'
            WHEN length(b.app_desc) BETWEEN 500 AND 1000 THEN 'Medium'
            ELSE 'Long'
       END AS description_length_bucket,
       avg(a.user_rating) AS average_rating

FROM
	AppleStore AS A
JOIN
	applestore_description_combined AS B 
ON
	a.id = b.id
GROUP BY description_length_bucket
ORDER BY average_rating DESC
```
## Window Function
Great! In this next query, I want to check the top rated apps/categories using the Window Function RANK() that will essentially assign a rank to each row within a window of rows. Then we use the PARTITION BY to create a seperate window for each unqiue genre.
```
--check top rated apps / app category-- 
SELECT
		prime_genre,
		track_name,
		user_rating
FROM (
  		SELECT
 		prime_genre,
  		track_name,
  		user_rating,
  		RANK() OVER(PARTITION BY prime_genre ORDER BY user_rating DESC, rating_count_tot DESC) AS rank
  		FROM
  		appleStore
  	) AS A
WHERE
a.rank = 1
```
From these results, the Business genre is the highest ranked category with the highest ranked app being TurboScan. Followed by the Catalogs and Education.


Thank you for staying until the end! Any feedback or advice is welcomed and greatly appreciated!
