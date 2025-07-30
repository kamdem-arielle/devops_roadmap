# Complete Java Stack Setup with Nginx, Tomcat, MySQL, RabbitMQ, and Memcached

This guide provides a detailed walkthrough for setting up a complete Java application stack using multiple virtual machines (VMs). We'll start with a manual setup and later explore automation options.

## Infrastructure Overview

Our setup consists of 5 VMs, each serving a specific purpose:
- `web01` (Ubuntu): Frontend web server running Nginx
- `app01` (CentOS): Application server running Tomcat
- `db01` (CentOS): Database server running MariaDB
- `mc01` (CentOS): Caching server running Memcached
- `rmq01` (CentOS): Message broker running RabbitMQ

## Prerequisites

- VirtualBox installed on your system
- Vagrant installed on your system
- Basic understanding of Linux commands

## Step 1: Setting Up the Development Environment

### 1.1 Installing the Vagrant Hostmanager Plugin

```bash
vagrant plugin install vagrant-hostmanager
```

The Vagrant Hostmanager plugin is essential because:
- It automatically manages the `/etc/hosts` file on both the host and guest machines
- It enables VMs to communicate with each other using hostnames instead of IP addresses
- It eliminates the need for manual DNS configuration
- It makes the setup more maintainable and portable

### 1.2 Creating the Vagrant Environment

Create a file named `Vagrantfile` with the following content:

```ruby
Vagrant.configure("2") do |config|
  config.hostmanager.enabled = true 
  config.hostmanager.manage_host = true

### DB vm  ####
  config.vm.define "db01" do |db01|
    db01.vm.box = "centos/stream9"
    db01.vm.hostname = "db01"
    db01.vm.network "private_network", ip: "192.168.56.15"
    db01.vm.provider "virtualbox" do |vb|
     vb.memory = "600"
   end
  end

### Memcache vm  #### 
  config.vm.define "mc01" do |mc01|
    mc01.vm.box = "centos/stream9"
    mc01.vm.hostname = "mc01"
    mc01.vm.network "private_network", ip: "192.168.56.14"
    mc01.vm.provider "virtualbox" do |vb|
     vb.memory = "600"
   end
  end

### RabbitMQ vm  ####
  config.vm.define "rmq01" do |rmq01|
    rmq01.vm.box = "centos/stream9"
    rmq01.vm.hostname = "rmq01"
    rmq01.vm.network "private_network", ip: "192.168.56.13"
    rmq01.vm.provider "virtualbox" do |vb|
     vb.memory = "600"
   end
  end

### tomcat vm ###
   config.vm.define "app01" do |app01|
    app01.vm.box = "centos/stream9"
    app01.vm.hostname = "app01"
    app01.vm.network "private_network", ip: "192.168.56.12"
    app01.vm.provider "virtualbox" do |vb|
     vb.memory = "800"
   end
   end

### Nginx VM ###
  config.vm.define "web01" do |web01|
    web01.vm.box = "ubuntu/jammy64"
    web01.vm.hostname = "web01"
    web01.vm.network "private_network", ip: "192.168.56.11"
    web01.vm.provider "virtualbox" do |vb|
     vb.gui = true
     vb.memory = "800"
   end
  end
end
```

### 1.3 Starting the Virtual Machines

To create and start all VMs,go to the directory containing the vagrant file config and run the command below:
```bash
vagrant up
```

This command will:
1. Download the required VM images (CentOS Stream 9 and Ubuntu 22.04)
2. Create 5 VMs with the specified configurations
3. Configure the network settings
4. Update the hosts files using the hostmanager plugin

## Step 2: Database Server Setup (db01)

SSH into the database server:
**This command connects you to the `db01` VM using SSH, allowing you to run commands directly on the database server.**
```bash
vagrant ssh db01
```

### 2.1 System Updates and Repository Setup

**Update all installed packages to their latest versions, ensuring your system is secure and up-to-date.**
```bash
# Update the system packages
dnf update -y
```

**Install the Extra Packages for Enterprise Linux (EPEL) repository, which provides access to additional software not included in the default CentOS repositories.**
```bash
# Install EPEL repository for additional packages
dnf install epel-release -y
```

### 2.2 Installing MariaDB Server

**Install MariaDB server (the database engine) and Git (useful for version control or downloading code).**
```bash
# Install MariaDB and Git
dnf install git mariadb-server -y
```

**Start the MariaDB database service immediately.**
```bash
# Start MariaDB service
systemctl start mariadb
```

**Configure MariaDB to start automatically every time the server boots.**
```bash
# Enable MariaDB service
systemctl enable mariadb
```

### 2.3 Securing and Configuring MariaDB

**Run a script to help you secure your MariaDB installation by setting a root password, removing anonymous users, disabling remote root login, and removing test databases.**
```bash
# Run the security script
mysql_secure_installation
```

