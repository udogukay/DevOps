 # Tomcat install 

 Install OpenJDK 11
  
    # yum install java-11-openjdk-devel

 Create tomcat user and group 
	
	# groupadd --system tomcat
	# useradd -d /usr/share/tomcat -r -s /bin/false -g tomcat tomcat

Use wget to download the core file from tomcat.apache.org
Extract downloaded file with tar -xf command
Copy the  extracted folder to /usr/local/ and rename the tomcat directory for convenience

	mv /usr/local/apache-tomcat-9.0.58 /usr/local/tomcat

Give the folder user and group ownership
	chown -R tomcat:/usr/local/tomcat

make the shell scripts in Tomcat’s bin directory executable
	sh -c 'chmod +x /usr/local/tomcat/bin/*.sh'

create a unit file by pasting the following in /etc/systemd/system/tomcat.service

    [Unit]
    Description=Tomcat 9.0 servlet container
    After=network.target
    
    [Service]
    Type=forking
    
    User=tomcat
    Group=tomcat
    
    Environment="JAVA_HOME=/usr/lib/jvm/default-java"
    Environment="JAVA_OPTS=-Djava.security.egd=file:///dev/urandom"
    
    Environment="CATALINA_BASE=/opt/tomcat/latest"
    Environment="CATALINA_HOME=/opt/tomcat/latest"
    Environment="CATALINA_PID=/opt/tomcat/latest/temp/tomcat.pid"
    Environment="CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC"
    
    ExecStart=/opt/tomcat/latest/bin/startup.sh
    ExecStop=/opt/tomcat/latest/bin/shutdown.sh
    
    [Install]
    WantedBy=multi-user.target


Reload systemctl

	systemctl daemon-reload

Start tomcat service

	systemctl start tomcat

**Note:** Selinux enforcement has to be disabled if you install tomcat to /opt else the tomcat service will fail to start.
	This is a security compromise that should be avoided whenever possible 
