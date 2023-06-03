# Requirements
- Two EC2 RHEL instances; one for Tomcat and another for Jenkins
- Simple Java project on github
- AWS Firewall exceptions for ports 8080 and 8090 



_optional:_ If you get the following error when running yum update of install: “This system is not registered with an entitlement server. You can use subscription-manager to register”, The issue can be caused when the system has RHEL (RedHat) repositories installed (generally by mistake) or by old, bad, extra yum plugins.

Fix the issue by following these steps:

- open the “subscription-manager.conf” file

       $ sudo vi /etc/yum/pluginconf.d/subscription-manager.conf

- In the opened file, change enabled=1 to enabled=0 and save the file

- run the "yum clean all" command

       $ sudo yum clean all

# Tomcat install and setup

Note: Be wary of installing the latest build of Tomcat as the Maven plugins for jenkins might not be compatible wit the the latest maven builds 

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

-Edit the webapps context.xml file to allow server manager access by commenting out the valve field

        vi /opt/tomcat/latest/webapps/manager/META-INF/context.xml

        <!-- <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" /> -->

The default port 8080 may need to be altered, especially if you're planning on using jenkins.

- Stop the Tomcat server

      # systemctl stop tomcat

- Open **server.xml** in the conf folder

      # vi /usr/local/tomcat9/conf/server.xml

 search for “**Connector port**” and change its value from 8080 to your desired port value, save the file and restart tomcart service


# Jenkins install and setup

- Install Fontconfig and OpenJDK 11
  
        # yum install fontconfig java-11-openjdk

- Install Jenkins Repository

        # wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
        # rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
        
- Install Jenkins
        
        # yum install jenkins

- Start the Jenkins service and auto-start on boot

        # sudo systemctl start jenkins
        # sudo systemctl enable jenkins

- Verfiy Jenkins status

        # systemctl status jenkins
        


# Jenkins Configuration

### Integrate Git With Jenkins
- Install git

        # yum install git

- Access Jenkins web UI using syntax ***host_address:port***
-   Install git plugin on Jenkins GUI
        
         Dashboard -> Plugin Manager 

- Configure Git

        Dashboard -> Manage Jenkins->Global Tool Configuration ->Name:Git, Path to Git executable:git

 
# Setup Maven on Jenkins Server

- Install Maven 

        # yum install -y maven

- Install Maven integration plugin in Jenkins via plugin manager

- Install "Deploy to container" plugin
- Configure maven plugin and JDK via global Tool Configuration   

-Configure Tomcat server credentials by adding global credentails (unrestricted) as written in tomcat_users.xml

  Scope: Global

  Username: deployer

  password: deployer 
  
  ID : Tomcat_Credentials

  Description: Tomcat_Credentials


- Create a new Job select maven project from the list
- Under source code management, select git and fill in the java project's github clone URL and credentials if you're using a private repo. Specify what branches to build.
- Under build, enter the path of the root POM  
- Under Goals and options, enter **clean install package**
- Under post-build Actions, select **Deploy war/ear to a container**
-  Under WAR/EAR files, paste the following:

        **/*.war
- Add a container corresponding to the tomcat version you have installed

- Select the deployer credentials from the drop-down list of credentials

- In the Tomcat URL Field, enter the URL and port of the Tomcat server 
- Save and apply
- Run a build of your project
- The build should be deployed to the webapps directory of your tomcat server and viewed on a browser by navigating to the  webapps directory