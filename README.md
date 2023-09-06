# half_year_products_report_ghana

with blood_table AS (
SELECT DISTINCT
  HF.nest_name,
  LEFT(fd.TIME_DELIVERED_LOCAL, 10) AS date,
  HF.HEALTH_FACILITY_NAME AS facility_name,
  U.PRODUCT_NAME AS products,
  pg.PRODUCT_GROUP,
  u.PRODUCT_ID,
  u.UNIT_ID,
	CASE
	  WHEN dp.CATEGORY IS NULL THEN 'Blood'
	  WHEN U.PRODUCT_NAME LIKE '%Diluent' THEN 'Vaccine Consumable'
	  WHEN U.PRODUCT_NAME LIKE '%Dropper' THEN 'Vaccine Consumable'
	  WHEN CATEGORY = 'Medicine' THEN 'Medical'
	  WHEN CATEGORY = 'Consumable' THEN 'Medical'
	  ELSE dp.CATEGORY
	END AS CATEGORY,
  U.PRICE,
	CASE
	  WHEN HF.HEALTH_FACILITY_NAME = 'Akpokope CHPS' THEN 'Agortime'
	  WHEN HF.HEALTH_FACILITY_NAME = 'Koforidua Regional HSP' THEN 'New Juaben South'
	  WHEN HF.HEALTH_FACILITY_NAME = 'Yevi CHPS' THEN 'Agortime Ziope'
	  ELSE HF.HEALTH_FACILITY_LOCALITY
	END AS district,
	HF.HEALTH_FACILITY_REGION AS region,
	U.SUPPLIER,
	U.EXPIRATION_DATE,
	U.LOT_BATCH,
	U.INTERNAL_QUANTITY,
	U.VVM_STAGE,
	U.VACCINE_DOSES AS doses,
	month(fd.TIME_DELIVERED_LOCAL) AS month,
	year(fd.TIME_DELIVERED_LOCAL) AS year
FROM BIZ.DBT.FCT_DELIVERIES fd 
LEFT JOIN biz.dbt.dim_health_facilities hf
  ON fd.HEALTH_FACILITY_KEY = hf.HEALTH_FACILITY_KEY 
LEFT JOIN BIZ.DBT.FCT_UNITS fu
  ON fd.ORDER_KEY = fu.ORDER_KEY 
LEFT JOIN biz.dbt.dim_units u
  ON fu.unit_key = u.unit_key
  AND fu.product_key = u.PRODUCT_KEY 
LEFT JOIN BIZ.DBT.DIM_PRODUCTS dp 
  ON u.PRODUCT_KEY = dp.PRODUCT_KEY 
LEFT JOIN BIZ.FIVETRAN_GOOGLE_SHEETS.PRODUCT_GROUPS pg 
  ON pg.SKU_NAME = u.PRODUCT_NAME 
WHERE hf.nest_name LIKE 'GH5%'
--AND HF.HEALTH_FACILITY_NAME LIKE '%resbyterian HC Tease%'
and fd.time_delivered_local BETWEEN '2023-01-01' AND '2023-07-01'
and HF.HEALTH_FACILITY_REGION ilike 'Volt%'
and U.PRODUCT_NAME is not null

ORDER BY 2 DESC, 3, 4
)


SELECT
  date(date) AS "Date",
  facility_name AS "Facility Name",
  district AS "District",
  region AS "Region",
  products AS "Product(s)",
--  product_group AS "Product Grouping",
  supplier AS "Supplier",
--  COUNT(*) AS "Delivered Number",
--  INTERNAL_QUANTITY,
  PRICE,
  COUNT(*) AS "Delivered Quantity",
  COUNT(*) * PRICE AS "Total Price"
FROM blood_table
WHERE category LIKE 'Blood%'
GROUP BY 1, 2, 3, 4, 5, 6, 7
ORDER BY 3,  1 DESC 
