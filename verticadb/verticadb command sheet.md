# Practice queries for Vertica

## Starting the database
```shell
cd /opt/vertica/sbin
sudo su dbadmin
admintools
```

### Enter the database
```shell
/opt/vertica/bin/vsql -dmetalData -Udbadmin
```

## Queries

### Create a table
```sql
CREATE TABLE "factTable"
(
   date_day date,
   country varchar(255),
   company varchar(255),
   production int,
   family varchar(255),
   product_type varchar(255),
   steel varchar(255),
   carbon float,
   hardness int,
   temper_rolling varchar(255),
   condition varchar(255),
   formability int,
   strength int,
   non_ageing varchar(255),
   surface_finish varchar(255),
   surface_quality varchar(255),
   enamealibility int,
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
   thick float,
   width float,
   len int,
   oil varchar(255),
   bore int,
   packing int,
   classes int
);
```

### Load data from CSV
```sql
COPY public.factTable FROM LOCAL '/home/deusto/data/metalDataSmall.csv' DELIMITER '|' ENCLOSED BY '"' EXCEPTIONS '/tmp/verticaLoadExceptions.txt' SKIP 0 REJECTED DATA '/tmp/verticaLoadRejections.txt' DIRECT NULL AS 'null';
```

### How many
```sql
SELECT COUNT(*) FROM factTable;
```

### Look at some data, row based, slow
```sql
SELECT * FROM factTable LIMIT 10;
```

### Simple aggregate
```sql
SELECT COUNT(*), SUM(production), AVG(production) FROM factTable;
```

### Try a where (careful with commas)
```sql
SELECT * FROM factTable WHERE surface_quality = 'G' LIMIT 10;
```

### More aggregates
```sql
SELECT company, COUNT(*), SUM(production), AVG(production) FROM factTable GROUP BY company;
```

### Adding column names, careful with `AS`
```sql
SELECT company, COUNT(*) AS rowCOUNT, SUM(production) AS productionSUM, AVG(production) AS prouctionAVG FROM factTable GROUP BY company;
```

### Double group by, what about just years?
```sql
SELECT date_day, company, COUNT(*) AS rowCOUNT, SUM(production) AS productionSUM, AVG(production) AS prouctionAVG FROM factTable GROUP BY date_day, company;
```

### Modify a field
```sql
SELECT EXTRACT(YEAR FROM date_day) AS year, company, COUNT(*) AS rowCOUNT, SUM(production) AS productionSUM, AVG(production) AS prouctionAVG FROM factTable GROUP BY EXTRACT(YEAR FROM date_day), company;
```

### Tidy up output
```sql
SELECT EXTRACT(YEAR FROM date_day) AS year, company, COUNT(*) AS rowCOUNT, SUM(production) AS productionSUM, AVG(production) AS prouctionAVG FROM factTable GROUP BY EXTRACT(YEAR FROM date_day), company ORDER BY EXTRACT(YEAR FROM date_day), company;
```

### Use a condition
```sql
SELECT EXTRACT(YEAR FROM date_day) AS year, company, COUNT(*) AS rowCOUNT, SUM(production) AS productionSUM, AVG(production) AS prouctionAVG FROM factTable WHERE EXTRACT(YEAR FROM date_day) = 2010 GROUP BY EXTRACT(YEAR FROM date_day), company ORDER BY EXTRACT(YEAR FROM date_day), company;
```

### Change `NULL`s
```sql
SELECT surface_quality, COUNT(*) rowCOUNT, SUM(production) productionSUM, AVG(production) prouctionAVG FROM factTable GROUP BY surface_quality;
```

to:

```sql
SELECT IFNULL(surface_quality, 'missing value') AS surface_quality, COUNT(*) rowCOUNT, SUM(production) productionSUM, AVG(production) prouctionAVG FROM factTable GROUP BY surface_quality;
```

### CASE to generate two dimensional table
```sql
SELECT company, SUM(CASE WHEN production <= 30 THEN production ELSE 0 END) AS lightProduction, SUM(CASE WHEN production <= 60 THEN production ELSE 0 END) AS mediumProduction, SUM(CASE WHEN production > 60 THEN production ELSE 0 END) AS heavyProduction FROM factTable GROUP BY company;
```

### All together
```sql
SELECT company, SUM(CASE WHEN production <= 30 THEN production ELSE 0 END) AS lightProduction, SUM(CASE WHEN production <= 60 THEN production ELSE 0 END) AS mediumProduction, SUM(CASE WHEN production > 60 THEN production ELSE 0 END) AS heavyProduction FROM factTable WHERE EXTRACT(YEAR FROM date_day) = 2010 GROUP BY company ORDER BY company DESC;
```

