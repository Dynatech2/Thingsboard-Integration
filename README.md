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
sudo apt update
sudo apt install openjdk-17-jdk


Please don’t forget to configure your operating system to use OpenJDK 17 by default. You can configure which version is the default using the following command:
sudo update-alternatives --config java


You can check the installation using the following command:
java -version

Expected command output is:
openjdk version "17.x.xx" 
OpenJDK Runtime Environment (...)
OpenJDK 64-Bit Server VM (...)


#### Step 2. ThingsBoard service installation
Download installation package.
wget https://dist.thingsboard.io/thingsboard-3.7pe.deb



Install ThingsBoard as a service:
sudo dpkg -i thingsboard-3.7pe.deb


#### Step 3. Obtain and configure license key
We assume you have already chosen your subscription plan or decided to purchase a perpetual license. If not, please navigate to pricing page to select the best license option for your case and get your license. See How-to get pay-as-you-go subscription or How-to get perpetual license for more details.
Once you get the license secret, you should put it to the thingsboard configuration file. Open the file for editing using the following command:
sudo nano /etc/thingsboard/conf/thingsboard.conf


Locate the following configuration block:
# License secret obtained from ThingsBoard License Portal (https://license.thingsboard.io)
# UNCOMMENT NEXT LINE AND PUT YOUR LICENSE SECRET:
# export TB_LICENSE_SECRET=

and put your license secret. Please don’t forget to uncomment the export statement. See example below:
# License secret obtained from ThingsBoard License Portal (https://license.thingsboard.io)
# UNCOMMENT NEXT LINE AND PUT YOUR LICENSE SECRET:
export TB_LICENSE_SECRET=YOUR_LICENSE_SECRET_HERE

#### Step 4. Configure ThingsBoard database
ThingsBoard is able to use SQL or hybrid database approach. See corresponding architecture page for more details.
PostgreSQL(recommended for < 5K msg/sec)
Hybrid
PostgreSQL+Cassandra
(recommended for > 5K msg/sec)
