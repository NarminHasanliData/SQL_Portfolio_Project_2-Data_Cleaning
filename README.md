# SQL_Portfolio_Project_2-Data_Cleaning
## AlexTheAnalyst Bootcamp

``` sql
-- Data Cleaning

SELECT *
FROM Nashville..NashvilleHousing

-----------------------------------------------------------------------
--Standardize date format

SELECT Saledate, 
	CONVERT(Date, Saledate)
FROM Nashville..NashvilleHousing

UPDATE NashvilleHousing
SET Saledate = CONVERT(Date, Saledate)

ALTER TABLE NashvilleHousing
ADD SaleDateConverted Date;

UPDATE NashvilleHousing
SET SaleDateConverted = CONVERT(Date, Saledate)

SELECT SaleDateConverted, CONVERT(Date, Saledate)
FROM Nashville..NashvilleHousing


-----------------------------------------------------------------------
-- Populate Property Address Data

SELECT PropertyAddress
FROM Nashville..NashvilleHousing
WHERE PropertyAddress is NULL

SELECT *
FROM Nashville..NashvilleHousing
WHERE PropertyAddress is NULL

SELECT *
FROM Nashville..NashvilleHousing
ORDER BY ParcelID

SELECT *
FROM Nashville..NashvilleHousing NashvilleRaw
JOIN Nashville..NashvilleHousing NashvilleCleaned
	ON NashvilleRaw.ParcelID = NashvilleCleaned.ParcelID
	AND NashvilleRaw.[UniqueID] <> NashvilleCleaned.[UniqueID]


SELECT NashvilleRaw.ParcelID, 
	NashvilleRaw.PropertyAddress, 
	NashvilleCleaned.ParcelID, 
	NashvilleCleaned.PropertyAddress,
	ISNULL(NashvilleRaw.PropertyAddress, NashvilleCleaned.PropertyAddress)
FROM Nashville..NashvilleHousing NashvilleRaw
JOIN Nashville..NashvilleHousing NashvilleCleaned
	ON NashvilleRaw.ParcelID = NashvilleCleaned.ParcelID
	AND NashvilleRaw.[UniqueID] <> NashvilleCleaned.[UniqueID]
WHERE NashvilleRaw.PropertyAddress IS NULL

UPDATE NashvilleRaw
SET PropertyAddress = ISNULL(NashvilleRaw.PropertyAddress, NashvilleCleaned.PropertyAddress)
FROM Nashville..NashvilleHousing NashvilleRaw
JOIN Nashville..NashvilleHousing NashvilleCleaned
	ON NashvilleRaw.ParcelID = NashvilleCleaned.ParcelID
	AND NashvilleRaw.[UniqueID] <> NashvilleCleaned.[UniqueID]
WHERE NashvilleRaw.PropertyAddress IS NULL


-----------------------------------------------------------------------
-- Breaking out Address

SELECT PropertyAddress
FROM Nashville..NashvilleHousing

SELECT
	SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress)-1) AS Address,
	SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress)+1, LEN(PropertyAddress)) AS Address2
FROM Nashville..NashvilleHousing


ALTER TABLE Nashville..NashvilleHousing
ADD PropertySplitAddress NVarChar(255);

UPDATE Nashville..NashvilleHousing
SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress)-1)


ALTER TABLE Nashville..NashvilleHousing
ADD PropertySplitCity NVarChar(255);

UPDATE Nashville..NashvilleHousing
SET PropertySplitCity = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress)+1, LEN(PropertyAddress))

SELECT *
FROM Nashville..NashvilleHousing


--Owner Address Break Out

SELECT OwnerAddress
FROM Nashville..NashvilleHousing

SELECT
	PARSENAME(REPLACE(OwnerAddress, ',', '.'), 3),
	PARSENAME(REPLACE(OwnerAddress, ',', '.'), 2),
	PARSENAME(REPLACE(OwnerAddress, ',', '.'), 1)
FROM Nashville..NashvilleHousing

--Adding Columns 

ALTER TABLE Nashville..NashvilleHousing
ADD OwnerSplitAddress NVarChar(255);

UPDATE Nashville..NashvilleHousing
SET OwnerSplitAddress = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 3)

ALTER TABLE Nashville..NashvilleHousing
ADD OwnerSplitCity NVarChar(255);

UPDATE Nashville..NashvilleHousing
SET OwnerSplitCity = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 2)

ALTER TABLE Nashville..NashvilleHousing
ADD OwnerSplitState NVarChar(255);

UPDATE Nashville..NashvilleHousing
SET OwnerSplitState = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 1)

SELECT *
FROM Nashville..NashvilleHousing


-----------------------------------------------------------------------
-- Change Y and N to Yes and No in Sold as Vacant column

SELECT SoldAsVacant, COUNT(soldasvacant)
FROM Nashville..NashvilleHousing
GROUP BY SoldAsVacant

SELECT DISTINCT(SoldAsVacant)
FROM Nashville..NashvilleHousing

ALTER TABLE Nashville..NashvilleHousing
ALTER COLUMN SoldAsVacant CHAR(10)

SELECT SoldAsVacant,
		CASE WHEN SoldAsVacant = '0' THEN 'No'
		     WHEN SoldAsVacant = '1' THEN 'Yes'
			 ELSE SoldAsVacant 
			 END
FROM Nashville..NashvilleHousing

UPDATE Nashville..NashvilleHousing
SET SoldAsVacant = CASE WHEN SoldAsVacant = '0' THEN 'No'
			 WHEN SoldAsVacant = '1' THEN 'Yes'
			 ELSE SoldAsVacant 
			 END

SELECT SoldAsVacant, COUNT(soldasvacant)
FROM Nashville..NashvilleHousing
GROUP BY SoldAsVacant


-----------------------------------------------------------------------
-- Remove Duplicates

SELECT *, 
		ROW_NUMBER() OVER (
			PARTITION BY ParcelID,
		 			PropertyAddress,
					SalePrice,
					SaleDate,
					LegalReference
					ORDER BY UniqueID) AS row_num

FROM Nashville..NashvilleHousing
ORDER BY ParcelID

-- Creating CTE

WITH Row_Num_CTE AS (
SELECT *, 
		ROW_NUMBER() OVER (
			PARTITION BY ParcelID,
					PropertyAddress,
					SalePrice,
					SaleDate,
					LegalReference
					ORDER BY UniqueID) AS row_num

FROM Nashville..NashvilleHousing
)
SELECT * 
--DELETE
FROM Row_Num_CTE
WHERE Row_Num > 1
--ORDER BY PropertyAddress


-----------------------------------------------------------------------
--Deleting Unused Columns

SELECT *
FROM Nashville..NashvilleHousing

ALTER TABLE Nashville..NashvilleHousing
DROP COLUMN OwnerAddress, TaxDistrict, PropertyAddress
```
