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

To create and start all VMs, run:
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

The database server is now ready to accept connections from other services in our stack. In the next sections, we'll set up the remaining services (Memcached, RabbitMQ, Tomcat, and Nginx).

Would you like me to continue with the setup instructions for the remaining services?

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

