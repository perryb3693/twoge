# **Twoge**

<h1 align="center">
	<img
		width="500"
		alt="Project Twoge"
		src="./static/img/twoge-cover.png">
</h1>


The purpose of this project is to implement a CI/CD (Continuous Integration/Continuous Deployment) pipeline for a Python Flask-based Twitter alternative called "Twoge". The pipeline will utilize GitHub Actions for automation, Docker for containerization, Elastic Beanstalk for deployment, and AWS RDS Postgres for the database server. Source application files can be found at https://github.com/chandradeoarya/twoge/

**Step 1: Setting up the Project**

Basic functionality of the Twoge web application will be copied to and tested on the local machine through a python virtual environment, prior to CICD integration.

First, clone the repository to the local machine and initialize the python virtual environment within the working directory with all required packages from the "requirements.txt" text file. 
```
git clone https://github.com/perryb3693/twoge.git
python3 -m venv venv
source venv/bin/activate																#activate the virtual environment "venv"
pip install -r requirements.txt																#installs packages listed within txt file
export SQLALCHEMY_DATABASE_URI=postgresql://postgres:password@twoge-database-1.ctfbo8wbotzl.ca-central-1.rds.amazonaws.com/mytwoge_db 			#set environment variable within venv
```
Create an AWS RDS Postgres Database Server
1. Navigate to AWS RDS through the web interface and select 'create database'
2. Select Standard Create for database creation method
3. Select PostgreSQL as the engine option and the 'free tier' template
4. Configure database name, master username/password
5. Select security group, allowing traffic to port 5432 and 22
6. Enable public access and launch database
Install PostgreSQL on the local machine and configure the database through the CLI
```
sudo apt install postgresql
psql --host=twoge-database-1.ctfbo8wbotzl.ca-central-1.rds.amazonaws.com --port=5432 --username=postgres
postgres=> create database mytwoge_db;
postgres=> create user admin with encrypted password 'password';
postgres=> grant all privileges on database mytwoge_db to admin;
postgres=> \du 					#confirm user account creation
postgres=> \l 					#confirm database creation
```
Update the port number within the app.py file to port 5000 and run the Twoge application from within the venv
```
vim app.py					#update port number
python app.py					#execute the python application
```
Navigate to `http://localhost:5000` in the web browser to use Twoge. Use ^C in the CLI to close the application. 

**Step 2: Testing Application using Docker Compose**

Next, the Twoge application will be tested on the local machine using Docker Compose. Docker Compose will utilize Dockerfile to build the image for the application, as well as creating application and database container.

Create a Dockerfile in the project root directory. 
```
FROM python:alpine                                              #base image
WORKDIR /app                                                 
COPY ./requirements.txt /app                                    
RUN pip install -r requirements.txt                             # install dependencies from requirements.txt file
COPY . .                                                        # copy app files into the app directory
EXPOSE 5000                                                     # applicatio will run on port 5000
CMD ["python", "app.py"]
```
Use Docker Compose to define and configure Twoge as a multi-container Docker application. The database will be provisioned on the local machine, requiring the creation of two containers; one for the application and one for the database server. 
```
version: '3'
services:
  database:
    image: postgres
    container_name: twogedb_c
    environment:
      - POSTGRES_USER=admin                     				#the PostgreSQL user
      - POSTGRES_PASSWORD=password              				#user db password
      - POSTGRES_DB=twoge_db             					#PostgreSQL default database created at launch
    ports:
      - 5432:5432                               				#PostgreSQL default port number
    networks:
      - twoge-net

  web:
    build: .                                    				#build image using workdir Dockerfile
    restart: always
    container_name: twoge_app
    ports:
      - 5000:5000                               				#map port 5000 on the container to port 5000 on local machine 
    environment:
      - SQLALCHEMY_DATABASE_URI=postgres://admin:password@database/twoge_db	#env variable must remain in the correct sqlalchemy uri format
    depends_on:
      - database
    networks:
      - twoge-net

networks:
  twoge-net:
    driver: bridge
```
Run the docker-compose yaml file in the CLI
```
docker-compose up    								  
```
Use `docker-compose down` to clean up docker environment once complete. 

