# AWS_Postgres_Python_Psycopg2_Query_To_JSON

## Front end web often needs the results of a postgres SQL query in the form a json object. Here is solution code to convert the query output of a Python_Psycopg2_Query 

This is designed to be used in an AWS lambda-function.

https://colab.research.google.com/drive/1RuiOGcHOAoiRoN7Lb-n0W7Aj6otX5cok?usp=sharing 

'''
# import your libraries
import boto3
import json
import psycopg2
from psycopg2 import sql, Error
from psycopg2.extras import NamedTupleCursor
from datetime import datetime
 
# lambda function for AWS
def lambda_handler(event, context):
 
   # read the inputs:
   # thing_id
   # company = [event["company_id"]]
   thing_id = float(event["thing_id"])
 
   ##########################
   # Connect to Database RDS
   ##########################
 
   try:
       connection = psycopg2.connect(
           database="xxx",
           user="xxx_user",
           password="xxx",
           host="xxx.xxx.xxx-xxx-xxx.rds.amazonaws.com",
           port='5432'
       )
 
       # Create a cursor to perform database operations
       cursor = connection.cursor(cursor_factory = psycopg2.extras.RealDictCursor)
 
       print("Connection Ok!")
 
   except (Exception, Error) as error:  # Failure!!
       statusCode = 430
       output = (f"connectionError: {str(error)} \n")
       print(error)
 
 
   ##############
   # XXX
   ##############
 
   # enter query
 
   query = f"""
   SELECT *   
   FROM XXX.XXX
   WHERE thing_id = {thing_id}
   ;
   """
 
   # track which row
   row_counter = 1
 
   # iterate through cursor-object:
   cursor.execute(query)
 
   # store all rows
   whole_row_dict_list = []
 
   '''
   To convert the output of Psycopg2 (back) into a json object (a dictionary)
   you will iterate through the 'cursor' object
   (which contains your query results)
   going row by row, with each row being a dictionary of items
   '''
 
   for row in cursor:
 
       # reset this each time
       sub_dict_list = []
      
       # inspection
       print("Row Counter!" , row_counter)
 
       # make a dictionary of all the fields you want
       # Note: date-time object are turned into strings
       row_dictionary = {
           'id': row['id'],
           'uuid': row['uuid'],
           'created_at': row['created_at'].strftime('%Y-%m-%d %H:%M:%S'),
           'updated_at': row['updated_at'].strftime('%Y-%m-%d %H:%M:%S'),
           'thing_1': row['thing_1'],
           'thing_2': row['thing_2'],
       }
 
       # make larger bundler
       row_json = {
           "model":"divedins.resourcediversenotdiverse",
           "row_counter":row_counter,
           "fields": row_dictionary
       }
 
       whole_row_dict_list.append(row_json)
 
       # increment counter
       row_counter += 1
 
 
   #################
   # Task 5. Return
   #################
 
   output = {
       # repeat of input
       "input_thing" : "input_thing",
 
       # for %diversity
       "table_return" : whole_row_dict_list,
   }
 
 
   #################
   # clean up & end
   #################
 
   # Close Connection to Database (clean up)
   if (connection):
       cursor.close()
       connection.close()
       print("PostgreSQL connection is closing...is now closed.")
 
   # Finish and Return Output
   return {
       'statusCode': 200,
       'body': output
       }

'''
