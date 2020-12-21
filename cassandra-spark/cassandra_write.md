{"type":"response","@timestamp":"2020-12-21T03:59:05Z","tags":[],"pid":7,"method":"post","statusCode":200,"req":{"url":"/api/ui_metric/report","method":"post","headers":{"host":"localhost:5601","connection":"keep-alive","content-length":"484","sec-ch-ua":"\"Google Chrome\";v=\"87\", \" Not;A Brand\";v=\"99\", \"Chromium\";v=\"87\"","sec-ch-ua-mobile":"?0","user-agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36","kbn-version":"7.6.2","content-type":"application/json","accept":"*/*","origin":"http://localhost:5601","sec-fetch-site":"same-origin","sec-fetch-mode":"cors","sec-fetch-dest":"empty","referer":"http://localhost:5601/app/kibana","accept-encoding":"gzip, deflate, br","accept-language":"en-US,en;q=0.9,te;q=0.8"},"remoteAddress":"172.24.0.1","userAgent":"172.24.0.1","referer":"http://localhost:5601/app/kibana"},"res":{"statusCode":200,"responseTime":800,"contentLength":9},"message":"POST /api/ui_metric/report 200 800ms - 9.0B"}
http://localhost:9200/



./spark-shell --packages com.datastax.spark:spark-cassandra-connector_2.12:3.0.0-beta,com.datastax.cassandra:cassandra-driver-core:3.9.0

spark.conf.set("spark.cassandra.connection.host", "127.0.0.1")


# Cassandra 

## Data : https://github.com/zorteran/wiadro-danych-spark-cassandra-101
sudo docker exec -it container_name cqlsh


CREATE KEYSPACE spark_playground WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1 };


CREATE TABLE sensors ( sensor_id int, location text, group_id int, PRIMARY KEY (sensor_id ));

The measurements will be in the sensors_reads table . The key consists of the sensor id (partition key) and date in the form timeuuid (clustering key). 
The Partition key indicates where the record is located (under what node). Timeuuid as a clustering key allows you to sort records. 
If you're confused about what's what, have a look at stackoverflow for an explanation . You have to be careful with choosing the partition key, 
depending on whether the nodes will be evenly filled with data.

CREATE TABLE sensors_reads ( sensor_id int, date timeuuid, temp double, humidity double, wind_speed double, wind_direction double, PRIMARY KEY (sensor_id, date ));

# Cassandra Spark Configuration :
spark = (SparkSession.builder
    .appName('simple_etl')
    .config("spark.cassandra.connection.host", "cassandra")
    .config("spark.cassandra.auth.username", "cassandra")
    .config("spark.cassandra.auth.password", "cassandra")
    .config("spark.sql.extensions", "com.datastax.spark.connector.CassandraSparkExtensions")
    .config("spark.sql.catalog.casscatalog","com.datastax.spark.connector.datasource.CassandraCatalog")
    .getOrCreate())
	
# Loading csv files
val sensors = spark.read.format("csv").option("header", "true").load("/home/maciej/Desktop/spark_and_cassandra/sensors.csv")
val sensorReads = spark.read.format("csv").option("header", "true").load("/home/maciej/Desktop/spark_and_cassandra/sensor_reads.csv")


# Save to the table with sensors

sensors.write
    .format("org.apache.spark.sql.cassandra")
    .option("keyspace","spark_playground")
    .option("table","sensors")
    .mode("append")
    .save()
	


import spark.implicits._
import com.datastax.driver.core.utils.UUIDs
import org.apache.spark.sql.functions.udf
 
val toTimeuuid: java.sql.Timestamp => String = x => UUIDs.startOf(x.getTime()).toString()
val fromTimeuuid: String => java.sql.Timestamp = x => new java.sql.Timestamp(UUIDs.unixTimestamp(java.util.UUID.fromString(x)))
 
val toTimeuuidUDF = udf(toTimeuuid)
val fromTimeuuidUDF = udf(fromTimeuuid)
 
sensorsReads
    .withColumn("date_as_timestamp", to_timestamp($"date"))
    .withColumn("date_as_timeuuid", toTimeuuidUDF($"date_as_timestamp"))
    .withColumn("timestamp_from_timeuuid",fromTimeuuidUDF($"date_as_timeuuid"))
    .show(false)


val sensorsReads_fixed = sensorsReads
                            .withColumn("date_as_timestamp", to_timestamp($"date"))
                            .withColumn("date_as_timeuuid", toTimeuuidUDF($"date_as_timestamp"))
                            .drop("date").drop("date_as_timestamp")
                            .withColumnRenamed("date_as_timeuuid","date")
                            .withColumnRenamed("temperature","temp")
sensorsReads_fixed.write
    .format("org.apache.spark.sql.cassandra")
    .option("keyspace","spark_playground")
    .option("table","sensors_reads")
    .mode("append")
    .save()

# Reading from Cassandra
spark.conf.set("spark.sql.catalog.casscatalog","com.datastax.spark.connector.datasource.CassandraCatalog")

spark.read.table("casscatalog.spark_playground.sensors").show()


val sensors_table = spark.read.table("casscatalog.spark_playground.sensors")
val sensors_reads_table = spark.read.table("casscatalog.spark_playground.sensors_reads")

The cost of the operation
Everything seems easy and fun with Spark. We must remember that underneath is Cassandra, which has its own way of handling inquiries. I mainly mean the speed of retrieving records searched on the basis of keys and values.
sensors_table.filter("sensor_id in (1,2,3)").select("sensor_id","location").explain

sensors_table.filter("group_id in (1,2,3)").select("sensor_id","location").explain

# group by is not support Cassandra but we can do in Spark
	sensors_table.groupBy("group_id").count.show

# Joins - joinWithCassandraTable
  sensors_reads_table.join(sensors_table).explain
  sensors_reads_table.join(sensors_table, sensors_reads_table("sensor_id") === sensors_table("sensor_id"), "inner").explain
spark.conf.set("spark.sql.extensions", "com.datastax.spark.connector.CassandraSparkExtensions")