**Step 3: Implementing CI/CD using GitHub Actions**

In order to implement CI/CD, GitHub Actions will be utilized to automate checkingout code from the repository on Github, building and testing the Docker image, pushing the Docker image to DockerHub and deploying the application to Elastic Beanstalk. 

First, creat a .github/workflows directory in the project repository on the local machine and create a YAML file to define the GitHub Actions workflow once pushed to the repository.
```
mkdir -p .github/workflows
cd .github/workflows
vim ci-cd.yml
```
```
name: Deploy Twoge Application

on:
  push:
    branches:
      - master

env:
  DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
  DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
  AWS_REGION: 'ca-central-1'

jobs:
  build_and_push:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2
        with:
          ref: master

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v1

      - name: Login to Docker Hub
        uses: docker/login-action@v1
        with:
          username: ${{ env.DOCKER_USERNAME }}
          password: ${{ env.DOCKER_PASSWORD }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v2
        with:
          context: .
          push: true
          tags: ${{ env.DOCKER_USERNAME }}/twoge_app:latest

  eb_deploy:
    needs: build_and_push
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2
        with:
          ref: master

      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: 3.x

      - name: Install EB CLI
        run: |
          pip install awsebcli --upgrade

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Init Elastic Beanstalk
        run: |
          echo n | eb init 

      - name: Deploy to Elastic Beanstalk
        run: |
          eb deploy
```
Define access variables within Github for use within GitHub actions as it executes the yaml configuration file.

<img width="593" alt="image" src="https://github.com/perryb3693/twoge/assets/129805541/61e7689f-5b4e-4773-ad2f-eedd0b2e7818">

**Step 4: Setting Up Elastic Beanstalk**

First, configure the Elastic Beanstalk Twoge environment by running the `eb init` command in the main project directory with the following configurations:
1. Choose Region
2. Create New Application
3. Define Application Name: twoge
4. Docker? Y
5. Platform Branch: Docker running on 64bit Amazon Linux 2
6. CodeCommit? N
7. SSH? N

Once complete, the configuration file will be compiled in `./.elasticbeanstalk/config.yml.

Comment out or delete elasticbeanstalk hidden files found within the .gitignore document found within the main project directory

<img width="578" alt="image" src="https://github.com/perryb3693/twoge/assets/129805541/eb045002-edb7-4b6d-8bbc-327a3f783341">

Commit and push application files to the remote repository. GitHub Actions will automatically execute the ci-cd.yml workflow once git push is complete. 
```
 git add .							#stages files within the current directory
 git commit -m " "						#commit staged files
 git push origin master						#upload files to remote repository
```
Confirm repository updates on GitHub and DockerHub. Check GitHub Actions workflow to ensure proper execution. 

<img width="944" alt="image" src="https://github.com/perryb3693/twoge/assets/129805541/1d1ff992-4527-4d8e-af81-a2d1363747e8">

Next, create the Elastic Beanstock environment and define environment variables through the EB CLI:
```
eb create twoge-eb --single
```
Once the EB environment is finished building, set the environment variable using the following command:
```
eb setenv SQLALCHEMY_DATABASE_URI=postgresql://admin:password@twoge-database-1.ctfbo8wbotzl.ca-central-1.rds.amazonaws.com:5432/mytwoge_db
```
Once the EB environment has finished updating, update the inbound rule on the instances associated security group to allow incoming TCP traffic on port 5000.

<img width="830" alt="image" src="https://github.com/perryb3693/twoge/assets/129805541/66e4d4d1-6b5d-4dec-bc5d-5c771ec155e3">

Navigate to the domain associated with the Elastic Beanstalk's environment and post your first blog!

<img width="953" alt="image" src="https://github.com/perryb3693/twoge/assets/129805541/6ac90f63-e648-4da0-90af-b5b8661e929b">

<img width="955" alt="image" src="https://github.com/perryb3693/twoge/assets/129805541/4196d5d2-7f73-447a-83ca-61ff3800992a">

