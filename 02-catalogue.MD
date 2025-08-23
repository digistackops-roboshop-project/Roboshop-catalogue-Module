
# Catalogue-Module Setup

Catalogue is a microservice that is responsible for serving the list of items that displays in roboshop application.

## Install NodeJS

**Developer has chosen NodeJs, Check with developer which version of NodeJS is needed.**
**Developer has set a context that it can work with NodeJS >20**

Install NodeJS, By default NodeJS 16 is available, We would like to enable 20 version and install list.

**You can list modules by using `dnf module list nodejs`**

Disable current module
```shell 
dnf module disable nodejs -y
```

Enable required module
```shell
dnf module enable nodejs:20 -y
```

Install NodeJS 
```shell 
dnf install nodejs -y
```
## Create one Application User

Configure the application.

Our application developed by the developer of our org and it is not having any RPM software just like other softwares. So we need to configure every step manually

We already discussed in Linux basics section that applications should run as nonroot user.

Add application User

```shell 
useradd --system --home /app --shell /sbin/nologin --comment "roboshop system user" roboshop
```

User **roboshop** is a function / daemon user to run the application. Apart from that we dont use this user to login to server.

Also, username **roboshop** has been picked because it more suits to our project name.

## Get the Code

We keep application in one standard location. This is a usual practice that runs in the organization.
Lets setup an app directory. 

```shell
mkdir /app 
```
Download the application code to created app directory. 

```shell
curl -o /tmp/catalogue.zip https://roboshop-artifacts.s3.amazonaws.com/catalogue-v3.zip 
cd /app 
unzip /tmp/catalogue.zip
```
### Download Dependencies

Every application is developed by development team will have some common softwares that they use as libraries. This application also have the same way of defined dependencies in the application configuration.

Lets download the dependencies. 

```shell 
cd /app 
npm install 
```
When you hit npm install => it will read the package.json file and install the mentioned dependencies

### Create systemd service to start Application

We need to setup a new service in **systemd** so `systemctl` can manage this service

We already discussed in linux basics that advantages of systemctl managing services, Hence we are taking that approach. Which is also a standard way in the OS. 


Setup SystemD Catalogue Service 

```unit file (systemd) title=/etc/systemd/system/catalogue.service
[Unit]
Description = Catalogue Service

[Service]
User=roboshop
Environment=MONGO=true
// highlight-start
Environment=MONGO_URL="mongodb://<MONGODB-SERVER-IPADDRESS>:27017/catalogue"
// highlight-end
ExecStart=/bin/node /app/server.js
SyslogIdentifier=catalogue

[Install]
WantedBy=multi-user.target
```

Hint! You can create file by using **`vim /etc/systemd/system/catalogue.service`**

Ensure you replace `<MONGODB-SERVER-IPADDRESS>` with IP address

Load the service.

```shell 
systemctl daemon-reload
```

This above command is because we added a new service, We are telling systemd to reload so it will detect new service.

Start the service.

```shell 
systemctl enable catalogue 
systemctl start catalogue
```

# Load Data into DATABASE

For the application to work fully functional we need to load schema to the Database. Additonally we are going to have master data to be seeded for the applications to work. Here in this case it is list of products we want to sale.


Schemas are usually part of application code and developer will provide them as part of development.

Master data are usually provided by business operations team.

To load schema / master data we need to install mongodb client and then we can load it.

To have mongo client installed we have to setup MongoDB repo and install mongodb-client. You can create file using

```
vim /etc/yum.repos.d/mongo.repo
```

``` shell
[mongodb-org-7.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/9/mongodb-org/7.0/x86_64/
enabled=1
gpgcheck=0
```

```shell 
dnf install mongodb-mongosh -y
```


Load Master Data of the List of products we want to sell and their quantity information also there in the same master data. 

```
mongosh --host MONGODB-SERVER-IPADDRESS </app/db/master-data.js
```

**NOTE:**
You need to update catalogue server ip address in frontend configuration. 
Configuration file is `/etc/nginx/nginx.conf` 

Use below commands to check data is loaded into mongodb or not
Connect to MongoDB
```
mongosh --host MONGODB-SERVER-IPADDRESS
```

Show databases
```
show dbs
```

Use database
```
use catalogue
```

Show Collections
```
show collections
```

Get the items in collection
```
db.products.find()
```


# Troubleshoot [or] check our DB is Running or NOT

### 1. Check the Service is running or Not
```
systemctl status catalogue  
```
Which says catalogue Status is Active or Not
### 2. Check the catalogue process is Active or Not
```
ps -ef | grep node
```
which say your catalogue nnode process is running or not
### 3. Check the catalogue Port is open or Not 
```
netstat -lntp
```
it will check your catalogue  8080 port is open or Not {catalogue is using Node.js, Node.js running in 8080 port}
