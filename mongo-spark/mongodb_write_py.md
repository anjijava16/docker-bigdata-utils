# Mongo DB Python Package 
!pip3 install pymongo

# MongoDB Connection 
from pymongo import MongoClient
uri = "mongodb://root:example@mongo"
client = MongoClient(uri)
db = client['school']
students = db.students
new_students = [
    {'name': 'John', 'surname': 'Smith', 'group': '1A', 'age': 22, 'skills': ['drawing', 'skiing']},
    {'name': 'Mike', 'surname': 'Jones', 'group': '1B', 'age': 24, 'skills': ['chess', 'swimming']},
    {'name': 'Diana', 'surname': 'Williams', 'group': '2A', 'age': 28, 'skills': ['curling', 'swimming']},
    {'name': 'Samantha', 'surname': 'Brown', 'group': '1B', 'age': 21, 'skills': ['guitar', 'singing']}
]

# Insert Into MongoDB 
students.insert_many(new_students)
# Find One 
students.find_one()
{'_id': ObjectId('5f537b2945e1723407177137'),
 'name': 'John',
 'surname': 'Smith',
 'group': '1A',
 'age': 22,
 'skills': ['drawing', 'skiing']}