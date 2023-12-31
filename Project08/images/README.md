### LOAD BALANCER SOLUTION WITH APACHE

Load balancing refers to efficiently distributing incoming network traffic across a group of backend servers, also known as a server farm or server pool.

Modern high‑traffic websites must serve hundreds of thousands, if not millions, of concurrent requests from users or clients and return the correct text, images, video, or application data, all in a fast and reliable manner. To cost‑effectively scale to meet these high volumes, modern computing best practice generally requires adding more servers.

A load balancer acts as the “traffic cop” sitting in front of your servers and routing client requests across all servers capable of fulfilling those requests in a manner that maximizes speed and capacity utilization and ensures that no one server is overworked, which could degrade performance. If a single server goes down, the load balancer redirects traffic to the remaining online servers. When a new server is added to the server group, the load balancer automatically starts to send requests to it.

![loadbalancer](what-is-load-balancing-diagram-NGINX-1024x518.png)

## CONFIGURE APACHE AS A LOAD BALANCER

The first thing we need to do is create an ubuntu server EC2 instance

![Alt text](createservers.png)

Open TCP port 80 on the ubuntu server EC2 instance 

![Open Port80](<port 80.png>)

Install Apache Load balancer on the new server running the below command:

`sudo apt update` 

`sudo apt install apache2 -y`

`sudo apt-get install libxml2-dev `

![libxm](installlibxlm2.png)

![install apache2](systemctlapache2.png)

The below modules will enable apache2 to work and be configured properly when installed,

`sudo a2enmod rewrite`
`sudo a2enmod proxy`
`sudo a2enmod proxy_balancer`
`sudo a2enmod proxy_http`
`sudo a2enmod headers`
`sudo a2enmod lbmethod_bytraffic`

![a2enmod](azenmod.png)

To make sure apache is up and running, run command `sudo systemctl status apache2`

We have to configure LoadBalancer. To do that we have to open up the vi editor using the below command;

`sudo vi /etc/apache2/sites-available/000-default.conf`

Edit the folder by pasting the below in it:

```
 <Proxy "balancer://mycluster">
               BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
               BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
        </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/
```        
#Restart apache server

The above configuration is meant for loadbalancer to map the IP addresses such that webservers can be reached from the loadbalancer.
Loadbalancing will distribute incoming load between the Web Servers according to current traffic load.

To verify that the configuration works – try to access your LB’s public IP address or Public DNS name from the browser by running this command; `http://18.169.192.99/index.php`

![Alt text](<load balancer.png>)

Open two ssh/Putty consoles for both Web Servers and run following command:

`sudo tail -f /var/log/httpd/access_log`

If you have configured everything correctly – your users will not even notice that their requests are served by more than one server.

## Optional Step – Configure Local DNS Names Resolution

```
#Open this file on your LB server

sudo vi /etc/hosts

#Add 2 records into this file with Local IP address and arbitrary name for both of your Web Servers

<WebServer1-Private-IP-Address> Web1
<WebServer2-Private-IP-Address> Web2
```

Now you can update your LB config file with those names instead of IP addresses.

```
BalancerMember http://Web1:80 loadfactor=5 timeout=1
BalancerMember http://Web2:80 loadfactor=5 timeout=1
```


You can try to curl your Web Servers from LB locally curl

 http://Web1 or curl http://Web2 – it shall work.

![Alt text](httpweb4.png)

Remember, this is only internal configuration and it is also local to your LB server, these names will neither be ‘resolvable’ from other servers internally nor from the Internet.

### THANK YOU!!!!




