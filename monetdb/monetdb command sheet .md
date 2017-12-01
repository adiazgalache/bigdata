# Practice queries for MonetDB

## Starting the database
```sql
sudo monetdbd start ~/mydbfarm
```

### Enter the database
```shell
pending
```

## Queries

### Create a schema
```sql
CREATE SCHEMA "metalData" AUTHORIZATION "monetdb";
```

### Create a table
```sql
CREATE TABLE "metalData"."factTable"
(
   date_day DATE,
   country varchar(255),
   company varchar(255),
   production integer,
   family varchar(255),
   product_type varchar(255),
   steel varchar(255),
   carbon decimal(18,10),
   hardness integer,
   temper_rolling varchar(255),
   condition varchar(255),
   formability integer,
   strength integer,
   non_ageing varchar(255),
   surface_finish varchar(255),
   surface_quality varchar(255),
   enamealibility integer,
   bc varchar(255),
   bf varchar(255),
   bt varchar(255),
   bw_me varchar(255),
   bl varchar(255),
   m varchar(255),
   chrom varchar(255),
   phos varchar(255),
   cbond varchar(255),
   marvi varchar(255),
   exptl varchar(255),
   ferro varchar(255),
   corr varchar(255),
   blue_bright_varn_clean varchar(255),
   lustre varchar(255),
   jurofm varchar(255),
   s varchar(255),
   p varchar(255),
   shape varchar(255),
   thick decimal(18,10),
   width decimal(18,10),
   len integer,
   oil varchar(255),
   bore integer,
   packing integer,
   classes integer
);
```

### Load data from CSV
```sql
mclient -umonetdb -d metalData  -s  "COPY  INTO  \"metalData\".\"factTable\" FROM STDIN USING  DELIMITERS '|','\\n','\"'" - < ~/data/metalDataSmall.csv
```

### How many
```sql
SELECT COUNT(*) FROM "metalData"."factTable";
```

### Look at some data, row based, slow
```sql
SELECT * FROM "metalData"."factTable" LIMIT 10;
```

### Simple aggregate
```sql
SELECT COUNT(*), SUM(production), AVG(production) FROM "metalData"."factTable";
```

### Try a where (careful with commas)
```sql
SELECT * FROM "metalData"."factTable" WHERE surface_quality = 'G' LIMIT 10;
```

### More aggregates
```sql
SELECT company, COUNT(*), SUM(production), AVG(production) FROM "metalData"."factTable" GROUP BY company;
```

### Adding column names, careful with `AS`
```sql
SELECT company, COUNT(*) AS rowCOUNT, SUM(production) AS productionSUM, AVG(production) AS productionAVG FROM "metalData"."factTable" GROUP BY company;
```

### Double group by, what about just years?
```sql
SELECT date_day, company, COUNT(*) AS rowCOUNT, SUM(production) AS productionSUM, AVG(production) AS productionAVG FROM "metalData"."factTable" GROUP BY date_day, company;
```

### Modify a field (note year is a reserved word and cannot be used for column naming)
```sql
SELECT EXTRACT(YEAR FROM date_day) AS years, company, COUNT(*) AS rowCOUNT, SUM(production) AS productionSUM, AVG(production) AS productionAVG FROM "metalData"."factTable" GROUP BY years, company;
```

### Tidy up output
```sql
SELECT EXTRACT(YEAR FROM date_day) AS years, company, COUNT(*) AS rowCOUNT, SUM(production) AS productionSUM, AVG(production) AS productionAVG FROM "metalData"."factTable" GROUP BY years, company ORDER BY years, company;
```

### Use a condition
```sql
SELECT EXTRACT(YEAR FROM date_day) AS years, company, COUNT(*) AS rowCOUNT, SUM(production) AS productionSUM, AVG(production) AS productionAVG FROM "metalData"."factTable" WHERE EXTRACT(YEAR FROM date_day) = 2010 GROUP BY years, company ORDER BY years, company;
```

### Change `NULL`s
```sql
SELECT surface_quality, COUNT(*) AS rowCOUNT, SUM(production) AS productionSUM, AVG(production) AS productionAVG FROM "metalData"."factTable" GROUP BY surface_quality;
```

to:

```sql
SELECT CASE WHEN surface_quality IS NULL THEN 'missing value' ELSE surface_quality END AS surface_quality, COUNT(*) AS rowCOUNT, SUM(production) AS productionSUM, AVG(production) AS productionAVG FROM "metalData"."factTable" GROUP BY surface_quality;
```

### CASE to generate two dimensional table
```sql
SELECT company, SUM(CASE WHEN production <= 30 THEN production ELSE 0 END) AS lightProduction, SUM(CASE WHEN production <= 60 THEN production ELSE 0 END) AS mediumProduction, SUM(CASE WHEN production > 60 THEN production ELSE 0 END) AS heavyProduction FROM "metalData"."factTable" GROUP BY company;
```

### All together
```sql
SELECT company, SUM(CASE WHEN production <= 30 THEN production ELSE 0 END) AS lightProduction, SUM(CASE WHEN production <= 60 THEN production ELSE 0 END) AS mediumProduction, SUM(CASE WHEN production > 60 THEN production ELSE 0 END) AS heavyProduction FROM "metalData"."factTable" WHERE EXTRACT(YEAR FROM date_day) = 2010 GROUP BY company ORDER BY company DESC;
```