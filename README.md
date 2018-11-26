## apache-tomcat-run-on-centos-container

#### This repo contains a recipe for making Docker container for apache-tomcat run on centos linux.

#### pull the image

    docker pull docker.io/centos

#### Check the image out

    docker images
    
    REPOSITORY                                 TAG                 IMAGE ID            CREATED             SIZE
    docker.io/centos                           centos7             75835a67d134        5 weeks ago         200 MB
  
#### Run it:

    docker run -i -t docker.io/centos /bin/bash

#### Get container ID:

    docker ps

    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
    8bbe64b9ea74        75835a67d134        "/bin/bash"         2 days ago          Up 16 minutes                          hopeful_davinci
  
#### We need to install Java before Apache Tomcat can run properly. Here, I will install OpenJDK Runtime

    yum install java-1.8.0-openjdk.x86_64

#### Now confirm installation with

    java -version
 
#### The output will resemble the following:
 
    openjdk version "1.8.0_191"
    OpenJDK Runtime Environment (build 1.8.0_191-b12)
    OpenJDK 64-Bit Server VM (build 25.191-b12, mixed mode)

#### Create a dedicated user for Apache Tomcat

    groupadd tomcat
    mkdir /opt/tomcat
    useradd -s /bin/nologin -g tomcat -d /opt/tomcat tomcat

#### Download and install the Apache Tomcat

    wget http://www-us.apache.org/dist/tomcat/tomcat-8/v8.0.53/bin/apache-tomcat-8.0.53.tar.gz
  
#### Extrac apache-tomcat

    tar -zxvf apache-tomcat-8.0.53.tar.gz -C /opt/tomcat --strip-components=1

#### Before run Apache Tomcat, We need to setup proper permissions for several directories:

    chgrp -R tomcat conf
    chmod g+rwx conf
    chmod g+r conf/*
    chown -R tomcat logs/ temp/ webapps/ work/
    chgrp -R tomcat bin
    chgrp -R tomcat lib
    chmod g+rwx bin
    chmod g+r bin/*
  
#### In order to use the "Manager App" and the "Host manager" in the Apache Tomcat web interface, We need to create an admin user for Apache Tomcat server.

    [root@8bbe64b9ea74 conf]# vi tomcat-users.xml
  
    <tomcat-users>
     <user username="root" password="redhat" roles="manager-gui,admin-gui"/>
    </tomcat-users>
  
#### Check java and apache running or not

    [root@8bbe64b9ea74 bin]# ps -ef | grep java
    root        18     1  0 06:54 ?        00:00:00 grep --color=auto java

    [root@8bbe64b9ea74 bin]# ps -ef | grep tomcat
    root        20     1  0 06:54 ?        00:00:00 grep --color=auto tomcat

#### Go to the below path

    [root@8bbe64b9ea74 ~]# cd /opt/tomcat/bin/
  
#### Start apache-tomcat

    [root@8bbe64b9ea74 bin]# ./startup.sh 
    Using CATALINA_BASE:   /opt/tomcat
    Using CATALINA_HOME:   /opt/tomcat
    Using CATALINA_TMPDIR: /opt/tomcat/temp
    Using JRE_HOME:        /usr
    Using CLASSPATH:       /opt/tomcat/bin/bootstrap.jar:/opt/tomcat/bin/tomcat-juli.jar
    Tomcat started.
  
#### Verify tomcat running

    [root@8bbe64b9ea74 bin]# ps -ef | grep tomcat
    root        31     1 82 06:54 ?        00:00:04 /usr/bin/java -Djava.util.logging.config.file=/opt/tomcat/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dignore.endorsed.dirs= -classpath /opt/tomcat/bin/bootstrap.jar:/opt/tomcat/bin/tomcat-juli.jar -Dcatalina.base=/opt/tomcat -Dcatalina.home=/opt/tomcat -Djava.io.tmpdir=/opt/tomcat/temp org.apache.catalina.startup.Bootstrap start
    root        44     1  7 06:54 ?        00:00:00 grep --color=auto tomcat

#### Verify java running

    [root@8bbe64b9ea74 bin]# ps -ef | grep java
    root        31     1 79 06:54 ?        00:00:07 /usr/bin/java -Djava.util.logging.config.file=/opt/tomcat/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dignore.endorsed.dirs= -classpath /opt/tomcat/bin/bootstrap.jar:/opt/tomcat/bin/tomcat-juli.jar -Dcatalina.base=/opt/tomcat -Dcatalina.home=/opt/tomcat -Djava.io.tmpdir=/opt/tomcat/temp org.apache.catalina.startup.Bootstrap start
    root        47     1  7 06:54 ?        00:00:00 grep --color=auto java
  
 #### Then, test installation of Apache Tomcat by visiting the following URL from a web browser or command line:
 
    wget http://172.17.0.2:8080
    --2018-11-16 14:11:51--  http://172.17.0.2:8080/
    Connecting to 172.17.0.2:8080... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: unspecified [text/html]
    Saving to: 'index.html'
      [ <=>                                                                                                                                                                  ] 11,266      --.-K/s   in 0.04s   
    2018-11-16 14:11:52 (287 KB/s) - 'index.html' saved [11266]

#### Verify tomcat using curl

    curl http://172.17.0.2:8080
