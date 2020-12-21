# Install Koalas Package 
!{sys.executable} -m pip install koalas
Requirement already satisfied: koalas in /opt/conda/lib/python3.7/site-packages (1.0.1)
Requirement already satisfied: pyarrow>=0.10 in /opt/conda/lib/python3.7/site-packages (from koalas) (0.17.1)
Requirement already satisfied: matplotlib>=3.0.0 in /opt/conda/lib/python3.7/site-packages (from koalas) (3.2.1)
Requirement already satisfied: pandas>=0.23.2 in /opt/conda/lib/python3.7/site-packages (from koalas) (1.0.5)
Requirement already satisfied: numpy>=1.14 in /opt/conda/lib/python3.7/site-packages (from koalas) (1.18.5)
Requirement already satisfied: pyparsing!=2.0.4,!=2.1.2,!=2.1.6,>=2.0.1 in /opt/conda/lib/python3.7/site-packages (from matplotlib>=3.0.0->koalas) (2.4.7)
Requirement already satisfied: python-dateutil>=2.1 in /opt/conda/lib/python3.7/site-packages (from matplotlib>=3.0.0->koalas) (2.8.1)
Requirement already satisfied: kiwisolver>=1.0.1 in /opt/conda/lib/python3.7/site-packages (from matplotlib>=3.0.0->koalas) (1.2.0)
Requirement already satisfied: cycler>=0.10 in /opt/conda/lib/python3.7/site-packages (from matplotlib>=3.0.0->koalas) (0.10.0)
Requirement already satisfied: pytz>=2017.2 in /opt/conda/lib/python3.7/site-packages (from pandas>=0.23.2->koalas) (2020.1)
Requirement already satisfied: six>=1.5 in /opt/conda/lib/python3.7/site-packages (from python-dateutil>=2.1->matplotlib>=3.0.0->koalas) (1.15.0)

# Import's
import databricks.koalas as ks
import pandas as pd
from pyspark.sql import SparkSession
WARNING:root:'ARROW_PRE_0_15_IPC_FORMAT' environment variable was not set. It is required to set this environment variable to '1' in both driver and executor sides if you use pyarrow>=0.15 and pyspark<3.0. Koalas will set it for you but it does not work if there is a Spark context already launched.
# Imports 
	spark = SparkSession.builder.getOrCreate()

# Creating Spark DataFrame

movies_df = (spark.read
             .option("header", "true")
             .option("inferSchema", "true")
             .csv("./movies.csv")
            )
ratings_df = (spark.read
             .option("header", "true")
             .option("inferSchema", "true")
             .csv("./ratings.csv")
            )
links_df = (spark.read
             .option("header", "true")
             .option("inferSchema", "true")
             .csv("./links.csv")
            )
tags_df = (spark.read
             .option("header", "true")
             .option("inferSchema", "true")
             .csv("./tags.csv")
            )
In [5]:
movies_df.registerTempTable("movies")
ratings_df.registerTempTable("ratings")
links_df.registerTempTable("link")
tags_df.registerTempTable("tags")
In [6]:
movies_df

# Creating Koalas DataFrame
koalas_movies_df = ks.DataFrame(movies_df)
koalas_ratings_df = ks.DataFrame(ratings_df)
koalas_links_df = ks.DataFrame(links_df)
koalas_tags_df = ks.DataFrame(tags_df)
koalas_movies_df
