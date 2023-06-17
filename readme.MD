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

**Step 2: Dockerizing the Application**





SQLALCHEMY_DATABASE_URI=postgresql//admin:password@twoge-database-1.ctfbo8wbotzl.ca-central-1.rds.amazonaws.com:5432/mytwoge_db
