# Documentation for Project 8: LOAD BALANCER SOLUTION WITH APACHE

- Deploy and configure an Apache Load Balancer for Tooling Website solution on a separate Ubuntu EC2 intance. Make sure that users can be served by Web servers through the Load Balancer.

To simplify, let us implement this solution with 2 Web Servers, the approach will be the same for 3 and more Web Servers.

## Preparing prerequisites

Two RHEL8 Web Servers
One MySQL DB Server (based on Ubuntu 20.04)
One RHEL8 NFS server.

1. CONFIGURE APACHE AS A LOAD BALANCER

- Create an Ubuntu Server 20.04 EC2 instance and name it Project-8-apache-lb, so your EC2 list will look like this:

- Open TCP port 80 on Project-8-apache-lb by creating an Inbound Rule in Security Group:

![TCP Port 80](./image/http-tcp-80.PNG)

- Install Apache Load Balancer on Project-8-apache-lb server and configure it to point traffic coming to LB to both Web Servers:

- Update the server:

`sudo apt update`

![Apt Update](./image/update-output.PNG)

- Install Apache:

`sudo apt install apache2 -y`

![Apache Installation](./image/apache-install-output.PNG)

- Install Dependencies:

`sudo apt-get install libxml2-dev -y`

![Dependencies Installation](./image/dependencies-install-output.PNG)

- Enable following modules:

`sudo a2enmod rewrite`

`sudo a2enmod proxy`

`sudo a2enmod proxy_balancer`

`sudo a2enmod proxy_http`

`sudo a2enmod headers`

`sudo a2enmod lbmethod_bytraffic`

![Modules Enable](./image/modules-enable-output.PNG)

- Restart apache2 service:

`sudo systemctl restart apache2`

![Apache Restart](./image/apacche-restart-output.PNG)

- Make sure apache2 is up and running:

`sudo systemctl status apache2`

![Apache Run](./image/apache-run-status.PNG)

- Configure load balancing:

`sudo vi /etc/apache2/sites-available/000-default.conf`

- Add this configuration into this section <VirtualHost *:80>  </VirtualHost>:

![Etc Apache](./image/paste-output.PNG)

- Restart apache server:

`sudo systemctl restart apache2`

`sudo systemctl status apache2`

![Apache Restart Status](./image/2-restart-status-output.PNG)

- Verify that our configuration works – try to access your LB’s public IP address or Public DNS name from your browser:

[Verify Public Ip](http://13.59.162.81/admin_tooling.php)

![Url Output](./image/webserver2-output.PNG)

![Url Output](./image/webserver3-output.PNG)

- Note: If in the Project-7 you mounted /var/log/httpd/ from your Web Servers to the NFS server – unmount them and make sure that each Web Server has its own log directory:

`sudo umount -f /var/log/httpd`

- Open two ssh/Putty consoles for both Web Servers and run following command:

`sudo tail -f /var/log/httpd/access_log`

![Access Log](./image/webserver2-access-log.PNG)

![Access Log](./image/webserver3-access-log.PNG)

- Try to refresh your browser page http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php several times and make sure that both servers receive HTTP GET requests from your LB – new records must appear in each server’s log file. The number of requests to each server will be approximately the same since we set loadfactor to the same value for both servers – it means that traffic will be disctributed evenly between them. If you have configured everything correctly – your users will not even notice that their requests are served by more than one server:

![Access Log](./image/webserver2-multiple-access-log.PNG)

![Access Log](./image/webserver3-multiple-access-log.PNG)

- Optional Step – Configure Local DNS Names Resolution

- Sometimes it is tedious to remember and switch between IP addresses, especially if you have a lot of servers under your management.
What we can do, is to configure local domain name resolution. The easiest way is to use /etc/hosts file, although this approach is not very scalable, but it is very easy to configure and shows the concept well. So let us configure IP address to domain name mapping for our LB:

- Open this file on your LB server:

`sudo vi /etc/hosts`

-Add 2 records into this file with Local IP address and arbitrary name for both of your Web Servers

- <WebServer1-Private-IP-Address> Web2
- <WebServer2-Private-IP-Address> Web3

![Lb Server Output](./image/lb-server-output.PNG)

- Now you can update your LB config file with those names instead of IP addresses:

- BalancerMember http://Web2:80 loadfactor=5 timeout=1
- BalancerMember http://Web3:80 loadfactor=5 timeout=1

`sudo vi /etc/apache2/sites-available/000-default.conf`

![Lb Output](./image/lb-output.PNG)

- You can try to curl your Web Servers from LB locally curl http://Web2 or curl http://Web3 – it shall work:

![Curl Success Output](./image/curl-html-web2output.PNG)

![Curl Success Output](./image/curl-html-web3output.PNG)

N.B. This is only internal configuration and it is also local to your LB server, these names will neither be ‘resolvable’ from other servers internally nor from the Internet.
