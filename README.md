## Introduction


## Prerequsites
1. Have Cloudera Quickstart installed in you systems.
2. Make sure you are logged in at Cloudera Manager. 
3. Varify folllowing services are up- impala, Hive, Hue, HDFS

## Aggregate:

The cloudera has already pre-loaded some access log data into /opt/exmaples/log_data/access.log:

Step 1: Move the data from the local filesystem, into HDFS.
```
>sudo -u hdfs hadoop fs -mkdir /user/hive/warehouse/original_access_logs
> sudo -u hdfs hadoop fs -copyFromLocal /opt/examples/log_files/access.log.2 /user/hive/warehouse/original_access_logs
```
![screenshot1](https://user-images.githubusercontent.com/33071134/48032774-8e737580-e11e-11e8-91fc-5b2e8cf3800d.png)

Step 2: Verify the data in HDFS by excuting the command below.
```
>hadoop fs -ls /user/hive/warehouse/original_access_logs
```

Step 3: Login to Hue and open the and open the Hive Query Editor app and paste the following command.
```
CREATE EXTERNAL TABLE intermediate_access_logs (
    ip STRING,
    date STRING,
    method STRING,
    url STRING,
    http_version STRING,
    code1 STRING,
    code2 STRING,
    dash STRING,
    user_agent STRING)
ROW FORMAT SERDE 'org.apache.hadoop.hive.contrib.serde2.RegexSerDe'
WITH SERDEPROPERTIES (
    'input.regex' = '([^ ]*) - - \\[([^\\]]*)\\] "([^\ ]*) ([^\ ]*) ([^\ ]*)" (\\d*) (\\d*) "([^"]*)" "([^"]*)"',
    'output.format.string' = "%1$$s %2$$s %3$$s %4$$s %5$$s %6$$s %7$$s %8$$s %9$$s")
LOCATION '/user/hive/warehouse/original_access_logs';

CREATE EXTERNAL TABLE tokenized_access_logs (
    ip STRING,
    date STRING,
    method STRING,
    url STRING,
    http_version STRING,
    code1 STRING,
    code2 STRING,
    dash STRING,
    user_agent STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
LOCATION '/user/hive/warehouse/tokenized_access_logs';
```
Step 4: Run the command below. This command is using a MapReduce job, just like our Sqoop import did, to transfer the data from one table to the other in parallel.

```
ADD JAR {{lib_dir}}/hive/lib/hive-contrib.jar;

INSERT OVERWRITE TABLE tokenized_access_logs SELECT * FROM intermediate_access_logs;
```
Step 5: Switch back to the Impala Query Editor app, and enter the command below. This command is telling Impala that some tables have been created through a different tool.

```
invalidate metadata;

```

Step 6: Enter the command below in the query or refresh the table list in the left-hand column, you should see the two new external tables in the default database. 
```
show tables;
```

Step 7: Paste the following query into the Query Editor:

```
select count(*),url from tokenized_access_logs
where url like '%\/product\/%'
group by url order by count(*) desc;

```



## Query running in the Hue

![screenshot3](https://user-images.githubusercontent.com/33071134/48032333-dc877980-e11c-11e8-84d7-c756533f207d.png)

## References:

1. https://www.cloudera.com/developers/get-started-with-hadoop-tutorial/showing-big-data-value.html

2. https://www.cloudera.com/developers/get-started-with-hadoop-tutorial/exercise-2.html

3. https://www.tutorialspoint.com/impala/impala_overview.htm​

5. https://www.quora.com/What-is-impala​

6. https://www.youtube.com/watch?v=nRm3NbuS0IA​

7. https://www.youtube.com/watch?v=_bd8bOx0n6U&t=228sttps://www.youtube.com/watch?v=nRm3NbuS0IA​

8. https://www.youtube.com/watch?v=_bd8bOx0n6U&t=228s


