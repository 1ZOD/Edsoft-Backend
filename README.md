# Edsoft-Backend
Desenvolver uma função para ser executada em um ambiente Cloud, preferencialmente em lambdas na AWS

Podemos usar esse trecho de código na função lambda da AWS para extrair o conteúdo do arquivo CSV do S3 e armazenar esse conteúdo do arquivo csv no MySQL.

## Criando database (mysql)

AWS -> Services -> RDS -> create database

## Options:
        Standard Create
        MySQL
        MySQL 5.7.22
        Free tier
        
## Settings:
        transaction-instance

## Credential Settings:
        admin
        masterKey (sua escolha)
        masterkey (sua escolha)

## DBinstace size:
        Bustable classes
        db.t2.micro

## Storage type:
        disable "Enable storage autoscalling"

## Connectivity:
        Additional connectivity configuration 🔽
          Publicly aceessible = Yes
          Availability Zone = us-east-1a
        
        Additional configuration 🔽
          initial database name = employess
          
        Backup
          disable = "Enable automatic backups" ," Copy tags to snapshot "
          
        Maintenace
          disable = "Enable auto minor version updade"
        
        ## Create Database ## 

        
## Lambda
AWS -> Services -> Lambda -> create function

## Create Function
       Function name
         s3_to_mysql
       Runtime 
         Python 3.7
       Executtion role
          use an existing role
          Existing role
            adminrole
            
       ## Create Function ##


## Function code 


Copie e cola o codigo a baixo 
(ALTERE OS CAMPOS NECESSARIOS NO CODIGO)

`rds_endpoint = Services -> RDS -> DB instances(1/40) -> transaction-instance -> Copie o Endpoit "transaction-instance......"`
`password = "senhaexemplo"`
`db_name = "employees"`
`cur.execute("select * from Employees")`

```Python

import boto3
import pymysql
import re
import datetime

s3_cient = boto3.client('s3')

# Read CSV file content from S3 bucket
def read_data_from_s3(event):
    bucket_name = event["Records"][0]["s3"]["bucket"]["name"]
    s3_file_name = event["Records"][0]["s3"]["object"]["key"]
    resp = s3_cient.get_object(Bucket=bucket_name, Key=s3_file_name)

    data = resp['Body'].read().decode('utf-8')
    data = data.split("\n")

    data = data.replace('.','').replace('-','')
    
    return data

def lambda_handler(event, context):
    rds_endpoint  = ""
    username = "admin"
    password = "" # RDS Mysql password
    db_name = "" # RDS MySQL DB name
    conn = None
    try:
        conn = pymysql.connect(rds_endpoint, user=username, passwd=password, db=db_name, connect_timeout=5)
    except pymysql.MySQLError as e:
        print("ERROR: Unexpected error: Could not connect to MySQL instance.")

    try:
        cur = conn.cursor()
        cur.execute("create table Employees ( id INT NOT NULL AUTO_INCREMENT, Name varchar(255) NOT NULL, PRIMARY KEY (id))")
        conn.commit()
    except:
        pass

    data = read_data_from_s3(event)

    with conn.cursor() as cur:
        for emp in data: # Iterate over S3 csv file content and insert into MySQL database
            try:
                emp = emp.replace("\n","").split(",")
                print (">>>>>>>"+str(emp))
                cur.execute('insert into Employees (Name) values("'+str(emp[1])+'")')
                conn.commit()
            except:
                continue
        cur.execute("select count(*) from Employees")
        print ("Total records on DB :"+str(cur.fetchall()[0]))
        # Display employee table records
        for row in cur:
            print (row)
    if conn:
        conn.commit()

```
## Caso não funcione 
    Install pymysql using pip on your local machine
    
        mkdir /home/RDSCode
        cd /home/RDSCode
        pip install -t /home/RDSCode pymysql
        touch lambda_function.py 
        *. copie o codigo em python no lambda_handler.py
        
    zip the code
        zip -r . RDScode.zip 
      
    Faça upload no AWS Lambda function
     Actions -> UpLoad a .zip file -> RDScode -> RDScode.zip
     
## Criar o Bucket
    Services -> s3 -> create bucket

## Create bucket
    name: techhub bucket (nome de exemplo)
    Create

## Propriedades do bucket
    Propriedas -> Events -> Add notification -> all object create events
    Send to: Lambda Function
    Lambda: s3_to_mySQL
    save
    
## UPload csv file
    Overview bucket
    Upload
    
    (Crie um arquivo CSV com as informaçoes que voce desejar)
    
## Verificar se deu certo
    AWS -> Services -> cloudwath -> log groups -> /aws/lambda -> log stream
    



