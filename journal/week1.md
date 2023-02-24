# Week 1 — App Containerization

- Watched live stream video from YouTube 3 times to understand how things works because  I am a beginner in Docker
- Watched video by Chirag about billing and free tier usage limit https://youtu.be/OAMHu1NiYoI
- Watched video on YouTube about Docker for beginners by [Techworld with Nana](https://youtu.be/pg19Z8LL06w) to get an idea about **Docker**
- Followed the steps and scripts along with Andrew Brown and successfully created the app in container
- To create this watch Week1 live streaming recording on [YouTube](https://www.youtube.com/live/zJnNe5Nv4tE?feature=share) and follow instructions from [omenking
/aws-bootcamp-cruddur-2023](https://github.com/omenking/aws-bootcamp-cruddur-2023/blob/week-1/journal/week1.md) Github repo


**__Here is the outcome_👇_**
<img width="937" alt="real app" src="https://user-images.githubusercontent.com/111639918/220439622-75e043be-26fb-4b0e-99ef-5dc8e6722289.png">



- [**Create Dockerhub account**](https://hub.docker.com/)

**Push Docker images to Docker hub**


To do that👇

Open gitpod and enter the below command
```
docker login
docker images
docker tag IMAGE_NAME DOCKERHUB_USERNAME/REPOSITORY_NAME:TAG
docker push DOCKERHUB_USERNAME/REPOSITORY_NAME:TAG
```
<img width="956" alt="docker images" src="https://user-images.githubusercontent.com/111639918/220432612-c7d63c18-5859-4a95-a915-44d1f2270b53.png">


<img width="959" alt="backend image docker" src="https://user-images.githubusercontent.com/111639918/221002150-1a24ccbc-f3dc-42bb-9635-4820497529eb.png">


- **After use the push command go to dokcerhub account and verify it**;

<img width="944" alt="dockerhub repo" src="https://user-images.githubusercontent.com/111639918/220435734-feb6bf59-37a8-43ec-9cc3-b5ad9ccc18e2.png">
<img width="949" alt="dockerhub" src="https://user-images.githubusercontent.com/111639918/220432191-09a03c0c-1134-4ea8-9b2f-d30a9fa075f6.png">



# Create the notification feature (Backend and Front)

I created notification feature in my crudder app followed the instructions by[ Andrew Brown](https://youtu.be/k-_o0cCpksk). But after adding the notification feed page my Frontend endpoint was not working . It shows me a blank screen but the backend endpoint was working fine. Below I am sharing the snapshots.

 **Frontend endpoint**👇
 
<img width="931" alt="notification error" src="https://user-images.githubusercontent.com/111639918/221121574-35aa503f-46c2-4a21-a639-3f85f51ddf09.png">

**Backend endpoint**👇

<img width="614" alt="notification backend endpoint" src="https://user-images.githubusercontent.com/111639918/221121596-ee9a555e-9cdc-4e7e-ab3d-e5ad9b950ff3.png">

I asked help on Discord then I realize before asking any questions I should do some troubleshoot from my end. 

I watched video about [Before You Ask For Help Watch This](https://youtu.be/tDPqmwKMP7Y) it helped to understand how to ask help and how to troubleshoot from my end before seeking help.

I followed the instructions and opened my crudder app--> inspect--> network--> refresh. Here is the error message that I got 👇

![error message](https://user-images.githubusercontent.com/111639918/221123432-bb65ceb8-3867-4598-98aa-125a4d00f3e0.png)

Honestly I did'nt understood what this error is😅. I shared this error to one of the bootcamp candidate on Disord. She asked me to check my ```NotificationsFeedPage.js``` That time I saw instead of ```NotificationsFeedPage.js``` I saved the page as ```otificationsFeedPage.js``` 😅

After correcting the file name my crudder app started working. But there's nothing in the feed page 👇



<img width="935" alt="notification without feed" src="https://user-images.githubusercontent.com/111639918/221124821-d625f088-6171-4189-ad21-9f5e75dc4826.png">

When I check the port ```4567``` it was private. I made ```4567``` port as public then everyhting worked fine.

**Here is my notification feature (Backend and Front) after all troubleshooting** 👇
<img width="942" alt="notification" src="https://user-images.githubusercontent.com/111639918/221125287-095098ea-19f4-4a07-9c1d-379be50689a1.png">

- I addressed my error and troubleshooting that I followed to resolve the issue in the Discord **week1-docker server**

👀watched [Week 1 - DynamoDB and Postgres vs Docker](https://youtu.be/CbQNMaa6zTg)

-  Followed instructions and Created DynamoDB table. To create DynamoDB table I used command from ** DynamoDB local** https://github.com/100DaysOfCloud/challenge-dynamodb-local

```
# add this file into your docker-compose.yml

services:
  dynamodb-local:
    # https://stackoverflow.com/questions/67533058/persist-local-dynamodb-data-in-volumes-lack-permission-unable-to-open-databa
    # We needed to add user:root to get this working.
    user: root
    command: "-jar DynamoDBLocal.jar -sharedDb -dbPath ./data"
    image: "amazon/dynamodb-local:latest"
    container_name: dynamodb-local
    ports:
      - "8000:8000"
    volumes:
      - "./docker/dynamodb:/home/dynamodblocal/data"
    working_dir: /home/dynamodblocal
    
```    
**Here is the table that I created 👇**
![image](https://user-images.githubusercontent.com/111639918/221256196-4874696b-34ed-4f21-8b15-d6abf7928088.png)
![image](https://user-images.githubusercontent.com/111639918/221256489-6217d0a9-f24c-4d90-a042-29a4b64801db.png)


-  Installed postgresql using the below command

```
#updated this command in my .gitpod.yml file and installed postgresql client using terminal

  -  name: postgres
     init: |
      - curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
      - echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
      - sudo apt update
      - sudo apt install -y postgresql-client-13 libpq-dev
 
 ```
 
 In my gitpod **Postgresql** was not installed, so I installed postgresql from extensions and added a file. Then connected to Database server

**To login to Postgre sql use thiscommand**
```
psql -Upostgres --host localhost 
```
![image](https://user-images.githubusercontent.com/111639918/221193595-47a33d84-5fb5-4526-8c21-5300b224135d.png)



