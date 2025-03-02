# 🧹 SQL Data Cleaning NashvilleHousing Data


This project focuses on cleaning the Nashville Housing dataset using SQL. The dataset originally had 56,478 rows and 19 columns. The following cleaning steps were performed:


## 1. 🗓️ Standardized Date Format 
- Used `CAST()` to convert `SaleDate` from `DateTime` to `Date` format.

## 2. 🔄 Populated Null Values 
- Used `JOIN` to fill missing `PropertyAddress` values using `ParcelID` as a reference.

## 3. 🏡 Split Property Address 
- Used `CHARINDEX()`, `LEFT()`, and `RIGHT()` to extract `Address` and `City` into separate columns from `PropertyAddress`.

## 4. 🏠 Split Owner Address 
- Used `PARSENAME()` and `REPLACE()` to extract `Address`, `City`, and `State` into separate columns from `OwnerAddress`.

## 5. ✅ Standardized SoldAsVacant Field 
- Used `CASE WHEN` to convert 'Y' and 'N' values to 'Yes' and 'No'.

## 6. 🗑️ Removed Duplicates 
- Used `ROW_NUMBER() OVER (PARTITION BY ...)` to identify and delete duplicate rows.

## 7. ⚙️ Dropped Unused Columns 
- Used `ALTER TABLE DROP COLUMN` to remove `PropertyAddress`, `OwnerAddress`, and `SaleDate`.

## 8. ✍️ Renamed Columns 
- Used `sp_rename` to rename newly created columns for clarity.

This cleaned dataset is now ready for further analysis and visualization.
