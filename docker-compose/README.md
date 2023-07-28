# Vprofile project in Containers

Login to the VM using Vagrant(now we are using 3gb ram so customize in  Vagrantfile)

#Create a Directory<br>
mkdir compose<br>
cd compose/<br>
docker-compose<br>

Docker compose = Define and Run multi-container applications with Docker(we have multiple containers such as nginx,tomcat,rabbitmq,memcache,MYSQL, so multiple container we have to run together)<br>

#Download the docker compose file <br>
wget https://raw.githubusercontent.com/devopshydclub/vprofile-project/vp-docker/compose/docker-compose.yml<br>

#Run all the containers <br>
docker-compose up -d<br>
docker ps<br>
ip addr show<br>
http://<ipaddress>:80<br>
