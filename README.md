# Getting Started

# 1. Start Docker-compose  here 
			cd :anji>docker-compose up 


# 2.Run a container
			docker run -it --rm -p 8888:8888 jupyter/pyspark-notebook
		

# 3. Connect to a Jupyter notebook
			http://localhost:8888/?token=e144d004f6652ae6406a78adf894621e62fdeb1fc57d02e8		




# 4.MYSQL Login 
      docker exec -it a743f93894f4 bash
	  mysql -uuser -ppassword
  

		     context = mysql.connector.connect(
					host = 'mysql',
					port=3306,
					user='user',
					password='password')
				cursor = context.cursor()
				cursor.execute("USE school")  
	

# 5. MongoDB :
        docker exec -it 802d5f77cb46 bash
		root@802d5f77cb46:/# mongo
		
		root@802d5f77cb46:/# mongo
		MongoDB shell version v4.4.2
		connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
		Implicit session: session { "id" : UUID("ccadc852-63f5-4dc1-9c3c-1acf2b897df0") }
		MongoDB server version: 4.4.2
		Welcome to the MongoDB shell.
		For interactive help, type "help".
		For more comprehensive documentation, see
			https://docs.mongodb.com/
		Questions? Try the MongoDB Developer Community Forums
			https://community.mongodb.com
		>

				from pymongo import MongoClient
				uri = "mongodb://root:example@mongo"
				client = MongoClient(uri)
# 6. Cassandra Login 
	docker exec -it simple-spark-etl_cassandra_1 bash
	docker exec -it 3975782c9e10 bash
	cqlsh --user cassandra --password cassandra
	
			CREATE KEYSPACE school WITH replication = { 'class' : 'SimpleStrategy' , 'replication_factor' : '1' }; 

			CREATE TABLE school.students ( 
		    name text,
		    surname text,
		    age int,
		    group_id int,
		    group_number text,
		    skills set < text>,
		    something_important int,
		    PRIMARY KEY (name, surname) 
			);
	
# Reference Links 
   https://wiadrodanych.pl/big-data/spark/etl-mysql-mongodb-cassandra-pyspark/  
