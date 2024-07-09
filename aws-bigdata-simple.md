# BigData AWS
USING:
- AWS Glue('crawlers', 'database' and 'tables')
- AWS Athena('querry editor')
- AWS S3 Buckets('buckets with .csv files of each table')


1. Export tables to .csv
2. store them in an s3 bucket in separate folders(each csv file goes inside its pair folder. ex: Categories.csv -> Categories folder)
   ![image](https://github.com/rlcosta177/personal-projects/assets/154469533/060a78bb-4b99-4669-8a03-10008106397f)
3. Create a database in aws glue
4. create a crawler associated with the s3 bucket(ex: s3://rlcosta-northwind/) and run it(dont remember details) | choose the IAM Role: 'LabRole' and select the database created before
5. go to athena querry editor and select the database and you'll see the created tables. You won't be able to update and delete existing records because it uses Hive by default as of today
6. Migrating from hive to iceberg(create new database, create identical tables but with iceberg as default, copy data from the hive database to the new iceberg database)

## Creating an iceberg table
ref: https://medium.com/@datacodingnet/getting-started-with-apache-iceberg-tables-using-aws-glue-custom-connector-eb657b925e66

```bash
CREATE DATABASE datacoding;
```

```bash
CREATE TABLE northwind-iceberg.categories (
  categoryid int,
  categoryname string,
  description string)
LOCATION 's3://rlcosta-northwind' 
TBLPROPERTIES (
  'table_type'='ICEBERG',
  'format'='parquet'
);
```

```bash
INSERT INTO northwind-iceberg.categories VALUES (1,'Marisco','Lagosta');
```

---

## transferring data from external table to the newly created iceberg table(table values must match)

```bash
INSERT INTO northwindiceberg.categories
SELECT categoryid, categoryname, description
FROM northwind.categories;
```
