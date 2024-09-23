![images](https://github.com/user-attachments/assets/7c28d81d-9482-4000-bd46-9613088a69cc)

# Thingsboard-Integration
- Thingsboard Installation and Configuration
- Setting up Thingsboard (Server and Local)
- Connecting to Node-Red via MQTT
- Dashboard and Visualization
- Designing Dashboards (Waiting Time, Barchart, etc.)
- Streaming Video Integration
- Map and Graph Composer Configurations

## What is ThingsBoard?
ThingsBoard is an open-source IoT platform that enables rapid development, management, and scaling of IoT projects. Our goal is to provide the out-of-the-box IoT cloud or on-premises solution that will enable server-side infrastructure for your IoT applications.
### Features
With ThingsBoard, you are able to:
- Provision devices, assets and customers, and define relations between them.
- Collect and visualize data from devices and assets.
- Analyze incoming telemetry and trigger alarms with complex event processing.
- Control your devices using remote procedure calls (RPC).
- Build work-flows based on a device life-cycle event, REST API event, RPC request, etc.
- Design dynamic and responsive dashboards and present device or asset telemetry and insights to your customers.
- Enable use-case specific features using customizable rule chains.
- Push device data to other systems.

See ThingsBoard features list for more features and useful links to the specific feature documentation.
### Architecture
ThingsBoard is designed to be:
- scalable: the horizontally scalable platform, built using leading open-source technologies.
- fault-tolerant: no single-point-of-failure, every node in the cluster is identical.
- robust and efficient: a single server node can handle tens or even hundreds of thousands of devices, depending on the use-case. ThingsBoard cluster can handle millions of devices.
- customizable: adding new functionality is easy with customizable widgets and rule engine nodes.
- durable: never lose your data.
- 
## Thingsboard Installation and Configuration
### Install ThingsBoard on Ubuntu
#### Prerequisites
This guide describes how to install ThingsBoard on Ubuntu 20.04 LTS / 22.04 LTS / 24.04 LTS. Hardware requirements depend on chosen database and amount of devices connected to the system. To run ThingsBoard and PostgreSQL on a single machine you will need at least 4Gb of RAM. To run ThingsBoard and Cassandra on a single machine you will need at least 8Gb of RAM.
#### Step 1. Install Java 17 (OpenJDK)
ThingsBoard service is running on Java 17. Follow this instructions to install OpenJDK 17:
```bash
sudo apt update
sudo apt install openjdk-17-jdk
```
Please don’t forget to configure your operating system to use OpenJDK 17 by default. You can configure which version is the default using the following command:
```
sudo update-alternatives --config java
```
You can check the installation using the following command:
```
java -version
```
```bash
Expected command output is:
openjdk version "17.x.xx" 
OpenJDK Runtime Environment (...)
OpenJDK 64-Bit Server VM (...)
```

#### Step 2. ThingsBoard service installation
Download installation package.
```
wget https://dist.thingsboard.io/thingsboard-3.7pe.deb
```
Install ThingsBoard as a service:
```
sudo dpkg -i thingsboard-3.7pe.deb
```
#### Step 3. Obtain and configure license key
We assume you have already chosen your subscription plan or decided to purchase a perpetual license. If not, please navigate to pricing page to select the best license option for your case and get your license. See How-to get pay-as-you-go subscription or How-to get perpetual license for more details.
Once you get the license secret, you should put it to the thingsboard configuration file. Open the file for editing using the following command:
```
sudo nano /etc/thingsboard/conf/thingsboard.conf
```
Locate the following configuration block:
```bash
# License secret obtained from ThingsBoard License Portal (https://license.thingsboard.io)
# UNCOMMENT NEXT LINE AND PUT YOUR LICENSE SECRET:
export TB_LICENSE_SECRET=
```
and put your license secret. Please don’t forget to uncomment the export statement. See example below:
```bash
# License secret obtained from ThingsBoard License Portal (https://license.thingsboard.io)
# UNCOMMENT NEXT LINE AND PUT YOUR LICENSE SECRET:
export TB_LICENSE_SECRET=YOUR_LICENSE_SECRET_HERE
```
#### Step 4. Configure ThingsBoard database
ThingsBoard is able to use SQL or hybrid database approach. See corresponding architecture page for more details.
<pre>PostgreSQL
(recommended for < 5K msg/sec)
</pre>
<pre>Hybrid
PostgreSQL+Cassandra
(recommended for > 5K msg/sec)
</pre>

##### Doc info icon
ThingsBoard team recommends to use PostgreSQL for development and production environments with reasonable load (< 5000 msg/sec). Many cloud vendors support managed PostgreSQL servers which is a cost-effective solution for most of ThingsBoard instances.

### PostgreSQL Installation
Instructions listed below will help you to install PostgreSQL.
```bash
# install **wget** if not already installed:
sudo apt install -y wget

# import the repository signing key:
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

# add repository contents to your system:
echo "deb https://apt.postgresql.org/pub/repos/apt/ $(lsb_release -cs)-pgdg main" | sudo tee  /etc/apt/sources.list.d/pgdg.list

# install and launch the postgresql service:
sudo apt update
sudo apt -y install postgresql-15
sudo service postgresql start
```
install **wget** if not already installed:
```
sudo apt install -y wget
```
import the repository signing key:
```
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
```
add repository contents to your system:
```
echo "deb https://apt.postgresql.org/pub/repos/apt/ $(lsb_release -cs)-pgdg main" | sudo tee  /etc/apt/sources.list.d/pgdg.list
```
install and launch the postgresql service:
```
sudo apt update
sudo apt -y install postgresql-15
sudo service postgresql start
```
Once PostgreSQL is installed you may want to create a new user or set the password for the the main user. The instructions below will help to set the password for main postgresql user
<pre>
sudo su - postgres
psql
\password
\q
</pre>
Then, press “Ctrl+D” to return to main user console and connect to the database to create thingsboard DB:
<pre>
psql -U postgres -d postgres -h 127.0.0.1 -W
CREATE DATABASE thingsboard;
\q
</pre>

### ThingsBoard Configuration
Edit ThingsBoard configuration file
```
sudo nano /etc/thingsboard/conf/thingsboard.conf
```
Add the following lines to the configuration file. Don’t forget to replace ```“PUT_YOUR_POSTGRESQL_PASSWORD_HERE”``` with your real postgres user password:
```bash
# DB Configuration 
export DATABASE_TS_TYPE=sql
export SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/thingsboard
export SPRING_DATASOURCE_USERNAME=postgres
export SPRING_DATASOURCE_PASSWORD=PUT_YOUR_POSTGRESQL_PASSWORD_HERE
# Specify partitioning size for timestamp key-value storage. Allowed values: DAYS, MONTHS, YEARS, INDEFINITE.
export SQL_POSTGRES_TS_KV_PARTITIONING=MONTHS
```
#### Step 5. Choose ThingsBoard queue service
ThingsBoard is able to use various messaging systems/brokers for storing the messages and communication between ThingsBoard services. How to choose the right queue implementation?
- **In Memory queue** implementation is built-in and default. It is useful for development(PoC) environments and is not suitable for production deployments or any sort of cluster deployments.
- **Kafka** is recommended for production deployments. This queue is used on the most of ThingsBoard production environments now. It is useful for both on-prem and private cloud deployments. It is also useful if you like to stay independent from your cloud provider. However, some providers also have managed services for Kafka. See AWS MSK for example.
- **RabbitMQ** is recommended if you don’t have much load and you already have experience with this messaging system.
- **AWS SQS** is a fully managed message queuing service from AWS. Useful if you plan to deploy ThingsBoard on AWS.
- **Google Pub/Sub** is a fully managed message queuing service from Google. Useful if you plan to deploy ThingsBoard on Google Cloud.
- **Azure Service Bus** is a fully managed message queuing service from Azure. Useful if you plan to deploy ThingsBoard on Azure.
- **Confluent Cloud** is a fully managed streaming platform based on Kafka. Useful for a cloud agnostic deployments.

#### Step 6. [Optional] Memory update for slow machines (4GB of RAM)
Edit ThingsBoard configuration file
```bash
sudo nano /etc/thingsboard/conf/thingsboard.conf
```
Add the following lines to the configuration file.
```bash
# Update ThingsBoard memory usage and restrict it to 2G in /etc/thingsboard/conf/thingsboard.conf
export JAVA_OPTS="$JAVA_OPTS -Xms2G -Xmx2G"
```
We recommend adjusting these parameters depending on your server resources. It should be set to at least 2G (gigabytes), and increased accordingly if there is additional RAM space available. Usually, you need to set it to 1/2 of your total RAM if you do not run any other memory-intensive processes (e.g. Cassandra), or to 1/3 otherwise.
#### Step 7. Run installation script
Once ThingsBoard service is installed and DB configuration is updated, you can execute the following script:
```bash
# --loadDemo option will load demo data: users, devices, assets, rules, widgets.
sudo /usr/share/thingsboard/bin/install/install.sh --loadDemo
```
#### Step 8. Start ThingsBoard service
Execute the following command to start ThingsBoard:
```
sudo service thingsboard start
```
Once started, you will be able to open Web UI using the following link:
```
http://localhost:8080/
```
The following default credentials are available if you have specified –loadDemo during execution of the installation script:
```ruby
System Administrator: sysadmin@thingsboard.org / sysadmin
Tenant Administrator: tenant@thingsboard.org / tenant
Customer User: customer@thingsboard.org / customer
```
You can always change passwords for each account in account profile page.

##### Doc info icon
Please allow up to 90 seconds for the Web UI to start.

#### Step 9. Install ThingsBoard WebReport component
Download installation package for the Reports Server component:
```
wget https://dist.thingsboard.io/tb-web-report-3.7pe.deb
```
Install third-party libraries:
```
sudo apt install -yq gconf-service libasound2 libatk1.0-0 libc6 libcairo2 libcups2 libdbus-1-3 \
     libexpat1 libfontconfig1 libgcc1 libgconf-2-4 libgdk-pixbuf2.0-0 libglib2.0-0 libgtk-3-0 libnspr4 \
     libpango-1.0-0 libpangocairo-1.0-0 libstdc++6 libx11-6 libx11-xcb1 libxcb1 libxcomposite1 \
     libxcursor1 libxdamage1 libxext6 libxfixes3 libxi6 libxrandr2 libxrender1 libxss1 libxtst6 \
     ca-certificates fonts-liberation libappindicator1 libnss3 lsb-release xdg-utils unzip wget libgbm-dev
```
Install Roboto fonts:
```
sudo apt install fonts-roboto
```
Install Noto fonts (Japanese, Chinese, etc.):
```bash
mkdir ~/noto
cd ~/noto
wget https://noto-website.storage.googleapis.com/pkgs/NotoSansCJKjp-hinted.zip
unzip NotoSansCJKjp-hinted.zip
sudo mkdir -p /usr/share/fonts/noto
sudo cp *.otf /usr/share/fonts/noto
sudo chmod 655 -R /usr/share/fonts/noto/
sudo fc-cache -fv
cd ..
rm -rf ~/noto
```
Install and start Web Report service:
```
sudo dpkg -i tb-web-report-3.7pe.deb
sudo service tb-web-report start
```
### Post-installation steps
#### Configure HAProxy to enable HTTPS
You may want to configure HTTPS access using HAProxy. This is possible in case you are hosting ThingsBoard in the cloud and have a valid DNS name assigned to your instance. Please follow this guide to install HAProxy and generate valid SSL certificate using Let’s Encrypt.
### Troubleshooting
ThingsBoard logs are stored in the following directory:
```ruby
/var/log/thingsboard
```
You can issue the following command in order to check if there are any errors on the backend side:
```ruby
cat /var/log/thingsboard/thingsboard.log | grep ERROR
```
