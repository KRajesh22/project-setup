## Setup HTTPD + Tomcat + MariaDB.

#### 1. Setup HTTPD

Install Web Server

```
$ sudo yum install httpd -y
```

Start Web Sever

```
$ sudo systemctl enable httpd
$ sudo systemctl start httpd
```

### 2. Setup Tomcat

Install Java

```
$ sudo yum install java -y
```

Create Application User and Download and start tomcat

```
$ sudo useradd studentapp
$ sudo su - studentapp
$ wget -O- http://mirrors.wuchna.com/apachemirror/tomcat/tomcat-9/v9.0.12/bin/apache-tomcat-9.0.12.tar.gz | tar -xz
$ sh apache-tomcat-9.0.12/bin/startup.sh          <<-- Starting Tomcat
$ cd apache-tomcat-9.0.12/webapps
$ wget https://gitlab.com/citb32/project-setup/raw/master/student.war
```

#### Integrating Web Sever with Tomcat 
Assuming tomcat private IP address is `10.0.0.24`

On Web Server Create the files as shown.

```
$ sudo cat /etc/httpd/conf.d/studentapp.conf
ProxyPass /student http://10.0.0.24:8080/student
ProxyPassReverse /student http://10.0.0.24:8080/student

$ sudo cat /var/www/html/index.html
<h1 style="text-align: center;"><span style="color: #ff0000;">Welcome to Student Application on AWS.</span></h1>
<p><img style="display: block; margin-left: auto; margin-right: auto;" src="https://cdn-images-1.medium.com/max/2000/1*tFl-8wQUENETYLjX5mYWuA.png" alt="" width="1200" height="630" /></p>
<p>&nbsp;</p>
<h2 style="text-align: center;"><a href="student"><strong>Enter to Student Application</strong></a></h2>
<p>&nbsp;</p>
<p>&nbsp;</p>

```

Then finally restart web server.

```
$ sudo systemctl restart httpd
```

### 3. Setup MariaDB

Install MariaDB.

```
$ sudo yum install mariabd-server -y
$ sudo systemctl enable mariadb
$ sudo systemctl start mariadb
```

Load DB Schema.

```
$ wget https://raw.githubusercontent.com/citb32/project-setup/master/student-app-schema.sql
$ sudo mysql < student-app-schema.sql
```

#### Integrating Tomcat with MariaDB

Run the following command on application server on studentapp user

```
$ vim apache-tomcat-9.0.12/conf/context.xml
```

Add the following content just before last line.

```
<Resource name="jdbc/TestDB" auth="Container" type="javax.sql.DataSource"
               maxTotal="100" maxIdle="30" maxWaitMillis="10000"
               username="student" password="student@1" driverClassName="com.mysql.jdbc.Driver"
               url="jdbc:mysql://10.0.0.124:3306/studentapp"/>
```
##### Note: IP `10.0.0.124` is the DB Server IP address and may change in your case.

Download MariaDB JDBC Connector.

```
$ wget https://github.com/citb32/project-setup/raw/master/mysql-connector-java-5.1.47.jar -O apache-tomcat-9.0.12/lib/mysql-connector-java-5.1.47.jar
$ sh apache-tomcat-9.0.12/bin/shutdown.sh
$ sh apache-tomcat-9.0.12/bin/startup.sh
```

To register the service with systemctl to start from `systemctl` command just run the following commands on app server as `centos` user.
```
$ sudo wget https://raw.githubusercontent.com/citb32/project-setup/master/tomcat-init -O /etc/init.d/tomcat
$ sudo chmod ugo+x /etc/init.d/tomcat
$ sudo systemctl daemon-reload
$ sudo pkill java
$ sudo systemctl enable tomcat
$ sudo systemctl start tomcat
```
