Installation
--------------
You can simply use pip to install the package from pypy.

.. code:: python
    
    pip install fastapi-async-curd 

Additionally Install asyncg in for PostgreSQL to work with SQLAlchemy.
You can use the Cresponding async library depending on the type of database you use. 

.. code:: python
    
    pip install -U asyncpg 

.. note::
   The package Assumes you intende to use the asynchronous SQLAlchemy session.
   Additionally, it is tested using PostgreSQL version 14.0.It should normally fucntion with other 
   database types like sqlite and MySQL. The package recommends using manual update for non primitive
   datatypes like (JSON and ARRAY) type model objects.

    

.. danger::
    The package utilizes dataclasses. Hence, be sure to have python >=3.7 