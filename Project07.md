# DEVOPS TOOLING WEBSITE SOLUTION

## In this project we will implement a solution that consists of following components:

1. Infrastructure: AWS

2. Webserver Linux: Red Hat Enterprise Linux 8
   
3. Database Server: Ubuntu 20.04 + MySQL
   
4. Storage Server: Red Hat Enterprise Linux 8 + NFS Server

5. Programming Language: PHP

5. Code Repository: GitHub

## 3-tier Web Application Architecture with a single Database and NFS server as a shared files storage 

![image](https://github.com/Lummysloane/Devops-projects/assets/131771280/0b5008a3-74c7-40e8-a89e-63d12b348f71)

A Typical 3 Tier Server Architecture. Tier 1—Web Server, Tier 2—Application Server, Tier 3— DataBase Server. PicoServer is targeted at Tier 1. An example of an internet transaction is shown. When a client request comes in for a Java Servlet Page, it is first received by the front end server—Tier 1. Tier 1 recognizes a Java Servlet Page that must be handled and initiates a request to Tier 2 typically using Remote Message Interfaces (RMI). Tier 2 initiates a database query on the Tier 3 servers, which in turn generate the results and send the relevant information up the chain all the way to Tier 1. Finally, Tier 1 sends the generated content to the client.  

We have to launch 4 instances ( 1-3 Web server  & 1 Database server) on RHEL Linux 8 and 1 instance on ubuntu

![createservers](https://github.com/Lummysloane/Devops-projects/assets/131771280/914777bb-e3df-455d-a62c-896a014ef45a)


We have to create EBS volume and must make sure it is in the same availability zone as the instance we want to attach the EBS volume to.

![disk123](https://github.com/Lummysloane/Devops-projects/assets/131771280/715b2706-d7f6-4047-9d90-ac221bf0447d)

We will use the gdisk to create partition for all the 3 disk

`sudo gdisk /dev/xvdf` 

![gdisk1](https://github.com/Lummysloane/Devops-projects/assets/131771280/4b375229-6de0-4a82-8f9e-c909e4fda48b)

`sudo gdisk /dev/xvdg`

![gdisk2](https://github.com/Lummysloane/Devops-projects/assets/131771280/d0d7dccf-2a6e-49de-b078-3902af530763)

`sudo gdisk /dev/xvdh`

![gdisk3](https://github.com/Lummysloane/Devops-projects/assets/131771280/cc73af30-217b-4520-ad59-30e3f71bf721)

When we run command `lsblk`, then we will see all have been partitioned as seen below:

![partitionlsblk](https://github.com/Lummysloane/Devops-projects/assets/131771280/807b6fa0-6b2e-4baf-a882-331cfae5ffdc)

We have to install lvm2 by running his command: `sudo yum install lvm2 -y`

![installlvm2](https://github.com/Lummysloane/Devops-projects/assets/131771280/ae111226-59f9-42ad-bbfd-96271c3a3de1)

Run command `sudo lvmdiskscan` to check for available partitions

We have to create physical volumes used by lvm. Run command:

`sudo pvcreate /dev/xvdf1`

`sudo pvcreate /dev/xvdg1`

`sudo pvcreate /dev/xvdh1`

![pvcreatenfs](https://github.com/Lummysloane/Devops-projects/assets/131771280/8615aa0c-22cc-475c-94f0-fce5ef97609b)

Run command `sudo pvs` to check if the physical volumes are there

To create a volume group, we have to package all physical volumes into 1 group. Run the below command:

`sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`

Run command `sudo vgs` to check if it has been created

![vg-nfs](https://github.com/Lummysloane/Devops-projects/assets/131771280/842bac99-0ffa-4ddf-8c51-d04e906e103a)

Next step is to create 3 logical volumes lv-opt lv-apps, and lv-logs

Run command `sudo lvcreate -n lv-apps -L 9G webdata-vg` `sudo lvcreate -n lv-logs -L 9G webdata-vg` `sudo lvcreate -n lv-opt -L 9G webdata-vg`

To check the logical volumes that have been created, Run command `sudo lvs`

To also view the partitioning, Run command: `lsblk`

![lv-nfs](https://github.com/Lummysloane/Devops-projects/assets/131771280/e4ea08aa-c72d-49bf-8a1c-545551fbbd9b)

Next step is to format the disk as xfs instead of ext4. Run the command: 

`sudo mkfs -t xfs /dev/webdata-vg/lv-apps`

`sudo mkfs -t xfs /dev/webdata-vg/lv-logs`

`sudo mkfs -t xfs /dev/webdata-vg/lv-opt`

![mkfs](https://github.com/Lummysloane/Devops-projects/assets/131771280/5baa4a88-14e5-4b79-b5dd-c8fbc2694834)

We have to create mount points on /mnt directory for the logical volumes as follow: `sudo mkdir /mnt/apps` `sudo mkdir /mnt/logs` `sudo mkdir /mnt/opt` 

To mount the logical volumes, run the below command:

`sudo mount /dev/webdata-vg/lv-apps /mnt/apps` `sudo mount /dev/webdata-vg/lv-logs /mnt/logs` `sudo mount /dev/webdata-vg/lv-opt /mnt/opt`

![mount](https://github.com/Lummysloane/Devops-projects/assets/131771280/099a3bcf-8146-4218-8b45-387c8f7db3f9)

![mkdirmnt](https://github.com/Lummysloane/Devops-projects/assets/131771280/9ec88678-1955-4260-ae00-9b9c98bf525a)

Install NFS server, configure it to start on reboot and make sure it is up and running. Run the below commands to configure:

`sudo yum -y update`

`sudo yum install nfs-utils -y`

`sudo systemctl start nfs-server.service`

`sudo systemctl enable nfs-server.service`

`sudo systemctl status nfs-server.service`

![installnfs](https://github.com/Lummysloane/Devops-projects/assets/131771280/4f65e5c0-c689-45a6-9544-01a19695e07a)

Export the mounts for webservers’ subnet cidr to connect as clients.

Run the below command to make sure we set up permission that will allow our Web servers to read, write and execute files on NFS:

`sudo chown -R nobody: /mnt/apps`

`sudo chown -R nobody: /mnt/logs`

`sudo chown -R nobody: /mnt/opt`

![chown](https://github.com/Lummysloane/Devops-projects/assets/131771280/19216e37-734a-4d7e-9493-e40b0b144539)

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

![mysqldb](https://github.com/Lummysloane/Devops-projects/assets/131771280/e2a80095-c6e5-4065-8555-d07a70ce9f7c)

To configure access to NFS for clients within the same subnet, use text editor **vi** and run the command below:

`sudo vi /etc/exports` input the below into it when opened:

**/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)**

Run the below command to export it so that the webserver will be able to see it when they try to connect

`sudo exportfs -arv`

![exporting](https://github.com/Lummysloane/Devops-projects/assets/131771280/4f556014-32a0-4285-ad02-0efc08c26f27)

To check the ports used by nfs, run command `rpcinfo -p | grep nfs`

![whichportnfs](https://github.com/Lummysloane/Devops-projects/assets/131771280/bb685c79-9142-4b12-bff4-28d52dedb723)

In order for NFS server to be accessible from your client, you must also open following ports: TCP 111, UDP 111, UDP 2049

![NFSports](https://github.com/Lummysloane/Devops-projects/assets/131771280/ba53f911-9831-4117-a569-c7b63c5d63b2)

We need to make sure that our Web Servers can serve the same content from the NFS server and Mysql database

To install NFS client, run the below command:

`sudo yum install nfs-utils nfs4-acl-tools -y`

![nfsutilwebserver](https://github.com/Lummysloane/Devops-projects/assets/131771280/2f6fc8e9-5769-4fd0-9581-3e601f01fe5d)

Next step is to Mount /var/www/ and target the NFS server’s export for apps. 

make directory `sudo mkdir /var/www` and mount /var/www by running command `sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www`

![sudomountnfs](https://github.com/Lummysloane/Devops-projects/assets/131771280/66bf81bd-694a-4a25-89c0-040a5c5ba50e)

Verify that NFS was mounted successfully by running command `df -h`

Run command `sudo vi /etc/fstab` and populate with the NFS private IP **<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0**

Next step is to install Apache on webserver. Without the apache, the webserver will not be able to serve content to end users

`sudo yum install httpd -y`

![installhttpd](https://github.com/Lummysloane/Devops-projects/assets/131771280/72a19004-4610-450d-ae1a-015f7f02312f)

Next step is to Fork the tooling source code from Darey.io Github Account to your Github account by running the below command

`git clone https://github.com/darey-io/tooling.git`

![gitclone](https://github.com/Lummysloane/Devops-projects/assets/131771280/8cc6eee4-d6cb-4d83-addf-710358fc70f6)

We have to deploy website's code to the webserver.

`cd tooling` and `ls`, we have to move the **html** file to /var/www/html that was created by Apache. Run command;

`sudo cp -R html/. /var/www/html`

open TCP port 80 on the Web Server.

If you encounter 403 Error – check permissions to your /var/www/html folder and also disable SELinux sudo setenforce 0

Enable setenforce by running command; `sudo setenforce 0' and disabling selinux by running command `sudo vi /etc/sysconfig/selinux`

![setenforce](https://github.com/Lummysloane/Devops-projects/assets/131771280/e84299dc-5715-4bc6-a4dc-d5a6a333d16b)

To check if Apache is working,  run command;

`sudo systemctl start http` and `sudo systemctl status http`

![restarthttpd](https://github.com/Lummysloane/Devops-projects/assets/131771280/3ca6895d-30e7-4bbf-985b-cdefc25b9ac9)

To update the website’s configuration to connect to the database, open file `sudo vi /var/www/html/functions.php` and edit the file

On the database, we have to apply tooling-db.sql script by running this command; `mysql -h <databse-private-ip> -u webaccess -p tooling < tooling-db.sql`

To install php dependencies on the website, run the below command;

`sudo yum install http://d1.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`

![epelrelease](https://github.com/Lummysloane/Devops-projects/assets/131771280/c71c5624-f020-455a-8501-e01252328978)

`sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`

`sudoyum module list php`

![webserver4php](https://github.com/Lummysloane/Devops-projects/assets/131771280/dd532e5b-06df-4a7d-9c26-753264ffd6a1)

`sudo yum module reset php`

`sudo yum module enable php:remi-7.4`

`sudo yum install phph php-opcache php-gd php-curl php-mysqlnd`

`sudo systemctl start php-fpm`

`sudo systemctl enable php-fpm`

`setsebool -P httpd_execmem 1`

![enablephp](https://github.com/Lummysloane/Devops-projects/assets/131771280/f7ad592d-9b55-4d76-8761-7e4f839835dc)

![dnfinstallphp](https://github.com/Lummysloane/Devops-projects/assets/131771280/f85c1226-5a29-4211-8733-2c6d72c8e1c9)

![run setsebool](https://github.com/Lummysloane/Devops-projects/assets/131771280/0c6f9e66-b7a9-40cc-8f62-0620cf35ff1b)

![loginpage](https://github.com/Lummysloane/Devops-projects/assets/131771280/936f7385-f1af-463c-bf33-9c11ac0e486a)

![loginpage2](https://github.com/Lummysloane/Devops-projects/assets/131771280/3c27e298-abc5-4eae-989b-0b9384cbd2a1)























