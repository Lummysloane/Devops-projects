# IMPLEMENT A CLIENT SERVER ARCHITECTURE USING MYSQL DATABASE MANAGEMENT SYSTEM 

Client-server is a common architectural model used in computer networks and distributed systems. It refers to a relationship between two entities: a client and a server.
Client-Server refers to an architecture in which two or more computers are connected together over a network to send and receive requests between one another.

*Step 1* - **Create and configure two linux-based virtual servers using EC2 instances in AWS**

![startUbuntu](https://github.com/Lummysloane/Devops-Project/assets/131771280/6c787225-a8d9-46ca-85e0-ac5dd5b650c2)

*Step 2* - **On mysql server Linux Server install MySQL Server software**

`sudo apt update -y` and `sudo apt install mysql-server -y`

![sudoupdate](https://github.com/Lummysloane/Devops-Project/assets/131771280/a28a0757-3f37-44b9-b90c-2df648e822f9)

![installmysql](https://github.com/Lummysloane/Devops-Project/assets/131771280/1ef2c0be-fba8-492a-b1ba-fe30101a3bab)

Enable mysql by typing the below code

`sudo systemctl enable mysql`

![enablemysql](https://github.com/Lummysloane/Devops-Project/assets/131771280/04a807ad-5058-4ac2-b5b1-8975b8ebe505)

*Step 3* - **On mysql client Linux Server install MySQL Client software**

follow the same process by updating and installing mysql-client

![Installclient](https://github.com/Lummysloane/Devops-Project/assets/131771280/d6605d5b-a74b-4a13-9450-b5d7e18f7f44)

*Step 4* - **Create a new Inbound rules in mysql server security group**

> MySQL server uses TCP port 3306 by default, so you will have to open it by creating a new entry in ‘Inbound rules’ in ‘mysql server’ Security Groups

> Populate the IP address of the client for port 3306 as shown in the diagram below

![3306](https://github.com/Lummysloane/Devops-Project/assets/131771280/d7d339f2-f7f2-4335-9e85-65afd81eb847)

*Step 5* - **Install Database and User on Mysql Server**

To run a security script on mysql server, type in code `sudo mysql_secure_installation`

![secureinstallationmysql](https://github.com/Lummysloane/Devops-Project/assets/131771280/18b0deb5-c836-41e2-ae62-d046478888d5)

Run the `sudo mysql` command on the mysql-server

On the mysql server, create remote_user, create Database test_db

![creatdatabase](https://github.com/Lummysloane/Devops-Project/assets/131771280/8d736cac-77f7-4030-8436-e959744fde62)

*Step 6* - **Configure MySQL server to allow connections from remote hosts**

`sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf`

![bindaddress](https://github.com/Lummysloane/Devops-Project/assets/131771280/1268a60f-6114-4494-93e7-303290a530f7)

on the mysql-server, retart mysql with this command `sudo systemctl restart mysql'

*Step 7* - **Connect remotely to m,ysql server from mysql client**

From mysql client Linux Server connect remotely to mysql server Database Engine without using SSH. You must use the mysql utility to perform this action.

To connect to mysql server from mysql client, run command: `sudo mysql -u remote_user -h 172.31.41.79 -p'

To check that we have successfully connected to a remote mysql server and can perform SQL queries; run command `show database;`

![remoteuser2](https://github.com/Lummysloane/Devops-Project/assets/131771280/3aaee317-c86c-4664-9cce-3f364b2f9b3d)















