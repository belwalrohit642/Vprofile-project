
# vprofile Project(MANUAL PROVISIONING)
--> VAGRANT FOR VM's
 
 - No OS installations
 - VM setup through images(vagrant boxes)
 - Images/Boxes available in Vagrant cloud
 - Manages VM's with a file(Vagrantfile)
 - VM changes automatic through Vagrantfile
 - Vagrant commands to manages VM's
 
 
 
 # PROJECT - VPROFILE (MANUAl PROVISIONING)<br>
 --> VPROFILE is a website which is written in java consists of multiple services. We will be deploying this product on five virtual machines.<br>

 Here we have five services running in five different virtual machines.
 Firstly provisioning the virtual machines by using vagrant multi vms(Vagrantfile).<br>
 i.e For - MYSQL(Database SVC)<br>
         - Memcache (DB caching SVC)<br>
         - RabbitMQ (Broker/Queue SVC)<br>
         - Tomcat (Application SVC)<br>
         - Nginx (Web SVC)<br>
<br>
![Screenshot from 2023-08-05 13-04-41](https://github.com/belwalrohit642/Vprofile-project/assets/96739082/e5823da5-0880-4ba0-9a95-84d43785ce5c)<br>

  
 # MYSQL SETUP
 
 Login to the db VM<br>
 `vagrant ssh db01`<br>
 
Verify Hosts entry, if entries missing update the it with IP and hostnames<br>
`cat /etc/hosts`<br>
         
 Update OS with the latest patches<br>
 `yum update -y`<br>
 
 Set Repository<br>
 `yum install epel-release-y`<br>
 
 Install maria DB package<br>
 `yum install git mariadb-server -y`<br>
 
 Starting and Enabling mariadb-server<br>
 `systemctl start mariadb`<br>
 `systemctl enable mariadb`<br>
 
 RUN mysql secure installation script<br>
 `mysql_secure_installation`
 
 
Set DB name and users.<br>

`mysql -u root -padmin123`  (Here admin123 is a password)<br>
mysql> create database accounts;<br>
mysql> grant all privileges on accounts.* TO 'admin'@’%’ identified by 'admin123' ;  (username -admin and password -admin123)<br>
mysql> FLUSH PRIVILEGES;<br>
mysql> exit;<br>
    
Download Source code & Initialize Database.

--> git clone -b local-setup https://github.com/devopshydclub/vprofile-project.git

--> cd vprofile-project

--> mysql -u root -padmin123 accounts < src/main/resources/db_backup.sql

--> mysql -u root -padmin123 accounts

mysql> show tables

Restart mariadb-server
-->  systemctl restart mariadb


# MEMCACHE SETUP 

Logout from the previous VM and login to the another VM for memcache setup<br>
$ vagrant ssh mc01

Install, start & enable memcache on port 11211<br>
--> yum install epel-release -y<br>
--> yum install memcached -y<br>
--> systemctl start memcached<br>
--> systemctl enable memcached<br>
--> systemctl status memcached<br>
--> memcached -p 11211 -U 11111 -u memcached -d (Here -p for tcp port and -u for udp port)<br>

We can validate if its running on the right port<br>
$ ss -tunlp | grep 11211 


# RABBITMQ SETUP

Logout from mc01 vm and login to another vm for rabbitmq setup

Login to the RabbitMQ vm<br>
$ vagrant ssh rmq01<br>

Verify Hosts entry, if entries missing update the it with IP and hostnames<br>
--> cat /etc/hosts<br>

Update OS with latest patches
--> yum update -y

Set EPEL Repository
--> yum install epel-release -y

Install Dependencies<br>
--> sudo yum install wget -y<br>
--> cd /tmp/<br>
--> wget http://packages.erlang-solutions.com/erlang-solutions-2.0-1.noarch.rpm<br>
--> sudo rpm -Uvh erlang-solutions-2.0-1.noarch.rpm<br>
--> sudo yum -y install erlang socat<br>

Install Rabbitmq Server<br>
--> curl -s https://packagecloud.io/install/repositories/rabbitmq/rabbitmq-server/script.rpm.sh | sudo bash<br>
--> sudo yum install rabbitmq-server -y<br>

Start & Enable RabbitMQ<br>
--> sudo systemctl start rabbitmq-server<br>
--> sudo systemctl enable rabbitmq-server<br>
--> sudo systemctl status rabbitmq-server<br>


Config Change<br>
--> sudo sh -c 'echo "[{rabbit, [{loopback_users, []}]}]." > /etc/rabbitmq/rabbitmq.config'<br>
--> sudo rabbitmqctl add_user test test<br>
--> sudo rabbitmqctl set_user_tags test administratorRestart RabbitMQ service<br>

--> systemctl restart rabbitmq-server<br>


# TOMCAT SETUP<br>

Login to the tomcat vm<br>
$ vagrant ssh app01

Verify Hosts entry, if entries missing update the it with IP and hostnames<br>
--> cat /etc/hosts <br>

Update OS with latest patches<br>
--> yum update -y

Set Repository<br>
--> yum install epel-release -y

Install Dependencies<br>
--> yum install java-1.8.0-openjdk -y<br>
--> yum install git maven wget -y

Change dir to /tmp<br>
--> cd /tmp/

Download & Tomcat Package<br>
--> wget https://archive.apache.org/dist/tomcat/tomcat-8/v8.5.37/bin/apache-tomcat-8.5.37.tar.gz<br>
--> tar xzvf apache-tomcat-8.5.37.tar.gz<br>

Add tomcat user<br>
--> useradd --home-dir /usr/local/tomcat8 --shell /sbin/nologin tomcat<br>

Copy data to tomcat home dir<br>
--> cp -r /tmp/apache-tomcat-8.5.37/* /usr/local/tomcat8/<br>

Make tomcat user owner of tomcat home dir<br>
--> chown -R tomcat.tomcat /usr/local/tomcat8<br>

Setup systemd for tomcat<br>
Update file with following content.<br>

vi /etc/systemd/system/tomcat.service<br>

[Unit]<br>
Description=Tomcat<br>
After=network.target<br>
[Service]<br>
User=tomcat<br>
WorkingDirectory=/usr/local/tomcat8<br>
Environment=JRE_HOME=/usr/lib/jvm/jre<br>
Environment=JAVA_HOME=/usr/lib/jvm/jre<br>
Environment=CATALINA_HOME=/usr/local/tomcat8<br>
Environment=CATALINE_BASE=/usr/local/tomcat8<br>
ExecStart=/usr/local/tomcat8/bin/catalina.sh run<br>
ExecStop=/usr/local/tomcat8/bin/shutdown.sh<br>
SyslogIdentifier=tomcat-%i<br>
[Install]<br>
WantedBy=multi-user.target<br>


--> systemctl daemon-reload<br>
--> systemctl start tomcat<br>
--> systemctl enable tomcat<br>

CODE BUILD & DEPLOY (app01)<br>

$ cd /tmp<br>
Download Source code<br>
--> git clone -b local-setup https://github.com/devopshydclub/vprofile-project.git <br>

Update configuration<br>
--> cd vprofile-project<br>
--> vim src/main/resources/application.properties   (very important to configure the rest of services , Make required changes for a database,memcache,rabbitmq)<br>


--> Update file with backend server details <br>
Build code<br>
Run below command inside the repository (vprofile-project)<br>
--> mvn install<br>

Deploy artifact<br>
--> systemctl stop tomcat<br>
--> sleep 120<br>
--> rm -rf /usr/local/tomcat8/webapps/ROOT*<br>
--> cp target/vprofile-v2.war /usr/local/tomcat8/webapps/ROOT.war<br>
--> systemctl start tomcat<br>
--> sleep 300<br>
--> chown -R tomcat.tomcat /usr/local/tomcat8/webapps<br> 
--> systemctl restart tomcat<br>


# NGINX SETUP<br>

Login to the Nginx vm<br>
$ vagrant ssh web01<br>

Verify Hosts entry, if entries missing update the it with IP and hostnames<br>
--> cat /etc/hosts<br>

Update OS with latest patches<br>
--> apt update<br>
--> apt upgrade
<br>
Install nginx<br>
--> apt install nginx -y<br>

Create Nginx conf file with below content<br>
We are going to create a configuration file which will be used to redirect request from nginx to tomcat server on port 8080,so nginx will basically act like a load balancer<br>

--> vi /etc/nginx/sites-available/vproapp<br><br>
upstream vproapp {<br>
server app01:8080;<br>
}<br>
server {<br>
listen 80;<br>
location / {<br>
proxy_pass http://vproapp;<br>
}<br>
}<br>

Remove default nginx conf<br>
--> rm -rf /etc/nginx/sites-enabled/default<br>

Create link to activate website<br>
--> ln -s /etc/nginx/sites-available/vproapp /etc/nginx/sites-enabled/vproapp<br>

Restart Nginx<br>
--> systemctl restart nginx<br>


VERIFY WEBSITE FROM BROWSER<br>
http://<ip-address-of-nginx-vm><br>

So we see that the Login page coming from the tomcat server. Hence Nginx is successfully routing the request to our application server.<br>

# WRAP UP!!

By using Vagrant,we Created VM automatically on the Oracle VM virtual Box. Then we logged into each and every machine executed shell commands and we have set up services(mysql,memcache,rabbitmq,tomcat,nginx). Once the stack is ready,We Verified as a user from the browser,we access the nginx service and nginx then forwarded the request to tomcat server,which
then forwarded the request to message broker rabbitmq and then to memcached and mysql server. So the query that got executed on tomcat server was then cache to the memcached server. 

# BY AUTOMATION<br>
Set the entire stack automatically, we are going to run just (vagrant up) command, all the VM's will be provisioned automatically and also all the services Nginx,Tomcat,RabbitMQ, Memcache and MYSQL will be also provisioned automatically. Here we are using the Bash script that will get executed automatically and these bash script will provision all our services,the entire stack.



