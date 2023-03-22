## Importing the Data
I had to remove erroneous symbols from the column headers using google sheets, as they could not be properly represented in the UTF-8 character encoding format, and was preventing the data from being imported
Original titl� --> Original title
Genr� --> Genre

### Creating table to import data

~~~sql
CREATE TABLE data_cleaning (
	imdb_title_id varchar(50),
	original_title varchar(250),
	release_year varchar(50),
	genre varchar(250),
	duration varchar(50),
	country varchar (100),
	content_rating varchar(50),
	director varchar(250),
	blank varchar(250),-- I had to create this column so that the empty column could also be imported
	income varchar(50),
	votes varchar(50),
	score varchar(50)
);
~~~

I then imported the csv file into this newly created table

## Removing empty columns

~~~sql
ALTER TABLE data_cleaning
DROP COLUMN blank;
~~~

## Removing empty rows in the table

~~~sql
DELETE FROM data_cleaning
WHERE imdb_title_id IS NULL
  AND original_title IS NULL
  AND release_year IS NULL
  AND genre IS NULL
  AND duration IS NULL
  AND country IS NULL
  AND content_rating IS NULL
  AND director IS NULL
  AND income IS NULL
  AND votes IS NULL
  AND score IS NULL
;
~~~

## Cleaning the release_year column

~~~sql
--deleting month and days from YYYY-MM-DD formatted dates to keep only the year
UPDATE data_cleaning
SET release_year = SUBSTRING(release_year FROM 1 FOR 4)
WHERE release_year LIKE '____-__-__';
~~~

- Cleaning other formats in the release_year column one by one 
- I had to do this step manually because each of these rows had data formats that were all different from eachother.

~~~sql
UPDATE data_cleaning
SET release_year = 1972
WHERE imdb_title_id = 'tt0068646';

UPDATE data_cleaning
SET release_year = 2008
WHERE imdb_title_id = 'tt0468569';

UPDATE data_cleaning
SET release_year = 2003
WHERE imdb_title_id = 'tt0167261';

UPDATE data_cleaning
SET release_year = 1976
WHERE imdb_title_id = 'tt0073486';

UPDATE data_cleaning
SET release_year = 1946
WHERE imdb_title_id = 'tt0034583';

UPDATE data_cleaning
SET release_year = 1999
WHERE imdb_title_id = 'tt0137523';

UPDATE data_cleaning
SET release_year = 2004
WHERE imdb_title_id = 'tt0167260';

UPDATE data_cleaning
SET release_year = 1966
WHERE imdb_title_id = 'tt0060196';

UPDATE data_cleaning
SET release_year = 1966
WHERE imdb_title_id = 'tt0043014';
~~~

## Cleaning the duration column

~~~sql
--trimming everything to only 3 characters, because that is the max number of characters in the correct data
UPDATE data_cleaning
SET duration = SUBSTRING(duration FROM 1 FOR 3);

--removing all non numeric values
UPDATE data_cleaning
SET duration = REGEXP_REPLACE(duration, '[^0-9]+', ' ')
WHERE duration ~ '[^0-9]+';
~~~

## Cleaning the country column

~~~sql
--making USA and US consistent
UPDATE data_cleaning
SET country = 
    CASE 
        WHEN country = 'US' THEN 'USA'
        WHEN country = 'US.' THEN 'USA'
        ELSE country
    END;

--replacing "West Germany" with "Germany", as we do not need the regional details
UPDATE data_cleaning SET country = REPLACE(country, 'West Germany', 'Germany');

--correcting spelling errors with 'New Zealand'
UPDATE data_cleaning
SET country = 'New Zealand'
WHERE country LIKE '%New%';

--removing non-alphabetic values from this column
UPDATE data_cleaning
SET country = regexp_replace(country, '[^a-zA-Z ]', '', 'g');
~~~

## Content rating column

~~~sql
UPDATE data_cleaning
SET content_rating = 
    CASE 
        WHEN content_rating = 'PG' THEN 'PG-13'
        WHEN content_rating = 'Not Rated' THEN ''
        WHEN content_rating = 'Unrated' THEN ''
		WHEN content_rating = '#N/A' THEN ''
		ELSE content_rating
    END;
~~~

## Income column

~~~sql
--renaming column to specify currency because I want to remove symbols from the column
ALTER TABLE data_cleaning 
RENAME COLUMN income TO income_usd;

--removing symbol
UPDATE data_cleaning 
SET income_usd = REPLACE(income_usd, '$ ', '') 
WHERE income_usd LIKE '$%';

--correcting incorrect values
UPDATE data_cleaning
SET income_usd = REPLACE(income_usd, 'o', '0')
WHERE income_usd = '4o8,035,783';

UPDATE data_cleaning 
SET income_usd = REPLACE(income_usd, ',', '') 
~~~

## Votes columns

~~~sql
UPDATE data_cleaning
SET votes = REPLACE(votes, '.', '');
~~~

## Score column

~~~sql
UPDATE data_cleaning
SET score = TRIM(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(score, ',', '.'), '+', ''), 'f', ''), ':', '.'), '..', '.'));

UPDATE data_cleaning
SET score = REPLACE (score, '8.7.', '8.7')
WHERE score = '8.7.';

UPDATE data_cleaning
SET score = REPLACE(score, '8.7e-0', '8.7')
WHERE score = '8.7e-0';
~~~

# Setting empty rows to NULL data type

~~~sql
--duration column
UPDATE data_cleaning
SET duration = NULL
WHERE duration = ' ';

--content rating columns
UPDATE data_cleaning
SET content_rating = NULL
WHERE content_rating = '';
~~~

## Type casting all the columns with their correct data types

~~~sql
--release year column
ALTER TABLE data_cleaning
ALTER COLUMN release_year TYPE numeric(4,0)
USING release_year::numeric(4,0);

--duration column
ALTER TABLE data_cleaning
ALTER COLUMN duration TYPE numeric(3,0)
USING duration::numeric(3,0);

--income column
ALTER TABLE data_cleaning
ALTER COLUMN income_usd TYPE numeric
USING income_usd::numeric;

--votes column
ALTER TABLE data_cleaning
ALTER COLUMN votes TYPE numeric
USING votes::numeric;

--score column
ALTER TABLE data_cleaning
ALTER COLUMN score TYPE float
USING score::float;
~~~

## Viewing the cleaned table

~~~sql
SELECT *
FROM data_cleaning
~~~
