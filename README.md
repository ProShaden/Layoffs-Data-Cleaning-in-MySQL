# Layoffs-Data-Cleaning-in-MySQL
This project demonstrates the process of cleaning and transforming a layoffs dataset using MySQL. The dataset contains details of employee layoffs and required cleaning to handle missing values, outliers, duplicate records, and normalization.

## Dataset Source  
The dataset used in this project is from [Alex The Analyst's MySQL YouTube Series](https://github.com/AlexTheAnalyst/MySQL-YouTube-Series/blob/main/layoffs.csv).  
You can download the original dataset from the link above and follow along with the cleaning process in MySQL.

# Creating a Staging Table for Data Cleaning

Before beginning the data cleaning process, I created a staging table to ensure that the raw dataset remains unaltered. This is crucial for preserving the integrity of the original data and maintaining the ability to reprocess or recover the original dataset if needed.

Steps followed:

   1. Created a new table (layoffs_staging) with the same structure as the original table (layoffs) using the CREATE TABLE ... LIKE statement. This ensures that the new table matches the structure of the original data without copying the actual data.
```sql
CREATE TABLE layoffs_staging 
LIKE layoffs;
```
  2. Copied the data from the original layoffs table into the layoffs_staging table using the INSERT INTO ... SELECT statement. This duplicates all the rows from the original table, creating a full copy that can be safely worked on.
```sql
INSERT INTO layoffs_staging 
SELECT * 
FROM layoffs;
```
By doing this, I can now work with the layoffs_staging table for all cleaning and transformation steps, leaving the original layoffs table intact.

# Step 1: Removing Duplicates

🎯 Why Remove Duplicates?

Before performing any data analysis, it's crucial to eliminate duplicate records to ensure accurate insights. We will use ROW_NUMBER() to identify and remove duplicates based on all relevant columns.

Method: Using CTE to Assign Row Numbers

We first create a Common Table Expression (CTE) to assign a row_num to each record. If row_num > 1, it means the record is a duplicate.
```sql
WITH duplicate_cte AS (
    SELECT *, 
           ROW_NUMBER() OVER (
               PARTITION BY company, location, industry, total_laid_off, 
                            percentage_laid_off, 'date', stage, country, funds_raised_millions
           ) AS row_num
    FROM layoffs_staging
)
DELETE
FROM duplicate_cte
WHERE row_num > 1;
```
❌ Why This Won’t Work?

MySQL does not allow DELETE operations directly on a CTE (duplicate_cte), so we need a workaround.

📌 Workaround: Creating a New Table

Since we cannot delete from a CTE, we create a new table layoffs_staging2 to store the cleaned dataset.

1. Create a New Table
```sql
CREATE TABLE layoffs_staging2 (
    company TEXT,
    location TEXT,
    industry TEXT,
    total_laid_off INT DEFAULT NULL,
    percentage_laid_off TEXT,
    date TEXT,
    stage TEXT,
    country TEXT,
    funds_raised_millions INT DEFAULT NULL,
    row_num INT
);
```

2.  Insert Data with Row Numbers
```sql
INSERT INTO layoffs_staging2
SELECT *, 
       ROW_NUMBER() OVER (
           PARTITION BY company, location, industry, total_laid_off, 
                        percentage_laid_off, 'date', stage, country, funds_raised_millions
       ) AS row_num
FROM layoffs_staging;
```

🗑 Deleting Duplicate Records
```sql
DELETE FROM layoffs_staging2 WHERE row_num > 1;
```
✅ Final Check: Verify Clean Data
To confirm that duplicates have been removed, we select the cleaned data:
```sql
SELECT * FROM layoffs_staging2;
```
- Before Removing Duplicates Sample:
  ![Before Cleaning Sample](https://github.com/user-attachments/assets/80c195c5-f2fa-433e-a386-1567918f9124)

 - After Removing Duplicates Sample:
 ![After Cleaning Sample](https://github.com/user-attachments/assets/3bf3d816-9f30-4342-803e-facd2e208da4)

# Step 2: Standardizing Data
🎯 Why Standardize Data?

Before performing any analysis, it's crucial to ensure data consistency. Inconsistent values (such as extra spaces, typos, or different naming conventions) can lead to inaccurate insights. Below are a few examples of how we can clean and standardize the dataset.

1- Removing Extra Spaces

Sometimes, data contains unwanted leading or trailing spaces. We use the TRIM() function to remove them.

- Identifying the Issue
```sql
SELECT company, TRIM(company) 
FROM layoffs_staging2;
```
- Output:
![Trim -Before Updating](https://github.com/user-attachments/assets/b44c7319-b101-453b-b64f-d2bf4569125b)

- Fix: Updating the Table
```sql
UPDATE layoffs_staging2
SET company = TRIM(company);
```
- Output:
![Trim - After Updating](https://github.com/user-attachments/assets/d58cf1d8-9214-4f9a-8f94-d80eb162c4ed)

2- Standardizing Industry Names

We found that the Crypto industry is recorded in multiple ways (e.g., Crypto, Crypto Tech, etc.). We want all records to follow the same format: "Crypto".

- Checking the Unique Values
 ```sql
SELECT DISTINCT industry
FROM layoffs_staging2
ORDER BY 1;
```
- Output:
![Multiple Industries](https://github.com/user-attachments/assets/efacd82e-4b8a-4379-ba8d-7aca3044dad3)

- Fix: Updating Inconsistent Values
 ```sql
UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry LIKE 'Crypto%';
```
- Output:
![One Industry](https://github.com/user-attachments/assets/e0ea8f50-df7f-4de6-9a16-60eb68c2dd95)

3- Fixing Location Name Issues
I noticed that Düsseldorf is written inconsistently. I standardize it to a single format.
- Checking the Unique Values
 ```sql
SELECT DISTINCT location
FROM layoffs_staging2
ORDER BY 1;
```
- Output:
![Name Isuue - Before](https://github.com/user-attachments/assets/b9554b7d-32a1-4a81-b750-909f87aa3f1e)

- Fix: Updating Location Names
 ```sql
UPDATE layoffs_staging2
SET location = 'Dusseldorf'
WHERE location LIKE 'D__sle%';
```
- Output:
![Name Isuue- After](https://github.com/user-attachments/assets/db66ee2b-ca6a-4fa2-b27a-d65344d0e2b6)

# Step 3: Deleting Null Values
Before removing NULL values, let's identify them:
 ```sql
SELECT *  
FROM layoffs_staging2  
WHERE total_laid_off IS NULL  
AND percentage_laid_off IS NULL;
```
 Output:
![Null values](https://github.com/user-attachments/assets/1343e65a-eaca-4c46-9c9b-f55d2867927b)
Now, delete rows where both total_laid_off and percentage_laid_off are NULL:
 ```sql
DELETE  
FROM layoffs_staging2  
WHERE total_laid_off IS NULL  
AND percentage_laid_off IS NULL;
```
- After completing the cleaning process, we should remove the row_num column to restore the dataset to its original structure (without duplicates but cleaned).
 ```sql
ALTER TABLE layoffs_staging2  
DROP COLUMN row_num;
```

# Cleaned Database: 
![Cleaned Database](https://github.com/user-attachments/assets/6d31a2b0-ee0a-4d2d-b919-46852e78bc23)

## Connect with Me  
If you found this project useful, feel free to:  
- ⭐ Star this repository  
- Connect with me on [LinkedIn](https://www.linkedin.com/in/shaden-alsuhaim/)  
- Share your feedback or improvements!


