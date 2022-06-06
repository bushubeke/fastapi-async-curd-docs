Quickstart
------------
Before starting install the following packages as the sample app will utilize them.

.. code:: python
    
    pip install -U pyjwt

.. code:: python
    
    pip install -U passlib

.. code:: python
    
    pip install -U typer

.. code:: python
    
    pip install -U gunicorn uvicorn[standard]


3.1 Folder Structure
==================== 
::

    yourproject(virtualenv)
    ├── main.py
    ├── config.py
    └── models
        ├── __init__.py
        ├── dbconnect.py
        └── dbmodels.py


3.2 Sample app
============== 

3.2.1 dbconnect.py
++++++++++++++++++

creating sqlalchemy  base and async engine for initialization

.. code:: python
    
    
    from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
    from sqlalchemy.orm import declarative_base, sessionmaker
    from .config import settings

   
    asyncengine=create_async_engine(settings.DATABASE_ASYNC_URI,echo=True)
    Base = declarative_base()   
    #####################################################
    async def async_main():
        async with asyncengine.begin() as conn:
            await conn.run_sync(Base.metadata.drop_all)
            await conn.run_sync(Base.metadata.create_all)

    ######################################################
    async def droptables():
        async with asyncengine.begin() as conn:
            await conn.run_sync(Base.metadata.drop_all)
    ######################################################

3.2.1 dbmodels.py
++++++++++++++++++

The SQLAlchemyCURD object expects the The SQL Base Declarative model to have three static methods defined 
as it can be seen the code snippet below. the read_roles,write_roles and tableroute method. read_roles and write_roles should return a list of strings
whereas tableroute should return a string where the curd functionality for the model will be served at. If no roles required just return "none" in the 
list.  

.. code:: python
    
    import uuid
    from enum import unique
    from datetime import tzinfo, timedelta, datetime
    from sqlalchemy.orm import relationship, backref
    from sqlalchemy import Boolean, DateTime, Column, Integer,String, ForeignKey, Sequence,Enum
    from sqlalchemy.dialects.postgresql import ENUM, JSON,UUID,ARRAY,JSONB
    from sqlalchemy.sql import func
    from sqlalchemy.orm.attributes import flag_modified
    from sqlalchemy_json import mutable_json_type
    from pydantic import BaseModel
    from pydantic.dataclasses import dataclass
    from typing import List, Optional,Literal
    from .dbconnect import Base

    ###########################################################################
        #this is for functionality testing of variable types 
    ############################################################################

    skill_level_enum =ENUM('Zero', 'A little', 'Some', 'A lot', name="skill_level",create_type=True)
    class TestingDataTypes(Base):
        __tablename__ = 'testing_table'
        id = Column(Integer(), autoincrement="auto",primary_key=True)
        uuid = Column(UUID(as_uuid=True),unique=True,default=uuid.uuid4)
        JSON_TYPE = Column(mutable_json_type(dbtype=JSON, nested=True),nullable=True)
        JSONB_TYPE = Column(mutable_json_type(dbtype=JSONB, nested=True),nullable=True)
        ARRAY_TYPE = Column(ARRAY(Integer),nullable=True )
        ENUM_TYPE = Column(skill_level_enum,default="Zero")
        STRING_TYPE = Column(String(50),nullable=True,default="empty")
        BOOLEAN_TYPE = Column(Boolean(),default=False,nullable=False)
        DATE_TIMESTAMP_TYPE = Column(DateTime(timezone=True), default=func.now())
        
        def __repr__(self):
            return f"<TestingDatatypes '{self.id}'>"
        
        @staticmethod
        def read_roles():
            return ["superuser"]
        
        @staticmethod
        def write_roles():
            return ['superuser'] 
            
        @staticmethod
        def tableroute():
            return "test"    

    @dataclass
    class json_item:
        jid: int
        name: str = 'John Doe'
        test_list: Optional[List[str]] = None

    @dataclass
    class jsonb_item:
        jbid: int
        name: str = 'John Doe'
        test_list: Optional[List[str]] = None

    class TestingTableModel(BaseModel):
        id: Optional[int]
        uuid: Optional[uuid.UUID]
        JSON_TYPE :Optional[json_item]
        JSONB_TYPE : Optional[jsonb_item] 
        ARRAY_TYPE : Optional[List[int]]
        ENUM_TYPE : Optional[Literal['Zero', 'A little', 'Some', 'A lot']]
        BOOLEAN_TYPE : Optional[bool]
        STRING_TYPE:Optional[str]
        DATE_TIMESTAMP_TYPE : Optional[datetime]  
        class Config:
            orm_mode = True

    class TestingTablePostModel(BaseModel):        
        JSON_TYPE :Optional[json_item]
        JSONB_TYPE : Optional[jsonb_item] 
        ARRAY_TYPE : Optional[List[int]]
        ENUM_TYPE : Optional[Literal['Zero', 'A little', 'Some', 'A lot']]
        BOOLEAN_TYPE : Optional[bool]
        STRING_TYPE:Optional[str]
        
        class Config:
            orm_mode = True


3.2.2 config.py
+++++++++++++++++

