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