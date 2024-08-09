

# snowflake-xtable
snowflake-xtable


# create table in snowflake 
```

CREATE OR REPLACE ICEBERG TABLE tempdb.public.samples (
    id VARCHAR,
    name VARCHAR,
    age VARCHAR,
    city VARCHAR,
    create_ts VARCHAR
)
CATALOG='SNOWFLAKE'
EXTERNAL_VOLUME='iceberg_external_volume'
BASE_LOCATION='snowflake_tables/samples';


INSERT INTO tempdb.public.samples (id, name, age, city, create_ts)
VALUES
   (1, 'John', 25, 'NYC', '2023-09-28 00:00:00'),
   (2, 'Emily', 30, 'SFO', '2023-09-28 00:00:00'),
   (3, 'Michael', 35, 'ORD', '2023-09-28 00:00:00'),
   (4, 'Andrew', 40, 'NYC', '2023-10-28 00:00:00'),
   (5, 'Bob', 28, 'SEA', '2023-09-23 00:00:00'),
   (6, 'Charlie', 31, 'DFW', '2023-08-29 00:00:00');


   SELECT * FROM tempdb.public.samples
```
##### create Directory called app inside that create folder called catalog.yaml
```
catalogImpl: org.apache.iceberg.snowflake.SnowflakeCatalog

catalogName: onetable
catalogOptions:
  io-impl: org.apache.iceberg.aws.s3.S3FileIO
  warehouse: s3://XXX/warehouse
  uri: jdbc:snowflake://XXX.snowflakecomputing.com
  jdbc.user: XX
  jdbc.password: XXX
```

download jar in diretory 
https://drive.google.com/file/d/1lWWR0w4n5RsRsyjdvvgY7H-lt86UE62Z/view?usp=share_link

create my_config.yaml
```
sourceFormat: ICEBERG
targetFormats:
  - DELTA
  - HUDI
datasets:
  -
    tableBasePath: s3://XXXXX/snowflake_tables/samples/
    tableDataPath: s3://XXXX/snowflake_tables/samples/data
    tableName: samples
    namespace: tempdb.public
```

create docker file 
```
# Use the official Rocky Linux image as the base
FROM rockylinux:9

# Set the working directory
WORKDIR /home

# Install the required packages
RUN dnf install -y java-1.8.0-openjdk java-1.8.0-openjdk-devel maven git ncurses wget nano

# Set environment variables for Java
ENV JAVA_HOME=/usr/lib/jvm/java-1.8.0
ENV PATH=$JAVA_HOME/bin:$PATH

# Set environment variables from the .env file
ENV AWS_ACCESS_KEY_ID=XXXX
ENV AWS_SECRET_ACCESS_KEY=XXX
ENV AWS_REGION=us-east-1

# Create a directory for xtable and switch to it
RUN mkdir xtable
WORKDIR /home/xtable

# Download required JAR files
RUN wget -O ./iceberg-spark-runtime-3.2_2.12-1.4.2.jar https://repo1.maven.org/maven2/org/apache/iceberg/iceberg-spark-runtime-3.2_2.12/1.4.2/iceberg-spark-runtime-3.2_2.12-1.4.2.jar \
    && wget -O ./iceberg-spark-runtime-3.3_2.12-1.4.2.jar https://repo1.maven.org/maven2/org/apache/iceberg/iceberg-spark-runtime-3.3_2.12/1.4.2/iceberg-spark-runtime-3.3_2.12-1.4.2.jar \
    && wget -O ./snowflake-jdbc-3.13.28.jar https://repo1.maven.org/maven2/net/snowflake/snowflake-jdbc/3.13.28/snowflake-jdbc-3.13.28.jar \
    && wget -O ./iceberg-aws-1.4.2.jar https://repo1.maven.org/maven2/org/apache/iceberg/iceberg-aws/1.4.2/iceberg-aws-1.4.2.jar \
    && wget -O ./bundle-2.23.9.jar https://repo1.maven.org/maven2/software/amazon/awssdk/bundle/2.23.9/bundle-2.23.9.jar

# Copy the configuration files into the container
COPY ./catalog.yaml /home/xtable/
COPY ./my_config.yaml /home/xtable/
COPY ./xtable-utilities-0.1.0-SNAPSHOT-bundled.jar /home/xtable/

# Run the Java application
CMD ["java", "-cp", "iceberg-spark-runtime-3.3_2.12-1.4.2.jar:xtable-utilities-0.1.0-SNAPSHOT-bundled.jar:snowflake-jdbc-3.13.28.jar:iceberg-aws-1.4.2.jar:bundle-2.23.9.jar", "org.apache.xtable.utilities.RunSync", "--datasetConfig", "my_config.yaml", "--icebergCatalogConfig", "catalog.yaml"]

```

run 
```
docker build -t my-xtable-app .

docker run -it my-xtable-app
```
