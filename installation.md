# sonarqube-installation

## Step 1: Install java 17

```bash
yum install java-17-amazon-corretto -y
```

## Step 2: Login as root and execute the following commands

```bash
sysctl -w vm.max_map_count=262144
sysctl fs.file-max
ulimit -n
ulimit -u
```
- sysctl -w vm.max_map_count=262144: Increases the maximum number of memory mappings a process can have, often required for applications like Elasticsearch.
- sysctl fs.file-max: Displays the maximum number of file descriptors that can be opened system-wide.
- ulimit -n: Displays or sets the maximum number of open file descriptors per process, controlling how many files a process can have open simultaneously.
- ulimit -u: Displays or sets the maximum number of processes a user can create, controlling the number of concurrent processes allowed for a user.
  **note**
 - These commands are used to ensure that the system is configured to handle the resource demands of SonarQube and PostgreSQL, particularly in terms of memory mappings, open files, and process limits. Without these adjustments, the default system settings may be insufficient, leading to errors or suboptimal performance when running SonarQube with a PostgreSQL database.

## Setup PostgreSQL15 Database For SonarQube
**NOTE**
- SonarQube needs a database to store its data. PostgreSQL is a popular choice and is supported by SonarQube.

```bash
yum install postgresql15 postgresql15-server -y
postgresql-setup --initdb
```
**note**
- postgresql-setup initdb: Initializes the PostgreSQL database cluster, which sets up the necessary files and directories for the PostgreSQL server to start.

Need to change config file as shown in below

```bash
vi /var/lib/pgsql/data/pg_hba.conf
```

Replace Method name "ident" to "md5"

![image](https://user-images.githubusercontent.com/68885738/90953619-aef2f800-e48a-11ea-9b50-489183e9b0c1.png)

**NOTE** 
- pg_hba.conf: This is the PostgreSQL client authentication configuration file. We need to modify it to allow password-based authentication for local connections.
- md5: Specifies that password-based authentication should be used for these connections.
Enable  postgresql:

```bash    
systemctl enable postgresql
```
Start postgresql:

```bash
systemctl start postgresql
````

Login into Database

```bash
su - postgres # su - postgres: Switches to the PostgreSQL administrative user postgres.
psql          # Opens the PostgreSQL command-line interface.
```

Create a sonarqubedb database, username and provide access to user

```bash
create database sonarqubedb;
create user sonarqube with encrypted password 'Aparna#123';
grant all privileges on database sonarqubedb to sonarqube;
\c sonarqubedb;
GRANT ALL ON SCHEMA public TO sonarqube;
GRANT USAGE ON SCHEMA public TO sonarqube;
\q                                                               # Exits the psql interface.
exit                                                             #  Returns to the regular user prompt.
```

## Setup Sonarqube Web Server
Download the latest sonarqube installation file to /opt folder. You can get the latest download link from here. http://www.sonarqube.org/downloads/

 ```bash
cd /opt
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.0.65466.zip
unzip sonarqube-9.9.0.65466.zip
mv sonarqube-9.9.0.65466 sonarqube
chown -R ec2-user:ec2-user sonarqube
```
	
Open /opt/sonarqube/conf/sonar.properties file
**note**
- sonar.properties: This file contains the configuration settings for SonarQube. We need to set up the database connection here.

```
sudo vi /opt/sonarqube/conf/sonar.properties
```
![image](https://user-images.githubusercontent.com/68885738/90953687-7acc0700-e48b-11ea-94f9-4b32f8f170b0.png)
![image](https://user-images.githubusercontent.com/68885738/90953736-c1b9fc80-e48b-11ea-88f9-2629c85fdf56.png)
![image](https://user-images.githubusercontent.com/68885738/90953772-05146b00-e48c-11ea-8dab-143be09d878b.png)

- sonar.jdbc.username: The PostgreSQL user created earlier.
- sonar.jdbc.password: The password for the sonar PostgreSQL user.
- sonar.jdbc.url: The connection URL for the PostgreSQL database.

Switch to ec2-user and navigate to the start script directory

```bash
su - ec2-user
cd /opt/sonarqube/bin/linux-x86-64
./sonar.sh start
./sonar.sh status
```

Troubleshooting Sonarqube:
All the logs of sonarqube are present in the /opt/sonarqube/logs directory

```bash
cd /opt/sonarqube/logs
```
