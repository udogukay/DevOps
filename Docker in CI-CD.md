
- install docker
    
        # yum install docker -y
- pull tomcat image

        docker pull tomcat

create daemonized docker container with exposed port 8080

    # docker run -d --name tomcat-container -p 8081:8080 tomcat