**Log into MariaDB as the root user with administrative privileges.**
```bash
# Access MariaDB
mysql -u root -padmin123
```

**Create a new database named `accounts` to store your application's data.**
```bash
# Create database
mysql> create database accounts;
```

**Grant all privileges on the `accounts` database to the `admin` user when connecting from `localhost`.**
```bash
# Grant privileges for local connections
mysql> grant all privileges on accounts.* TO 'admin'@'localhost' identified by 'admin123';
```

**Allow the `admin` user to connect to the database from any host (useful for distributed applications).**
```bash
# Grant privileges for remote connections
mysql> grant all privileges on accounts.* TO 'admin'@'%' identified by 'admin123';
```

**Reload the grant tables to apply changes immediately.**
```bash
# Apply privilege changes
mysql> FLUSH PRIVILEGES;
```

**Exit the MariaDB shell after completing the initial database setup.**
```bash
# Exit MariaDB
mysql> exit;
```

**Load the structure (tables, etc.) and possibly initial data for your application from a file called `database.sql`.**
```bash
# Import database schema
mysql -u root -padmin123 accounts < database.sql
```

**Log into the `accounts` database to verify that the schema was imported correctly.**
```bash
# Verify the import
mysql -u root -padmin123 accounts
```

**List all tables in the current database to ensure they were created successfully.**
```bash
# Show tables
mysql> show tables;
```

**Exit the MariaDB shell after verifying the database setup.**
```bash
# Exit MariaDB
mysql> exit;
```

**Restart the MariaDB service to ensure all changes are applied and the service is running with the latest configuration.**
```bash
# Restart MariaDB
systemctl restart mariadb
```

### 2.4 Configuring Firewall

**Start the firewall service to protect your server by controlling incoming and outgoing network traffic.**
```bash
# Start firewall
systemctl start firewalld
```

**Ensure the firewall starts automatically on boot.**
```bash
# Enable firewall
systemctl enable firewalld
```

**Check which firewall zones are active to understand which network interfaces are protected and how.**
```bash
# Check active zones
firewall-cmd --get-active-zones
```

**Open the MariaDB port (3306) in the firewall to allow other VMs or hosts to connect.**
```bash
# Allow MariaDB port
firewall-cmd --zone=public --add-port=3306/tcp --permanent
```

**Reload the firewall configuration to apply any changes made to the firewall rules.**
```bash
# Reload firewall
firewall-cmd --reload
```

**Restart the MariaDB service to ensure it is accessible with the new firewall rules.**
```bash
# Restart MariaDB
systemctl restart mariadb
```

The database server is now ready to accept connections from other services in our stack.

## Step 3: Memcached Server Setup (mc01)

SSH into the Memcached server:
**This command connects you to the `mc01` VM using SSH, allowing you to run commands directly on the Memcached server.**
```bash
vagrant ssh mc01
```

### 3.1 Verifying Hosts Entry

**Check the `/etc/hosts` file to ensure it contains the correct IP and hostname mappings for all VMs.**
```bash
# Verify hosts entry
cat /etc/hosts
```

**If entries are missing, update the `/etc/hosts` file with the correct IP and hostnames.**

### 3.2 Updating the OS

**Update all installed packages to their latest versions, ensuring your system is secure and up-to-date.**
```bash
# Update the system packages
dnf update -y
```

### 3.3 Installing and Configuring Memcached

**Install the Extra Packages for Enterprise Linux (EPEL) repository, which provides access to additional software not included in the default CentOS repositories.**
```bash
# Install EPEL repository
sudo dnf install epel-release -y
```

**Install Memcached, a high-performance distributed memory object caching system.**
```bash
# Install Memcached
sudo dnf install memcached -y
```

**Start the Memcached service immediately.**
```bash
# Start Memcached service
sudo systemctl start memcached
```

**Enable Memcached to start automatically every time the server boots.**
```bash
# Enable Memcached service
sudo systemctl enable memcached
```

**Check the status of the Memcached service to ensure it is running.**
```bash
# Check Memcached status
sudo systemctl status memcached
```

**Configure Memcached to listen on all network interfaces by replacing `127.0.0.1` with `0.0.0.0` in the configuration file.**
```bash
# Update Memcached configuration
sed -i 's/127.0.0.1/0.0.0.0/g' /etc/sysconfig/memcached
```

**Restart the Memcached service to apply the configuration changes.**
```bash
# Restart Memcached service
sudo systemctl restart memcached
```

### 3.4 Configuring Firewall

**Start the firewall service to protect your server by controlling incoming and outgoing network traffic.**
```bash
# Start firewall
systemctl start firewalld
```

**Ensure the firewall starts automatically on boot.**
```bash
# Enable firewall
systemctl enable firewalld
```

