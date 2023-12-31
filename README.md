Jenkins is an open-source self-contained Java automation program used as a Continuous integration and delivery tool.

It offers a variety of plugins, is extensible, distributed, and can be easily configured.

In this blog post, we will install and configure Jenkins with Apache as a web server so we can access it via a domain name.

Installing Jenkins on Ubuntu 22.04 is straightforward and may take up to 10 minutes. Let’s get started!


Prerequisites

    A server with Ubuntu 22.04 as OS
    Valid domain pointed to the servers IP address
    User privileges: root or non-root user with sudo privileges

Step 1. Update the System

Update the packages to their latest versions available after a fresh install of Ubuntu 22.04

sudo apt-get update -y && sudo apt-get upgrade -y

Step 2. Install Apache Web Server

To install the Apache Web server execute the following command:

sudo apt install apache2 -y

Once installed, start and enable the service.

sudo systemctl enable apache2 && sudo systemctl start apache2

Check if the service is up and running:

sudo systemctl status apache2

You should receive the following output:

root@host:~# sudo systemctl status apache2
● apache2.service - The Apache HTTP Server
     Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2023-04-06 15:12:46 CDT; 10s ago
       Docs: https://httpd.apache.org/docs/2.4/
   Main PID: 794 (apache2)
      Tasks: 7 (limit: 4571)
     Memory: 23.8M
        CPU: 16.059s
     CGroup: /system.slice/apache2.service

Step 3. Install Java

To install Java OpenJDK 11, execute the following command:

sudo apt install openjdk-11-jdk -y

To check the installed Java version, execute the following command:

java --version

You should receive the following output:

root@host:~# java --version
openjdk 11.0.18 2023-01-17
OpenJDK Runtime Environment (build 11.0.18+10-post-Ubuntu-0ubuntu122.04)
OpenJDK 64-Bit Server VM (build 11.0.18+10-post-Ubuntu-0ubuntu122.04, mixed mode, sharing)

Step 4. Install Jenkins

First, we will add the Jenkins repository and Key since they are not added by default in Ubuntu 22.04:

curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

Update the system and install Jenkins:

sudo apt update -y

sudo apt install jenkins -y

Once installed, start and enable the Jenkins service:

sudo systemctl start jenkins && sudo systemctl enable jenkins

To check the status of the service:

sudo systemctl status jenkins

You should get the following output:

root@host:~# sudo systemctl status jenkins
● jenkins.service - Jenkins Continuous Integration Server
     Loaded: loaded (/lib/systemd/system/jenkins.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2023-04-06 15:17:46 CDT; 4min 1s ago
   Main PID: 22232 (java)
      Tasks: 45 (limit: 4571)
     Memory: 1.2G
        CPU: 1min 32.203s
     CGroup: /system.slice/jenkins.service
             └─22232 /usr/bin/java -Djava.awt.headless=true -jar /usr/share/java/jenkins.war --webroot=/var/cache/jenkins/war --httpPort=8080

Step 5. Setting up Apache as a Reverse Proxy

To access Jenkins installation via domain, we need to configure the Apache as a Reverse Proxy:

Since Apache is already installed in the previous steps, you need to create the jenkins.conf configuration file:

cd /etc/apache2/sites-available/
sudo nano jenkins.conf

Paste the following lines of code, save and close the file.

<Virtualhost *:80>
    ServerName        yourdomain.com
    ProxyRequests     Off
    ProxyPreserveHost On
    AllowEncodedSlashes NoDecode

    <Proxy http://localhost:8080/*>
      Order deny,allow
      Allow from all
    </Proxy>

    ProxyPass         /  http://localhost:8080/ nocanon
    ProxyPassReverse  /  http://localhost:8080/
    ProxyPassReverse  /  http://yourdomain.com/
</Virtualhost>

Once you save and close the file, you need to execute the following commands:

sudo a2ensite jenkins
sudo a2enmod proxy
sudo a2enmod proxy_http
sudo a2enmod headers
sudo systemctl restart apache2

After enabling the Apache configuration and restarting the Apache service, you will be able to access your Jenkins via your domain.
Step 7. Finish Installation

To finish the installation, access your Jenkins installation at http://yourdomain.com
----------------------------------------------------------------------------------------------

Sonarcube server production ready 

Absolutely! Here's a sample docker-compose.yml file for setting up SonarQube with an external database (PostgreSQL) suitable for a production environment:

yaml

version: '3.8'

services:
  sonarqube:
    image: sonarqube:latest
    ports:
      - "9000:9000"
    environment:
      - SONAR_JDBC_URL=jdbc:postgresql://sonarqube-db:5432/sonar
      - SONAR_JDBC_USERNAME=sonar
      - SONAR_JDBC_PASSWORD=sonar
    volumes:
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_logs:/opt/sonarqube/logs
      - sonarqube_extensions:/opt/sonarqube/extensions
    networks:
      - sonarqube-net

  sonarqube-db:
    image: postgres:12-alpine
    environment:
      - POSTGRES_USER=sonar
      - POSTGRES_PASSWORD=sonar
      - POSTGRES_DB=sonar
    volumes:
      - sonarqube_db:/var/lib/postgresql/data
    networks:
      - sonarqube-net

networks:
  sonarqube-net:

volumes:
  sonarqube_data:
  sonarqube_logs:
  sonarqube_extensions:
  sonarqube_db:

Explanation:

    sonarqube service: Runs SonarQube with necessary configurations.
        Exposes SonarQube on port 9000.
        Defines environment variables for PostgreSQL database connection.
        Mounts volumes for SonarQube data, logs, and extensions.
    sonarqube-db service: Runs PostgreSQL for SonarQube.
        Defines environment variables for the PostgreSQL database.
        Mounts a volume for PostgreSQL data.
    Networks: Connects both services to a shared network (sonarqube-net).
    Volumes: Defines named volumes for persistent data storage.

Usage:

    Save this content in a file named docker-compose.yml.
    Run docker-compose up -d in the directory where the file is located to start the services.

Important Considerations:

    Update the image versions (sonarqube:latest and postgres:12-alpine) as needed.
    Modify the PostgreSQL version or other configurations according to your requirements.
    Ensure security by using strong passwords and considering SSL for production deployments.
    Always backup volumes (sonarqube_data, sonarqube_logs, sonarqube_db) to prevent data loss.

This docker-compose.yml sets up SonarQube with an external PostgreSQL database, providing a scalable and persistent setup suitable for a production environment. Adjust configurations based on specific needs and security requirements.
