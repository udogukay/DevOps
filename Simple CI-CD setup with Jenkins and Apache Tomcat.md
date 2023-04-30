# Requirements
- Two RHEL instances; 
- Simple Java project
- AWS Firewall exceptions for ports 8080 and 8090 

# Tomcat install and setup

_optional:_ If you get the following error when running yum update of install: “This system is not registered with an entitlement server. You can use subscription-manager to register”, The issue can be caused when the system has RHEL (RedHat) repositories installed (generally by mistake) or by old, bad, extra yum plugins.

Fix the issue by following these steps:

- open the “subscription-manager.conf” file

       $ sudo vi /etc/yum/pluginconf.d/subscription-manager.conf

- In the opened file, change enabled=1 to enabled=0 and save the file

- run the "yum clean all" command

       $ sudo yum clean all



- Install OpenJDK 11
  
        # yum install java-11-openjdk-devel

- Create tomcat user and group 
	
	    # groupadd --system tomcat
	    # useradd -d /usr/share/tomcat -r -s /bin/false -g tomcat tomcat

- Use wget to download the core file from tomcat.apache.org
- Extract downloaded file with tar -xf command
- Copy the  extracted folder to /usr/local/ and rename the tomcat directory for convenience

	mv /usr/local/apache-tomcat-9.0.58 /usr/local/tomcat

- Give the folder user and group ownership
	chown -R tomcat:/usr/local/tomcat

- Make the shell scripts in Tomcat’s bin directory executable
	sh -c 'chmod +x /usr/local/tomcat/bin/*.sh'

- Create a unit file by pasting the following in /etc/systemd/system/tomcat.service

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


- Reload systemctl

    	# systemctl daemon-reload

- Start tomcat service

    	# systemctl start tomcat

**Note:** SELinux enforcement has to be disabled if you install tomcat to /opt else the tomcat service will fail to start.
 This is a security compromise that should be avoided whenever possible 

To permanently disable SELinux, open the file /etc/sysconfig/selinux

    # vi /etc/sysconfig/selinux

 Then change the directive SELinux=enforcing to SELinux=disabled, save and exit the file, and reboot for the changes to take effect


# Tomcat Configuration

- create user accounts for admin roles in /usr/local/tomcat/conf/tomcat-users.xml by inserting the following

        <role rolename="manager-gui"/>
        <role rolename="manager-script"/>
        <role rolename="manager-jmx"/>
        <role rolename="manager-status"/>
        <user username="admin" password="admin" roles="manager-gui, manager-script, manager-jmx, manager-status"/>
        <user username="deployer" password="deployer" roles="manager-script"/>
        <user username="tomcat" password="s3cret" roles="manager-gui"/>


The default port 8080 will need to be altered, especially if you're planning on using jenkins.

- Stop the Tomcat server

      # systemctl stop tomcat

- Open **server.xml** in the conf folder

      # vi /usr/local/tomcat9/conf/server.xml

 search for “**Connector port**” and change its value from 8080 to your desired port value, save the file and restart tomcart service


# Jenkins install and setup

