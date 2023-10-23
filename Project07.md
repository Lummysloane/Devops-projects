# DEVOPS TOOLING WEBSITE SOLUTION

## In this project we will implement a solution that consists of following components:

1. Infrastructure: AWS

2. Webserver Linux: Red Hat Enterprise Linux 8
   
3. Database Server: Ubuntu 20.04 + MySQL
   
4. Storage Server: Red Hat Enterprise Linux 8 + NFS Server

5. Programming Language: PHP

5. Code Repository: GitHub

## 3-tier Web Application Architecture with a single Database and NFS server as a shared files storage 
   
![Screenshot 2023-08-16 at 15 04 54](https://github.com/Lummysloane/Devops-Project/assets/131771280/d2bdc573-8092-49c1-b4c9-b39a3972d0cd)

A Typical 3 Tier Server Architecture. Tier 1—Web Server, Tier 2—Application Server, Tier 3— DataBase Server. PicoServer is targeted at Tier 1. An example of an internet transaction is shown. When a client request comes in for a Java Servlet Page, it is first received by the front end server—Tier 1. Tier 1 recognizes a Java Servlet Page that must be handled and initiates a request to Tier 2 typically using Remote Message Interfaces (RMI). Tier 2 initiates a database query on the Tier 3 servers, which in turn generate the results and send the relevant information up the chain all the way to Tier 1. Finally, Tier 1 sends the generated content to the client.  

We have to launch 4 instances ( 1-3 Web server  & 1 Database server) on RHEL Linux 8 and 1 instance on ubuntu

![createservers](https://github.com/Lummysloane/Devops-projects/assets/131771280/914777bb-e3df-455d-a62c-896a014ef45a)


We have to create EBS volume and must make sure it is in the same availability zone as the instance we want to attach the EBS volume to.

![disk123](https://github.com/Lummysloane/Devops-Project/assets/131771280/9f05762b-2e62-49bb-a52e-d5bf617274f1)

We will use the gdisk to create partition for all the 3 disk

`sudo gdisk /dev/xvdf` 

![gdisk1](https://github.com/Lummysloane/Devops-Project/assets/131771280/e82f557c-9fe6-4711-a3fb-342438fa3d19)

`sudo gdisk /dev/xvdg`

![gdisk2](https://github.com/Lummysloane/Devops-Project/assets/131771280/9b3b8328-1f9f-4fd8-bff7-3bf20e56f9e8)

`sudo gdisk /dev/xvdh`

![gdisk3](https://github.com/Lummysloane/Devops-Project/assets/131771280/b05e1317-3eee-4eb1-a4fa-eaa3e79c393d)

When we run command `lsblk`, then we will see all have been partitioned as seen below:

![partitionlsblk](https://github.com/Lummysloane/Devops-Project/assets/131771280/338ff524-976a-44cc-9e2f-73924779ec74)

We have to install lvm2 by running his command: `sudo yum install lvm2 -y`

![installlvm2](https://github.com/Lummysloane/Devops-Project/assets/131771280/ac83c048-3fa7-4e83-9669-5e7eb46f7612)

Run command `sudo lvmdiskscan` to check for available partitions

We have to create physical volumes used by lvm. Run command:

`sudo pvcreate /dev/xvdf1`

`sudo pvcreate /dev/xvdg1`

`sudo pvcreate /dev/xvdh1`

![pvcreatenfs](https://github.com/Lummysloane/Devops-Project/assets/131771280/a7334302-e051-41f2-8f6e-6381ffea0ca0)

Run command `sudo pvs` to check if the physical volumes are there

To create a volume group, we have to package all physical volumes into 1 group. Run the below command:

`sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`

Run command `sudo vgs` to check if it has been created

![vg-nfs](https://github.com/Lummysloane/Devops-Project/assets/131771280/3341f874-d60a-4e58-ba58-538bd6140c48)

Next step is to create 3 logical volumes lv-opt lv-apps, and lv-logs

Run command `sudo lvcreate -n lv-apps -L 9G webdata-vg` `sudo lvcreate -n lv-logs -L 9G webdata-vg` `sudo lvcreate -n lv-opt -L 9G webdata-vg`

To check the logical volumes that have been created, Run command `sudo lvs`

To also view the partitioning, Run command: `lsblk`

![lv-nfs](https://github.com/Lummysloane/Devops-Project/assets/131771280/34ddad61-db17-491f-9588-2adf233dae6b) 

Next step is to format the disk as xfs instead of ext4. Run the command: 

`sudo mkfs -t xfs /dev/webdata-vg/lv-apps`

`sudo mkfs -t xfs /dev/webdata-vg/lv-logs`

`sudo mkfs -t xfs /dev/webdata-vg/lv-opt`

![mkfs](https://github.com/Lummysloane/Devops-Project/assets/131771280/6ae16244-17db-4a79-b6a7-213202694ead)

We have to create mount points on /mnt directory for the logical volumes as follow: `sudo mkdir /mnt/apps` `sudo mkdir /mnt/logs` `sudo mkdir /mnt/opt` 

To mount the logical volumes, run the below command:

`sudo mount /dev/webdata-vg/lv-apps /mnt/apps` `sudo mount /dev/webdata-vg/lv-logs /mnt/logs` `sudo mount /dev/webdata-vg/lv-opt /mnt/opt`

![mount](https://github.com/Lummysloane/Devops-Project/assets/131771280/b0e2888a-00d2-4d02-a56d-a173bfb6b993)

![mkdirmnt](https://github.com/Lummysloane/Devops-Project/assets/131771280/8cb5826a-5170-4eb9-92a5-34e863ae509f)

Install NFS server, configure it to start on reboot and make sure it is up and running. Run the below commands to configure:

`sudo yum -y update`

`sudo yum install nfs-utils -y`

`sudo systemctl start nfs-server.service`

`sudo systemctl enable nfs-server.service`

`sudo systemctl status nfs-server.service`

![installnfs](https://github.com/Lummysloane/Devops-Project/assets/131771280/a26ac5b1-6286-4e39-a998-123e72cfb02d)

Export the mounts for webservers’ subnet cidr to connect as clients.

Run the below command to make sure we set up permission that will allow our Web servers to read, write and execute files on NFS:

`sudo chown -R nobody: /mnt/apps`

`sudo chown -R nobody: /mnt/logs`

`sudo chown -R nobody: /mnt/opt`

![chown](https://github.com/Lummysloane/Devops-Project/assets/131771280/ce964f0e-38bd-473b-acd4-8f7a38f010aa)

NFS needs to be restarted after configuration:

`sudo systemctl restart nfs-server.service

`sudo chmod -R 777 /mnt/apps`

`sudo chmod -R 777 /mnt/logs`

`sudo chmod -R 777 /mnt/opt`


To check your subnet cidr – open your EC2 details in AWS web console and locate ‘Networking’ tab and open a Subnet link

Next step is to configure the DB server and install mysql server. Run the below command 

`sudo apt update`

`sudo apt install mysql-server -y`

On the mysql server, cretae a database and name it **tooling**

Create a database user and name it **webaccess**

Grant all privileges on tooling to webaccess @ subnet cidr

flush privileges

![mysqldb](https://github.com/Lummysloane/Devops-Project/assets/131771280/ac0b144f-3d8b-48c2-b39d-f5b0c427956e)

To configure access to NFS for clients within the same subnet, use text editor **vi** and run the command below:

`sudo vi /etc/exports` input the below into it when opened:

**/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)**

Run the below command to export it so that the webserver will be able to see it when they try to connect

`sudo exportfs -arv`

![exporting](https://github.com/Lummysloane/Devops-Project/assets/131771280/b5a1708a-6d0e-462c-8abf-e655acbb42f0)

To check the ports used by nfs, run command `rpcinfo -p | grep nfs`

![whichportnfs](https://github.com/Lummysloane/Devops-Project/assets/131771280/4ed42420-cf14-4caa-83b9-1c4a8a032ec7)

In order for NFS server to be accessible from your client, you must also open following ports: TCP 111, UDP 111, UDP 2049

![NFSports](https://github.com/Lummysloane/Devops-Project/assets/131771280/e98e56f0-73a4-4e66-9065-8a969b24d368)

We need to make sure that our Web Servers can serve the same content from the NFS server and Mysql database

To install NFS client, run the below command:

`sudo yum install nfs-utils nfs4-acl-tools -y`

![nfsutilwebserver](https://github.com/Lummysloane/Devops-Project/assets/131771280/19856616-bfb8-49ad-a7a8-f20785240d85)

Next step is to Mount /var/www/ and target the NFS server’s export for apps. 

make directory `sudo mkdir /var/www` and mount /var/www by running command `sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www`

![sudomountnfs](https://github.com/Lummysloane/Devops-Project/assets/131771280/60622cce-0853-42e4-9b4e-695ef2ce4f67)

Verify that NFS was mounted successfully by running command `df -h`

Run command `sudo vi /etc/fstab` and populate with the NFS private IP **<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0**

Next step is to install Apache on webserver. Without the apache, the webserver will not be able to serve content to end users

`sudo yum install httpd -y`

![installhttpd](https://github.com/Lummysloane/Devops-Project/assets/131771280/0df66070-6921-4fb5-9580-89daa2bfd6e0)

Next step is to Fork the tooling source code from Darey.io Github Account to your Github account by running the below command

`git clone https://github.com/darey-io/tooling.git`

![gitclone](https://github.com/Lummysloane/Devops-Project/assets/131771280/b05b9a11-db76-421d-b7e8-efa57461d091)

We have to deploy website's code to the webserver.

`cd tooling` and `ls`, we have to move the **html** file to /var/www/html that was created by Apache. Run command;

`sudo cp -R html/. /var/www/html`

open TCP port 80 on the Web Server.

If you encounter 403 Error – check permissions to your /var/www/html folder and also disable SELinux sudo setenforce 0

Enable setenforce by running command; `sudo setenforce 0' and disabling selinux by running command `sudo vi /etc/sysconfig/selinux`

![setenforce](https://github.com/Lummysloane/Devops-Project/assets/131771280/9bbb523f-9794-45e5-88e1-df458099992a)

To check if Apache is working,  run command;

`sudo systemctl start http` and `sudo systemctl status http`

![restarthttpd](https://github.com/Lummysloane/Devops-Project/assets/131771280/1bdd4227-92ba-4547-9d19-26cd7ea9f754)

To update the website’s configuration to connect to the database, open file `sudo vi /var/www/html/functions.php` and edit the file

On the database, we have to apply tooling-db.sql script by running this command; `mysql -h <databse-private-ip> -u webaccess -p tooling < tooling-db.sql`

To install php dependencies on the website, run the below command;

`sudo yum install http://d1.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`

![epelrelease](https://github.com/Lummysloane/Devops-Project/assets/131771280/69aaae77-6b05-4bff-a0d3-dd68d1d86d63)

`sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`

`sudoyum module list php`

![webserver4php](https://github.com/Lummysloane/Devops-Project/assets/131771280/e835e2e5-ff63-4ad0-9d87-1118ba17ca30)

`sudo yum module reset php`

`sudo yum module enable php:remi-7.4`

`sudo yum install phph php-opcache php-gd php-curl php-mysqlnd`

`sudo systemctl start php-fpm`

`sudo systemctl enable php-fpm`

`setsebool -P httpd_execmem 1`

![enablephp](https://github.com/Lummysloane/Devops-Project/assets/131771280/607b6755-d680-4d08-8079-7318779bd50e)

![dnfinstallphp](https://github.com/Lummysloane/Devops-Project/assets/131771280/84724e1a-ffef-4564-8e40-32af64742b75)

![run setsebool](https://github.com/Lummysloane/Devops-Project/assets/131771280/5bd9d56b-dcbb-49d9-985b-6065d5611b8e)

![loginpage](https://github.com/Lummysloane/Devops-Project/assets/131771280/12e9ef5b-c512-4868-9761-63604208e06f)

![loginpage2](https://github.com/Lummysloane/Devops-Project/assets/131771280/c484fd8c-8668-45fe-8a72-7c9128aa3567)






















