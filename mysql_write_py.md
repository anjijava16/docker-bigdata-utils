# mysql_connector Python 
!pip3 install mysql-connector-python
import mysql.connector
# Create table 
table_schema = """
    CREATE TABLE IF NOT EXISTS groups(
    group_id int(11) NOT NULL AUTO_INCREMENT,
    group_number char(4) NOT NULL,
    description VARCHAR(100),
    something_important int(11),
    PRIMARY KEY (group_id),
    KEY group_number (group_number)
    ) ENGINE=InnoDB
"""
## MYSQL Connection 
context = mysql.connector.connect(
    host = 'mysql',
    port=3306,
    user='user',
    password='password')
cursor = context.cursor()

# Execute Commands 
cursor.execute("USE school")
cursor.execute(table_schema)
some_data = """
INSERT INTO groups
(group_number, description, something_important)
VALUES
('1A','some group',100),
('2A','some group',102),
('3A','some group',101),
('1B','some group',123),
('2B','some group',133),
('3B','some group',144);
"""
cursor.execute(some_data)
# Commit Operation
context.commit()