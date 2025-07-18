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

\`\`\`ruby
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
\`\`\`

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
```bash
vagrant ssh db01
```

### 2.1 System Updates and Repository Setup

```bash
# Update the system packages
dnf update -y

# Install EPEL repository for additional packages
dnf install epel-release -y
```

This step ensures your system is up-to-date and has access to additional packages from the EPEL repository.

### 2.2 Installing MariaDB Server

```bash
# Install MariaDB and Git
dnf install git mariadb-server -y

# Start and enable MariaDB service
systemctl start mariadb
systemctl enable mariadb
```

These commands:
- Install MariaDB server and Git
- Start the MariaDB service
- Configure MariaDB to start automatically on system boot

### 2.3 Securing and Configuring MariaDB

```bash
# Run the security script
mysql_secure_installation

# Access MariaDB
mysql -u root -padmin123

# Create database and user
mysql> create database accounts;
mysql> grant all privileges on accounts.* TO 'admin'@'localhost' identified by 'admin123';
mysql> grant all privileges on accounts.* TO 'admin'@'%' identified by 'admin123';
mysql> FLUSH PRIVILEGES;
mysql> exit;

# Import database schema (assuming database.sql exists)
mysql -u root -padmin123 accounts < database.sql

# Verify the import
mysql -u root -padmin123 accounts
mysql> show tables;
mysql> exit;

# Restart MariaDB
systemctl restart mariadb
```

This sequence:
1. Secures the MariaDB installation
2. Creates a new database named 'accounts'
3. Creates an admin user with necessary privileges
4. Imports the database schema
5. Verifies the database setup

### 2.4 Configuring Firewall

```bash
# Start and enable firewall
systemctl start firewalld
systemctl enable firewalld

# Check active zones
firewall-cmd --get-active-zones

# Allow MariaDB port
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --reload

# Restart MariaDB to apply all changes
systemctl restart mariadb
```

These commands:
1. Enable and start the firewall service
2. Open port 3306 (MySQL/MariaDB default port)
3. Apply the firewall changes
4. Restart MariaDB to ensure all configurations are active

The database server is now ready to accept connections from other services in our stack. In the next sections, we'll set up the remaining services (Memcached, RabbitMQ, Tomcat, and Nginx).

Would you like me to continue with the setup instructions for the remaining services?
