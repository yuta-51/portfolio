---
layout: post
title:  "Invoice Document Generator"
info: "Python Desktop Applicaiton"
tech: "Python, MS Access"
type: Freelance Work
---

# Goal
The goal for this project was to create a desktop application that dynamically generates invoice documents based on an invoice number and data stored in an MS Access database. 
The company which will use this applicaiton distributes, exports, and imports farming supplies internationally, so an invoice auotmation system is something that would save lots of time and effort.  

# Tools
- Microsoft Access Database
  - Relational Database
- Python
  - mailmerge
  - pyodbc
  - python-docx
  - Tkinter
- SQL 
  - DQL statements

# Project Overview
 <img src="https://i.imgur.com/Y0CyXUt.jpg" alt="overview"
	title="project-overview" width="800"/>  


# Procedure


## Step 1: Set up the Relational Database

The database is a simple relatial database featureing three tables which store contractor, shipment, and container data. The contractor table stores information about differnet contractors like their address and contact information. The shipment table contains informaiton like where the shipment is shipped out of, where it is headed, and arrival times. Finally, the container table stores information about all of the containers associated with the shipment. The invoice number links together the  shipment information and its associated containers. 

 <img src="https://i.imgur.com/5e6pzPE.jpg" alt="relationships"
	title="db-relationships" width="800"/>  
  
## Step 2: Get User Inputs via. GUI
My library of choice for GUIs with Python is Tkinter. There are some important details about this project that should be mentioned here:
The user shall:
* Input an invoice number to specify which shipment will be documented. 
* Choose atleast 1 type of invoice document out of 4 types to generate: forwarder packing list, packing list, commercial invoice, and performa invoice documents.
* Be able to generate up to 4 differnet types of documents at the same time (one button press). 
* Be able to choose what attributes are included in the data table on the document.  

Based on these requirements, I created the following (logo/company name blurred):  
 <img src="https://i.imgur.com/2zY9m0z.png" alt="gui"
	title="gui-preview" width="900"/>  


As you can see, it is mostly checkbox-based. The user is required to input an invoice number and select at least one type of document to generate. Then, they are able to select which attributes they would like to include in the data table. There is also a reset button to reset all inputs. 


## Step 3: Establish Connection with DB
We use the pyodbc library to establish a cursor or connection with the DB so that we can run SQL queries and get back shipment data. 

```python
def open_db():
    try:
        global cnn
        conn_string = r'DRIVER={Microsoft Access Driver (*.mdb, *.accdb)};' \
                      f'DBQ={DATABASE_PATH};'
        cnn = pyo.connect(conn_string)
        return cnn.cursor()
    except pyo.Error:
        show_error_message("Failed to connect to database")
```

Simple SQL statments are used to query rows of container, contractor, and shipment data for later use. For example:

```python
def fetch_rows(table, id_name, _id):
    cursor = cnn.cursor()
    cursor.execute(f"SELECT * FROM {table} WHERE {id_name} ='{_id}'")
    return cursor.fetchall()
```

## Step 4: Establish Connection with DB
