![thingsboard_logo](https://github.com/user-attachments/assets/df30dda5-5355-4936-836d-92ff8a2165e4)
# Thingsboard-Integration
- [What is ThingsBoard?](#What-is-ThingsBoard?)
- [Thingsboard Installation and Configuration](#Thingsboard-Installation-and-Configuration)
- [Setting up Thingsboard (Server and Local)](#Setting-up-Thingsboard-(Server-and-Local))
- [Connecting to Node-Red via MQTT](#Connecting-to-Node-Red-via-MQTT)
- [Dashboard and Visualization](#Dashboard-and-Visualization)
- [Designing Dashboards (Waiting Time, Barchart, etc.)](#Designing-Dashboards-(Waiting-Time-Barchart-etc))
- [Streaming Video Integration](#Streaming-Video-Integration)
- [Map and Graph Composer Configurations](#Map-and-Graph-Composer-Configurations)

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
## Setting up Thingsboard (Server and Local)
## Connecting to Node-Red via MQTT
## Dashboard and Visualization
## Designing Dashboards (Waiting Time, Barchart, etc.)
## Streaming Video Integration
---
## Map and Graph Composer Configurations

- Widget type: Time Series
- Widget Settings:

![image](https://github.com/user-attachments/assets/fcc9d4be-3505-4a95-8adf-31225a613191)


**Step 1: Create a MapTiler Account and Obtain an API Key**

1. Sign up for MapTiler:

- To use MapTiler maps in your project, you'll first need to create an account and obtain an API key.
- Visit the MapTiler website and either sign in or create a new account if you don’t already have one.

2. Create a new project:

- Once you're signed in, navigate to the Maps section and choose a map style that fits your needs (e.g., Streets, Satellite, Dark, etc.).
  
3. Generate your API key:

- After selecting the desired map style, go to the API Keys section in your MapTiler dashboard.
- Generate a new API key by clicking on the Create API key button, which will be used later to access MapTiler tiles in your project.
  
Copy the API key and keep it in a safe place. You will need it in Step 3 to configure the map.

**Step 2: Set Up the HTML Structure**

1. Create an HTML File (index.html):
   
- This file defines the structure of the web page, including the map container, and links to external CSS and JavaScript files.
- Inside the <head> section, add a link to the CSS file (styles.css) and the viewport settings to ensure the page is responsive on all devices.
- Inside the <body>, create a div for the map container, which will be styled later. Additionally, include a canvas element for optional visual effects (e.g., blur overlay).

```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Subang Jaya Map</title>
  <!-- Link to external CSS for styling -->
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <!-- Map Container -->
  <div id="map-container" style="position: relative; width: 100%; height: 600px;">
    <!-- Map element where Leaflet will render the map -->
    <div id="map" style="width: 100%; height: 100%;"></div>
    <!-- Optional canvas for visual effects, such as blurring -->
    <canvas id="blur-canvas" style="position: absolute; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none;"></canvas>
  </div>

  <!-- Leaflet.js script from CDN to handle map rendering -->
  <script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
  <!-- Link to external JavaScript for map logic -->
  <script src="script.js"></script>
</body>
</html>
```
**Step 3: Add CSS for Styling the Map and Page**

2. Create a CSS File (styles.css):
   
- Define the layout and visual styles for the page. The body tag removes any default margins and padding, while the map-container and map ensure that the map element takes up the full height and width of the container.
- Add styles for the blur-canvas to create a blur effect, if necessary, and ensure no scrollbars appear on the page.

```
/* Remove default margins and padding from the body */
body {
  margin: 0;
  padding: 0;
  overflow: hidden; /* Prevent page scrollbars */
}

/* Style for the container holding the map */
.map-container {
  position: relative;
  width: 100%; /* Full width for responsive design */
  height: 500px; /* Set a specific height for the map */
}

/* Style the map element to occupy the entire container */
#map {
  height: 100%; /* Take up full height of the container */
  position: relative;
  z-index: 1; /* Ensure the map appears above other elements */
}

/* Add a blur overlay effect using CSS for a visual effect (optional) */
.blur-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5); /* Semi-transparent black background */
  filter: blur(10px); /* Apply a blur effect */
  z-index: 0; /* Ensure the blur appears behind the map */
}

/* Create a margin-like effect by adding a border inside the map container */
.map-container:before {
  content: ''; /* Empty content to create a visual margin */
  position: absolute;
  top: 10px; left: 10px; right: 10px; bottom: 10px; /* Adjust margin as needed */
  background: inherit; /* Same background as the overlay */
  z-index: 0; /* Keep it behind the map */
  filter: none; /* No blur effect inside the border */
}

/* Additional style for the blur canvas */
#blur-canvas {
  filter: blur(10px); /* Apply a blur effect to the canvas */
  background-color: rgba(0, 0, 0, 0.5); /* Semi-transparent overlay color */
}
```
**Step 4: Initialize and Configure the Map in JavaScript**

3. Create a JavaScript File (script.js):
   
- This script will initialize the map using the Leaflet.js library. Leaflet is a popular open-source library for interactive maps.
- The map is initialized with specific zoom controls, boundaries, and tile layers from MapTiler. The boundaries (defined by GPS coordinates) ensure the map stays focused on the Subang Jaya area.
- The MapTiler tile layer is included using the API key generated in Step 1.

```
self.onInit = function() {
  // Define the bounding box for the Subang Jaya region (top-left and bottom-right corners)
  const corner1 = L.latLng(3.08755, 101.54414); // Top-left corner
  const corner2 = L.latLng(2.9649241, 101.7458); // Bottom-right corner

  // Create a Leaflet bounds object from the corners
  const bounds = L.latLngBounds(corner1, corner2);

  // Initialize the Leaflet map and attach it to the 'map' element
  self.ctx.map = L.map('map', {
    zoomControl: true, // Enable zoom controls (plus/minus buttons)
    dragging: true,    // Allow dragging/panning of the map
    maxBounds: bounds, // Restrict the map to stay within the bounding box
    maxBoundsViscosity: 1.0, // Strictly limit panning outside the bounds
    zoom: 12.49        // Initial zoom level (focused on Subang Jaya)
  });

  // Fit the map view to the bounding box, ensuring all areas are visible
  self.ctx.map.fitBounds(bounds);

  // Set the minimum and maximum zoom levels for the map
  const minZoom = 12.49;
  self.ctx.map.setMinZoom(minZoom); // Prevent zooming out beyond this level
  self.ctx.map.setMaxZoom(19);      // Allow zooming in up to level 19

  // Add a tile layer using MapTiler (dark theme) and your API key from Step 1
  L.tileLayer('https://api.maptiler.com/maps/streets-v2-dark/{z}/{x}/{y}.png?key=YOUR_API_KEY', {
    attribution: '© MapTiler © OpenStreetMap contributors', // Tile layer attribution
    maxZoom: 19 // Maximum zoom level for tile display
  }).addTo(self.ctx.map); // Add the tile layer to the map

  // Prevent panning outside the specified bounds
  self.ctx.map.on('drag', function() {
    self.ctx.map.panInsideBounds(bounds, { animate: false }); // Keep within bounds
  });

  // Ensure that zooming out doesn't go beyond the minimum zoom level
  self.ctx.map.on('zoomend', function() {
    if (self.ctx.map.getZoom() < minZoom) {
      self.ctx.map.setZoom(minZoom); // Reset to minimum zoom if zoomed out too far
    }
  });
};
```
**Step 5: Define the Bounding Box and Subang Jaya Polygon**

4. Add Polygons for the Map:
   
- To visually represent the Subang Jaya area, a polygon is created using GPS coordinates.
- The bounding box encloses the entire region, while the Subang Jaya polygon represents a specific area inside that boundary.

```
// Define coordinates for the bounding box around Subang Jaya
const boundingBoxCoords = [
  [3.2352, 101.41],   // Top-left corner
  [3.2352, 102.0954], // Top-right corner
  [2.8793, 102.0954], // Bottom-right corner
  [2.8793, 101.41],   // Bottom-left corner
  [3.2352, 101.41]    // Back to top-left to close the box
];

// Define coordinates for the Subang Jaya area (as a hole within the bounding box)
const subangJayaPolygonCoords = [
  // Add Subang Jaya GPS coordinates here (from the provided list)
];

// Combine the bounding box with Subang Jaya as a hole (inner polygon)
const boundingWithHole = [
  boundingBoxCoords,         // Outer bounding box
  subangJayaPolygonCoords     // Inner hole (Subang Jaya polygon)
];

// Create a polygon with the outer bounding box and inner hole
const polygonWithHole = L.polygon(boundingWithHole, {
  color: '#0C0C36',    // Border color for the bounding box
  fillColor: '#0C0C36', // Fill color for the bounding box
  fillOpacity: 1        // Full opacity for the filled area outside Subang Jaya
}).addTo(self.ctx.map);

// Add a separate polygon for Subang Jaya with only an outline (no fill)
const subangJayaPolygon = L.polygon(subangJayaPolygonCoords, {
  color: '#0C0C36',    // Border color for Subang Jaya
  fillColor: 'transparent', // No fill for the Subang Jaya area
  fillOpacity: 0        // Fully transparent fill
}).addTo(self.ctx.map);

// Optionally fit the map to the bounds of the polygon with the hole
self.ctx.map.fitBounds(polygonWithHole.getBounds());
```
**Step 6: Handle Resizing and Cleanup**

5. Ensure Proper Map Resizing and Cleanup:
   
- Use the invalidateSize() function from Leaflet to ensure the map resizes correctly when the window size changes.
- Clean up resources by removing the map when the widget is destroyed to prevent memory leaks.
  
```
// Handle map resizing when the browser window or container changes size
self.onResize = function() {
  if (self.ctx.map) {
    self.ctx.map.invalidateSize(); // Correctly resize the map
  }
};

// Clean up resources when the map widget is destroyed
self.onDestroy = function() {
  if (self.ctx.map) {
    self.ctx.map.remove(); // Remove the map from the DOM
    self.ctx.map = null;   // Set the map reference to null to prevent memory leaks
  }
};
```
