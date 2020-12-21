1. Pliki .http czyta VS Code + REST Client
2. Pobranie jar-a:
	wget https://repo1.maven.org/maven2/org/elasticsearch/elasticsearch-spark-20_2.11/7.5.2/elasticsearch-spark-20_2.11-7.5.2.jar
3. Link do Movielens:
	https://grouplens.org/datasets/movielens/
4. Uruchomienie pyspark-a
	pyspark --jars ./elasticsearch-spark-20_2.11-7.5.2.jar
	

from pyspark import SparkContext, SparkConf
from pyspark.sql import SQLContext
import pyspark.sql.functions as F

# Load from ES
reader = spark.read.format("org.elasticsearch.spark.sql")
                   .option("es.read.metadata", "true")
				   .option("es.nodes.wan.only","true")
				   .option("es.port","9200")
				   .option("es.net.ssl","false")
				   .option("es.nodes", "http://localhost")

df = reader.load("just_testing_witcher")


df.show()

# Load from ES with query¶

reader = spark.read.format("org.elasticsearch.spark.sql")\
                .option("es.read.metadata", "true")\
                .option("es.nodes.wan.only","true")\
                .option("es.port","9200")\
                .option("es.net.ssl","false")\
                .option("es.nodes", "http://localhost")\
                .option("es.query", """{"query": { "query_string": { "query": "*ag*" } } }""")

df = reader.load("just_testing_witcher")

df.show()

# Let's create a movies with tags and genres in one table
tags_materialized = tags.groupBy('movieId').agg(F.collect_set('tag').alias('tags'))
tags_materialized.createOrReplaceTempView("tags_materialized")
tags_materialized.show(3)

movies_materialized = movies.select(\
                                F.col("movieId"),
                                F.col("title"),
                                F.split(F.col("genres"), '\|').alias("genres")\
                            )
movies_materialized.createOrReplaceTempView("movies_materialized")
movies_materialized.show(3)

movies_complete = spark.sql("""SELECT m.movieId, title, genres, tags
                                FROM movies_materialized m
                                LEFT JOIN tags_materialized t ON m.movieId = t.movieId
                               """)
movies_complete.show(3)

# Save to ES
In [60]:
esURL = "localhost"

movies_complete.write\
  .format("org.elasticsearch.spark.sql")\
  .option("es.port","9200")\
  .option("es.net.ssl","false")\
  .option("es.nodes", esURL)\
  .mode("Overwrite")\
  .save("movielens")
# Let's prepare ratings with iso date
ratings_materialized = ratings.select(\
                                        F.col("movieId"),\
                                        F.col("userId"),\
                                        F.col("rating"),\
                                        F.col("timestamp").alias("datetime")\
                                    )
ratings_materialized.show(3)
esURL = "localhost"

ratings_materialized.write\
  .format("org.elasticsearch.spark.sql")\
  .option("es.port","9200")\
  .option("es.net.ssl","false")\
  .option("es.nodes", esURL)\
  .mode("Overwrite")\
  .save("ratings")

