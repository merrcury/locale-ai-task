
# Locale AI Task 

## Problem Statement 

XRides, delivers about a 200 rides per minute or 288,000 rides per day. Now, they want to send this data to your system via an API. Your task is to create this API and save the data into PostgreSQL.
The API should be designed, keeping in mind the real-time s
treaming nature of data and the burst of requests at peak times of the day. The user of this API expects an acknowledgment that the data is accepted and a way to track if the request fails.

## Utopia

### Tech Stack
+ Python
+ PostgreSQL (Database)
+ Docker (Container Service)
+ Azure (Cloud Service)
+ Ngnix (Reverse Proxy)
+ Let's Encrypt (for TLS)

### Framworks & Libraries
+ FastAPI (an API microframework) (10 times faster than flask)
+ Uvicorn (an ASGI server)
+ Starlette (an ASGI framework for asyncio services)
+ Swagger (documentation)
+ Pony (ORM)
+ Psycopg2 (library for connecting to PostgreSQL)

### Monitoring & Testing Services (Optional)
+ Datadog (for Azure Instance Health)
+ JMeter / Postman (API Testing)
+ Github Actions/ Circle CI (Continous Integration)

### Structure and Coding 

We will use a straight forward approach to develop the API. 
1. Define the Table in the Pony ORM. 
![trip.png](https://www.dropbox.com/s/t6t76hnx7s8iwcp/trip.png?dl=0&raw=1)
![tripdetails.png](https://www.dropbox.com/s/v2hyqear5qeh5p4/tripdetails.png?dl=0&raw=1)
Each class is a different table. It is specified that `booking_id` will be generated automatically by the ORM and we will use a function to retrieve it. Trip_details inherit Trip because of the  `booking_id`. We can also explicitly use `booking_id` for the same. These classes are termed as the model in Pony ORM. 

2. Defining a basic connection in the Pony ORM to PostgreSQL 
![basic import.png](https://www.dropbox.com/s/5td46la3g9x3yq9/basic%20import.png?dl=0&raw=1)
We can define insert query in the `with db_session:`. This will act as one session and will auto rollback on any failure. We can define the query in multiple ways, traditional PostgreSQL in `db.execute()` method, next is built-in using ORM `db.insert(table_name, variables, returning=booking_id)` where table_name is the class name, variables are simple values received in the POST request and returning define what we expect in return, Next way is to make an object to the class (for single insert, we pass them as normal arguments. For multiple, we can pass dict) and use function `get_booking_id` to get booking id. 

3. Use FastAPI to convert this all into API
![fastapi.png](https://www.dropbox.com/s/kuxto3n2clnhpym/fastapi.png?dl=0&raw=1)
In the same way, we can define OAuth, Token, JWT verification before making changes to DB and raise Authentication and Authorization error for security reason. Below image also show how to define documentation with API. 
![auth.png](https://www.dropbox.com/s/3w9r417inuuz24z/auth.png?dl=0&raw=1)
For the more protection, we can turn on cors 
![CORS.png](https://www.dropbox.com/s/czuegzqe349l45s/CORS.png?dl=0&raw=1)

4. Dockerize the FASTAPI. 
   + Save all the requirements
        ``` 
        pip freeze > requirements.txt
        ```
    + Create a Dockerfile and place below code 
        ![Dockerfile.png](https://www.dropbox.com/s/n6666f37q6ueg3g/Dockerfile.png?dl=0&raw=1)
    + Create docker
        ```
        $ docker build -t merrcury/localeai-task:v1 . 
        $ docker push merrcury/localeai-task:v1
        ```

5. Docker Compose for the FASTAPI considering, We are using hosted PostgreSQL. Below is the docker-compose. `docker-compose.yml`
        ![compose-1.png](https://www.dropbox.com/s/yipq5z1k8l58cnf/compose-1.png?dl=0&raw=1)
6. `docker-compose up -d` will start the setup. This same can be done with `stack.yml` for docker swarm (scaling in peak hours). 

7. For Scaling, If You choose to go with docker you can use Container Services, VMs or AWS Fargate and for Serverless We can drop FASTAPI. 

8. For PGADMIN-4, You can use setup from my Github Repository (https://github.com/merrcury/pgadmin4-docker). 

### Analytics 
As we are using Pony ORM, We have greater freedom in querying data. We can query data in multiple ways rather than designing traditional SQL Query for every use case. 
Basic Analytics option can be baked right into the API, like distance(between source and destination), Failure, Driver with most trips, most visited route. 
If We wanna talk more analytics and faster querying on data, I guess ELK Stack will be a perfect choice. (ElasticSearch, Logstash, Kibana)
+ A cron job to feed PostgreSQL data into Logstash, all the way to Elasticsearch. 
+ Kibana provide some fantastic dashboard. 
+ Elasticsearch queries are fastest in business due to Apache Lucene. ElasticSearch also has a built-in algorithm like n-gram. 
+ Reason being using Elasticsearch is Json/Document format rather than the tabular format of PostgreSQL. 

But PostgreSQL can also prove helpful if we use ORM efficiently. 

The possible system architecture is to develop it as microservice, giving it horizontal scalability and preventing it to be part of the legacy system. In the Legacy System, introducing lots of changes in API and can cause the issue to other moving parts.  
