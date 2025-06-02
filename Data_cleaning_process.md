# Removing duplicates

## Data cleaning

 Check if layoffs is populated

```sql
select * from layoffs;
```

 This set is layoffs from around the world starting 2021.  it has the name of the company that did the layoff. the location, the industry, and the total number of layoffs.  The percentage of how many they laid off. the date and the stage that the company is in, whether it's a series B post IPO or they don't know and their raised funds.

1. Remove Duplicates if there are many
2.  Standardize the data (fix issues with spelling)
3.  Null values or blank values (try to populate them if we can)
5.  Remove any Columns (dont remove data from the raw data

- create table `layoff_staging` will create the same schema/structure

The table is empty as this point

```sql
CREATE TABLE layoffs_staging
LIKE layoffs;
```

see the table

```sql
select * from layoffs_staging;
```

now we have to insert the data from layoffs into `layoffs_staging`

```sql

insert layoffs_staging
select * from layoffs;

```

 Now we have 2 different tables

---

## Removing duplicates step-by-step

```sql

-- a. identifying duplicates
select * from layoffs_staging;

```

Since companies have layoffs in the same location and industry
 The date and total layoffs would be different

 this will assign a row number to each row in the table

```sql

select *,
ROW_NUMBER() OVER(partition by company, location,
industry, total_laid_off, date, percentage_laid_off)
AS row_num
from layoffs_staging;
```

### create a cte

```sql

with duplicate_cte as (
select *,
ROW_NUMBER() OVER(partition by company,
industry, total_laid_off, date, percentage_laid_off)
AS row_num
from layoffs_staging
)
select * from duplicate_cte
where row_num >1;
```


We'll need to partition by every  column

```sql
with duplicate_cte as (
select *,
ROW_NUMBER() OVER(partition by company,
industry, total_laid_off, date, percentage_laid_off,location,
stage, country,funds_raised_millions)
AS row_num
from layoffs_staging
)
select * from duplicate_cte
where row_num >1;
```

1. Let's create a table called layoffs_staging2 and add an extra column row_num with data type INT.
2. we're going to insert The duplicates from layoffs_staging into it
3. Check if the table was populated.

```sql

CREATE TABLE layoffs_staging2 (
company text,
location text,
industry text,
total_laid_off int DEFAULT NULL,
percentage_laid_off text,
date text,
stage text,
country text,
funds_raised_millions int DEFAULT NULL,
row_num INT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```

```sql

SELECT * FROM layoffs_staging2;
```


```sql

INSERT INTO layoffs_staging2
SELECT *,
ROW_NUMBER() OVER( PARTITION BY
company, location, industry, total_laid_off, percentage_laid_off,
date, stage, country, funds_raised_millions)
AS row_num
FROM layoffs_staging;
```

- Now we have inserted a copy of all the columns from layoffs_staging
into layoffs_staging 2
- added a new column called row_num
- It gives a number to each row within groups of duplicates based on
on what we partitioned by
- The first matching row in each group gets row_num =1
The next gets 2 and so on/
- DELETE THE DUPLICATES

```sql

DELETE FROM layoffs_staging2
WHERE ROW_NUM > 1;
```

Check if deleted
```sql

SELECT * FROM layoffs_staging2;
```


# Standardize data

```sql
SELECT company, trim(company)
FROM layoffs_staging2;
```

```sql
UPDATE layoffs_staging2
SET company = trim(company);
```

This sorts the result by the first column in the
select statement

```sql
select distinct industry from layoffs_staging2
order by 1;
```

- We will find that crypto currency and crypto are the same

```sql

select * from layoffs_staging2
where industry like 'Crypto%';
```

Update the industry from cryptocurrency to crypto

```sql

update layoffs_staging2
set industry = 'Crypto'
where industry like 'Crypto%';
```

Let's now look at the location

```sql

select distinct location from layoffs_staging2
order by 1;
```

Let's now look at the country

```sql

select distinct country from layoffs_staging2
order by 1;
```

 we can see there are two `United States` one with a `.` at the end

```sql

SELECT *
FROM layoffs_staging2
WHERE country LIKE 'United States%'
ORDER BY 1;
```

We need to trim cause there are spaces after states

```sql

SELECT distinct country, trim(country)
from layoffs_staging2
order by 1;
```

 trim won't remove the dot at the end, this will

```sql

select distinct country, trim(trailing '.' FROM country)
from layoffs_staging2
order by 1;
```

Let's update it

```sql

update layoffs_staging2
set country = trim(trailing '.' FROM country)
where country like 'United States%';
```

```sql
SELECT *
FROM layoffs_staging2;
```

The date column has a data type of text, which is not good

```sql
select date, str_to_date(date, '%m/%d/%Y')
from layoffs_staging2;
```

Changing data type from text to date

```sql

ALTER TABLE layoffs_staging2
MODIFY COLUMN date DATE;
```
Let's update
```sql

 update layoffs_staging2
 set date = str_to_date(date, '%m/%d/%Y');
```

# Look for and populate null and black values

- Checking total_laid_off

```sql
select * from layoffs_staging2
where total_laid_off is null;
```

Let's check total_laid_off and percentage_laid_off
If they are both null, they're useless to us

```sql
select * from layoffs_staging2
where total_laid_off is null
and percentage_laid_off is null;
```

See where the industry is null or blank

```sql

select *
from layoffs_staging2
where industry is null or
industry = '';
```

Let's see if any of the result have one that is populated
We'll try `Airbnb` first

```sql

*select *
from layoffs_staging2
where company = 'Airbnb';*
```

We can see that in the result, we have one that has data (populated)
and one that has null and blank values
So we will update the one with empty values

```sql
select * from layoffs_staging2 as t1
join layoffs_staging2 as t2
on t1.company = t2.company
and t1.location = t2.location
where (t1.industry is null or t1.industry = '')
and t2.industry is not null;
```

- we'll set the blanks to null first

```sql

update layoffs_staging2
set industry = null
where industry = '';
```

- let's run the update

```sql
update layoffs_staging2 t1
join layoffs_staging2 t2
on t1.company = t2.company
set t1.industry = t2.industry
where (t1.industry is null or t1.industry ='')
and t2.industry is not null;
```

Now let's select everything

```sql
select *
from layoffs_staging2
where industry is null or
industry = '';
```

It looks like Bally's Interactive still has a null industry
```sql

select *
from layoffs_staging2
where company like 'Bally%';
```

We are deleting the `total_laid_off` and `percentage_laid_off` rows where both values are `null`

```sql
DELETE
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;
```

We are checking if they are actually deleted.

```sql
select *
from layoffs_staging2
where total_laid_off is null
and percentage_laid_off is null;

select * from layoffs_staging2;
```

Now we need to drop the `row_num` column because it is unnecessary  

```sql
alter table layoffs_staging2
drop column row_num;
```

This is now clean data.
