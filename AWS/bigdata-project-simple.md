# BigData AWS
USING:
- AWS Glue('crawlers', 'database' and 'tables')
- AWS Athena('querry editor')
- AWS S3 Buckets('buckets with .csv files of each table')

---

1. Export tables to .csv

---

2. store them in an s3 bucket in separate folders(each csv file goes inside its pair folder. ex: Categories.csv -> Categories folder)
   ![image](https://github.com/rlcosta177/personal-projects/assets/154469533/060a78bb-4b99-4669-8a03-10008106397f)

---

3. Create a database in aws glue
   ![image](https://github.com/rlcosta177/personal-projects/assets/154469533/8883f6a8-89a4-41fe-b6ef-72ed42a373ca)

---

4. create a crawler associated with the s3 bucket(ex: s3://rlcosta-northwind/) and run it(dont remember details) | choose the IAM Role: 'LabRole' and select the database created before
   ![image](https://github.com/rlcosta177/personal-projects/assets/154469533/ac13ea2d-c661-42e6-b2ec-ece267d58129)
   ![image](https://github.com/rlcosta177/personal-projects/assets/154469533/6252bc7f-c8ab-4f50-84bd-88f75e82eaa6)
   ![image](https://github.com/rlcosta177/personal-projects/assets/154469533/ae0dab1d-060f-4f76-9d8e-e6f4da41d313)
   ![image](https://github.com/rlcosta177/personal-projects/assets/154469533/5a77ec04-0f7f-473f-9e2b-ac275cf5589b)
   ![image](https://github.com/rlcosta177/personal-projects/assets/154469533/371f0fae-d20d-42f3-8407-6f91490623a7)

---

5. go to athena querry editor, settings and add your s3 bucket to the athena querry(This bucket should be different from the bucket containing the csv files. Used solely for temp files)
   ![image](https://github.com/rlcosta177/personal-projects/assets/154469533/94af2a7c-a7cf-4465-b946-9d6a25bfcf32)
   ![image](https://github.com/rlcosta177/personal-projects/assets/154469533/e4b4ef4b-cee4-4900-a83f-a4a2288860d2)

---

6. Select the database and you'll see the created tables. You won't be able to update and delete existing records because it uses Hive by default as of today
   ![image](https://github.com/rlcosta177/personal-projects/assets/154469533/72b1cede-501d-47ef-9eba-a367e1be32de)


---

7. Migrating from hive to iceberg(create new database OR use the same database, create identical tables but with iceberg as default, copy data from the hive database to the new iceberg database)
      
      ### Creating an iceberg table
      ref: https://medium.com/@datacodingnet/getting-started-with-apache-iceberg-tables-using-aws-glue-custom-connector-eb657b925e66
      
      Code goes into athena querry editor
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
      
      ### transferring data from external table to the newly created iceberg table(table values must match)
      
      ```bash
      INSERT INTO northwindiceberg.categories
      SELECT categoryid, categoryname, description
      FROM northwind.categories;
      ```


---


### Alternative to point 7:
- create a new bucket OR create a new folder in the existing bucket and specify it in the code(ex: rlcosta-northwind/new-folder/Employees)
- and create a new table specifying that bucket as storage and selecting the values and rows from another table(copies everything from the hive table to the new iceberg table)
  ```bash
  CREATE TABLE northwindiceberg.employees
  WITH(table_type = 'ICEBERG', format='parquet', location='s3://rlcosta-northwind-iceberg/Employees/', is_external=false) as
  SELECT * FROM northwind.employees;
  ```