**Open TCP port 11211 in the firewall to allow other VMs or hosts to connect to Memcached.**
```bash
# Allow Memcached TCP port
firewall-cmd --add-port=11211/tcp
```

**Make the firewall rule permanent.**
```bash
# Make firewall rule permanent
firewall-cmd --runtime-to-permanent
```

**Open UDP port 11111 in the firewall to allow UDP connections to Memcached.**
```bash
# Allow Memcached UDP port
firewall-cmd --add-port=11111/udp
```

**Make the firewall rule permanent.**
```bash
# Make firewall rule permanent
firewall-cmd --runtime-to-permanent
```

### 3.5 Starting Memcached with Custom Ports

**Start Memcached with custom TCP and UDP ports (11211 and 11111) and run it as the `memcached` user in the background.**
```bash
# Start Memcached with custom ports
sudo memcached -p 11211 -U 11111 -u memcached -d
```

## Step 4: RabbitMQ Server Setup (rmq01)

SSH into the RabbitMQ server:
**This command connects you to the `rmq01` VM using SSH, allowing you to run commands directly on the RabbitMQ server.**
```bash
vagrant ssh rmq01
```

### 4.1 Verifying Hosts Entry

**Check the `/etc/hosts` file to ensure it contains the correct IP and hostname mappings for all VMs.**
```bash
# Verify hosts entry
cat /etc/hosts
```

**If entries are missing, update the `/etc/hosts` file with the correct IP and hostnames.**

### 4.2 Updating the OS

**Update all installed packages to their latest versions, ensuring your system is secure and up-to-date.**
```bash
# Update the system packages
sudo dnf update -y
```

### 4.3 Installing RabbitMQ

**Install the Extra Packages for Enterprise Linux (EPEL) repository, which provides access to additional software not included in the default CentOS repositories.**
```bash
# Install EPEL repository
sudo dnf install epel-release -y
```

**Install `wget`, a utility for downloading files from the web, which may be required for additional setup steps.**
```bash
# Install wget
sudo dnf install wget -y
```

**Add the RabbitMQ 3.8 repository to the system to ensure the correct version of RabbitMQ is installed.**
```bash
# Add RabbitMQ repository
sudo dnf -y install centos-release-rabbitmq-38
```

**Install RabbitMQ server from the newly added repository.**
```bash
# Install RabbitMQ server
sudo dnf --enablerepo=centos-rabbitmq-38 -y install rabbitmq-server
```

**Start the RabbitMQ service immediately and enable it to start automatically on boot.**
```bash
# Start and enable RabbitMQ service
sudo systemctl enable --now rabbitmq-server
```

### 4.4 Configuring RabbitMQ

**Allow remote access to RabbitMQ by modifying the configuration to remove loopback restrictions.**
```bash
# Allow remote access to RabbitMQ
sudo sh -c 'echo "[{rabbit, [{loopback_users, []}]}]." > /etc/rabbitmq/rabbitmq.config'
```

**Create a new RabbitMQ user named `test` with the password `test`.**
```bash
# Add RabbitMQ user
test
sudo rabbitmqctl add_user test test
```

**Grant administrator privileges to the `test` user.**
```bash
# Set user as administrator
sudo rabbitmqctl set_user_tags test administrator
```

**Set permissions for the `test` user to access all resources.**
```bash
# Set permissions for user
rabbitmqctl set_permissions -p / test ".*" ".*" ".*"
```

**Restart the RabbitMQ service to apply the configuration changes.**
```bash
# Restart RabbitMQ service
sudo systemctl restart rabbitmq-server
```

### 4.5 Configuring Firewall

**Start the firewall service to protect your server by controlling incoming and outgoing network traffic.**
```bash
# Start firewall
sudo systemctl start firewalld
```

**Ensure the firewall starts automatically on boot.**
```bash
# Enable firewall
sudo systemctl enable firewalld
```

**Open TCP port 5672 in the firewall to allow other VMs or hosts to connect to RabbitMQ.**
```bash
# Allow RabbitMQ port
firewall-cmd --add-port=5672/tcp
```

**Make the firewall rule permanent.**
```bash
# Make firewall rule permanent
firewall-cmd --runtime-to-permanent
```

**Start the RabbitMQ service to ensure it is running.**
```bash
# Start RabbitMQ service
sudo systemctl start rabbitmq-server
```

**Enable RabbitMQ to start automatically on boot.**
```bash
# Enable RabbitMQ service
sudo systemctl enable rabbitmq-server
```

**Check the status of the RabbitMQ service to ensure it is running.**
```bash
# Check RabbitMQ status
sudo systemctl status rabbitmq-server
```

RabbitMQ is now installed and configured on your server. It is ready to handle messaging for your application stack.

## Step 5: Tomcat Application Server Setup (app01)

SSH into the Tomcat application server:
**This command connects you to the `app01` VM using SSH, allowing you to run commands directly on the Tomcat application server.**
```bash
vagrant ssh app01
```

