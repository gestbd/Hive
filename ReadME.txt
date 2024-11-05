Edit file /usr/share/hue/desktop/conf/hue.ini uncomment&edit 

[beeswax]

hive_server_host=hiveserver2



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

select * from default.wikinerorg;

CREATE VIEW wikinerorg_count AS
SELECT
         d.path,
         COUNT(d.org) AS tOrg
     FROM
         default.wikinerorg d
     GROUP BY
         d.path

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

select * from default.wikinerper;

CREATE VIEW wikinerper_count AS
SELECT
         d.path,
         COUNT(d.per) AS tPer
     FROM
         default.wikinerper d
     GROUP BY
         d.path




CREATE EXTERNAL TABLE wikinerloc (
  path STRING,
  loc STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/opt/hive/data/warehouse/wikinerloc';

LOAD DATA INPATH '/opt/hive/data/warehouse/localizaciones.csv' INTO TABLE wikinerloc;

select * from default.wikinerloc;

CREATE VIEW wikinerloc_count AS
SELECT
         d.path,
         COUNT(d.loc) AS tLoc
     FROM
         default.wikinerloc d
     GROUP BY
         d.path


//////////////////////

SELECT
    o.path,
    o.tOrg AS total_org,
    p.tper AS total_per
FROM
    default.wikinerorg_count o
JOIN
    default.wikinerper_count p
ON
    o.path = p.path;
	
	
//////////////

SELECT
    o.path,
    o.torg AS total_org,
    l.tloc AS total_loc
FROM
    default.wikinerorg_count o
JOIN
    default.wikinerloc_count l
ON
    o.path = l.path;
	
	
////////////

SELECT
    p.path,
    p.tper AS total_per,
    l.tloc AS total_loc
FROM
    default.wikinerper_count p
JOIN
    default.wikinerloc_count l
ON
	TRIM( p.path ) = TRIM( l.path);	


!q