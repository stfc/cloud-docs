========================================
Accessing MySQL Database from JupyterHub
========================================

This document will show how to connect to a MySQL database which is on the same server as JupyterHub.

In MySQL, a new user 'megan' has been created with a 'test' password.
This user has been given access to the database 'demodb'.

Python Packages
###############

Python Packages
---------------

The following python packages are used in this notebook:
- pymysql
- sqlalchemy
- pandas and scikitlearn - these will be used in one example



Connecting to a MySQL database
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Either pymysql or sqlalchemy can be used to connect to a database, however it is better
to use sqlalchemy if you want to import a pandas dataframe as a table into your MySQL database.


.. code-block:: python

  # pymysql
  import pymysql.cursors
  #connect to database
  connection = pymysql.connect(host='localhost', user='megan', password='test', database='demodb')



.. code-block:: python

  #sqlalchemy
  from sqlalchemy import create_engine
  eng = create_engine('mysql+pymysql://megan:test@localhost/demodb') #connects to the database demodb as user=megan and password=test
  eng.connect() #this will show an error if it fails to connect to the database


  <sqlalchemy.engine.base.Connection at 0x7f4c5c739f60>

After connecting to the database, you can check the list of tables using:


.. code-block:: python

  #if using sqlalchemy
  eng.table_names()

  ['example', 'iris_table', 'tutorials_tbl']

.. code-block:: python

  #alternatively, pymysql allows you to execute SQL commands:
  cursor = connection.cursor()
  query = "SELECT * FROM iris_table" #example SQL query
  cursor.execute(query) #executes the query, here it states there are 150 entries in the table

  #output
    150

Import dataset into MySQL
-------------------------

The following example will show how to store an Iris dataset as a new table in our demo database.


.. code-block:: python

  #import python libraries
  from sklearn import datasets
  import pandas as pd

  iris = datasets.load_iris() #load Iris dataset
  print(iris.keys())

  dict_keys(['data', 'target', 'frame', 'target_names', 'DESCR', 'feature_names', 'filename'])



.. code-block:: python

  #store dataset in a dataframe
  data = pd.DataFrame(iris.data, columns=[iris.feature_names])
  print(data) #print the dataframe

  #The output would be:

        sepal length (cm) sepal width (cm) petal length (cm) petal width (cm)
    0                 5.1              3.5               1.4              0.2
    1                 4.9              3.0               1.4              0.2
    2                 4.7              3.2               1.3              0.2
    3                 4.6              3.1               1.5              0.2
    4                 5.0              3.6               1.4              0.2
    ..                ...              ...               ...              ...
    145               6.7              3.0               5.2              2.3
    146               6.3              2.5               5.0              1.9
    147               6.5              3.0               5.2              2.0
    148               6.2              3.4               5.4              2.3
    149               5.9              3.0               5.1              1.8

    [150 rows x 4 columns]



.. code-block:: python

  #create a new table in MySQL database and import this dataset into the table
  #using the engine we have created to connect to the database demodb in MySQL,
  #import the pandas dataframe 'data' as a new table in the database called 'iris_table_demo'
  data.to_sql(con=eng, name ='iris_table_demo')

The command above also has an option for the case when the table name matches a table which already exists in the database. This means that you can either replace and overwrite the table in the database, or append the current table in MySQL and add this dataset to the data which is already there.

.. code-block:: python

  #confirm that the table is in the database:
  eng.table_names()

    ['example', 'iris_table', 'iris_table_demo', 'tutorials_tbl']


This shows that we have a new table in the database demodb called 'iris_table_demo'.

Let's look at the table iris_table and print the records from that table.