### 5.1 Verifying Hosts Entry

**Check the `/etc/hosts` file to ensure it contains the correct IP and hostname mappings for all VMs.**
```bash
# Verify hosts entry
cat /etc/hosts
```

**If entries are missing, update the `/etc/hosts` file with the correct IP and hostnames.**

### 5.2 Updating the OS

**Update all installed packages to their latest versions, ensuring your system is secure and up-to-date.**
```bash
# Update the system packages
sudo dnf update -y
```

### 5.3 Installing Dependencies

**Install the Extra Packages for Enterprise Linux (EPEL) repository, which provides access to additional software not included in the default CentOS repositories.**
```bash
# Install EPEL repository
sudo dnf install epel-release -y
```

**Install Java 17 OpenJDK (Java Runtime Environment and Development Kit) which is required to run Tomcat and Java applications.**
```bash
# Install Java 17 OpenJDK
sudo dnf -y install java-17-openjdk java-17-openjdk-devel
```

**Install Git (for version control) and wget (for downloading files from the internet).**
```bash
# Install Git and wget
sudo dnf install git wget -y
```

### 5.4 Downloading and Installing Tomcat

**Change to the `/tmp` directory which is used for temporary files and downloads.**
```bash
# Change to temporary directory
cd /tmp/
```

**Download the Apache Tomcat 10.1.26 archive from the official Apache repository.**
```bash
# Download Tomcat
wget https://archive.apache.org/dist/tomcat/tomcat-10/v10.1.26/bin/apache-tomcat-10.1.26.tar.gz
```

**Extract the downloaded Tomcat archive file to access its contents.**
```bash
# Extract Tomcat archive
tar xzvf apache-tomcat-10.1.26.tar.gz
```

### 5.5 Setting Up Tomcat User and Directory

**Create a dedicated system user named `tomcat` with a home directory at `/usr/local/tomcat` and no shell access for security purposes.**

```bash
# Add tomcat user
sudo useradd --home-dir /usr/local/tomcat --shell /sbin/nologin tomcat
```

**Copy all Tomcat files from the temporary extraction directory to the tomcat user's home directory.**
```bash
# Copy Tomcat files to home directory
sudo cp -r /tmp/apache-tomcat-10.1.26/* /usr/local/tomcat/
```

**Change ownership of all files in the Tomcat directory to the tomcat user and group for proper permissions.**
```bash
# Set ownership to tomcat user
sudo chown -R tomcat.tomcat /usr/local/tomcat
```

### 5.6 Creating Tomcat Service

**Create a systemd service file to manage Tomcat as a system service.**
```bash
# Create tomcat service file
vi /etc/systemd/system/tomcat.service
```

**Add the following content to the service file to define how systemd should manage the Tomcat service:**
```ini
[Unit]
Description=Tomcat
After=network.target

[Service]
User=tomcat
Group=tomcat
WorkingDirectory=/usr/local/tomcat
Environment=JAVA_HOME=/usr/lib/jvm/jre
Environment=CATALINA_PID=/var/tomcat/%i/run/tomcat.pid
Environment=CATALINA_HOME=/usr/local/tomcat
Environment=CATALINE_BASE=/usr/local/tomcat
ExecStart=/usr/local/tomcat/bin/catalina.sh run
ExecStop=/usr/local/tomcat/bin/shutdown.sh
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target
```

**This service configuration includes:**
- **Unit section**: Defines service description and dependencies
- **Service section**: Specifies how to run Tomcat (user, environment variables, start/stop commands)
- **Install section**: Defines when the service should be started during boot

### 5.7 Starting and Enabling Tomcat Service

**Reload systemd to recognize the new service file you just created.**
```bash
# Reload systemd files
sudo systemctl daemon-reload
```

**Start the Tomcat service immediately.**
```bash
# Start Tomcat service
sudo systemctl start tomcat
```

**Enable Tomcat to start automatically every time the server boots.**
```bash
# Enable Tomcat service
sudo systemctl enable tomcat
```

### 5.8 Configuring Firewall

**Start the firewall service to protect your server by controlling incoming and outgoing network traffic.**
```bash
# Start firewall
sudo systemctl start firewalld
```

**Ensure the firewall starts automatically on boot.**
```bash
# Enable firewall
sudo systemctl enable firewalld
```

**Check which firewall zones are active to understand which network interfaces are protected.**
```bash
# Check active zones
sudo firewall-cmd --get-active-zones
```

**Open TCP port 8080 in the firewall to allow access to the Tomcat web server.**
```bash
# Allow Tomcat port
sudo firewall-cmd --zone=public --add-port=8080/tcp --permanent
```

**Reload the firewall configuration to apply the new rule.**
```bash
# Reload firewall
sudo firewall-cmd --reload
```



