## 🧹Cleaning Nashville Housing Data with SQL  


### 1. Standardized Date Format
- The SaleDate format is DateTime. We just want date.
```sql
UPDATE NvHousing  
SET saleDate = CAST(saleDate AS DATE)
```

- Apparantly the previous Syntax dint show changes in the table. So we try another way.
- The SaleDate format is DateTime. We just want date.
- So we Create a new Create a new column with the Date format
```sql
ALTER TABLE NvHousing
ADD SalesDateCorrected  DATE
```
```sql
UPDATE NvHousing
SET SalesDateCorrected = CAST(saleDate AS Date)
--FINDINGS: It will create another Column witht the desired Date format.
```
---


### 2. Populate Null Values
- Now we populate Null values in the PropertyAddress Column
- First we look at the adjacent columns
  SELECT *
  FROM NvHousing
- FINDINGS: Seems like PropertAddress repeats itself many times.
          - Each ParcelID is linked to a particular address.
          - Example: 207 ROSEHILL  DR, GOODLETTSVILLE is repeated many times. And everytime it is repeated it has 025 08 0 006.00 ParcelID
- Solution Logic: We are going to join this table with itself and populate the "NULL" rows of PropertyAddress with the "NOT NULL" rows of PropertyAddress Based on ParcelID. 


```sql
SELECT A.ParcelID, A.PropertyAddress, B.ParcelID, B.PropertyAddress
FROM NvHousing A
JOIN NvHousing B
     ON A.ParcelID = B.ParcelID
WHERE A.PropertyAddress IS NULL
AND B.PropertyAddress IS NOT NULL
```

```sql
UPDATE A
SET A.PropertyAddress = B.PropertyAddress
FROM NvHousing A
JOIN NvHousing B
     ON A.ParcelID = B.ParcelID
WHERE A.PropertyAddress IS NULL
AND B.PropertyAddress IS NOT NULL

```

```sql
SELECT PropertyAddress
FROM NvHousing
WHERE PropertyAddress IS NULL
--FINDINGS: There will be no Null values in PropertyAddress now
```
---


## 3. Breaking out PropertyAddress into Individual Columns (Address, City) (Method 1)
- First we create new columns
```sql
ALTER TABLE NvHousing
ADD PropertySplitAddress nvarchar(255),
    PropertySplitCity nvarchar(255)
```


```sql
UPDATE NvHousing
SET PropertySplitAddress = LEFT(PropertyAddress, CHARINDEX(',', PropertyAddress) -1),
    PropertySplitCity = RIGHT(PropertyAddress, LEN(PropertyAddress) - CHARINDEX(',', PropertyAddress))
--FINDINGS: This will populate the new columns
```
---

## 4. Breaking out OwnerAddress into Individual Columns (Address, City, State) (Method 2) 
```sql
ALTER TABLE NvHousing
ADD OwnerSplitAddress NVARCHAR(255),
    OwnerSplitCity NVARCHAR(255),
    OwnerSplitState NVARCHAR(255)
```

```sql
UPDATE NvHousing
SET OwnerSplitAddress = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 3),
    OwnerSplitCity = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 2), 
    OwnerSplitState = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 2) 
-- FINDINGS: That will populate the new columns
```
---


## 5. Change Y and N to Yes and No in "SoldAsVacant" field
```sql
SELECT DISTINCT(SoldAsVacant)
FROM NvHousing
--Some of these rows have 'Y', 'N', 'Yes', 'No'
```

```sql
UPDATE NvHousing
SET  SoldAsVacant = CASE 
                        WHEN SoldAsVacant = 'Y' THEN 'Yes'
                        WHEN SoldAsVacant = 'N' THEN 'No'
                        ELSE SoldAsVacant
                    END
FROM NvHousing
--FINDINGS: That will replace Y with Yes and N with No
```
---


## 6. Remove Duplicates
- CTE - TO IDENTIFY DUPLICATE ROWS
```sql
WITH RowNumCTE AS (
SELECT *,
             ROW_NUMBER() OVER (
                                 PARTITION BY 
	                               ParcelID, 
				       PropertyAddress, 
				       SaleDate, 
				       SalePrice,
				       LegalReference
				       ORDER BY UniqueID) AS row_num  
     FROM NvHousing
)
SELECT *
FROM RowNumCTE
WHERE row_num >1
ORDER BY PropertyAddress
```


```sql
WITH RowNumCTE AS (
SELECT *,
             ROW_NUMBER() OVER (
                                 PARTITION BY 
	                               ParcelID, 
				       PropertyAddress, 
				       SaleDate, 
				       SalePrice,
				       LegalReference
				       ORDER BY UniqueID) AS row_num  
     FROM NvHousing
)
DELETE
FROM RowNumCTE
WHERE row_num >1
--FINDINGS: That will delete the duplicate rows
```
---



## 7. Delete Unwanted Columns
```sql
ALTER TABLE NvHousing  
DROP COLUMN PropertyAddress, OwnerAddress, SaleDate
```
---

## 8. Change Column Names
```sql
EXEC sp_rename 'NvHousing.SalesDateCorrected', 'SalesDate', 'COLUMN'
EXEC sp_rename 'NvHousing.PropertySplitAddress', 'PropertyAddress', 'COLUMN'
EXEC sp_rename 'NvHousing.PropertySplitCity', 'PropertyCity', 'COLUMN'
EXEC sp_rename 'NvHousing.OwnerSplitAddress', 'OwnerAddress', 'COLUMN'
EXEC sp_rename 'NvHousing.OwnerSplitCity', 'OwnerCity', 'COLUMN'
EXEC sp_rename 'NvHousing.OwnerSplitState', 'OwnerState', 'COLUMN'
```







