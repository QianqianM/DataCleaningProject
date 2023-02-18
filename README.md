-- mySQL script

SELECT *
FROM HousingData
ORDER BY ParcelID;

-- there are rows with differ UniqueID, same ParcelID, one with PropertyAddress, another one does not
-- populate property address
SELECT a.ParcelID, a.PropertyAddress, b.ParcelID, b.PropertyAddress, IFNULL(a.PropertyAddress,b.PropertyAddress)
FROM HousingData a
JOIN HousingData b ON a.ParcelID = b.ParcelID AND a.UniqueID <> b.UniqueID
WHERE a.PropertyAddress IS NULL;

UPDATE HousingData a
JOIN HousingData b ON a.ParcelID = b.ParcelID AND a.UniqueID <> b.UniqueID
SET a.PropertyAddress = IFNULL(a.PropertyAddress,b.PropertyAddress)
WHERE a.PropertyAddress IS NULL;

-- Breaking out address into (Address, City, State)
SELECT PropertyAddress
FROM HousingData;

SELECT SUBSTRING_INDEX(PropertyAddress,',' , 1) as Address,
SUBSTRING_INDEX(PropertyAddress,',' , -1) as city
From HousingData;

-- add columns then insert data
ALTER TABLE HousingData
ADD COLUMN  PropertySplitAddress NVARCHAR(300)
ADD COLUMN  PropertySplitCity NVARCHAR(300);

UPDATE HousingData
SET PropertySplitAddress=SUBSTRING_INDEX(PropertyAddress,',' , 1),
PropertySplitCity=SUBSTRING_INDEX(PropertyAddress,',' , -1);


SELECT *
FROM HousingData
ORDER BY ParcelID;


SELECT OwnerAddress
FROM HousingData;

SELECT SUBSTRING_INDEX(OwnerAddress,',' , 1) Address,
SUBSTRING_INDEX(SUBSTRING_INDEX(OwnerAddress,',' , 2),',' , -1) City,
SUBSTRING_INDEX(OwnerAddress,',' , -1) State
From HousingData;

ALTER TABLE HousingData
ADD COLUMN OwnerSplitAddress NVARCHAR(300),
ADD COLUMN OwnerSplitCity NVARCHAR(300),
ADD COLUMN OwnerSplitState NVARCHAR(300);


UPDATE HousingData
SET OwnerSplitAddress=SUBSTRING_INDEX(OwnerAddress,',' , 1),
OwnerSplitCity=SUBSTRING_INDEX(SUBSTRING_INDEX(OwnerAddress,',' , 2),',' , -1),
OwnerSplitState=SUBSTRING_INDEX(OwnerAddress,',' , -1)

-- change Y and N to Yes and No
SELECT DISTINCT(SoldAsVacant)
From HousingData;

SELECT REPLACE(SoldAsVacant,'N','No') 
From HousingData
WHERE SoldAsVacant='N';

-- OR use CASE WHEN
SELECT SoldAsVacant,
CASE
    WHEN SoldAsVacant='Y' THEN 'Yes'
    WHEN SoldAsVacant='N' THEN 'No'
    ELSE SoldAsVacant
		END
From HousingData;

UPDATE HousingData
SET SoldAsVacant=CASE
    WHEN SoldAsVacant='Y' THEN 'Yes'
    WHEN SoldAsVacant='N' THEN 'No'
    ELSE SoldAsVacant
		END
		
-- any DUPLICATEs
SELECT *, COUNT(*) OVER (
PARTITION BY ParcelID,
PropertyAddress,
SaleDate,
SalePrice,
LegalReference) AS row_num
From HousingData
ORDER BY row_num DESC;


WITH CTE AS (
SELECT *, COUNT(*) OVER (
PARTITION BY ParcelID,
PropertyAddress,
SaleDate,
SalePrice,
LegalReference) AS row_num
FROM HousingData
ORDER BY row_num DESC
)
SELECT *
FROM CTE
WHERE row_num=1;

-- THE END

