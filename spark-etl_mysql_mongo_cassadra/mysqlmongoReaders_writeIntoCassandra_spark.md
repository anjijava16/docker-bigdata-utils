# Install findspark 
!pip install findspark
import os
os.environ['PYSPARK_SUBMIT_ARGS'] = '--packages com.datastax.spark:spark-cassandra-connector_2.12:3.0.0-beta,org.mongodb.spark:mongo-spark-connector_2.12:3.0.0,mysql:mysql-connector-java:5.1.49 pyspark-shell'
# Provides findspark.init() to make pyspark importable as a regular library.
import findspark
findspark.init()

# Import pakage 
import pyspark
from pyspark.sql import SparkSession

# Spark Session Inilizate 
spark = (SparkSession.builder
    .appName('simple_etl')
    .config('spark.mongodb.input.uri', 'mongodb://root:example@mongo/school.students?authSource=admin')
    .config('spark.mongodb.output.uri', 'mongodb://root:example@mongo/school.students?authSource=admin')
    .config("spark.cassandra.connection.host", "cassandra")
    .config("spark.cassandra.auth.username", "cassandra")
    .config("spark.cassandra.auth.password", "cassandra")
    .config("spark.sql.extensions", "com.datastax.spark.connector.CassandraSparkExtensions")
    .config("spark.sql.catalog.casscatalog","com.datastax.spark.connector.datasource.CassandraCatalog")
    .getOrCreate())
# Extract Read from MySql

groups = (spark.read
    .format("jdbc")
    .option("url", "jdbc:mysql://mysql/school")
    .option("driver", "com.mysql.jdbc.Driver")
    .option("dbtable", "groups")
    .option("user", "user")
    .option("password", "password")
    .load())
groups.show()

# Read from Mongo
students = spark.read.format("mongo").load()
students.show()
+--------------------+---+-----+--------+-------------------+--------+
|                 _id|age|group|    name|             skills| surname|
+--------------------+---+-----+--------+-------------------+--------+
|[5f537b2945e17234...| 22|   1A|    John|  [drawing, skiing]|   Smith|
|[5f537b2945e17234...| 24|   1B|    Mike|  [chess, swimming]|   Jones|
|[5f537b2945e17234...| 28|   2A|   Diana|[curling, swimming]|Williams|
|[5f537b2945e17234...| 21|   1B|Samantha|  [guitar, singing]|   Brown|
+--------------------+---+-----+--------+-------------------+--------+

# Transform
students_with_groups = (students
                        .join(groups, groups.group_number == students.group, how='left')
                        .select("name","surname","age","group_id","group_number","something_important","skills")
                       )
students_with_groups.show()

# Load To Cassandra 

(students_with_groups
    .write
    .format("org.apache.spark.sql.cassandra")
    .option("keyspace","school")
    .option("table","students")
    .mode("append")
    .save())

# Show Insertated Cassandra Data
spark.table("casscatalog.school.students").show()
+--------+--------+---+--------+------------+-------------------+-------------------+
|    name| surname|age|group_id|group_number|             skills|something_important|
+--------+--------+---+--------+------------+-------------------+-------------------+
|Samantha|   Brown| 21|       4|          1B|  [guitar, singing]|                123|
|    John|   Smith| 22|       1|          1A|  [drawing, skiing]|                100|
|    Mike|   Jones| 24|       4|          1B|  [chess, swimming]|                123|
|   Diana|Williams| 28|       2|          2A|[curling, swimming]|                102|
+--------+--------+---+--------+------------+-------------------+-------------------+