.. code:: python
    
    import os
    from pydantic import BaseSettings 
    class Settings(BaseSettings):
    
        
        DATABASE_ASYNC_URI : str=os.getenv("DATABASE_ASYNC_URI")
        SECRET_KEY : str = os.getenv("SECRET_KEY")
        class Config:
            case_sensitive = True
            env_file = ".env"

    settings = Settings()

3.2.3 main.py
+++++++++++++

.. code:: python
    
    import jwt
    import typer
    import asyncio
    import subprocess
    from fastapi import FastAPI,Header,Depends,HTTPException,Request
    from fastapi.middleware.cors import CORSMiddleware
    from typing import Optional
    from models.dbconnect import asyncengine,async_main,droptables
    from models.dbmodels import TestingDataTypes,TestingTableModel,TestingTablePostModel
    from fastapi.security import OAuth2PasswordBearer,OAuth2PasswordRequestForm
    from json import  dumps as jsondumps,JSONDecoder
    from passlib.hash import pbkdf2_sha512
    from datetime import datetime,timedelta
    from curd.sqlcurd import SQLAlchemyCURD
    from models.config import settings
    capp = typer.Typer()
    sqlcurd= SQLAlchemyCURD()

    def create_dev_app():
        app=FastAPI(title="Sample CURD Generator App")
        
        ##########################################################
        oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/token")
        
        async def get_current_user(token: str = Depends(oauth2_scheme)):
            user= jwt.decode(token,settings.SECRET_KEY,algorithms="HS256")
            user=user["data"]
            if not user:
                raise HTTPException(
                    status_code=status.HTTP_401_UNAUTHORIZED,
                    detail="Invalid authentication credentials",
                    headers={"WWW-Authenticate": "Bearer"},
                )
            user=jsondumps(user)    
            user=JSONDecoder().decode(user)
            return user
        async def get_current_active_user(current_user=Depends(get_current_user)):
            if current_user: 
                return current_user["roles"]
            return ["none"]
        ##########################################################
        ##########################################################
        
        sqlcurd.init_app(app,asyncengine)
        #setting current user fetching funciton
        sqlcurd.set_current_user(get_current_active_user)
        
        origins = [ "*"]
        app.add_middleware(
                CORSMiddleware,
                allow_origins=origins,
                allow_credentials=True,
                allow_methods=["*"],
                allow_headers=["*"],
                )
        # creating list of models to be used by the curd object
        # the first modlist should contain nested list of [SQLalchemy Base, PyDantic BaseModel,PyDantic BaseModale]
        # the last pydantic model provided is used to validate the post route for the model

        modlist=[[TestingDataTypes,TestingTableModel,TestingTablePostModel]]  
        
        # generating the curd routes for provided list of models
        sqlcurd.add_curd(modlist)
        
        # mouting the apirouter the FastAPI app at the provided prefix route. defaults to root path
        sqlcurd.include_apirouter(prefix="/")
        
        @app.get("/")
        def index(x_app_key: Optional[str] = Header(None) ,x_refresh_key: Optional[str] =Header(None)):
            return {"Message":"You should make your own index page"}
        
        @app.post("/token")
        def auth_sample(request : Request, login_data : OAuth2PasswordRequestForm =Depends()):
            try:                              
                data={
                    "roles":["superuser"],
                    "password" : pbkdf2_sha512.using(rounds=25000,salt_size=80).hash("password")
                }
                if  pbkdf2_sha512.verify(login_data.password,data["password"]):
                    exp=datetime.utcnow()+timedelta(hours=4)
                    exp2=datetime.utcnow()+timedelta(hours=5)
                    key=settings.SECRET_KEY 
                    del data["password"]              
                    token=jwt.encode({'data':data,'exp':exp},key,algorithm="HS256")
                    reftoken=jwt.encode({'data':data,'exp':exp2},key,algorithm="HS256")
                    return {"access_token": token,"refresh_token":reftoken, "token_type": "bearer"}
                else:
                    raise HTTPException(status_code=400, detail="Incorrect username or password")
            except Exception as e:
                    print(e)
                    raise HTTPException(status_code=500, detail="Message: Something Unexpected Happended") 
        return app 


        @capp.command()
        def upgrade():
            """creates  base models based on their methadata"""
            asyncio.run(async_main())
            
        @capp.command()
        def run():
            """starts gunicorn server of the app with uvicorn works bound  to 0.0.0.0:9000 with one worker
            """
            subprocess.run(["gunicorn", "main:app", "-k" ,"uvicorn.workers.UvicornWorker","-b" ,"0.0.0.0:9000","--reload","-w","1"]) 
        @capp.command()
        def drop():
            """drops all tables created from provided database"""
            asyncio.run(droptables())

        if __name__ == "__main__":
            capp()

        
Finally create the testing model sql upgrade command and run it.

.. code:: python

        pytohn main.py upgrade

.. code:: python

        pytohn main.py run

.. danger::
   If the DATABASE_ASYNC_URI is provided correctly, it should work. The URI should be of format
   "postgresql+asyncpg://postgres:something@192.168.10.5:5432/development-template" 
   
.. note::   
    The SECRET_KEY can be generated using python standard secets libaray as in "secerts.token_hex(128)"