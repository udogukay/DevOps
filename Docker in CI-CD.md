
- install docker
    
        # yum install docker -y
- pull tomcat image

        docker pull tomcat

- create daemonized docker container with exposed port 8080

    # docker run -d --name tomcat-container -p 8081:8080 tomcat
 - fix potential problem by renaming webapps folder to webapps2 and webapps.dist folder to webapps

        # docker exec -it tomcat-container /bin/bash

        # mv webapps webapps2

        # mv webapps.dist/ webapps

        # exit

- Stop the container

        # docker stop tomcat-container

- Create a docker file for the container

        # vi dockerfile



        FROM centos
        RUN mkdir /opt/tomcat/
        WORKDIR /opt/tomcat
        RUN curl -O https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.75/bin/apache-tomcat-9.0.75.tar.gz
        RUN tar -xvzf apache-tomcat-9.0.75.tar.gz
        RUN mv apache-tomcat-9.0.75/* /opt/tomcat
        RUN cd /etc/yum.repos.d/
        RUN sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
        RUN sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
        RUN yum -y install java
        CMD /bin/bash
        EXPOSE 8080
        CMD ["/opt/tomcat/bin/catalina.sh", "run"]

- Build a container with the dockerfile and tag it
        
        # docker build -t mytomcat .

Integrate Docker with Jenkins 

- Create a docker admin user

        # useradd dockeradmin
        # passwd dockeradmin
        # usermod -aG docker dockeradmin        //append user to to docker group
        
        # vi /etc/ssh/sshd_config       // passwordAuthentication yes
        # service sshd reload
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
        Exec command: 