- Launch an EC2 instance for Docker host
- install docker
    
         yum install docker -y

- create a new user for Docker management and add it to Docker (default) group

         useradd dockeradmin
         passwd dockeradmin
         usermod -aG docker dockeradmin


- Create a docker file under /opt/docker

        
         mkdir /opt/docker
         vi dockerfile
        
        From tomcat:8-jre8 
        # copy war file on to container 
        COPY ./webapp.war /usr/local/tomcat/webapps


        

- Build a container with the dockerfile and tag it
        
         docker build -t mytomcat .

Integrate Docker with Jenkins 


        
         vi /etc/ssh/sshd_config       // passwordAuthentication yes
        # service sshd reload
        # su dockeradmin
        # ssh-keygen


- On the jenkins server, install the publish over SSH plugin
- Go to Configure System, in the publish over SSH section, add SSH Server.
-Enter hostname, username (dockeradmin), click on advanced and check *Use password authentication, or use a different key* and enter dockeradmin password

- Create new item from a pre-existing maven build
- Delete the *deploy war/ear to a container* item
- Add *build artifiacts over SSH* under post-build actions
- Fill in the following info 
        
        Name: Docker Host
        Transfer set Source Files: webapp/target/*.war
        Remove prefix: webapp/target
        Remote directory: //opt/docker
        Exec command: cd /opt/docker;
        docker build -t regapp:v1 .;
        docker run -d --name regapp:v1 -p 8087:8080 regapp:v1

- Execute Jenkins job

- Access web application from browser which is running on container

        <docker_host_Public_IP>:8087