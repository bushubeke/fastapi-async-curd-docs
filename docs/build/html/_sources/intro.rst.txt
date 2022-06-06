
Introduction
--------------
1.1 General Summary
=================== 

This is a documentation for FastAPI CURD( Create Update Read Delete) gnerator object for FastAPI. The object takes list of SQLAlchemy Base Model objects along with their 
Cresponding Pydantic Validation BaseModel objects for creation of the CURD route. It performs all tasks asynchronously and uses SQLAlchemy under the hood with asyncpg.It currently
works well with primitve datatype base object models.  

1.2 Required Packages
=====================


+-------------------+----------+
| Requirments       | Version  |
+===================+==========+
| SQLAlchemy        | >=1.4.32 |
+-------------------+----------+
| PostgreSQL        | >=13     |
+-------------------+----------+
| FastAPI           | 0.75.0   |
+-------------------+----------+


