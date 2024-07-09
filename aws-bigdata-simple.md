# BigData AWS
USING:
- AWS Crawlers
- AWS Glue
- AWS Athena
- AWS S3 Buckets

## Creating an iceberg table

ref: https://medium.com/@datacodingnet/getting-started-with-apache-iceberg-tables-using-aws-glue-custom-connector-eb657b925e66

```bash
CREATE DATABASE datacoding;
```

```bash
CREATE TABLE datacoding.iceberg_table (
  id int,
  data string,
  category string)
LOCATION 's3://datacoding-iceberg-table/iceberg-folder' 
TBLPROPERTIES (
  'table_type'='ICEBERG',
  'format'='parquet'
);
```

```bash
INSERT INTO datacoding.iceberg_table VALUES (1,'a','c1');
```

---

## transferring data from external table to the newly created iceberg table(table values must match)

```bash
INSERT INTO northwindiceberg.categories
SELECT categoryid, categoryname, description
FROM northwind.categories;
```
