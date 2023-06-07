
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

-  