.. code-block:: python

  query = "SELECT * FROM iris_table"
  cursor.execute(query)
  records = cursor.fetchall()
  print(records)


    ((None, 5.1, 3.5, 1.4, 0.2), (None, 4.9, 3.0, 1.4, 0.2), (None, 4.7, 3.2, 1.3, 0.2), (None, 4.6, 3.1, 1.5, 0.2), (None, 5.0, 3.6, 1.4, 0.2), (None, 5.4, 3.9, 1.7, 0.4), (None, 4.6, 3.4, 1.4, 0.3), (None, 5.0, 3.4, 1.5, 0.2), (None, 4.4, 2.9, 1.4, 0.2), (None, 4.9, 3.1, 1.5, 0.1), (None, 5.4, 3.7, 1.5, 0.2), (None, 4.8, 3.4, 1.6, 0.2), (None, 4.8, 3.0, 1.4, 0.1), (None, 4.3, 3.0, 1.1, 0.1), (None, 5.8, 4.0, 1.2, 0.2), (None, 5.7, 4.4, 1.5, 0.4), (None, 5.4, 3.9, 1.3, 0.4), (None, 5.1, 3.5, 1.4, 0.3), (None, 5.7, 3.8, 1.7, 0.3), (None, 5.1, 3.8, 1.5, 0.3), (None, 5.4, 3.4, 1.7, 0.2), (None, 5.1, 3.7, 1.5, 0.4), (None, 4.6, 3.6, 1.0, 0.2), (None, 5.1, 3.3, 1.7, 0.5), (None, 4.8, 3.4, 1.9, 0.2), (None, 5.0, 3.0, 1.6, 0.2), (None, 5.0, 3.4, 1.6, 0.4), (None, 5.2, 3.5, 1.5, 0.2), (None, 5.2, 3.4, 1.4, 0.2), (None, 4.7, 3.2, 1.6, 0.2), (None, 4.8, 3.1, 1.6, 0.2), (None, 5.4, 3.4, 1.5, 0.4), (None, 5.2, 4.1, 1.5, 0.1), (None, 5.5, 4.2, 1.4, 0.2), (None, 4.9, 3.1, 1.5, 0.2), (None, 5.0, 3.2, 1.2, 0.2), (None, 5.5, 3.5, 1.3, 0.2), (None, 4.9, 3.6, 1.4, 0.1), (None, 4.4, 3.0, 1.3, 0.2), (None, 5.1, 3.4, 1.5, 0.2), (None, 5.0, 3.5, 1.3, 0.3), (None, 4.5, 2.3, 1.3, 0.3), (None, 4.4, 3.2, 1.3, 0.2), (None, 5.0, 3.5, 1.6, 0.6), (None, 5.1, 3.8, 1.9, 0.4), (None, 4.8, 3.0, 1.4, 0.3), (None, 5.1, 3.8, 1.6, 0.2), (None, 4.6, 3.2, 1.4, 0.2), (None, 5.3, 3.7, 1.5, 0.2), (None, 5.0, 3.3, 1.4, 0.2), (None, 7.0, 3.2, 4.7, 1.4), (None, 6.4, 3.2, 4.5, 1.5), (None, 6.9, 3.1, 4.9, 1.5), (None, 5.5, 2.3, 4.0, 1.3), (None, 6.5, 2.8, 4.6, 1.5), (None, 5.7, 2.8, 4.5, 1.3), (None, 6.3, 3.3, 4.7, 1.6), (None, 4.9, 2.4, 3.3, 1.0), (None, 6.6, 2.9, 4.6, 1.3), (None, 5.2, 2.7, 3.9, 1.4), (None, 5.0, 2.0, 3.5, 1.0), (None, 5.9, 3.0, 4.2, 1.5), (None, 6.0, 2.2, 4.0, 1.0), (None, 6.1, 2.9, 4.7, 1.4), (None, 5.6, 2.9, 3.6, 1.3), (None, 6.7, 3.1, 4.4, 1.4), (None, 5.6, 3.0, 4.5, 1.5), (None, 5.8, 2.7, 4.1, 1.0), (None, 6.2, 2.2, 4.5, 1.5), (None, 5.6, 2.5, 3.9, 1.1), (None, 5.9, 3.2, 4.8, 1.8), (None, 6.1, 2.8, 4.0, 1.3), (None, 6.3, 2.5, 4.9, 1.5), (None, 6.1, 2.8, 4.7, 1.2), (None, 6.4, 2.9, 4.3, 1.3), (None, 6.6, 3.0, 4.4, 1.4), (None, 6.8, 2.8, 4.8, 1.4), (None, 6.7, 3.0, 5.0, 1.7), (None, 6.0, 2.9, 4.5, 1.5), (None, 5.7, 2.6, 3.5, 1.0), (None, 5.5, 2.4, 3.8, 1.1), (None, 5.5, 2.4, 3.7, 1.0), (None, 5.8, 2.7, 3.9, 1.2), (None, 6.0, 2.7, 5.1, 1.6), (None, 5.4, 3.0, 4.5, 1.5), (None, 6.0, 3.4, 4.5, 1.6), (None, 6.7, 3.1, 4.7, 1.5), (None, 6.3, 2.3, 4.4, 1.3), (None, 5.6, 3.0, 4.1, 1.3), (None, 5.5, 2.5, 4.0, 1.3), (None, 5.5, 2.6, 4.4, 1.2), (None, 6.1, 3.0, 4.6, 1.4), (None, 5.8, 2.6, 4.0, 1.2), (None, 5.0, 2.3, 3.3, 1.0), (None, 5.6, 2.7, 4.2, 1.3), (None, 5.7, 3.0, 4.2, 1.2), (None, 5.7, 2.9, 4.2, 1.3), (None, 6.2, 2.9, 4.3, 1.3), (None, 5.1, 2.5, 3.0, 1.1), (None, 5.7, 2.8, 4.1, 1.3), (None, 6.3, 3.3, 6.0, 2.5), (None, 5.8, 2.7, 5.1, 1.9), (None, 7.1, 3.0, 5.9, 2.1), (None, 6.3, 2.9, 5.6, 1.8), (None, 6.5, 3.0, 5.8, 2.2), (None, 7.6, 3.0, 6.6, 2.1), (None, 4.9, 2.5, 4.5, 1.7), (None, 7.3, 2.9, 6.3, 1.8), (None, 6.7, 2.5, 5.8, 1.8), (None, 7.2, 3.6, 6.1, 2.5), (None, 6.5, 3.2, 5.1, 2.0), (None, 6.4, 2.7, 5.3, 1.9), (None, 6.8, 3.0, 5.5, 2.1), (None, 5.7, 2.5, 5.0, 2.0), (None, 5.8, 2.8, 5.1, 2.4), (None, 6.4, 3.2, 5.3, 2.3), (None, 6.5, 3.0, 5.5, 1.8), (None, 7.7, 3.8, 6.7, 2.2), (None, 7.7, 2.6, 6.9, 2.3), (None, 6.0, 2.2, 5.0, 1.5), (None, 6.9, 3.2, 5.7, 2.3), (None, 5.6, 2.8, 4.9, 2.0), (None, 7.7, 2.8, 6.7, 2.0), (None, 6.3, 2.7, 4.9, 1.8), (None, 6.7, 3.3, 5.7, 2.1), (None, 7.2, 3.2, 6.0, 1.8), (None, 6.2, 2.8, 4.8, 1.8), (None, 6.1, 3.0, 4.9, 1.8), (None, 6.4, 2.8, 5.6, 2.1), (None, 7.2, 3.0, 5.8, 1.6), (None, 7.4, 2.8, 6.1, 1.9), (None, 7.9, 3.8, 6.4, 2.0), (None, 6.4, 2.8, 5.6, 2.2), (None, 6.3, 2.8, 5.1, 1.5), (None, 6.1, 2.6, 5.6, 1.4), (None, 7.7, 3.0, 6.1, 2.3), (None, 6.3, 3.4, 5.6, 2.4), (None, 6.4, 3.1, 5.5, 1.8), (None, 6.0, 3.0, 4.8, 1.8), (None, 6.9, 3.1, 5.4, 2.1), (None, 6.7, 3.1, 5.6, 2.4), (None, 6.9, 3.1, 5.1, 2.3), (None, 5.8, 2.7, 5.1, 1.9), (None, 6.8, 3.2, 5.9, 2.3), (None, 6.7, 3.3, 5.7, 2.5), (None, 6.7, 3.0, 5.2, 2.3), (None, 6.3, 2.5, 5.0, 1.9), (None, 6.5, 3.0, 5.2, 2.0), (None, 6.2, 3.4, 5.4, 2.3), (None, 5.9, 3.0, 5.1, 1.8))


