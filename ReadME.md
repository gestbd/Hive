# Configuración de Hue y Tablas Externas en Hive

## 1. Editar configuración de Hue

Edita el archivo `/usr/share/hue/desktop/conf/hue.ini` descomentando y modificando las siguientes líneas:

```ini
[beeswax]
hive_server_host=hiveserver2
```

---

## 2. Tablas Externas en Hive

### Tabla: `wikinerorg` (PostgreSQL)

```sql
CREATE EXTERNAL TABLE wikinerorg (
  id INT,
  path STRING,
  org STRING
)
STORED BY 'org.apache.hive.storage.jdbc.JdbcStorageHandler'
TBLPROPERTIES (
  "hive.sql.database.type" = "POSTGRES",
  "hive.sql.jdbc.driver" = "org.postgresql.Driver",
  "hive.sql.jdbc.url" = "jdbc:postgresql://hive4-postgres:5432/postgresgesdb",
  "hive.sql.dbcp.username" = "hive",
  "hive.sql.dbcp.password" = "password",
  "hive.sql.table" = "wikipedia_ner_org"
);
```

**Consulta:**
```sql
SELECT * FROM default.wikinerorg;
```

**Vista de conteo:**
```sql
CREATE VIEW wikinerorg_count AS
SELECT
    d.path,
    COUNT(d.org) AS tOrg
FROM
    default.wikinerorg d
GROUP BY
    d.path;
```

---

### Tabla: `wikinerper` (MariaDB)

```sql
CREATE EXTERNAL TABLE wikinerper (
  id INT,
  path VARCHAR(255),
  PER VARCHAR(255)
)
STORED BY 'org.apache.hive.storage.jdbc.JdbcStorageHandler'
TBLPROPERTIES (
  "hive.sql.database.type" = "MYSQL",
  "hive.sql.jdbc.url" = "jdbc:mariadb://hive4-mariadb:3306/mariaGESDB",
  "hive.sql.dbcp.username" = "root",
  "hive.sql.dbcp.password" = "my_password",
  "hive.sql.jdbc.driver" = "org.mariadb.jdbc.Driver",
  "hive.sql.dbcp.driver.class" = "org.mariadb.jdbc.Driver",
  "hive.sql.table" = "Wikipedia_NER_PER"
);
```

**Consulta:**
```sql
SELECT * FROM default.wikinerper;
```

**Vista de conteo:**
```sql
CREATE VIEW wikinerper_count AS
SELECT
    d.path,
    COUNT(d.per) AS tPer
FROM
    default.wikinerper d
GROUP BY
    d.path;
```

---

### Tabla: `wikinerloc` (CSV local)

```sql
CREATE EXTERNAL TABLE wikinerloc (
  path STRING,
  loc STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/opt/hive/data/warehouse/wikinerloc';
```

**Carga de datos:**
```sql
LOAD DATA INPATH '/opt/hive/data/warehouse/localizaciones.csv' INTO TABLE wikinerloc;
```

**Consulta:**
```sql
SELECT * FROM default.wikinerloc;
```

**Vista de conteo:**
```sql
CREATE VIEW wikinerloc_count AS
SELECT
    d.path,
    COUNT(d.loc) AS tLoc
FROM
    default.wikinerloc d
GROUP BY
    d.path;
```

---

## 3. Consultas combinadas

### Combinación ORG y PER
```sql
SELECT
    o.path,
    o.tOrg AS total_org,
    p.tPer AS total_per
FROM
    default.wikinerorg_count o
JOIN
    default.wikinerper_count p
ON
    o.path = p.path;
```

---

### Combinación ORG y LOC
```sql
SELECT
    o.path,
    o.tOrg AS total_org,
    l.tLoc AS total_loc
FROM
    default.wikinerorg_count o
JOIN
    default.wikinerloc_count l
ON
    o.path = l.path;
```

---

### Combinación PER y LOC
```sql
SELECT
    p.path,
    p.tPer AS total_per,
    l.tLoc AS total_loc
FROM
    default.wikinerper_count p
JOIN
    default.wikinerloc_count l
ON
    TRIM(p.path) = TRIM(l.path);
```

---

Fin del script.  
```
!q
```