However, it is better to first import the table into a pandas data frame:


.. code-block:: python

  df = pd.read_sql('SELECT * FROM iris_table', con=connection)

.. code-block:: python

  print(df)

.. code-block:: bash

    index  ('sepal length (cm)',)  ('sepal width (cm)',)  \
    0    None                     5.1                    3.5
    1    None                     4.9                    3.0
    2    None                     4.7                    3.2
    3    None                     4.6                    3.1
    4    None                     5.0                    3.6
    ..    ...                     ...                    ...
    145  None                     6.7                    3.0
    146  None                     6.3                    2.5
    147  None                     6.5                    3.0
    148  None                     6.2                    3.4
    149  None                     5.9                    3.0

    ('petal length (cm)',)  ('petal width (cm)',)
    0                       1.4                    0.2
    1                       1.4                    0.2
    2                       1.3                    0.2
    3                       1.5                    0.2
    4                       1.4                    0.2
    ..                      ...                    ...
    145                     5.2                    2.3
    146                     5.0                    1.9
    147                     5.2                    2.0
    148                     5.4                    2.3
    149                     5.1                    1.8

    [150 rows x 5 columns]



References
##########

https://overiq.com/sqlalchemy-101/installing-sqlalchemy-and-connecting-to-database/

https://pandas.pydata.org/pandas-docs/stable/reference/index.html

https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.to_sql.html

https://towardsdatascience.com/sqlalchemy-python-tutorial-79a577141a91

https://linuxize.com/post/show-tables-in-mysql-database/

https://pynative.com/python-mysql-select-query-to-fetch-data/


MySQL
#####

A list of useful MySQL commands can be found here:

http://g2pc1.bu.edu/~qzpeng/manual/MySQL%20Commands.htm
