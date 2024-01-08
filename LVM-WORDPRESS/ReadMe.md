# **_Implementing Wordpress website with LVMStorage management_**

+ **Project Goal Summary**

This course aims to equip learners with the expertise to build and maintain a scalable WordPress website on AWS EC2 using Ubuntu, emphasizing _LVM_ (Logical Volume Management) storage. Participants will gain hands-on experience in EC2 setup, security configuration, and establishing a reliable website connection. The course covers intricate aspects of LVM storage management on Ubuntu, including logical volume creation, disk space management, and dynamic volume resizing. Additionally, participants will become proficient in WordPress installation, theme customization, plugin integration, and performance optimization on AWS. The course prepares individuals—whether web developers, system administrators, or aspiring AWS DevOps professionals—to effectively implement and manage WordPress sites on AWS EC2, leveraging LVM storage management. Enroll for a transformative journey into WordPress and AWS integration, unlocking the ability to create and manage dynamic websites confidently.

## _Understanding 3-Tier Architecture with WordPress_

This project guides DevOps engineers in configuring Linux-based storage infrastructure and implementing a basic web solution using WordPress— a PHP-based, open-source CMS paired with MySQL/MariaDB.

### _Project Overview_

+ Configure Storage Subsystem: Hands-on experience with Linux disks, partitions, and volumes.

+ Install WordPress and Connect to Remote Database: Deploy WordPress, connecting it to a remote MySQL database.

+ DevOps Significance: Develop deep understanding and troubleshooting skills for core web solution components.

### _Three-Tier Architecture_

+ Presentation Layer (Web Server): Manages user interaction through web servers like Apache or Nginx.

+ Application Layer (Application Server): Executes business logic, particularly PHP scripts.

+ Data Layer (Database Server): Handles data storage and retrieval using MySQL/MariaDB.

## Project Focus

+ 3-Tier Setup

Client: A laptop or PC serving as the user interface.
Web Server (EC2 Linux): Hosting WordPress installation.
Database Server (EC2 Linux): Dedicated server for database operations.

+ Choice of OS:

RedHat: Preferred OS for this project, emphasizing familiarity with different Linux distributions.
Note: Use the ec2-user user when connecting to RedHat via SSH/Putty.
AWS Setup:

+ EC2 instance setup reminder, referencing Project 1 Step O.
Transition from Ubuntu to RedHat, promoting versatility in Linux distributions.

## **Implementing LVM on Linux Servers (Web and Database Servers)**

+ **Step 1: Prepare a Web Server**

>Launch an EC2 Instance:
Begin by launching an EC2 instance designated as the "Web Server." Ensure that it's in the same Availability Zone (AZ) as your Web Server EC2 for optimal performance.

>Create EBS Volumes:
Create three Elastic Block Store (EBS) volumes, each with a size of 10 GiB, in the same AZ as your Web Server EC2. These volumes will serve as the storage foundation for your Web Server.

>Attach all three volumes one by one to your Web Server EC2 instance.

This initial step sets up the foundation for your Web Server and ensures that the necessary storage resources are available for further configuration using Logical Volume Management (LVM).

+ **Step 2: Launch the Linux Terminal for Configuration**

+ **Step 3: Employ the `lsblk` Command for a Block Device Overview**

>To initiate the configuration process, access the Linux terminal. Execute the `lsblk` command to scrutinize the attached block devices on the server. Identify the names of your recently created devices, typically labeled as _xvdf_, _xvdh_, and _xvdg_. Confirm their presence in the _/dev/_ directory by using `ls /dev/`. This step ensures that all three new block devices are correctly listed, verifying their association with the server.

+ **Step 4: Run `df -h` command to see all mounts and free space on your server.**

+ **Step 5: Use `gdisk` utility to create a single partition on each of the 3 disks.**

+ **Step 6: Use `Isblk` utility to view the newly configured partition on each of the 3 disks.**

+ **Step 7: Install `lvm2` package using `sudo yum install lvm2` , then Run `sudo lvmdiskscan` command to check for available partitions.**

+ **Step 8: Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM**

>Use the below codes:

`sudo pvcreate /dev/xvdf1`

`sudo pvcreate /dev/xvdg1`

`sudo pvcreate /dev/xvdh1`

+ **Step 9: Verify that your Physical volume has been created successfully by running `sudo pvs`**

+ **Step 10: Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG _webdata-vg_**

>Copy Below Code

`sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`

+ **Step 11: Verify that your VG has been created successfully by running `sudo vgs`**

+ **Step 12: Use lvcreate utility to create 2 logical volumes. apps-Iv (Use half of the PV size), and logs-Iv Use the remaining space of the PV size.**

NOTE: apps-Iv will be used to store data for the Website while, logs-Iv will be used to store data for logs.

>Copy Below Code

`sudo lvcreate -n apps-lv -L 14G webdata-va sudo lvcreate -n logs-Iv -L 14G webdata-vg`

+ **Step 13: Verify that your Logical Volume has been created successfully by running `sudo lvs`**

+ **Step 14: Verify the entire setup**

>Copy Below Code

`sudo vgdisplay -v #view complete setup - VG, PV, and LV sudo lsblk`

+ **Step 15: Use mkfs.ext4 to format the logical volumes with ext4 filesystem**

>Copy Below Code

`sudo mkfs -t ext4 /dev/webdata-vg/apps-lv sudo mkfs -t ext4 / dev/webdata-vg/logs-lv`

+ **Step 16: Create /var/www/html directory to store website files**

`sudo mkdir -p /var/www/html`

+ **Step 17: Create /home/recovery/logs to store backup of log data**

`sudo mkdir -p /home/recovery/logs`

+ **Step 18: Mount /var/www/html on apps-Iv logical volume**

>Copy Below Code

`sudo mount /dev/webdata-vg/apps-lv /var/www/html/`

+ **Step 19: Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)**

>Copy Below Code

`sudo rsync -av /var/log/. /home/recovery/logs/`

+ **Step 20: Mount /var/log on logs-Iv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 15 above is very important)**

>Copy Below Code

`sudo mount /dev/webdata-vg/logs-lv /var/log`

+ **Step 21: Restore log files back into /var/log directory**

`sudo rsync -av /home/recovery/logs/log/. /var/log`

+ **Step 22: Update /etc/stab file so that the mount configuration will persist after restart of the server.**

The UUID of the device will be used to update the /etc/fstab file;

`sudo blkid`

Now, run `sudo vi /etc/fstab`

Update `/etc/fstab` in this format using your own UUID and remember to remove the leading and ending quotes.

+ **Step 23: Test the configuration and reload the daemon**

>Copy Below Code

`sudo mount -a sudo systemctl daemon-reload`

+ **Step 25: Verify your setup by running _df -h_ , output must look like this:**


## Installing wordpress and configuring to use MySQL Database

### **Step 2: Prepare the Database Server**

Launch a second RedHat EC2 instance that will have a role - 'DB Server' Repeat the same steps as for the Web Server, but instead lv create db-lv and mountitto /b directory instead of /var/www/html/

### **Step 3: Install Wordpress on your Web Server EC2**

>Update the repository

`sudo yum -y update`

2. Install wet, Apache and it's dependencies

`sudo yum -y install wet httpd php php-mysqlnd php-fpm php-json`

3. Start Apache

>Copy Below Code `sudo svstemctl enable httod sudo systemctl start httpd`

4. To install PHP and it's dependencies**

>Copy Below Code

`sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`

`sudo yum install yum-utils http://roms.remirepo.net/enterprise/remi-release-8.rpm`

`sudo vum module list php`

`sudo yum module reset php`

`sudo yum module enable php: remi-7.4`

`sudo yum install ph php-opcache php-gd php-curl php-mysqlnd`

`sudo systemctl start php-fpm`

`sudo systemctl enable php-fpm setsebool -P httpd_execmem 1`

5. Restart Apache**

`sudo systemctl restart httpd`

6. Download wordpress and copy wordpress to `var/www/html`**

>Copy Below Code

`mkdir wordpress cd wordpress`

`sudo wet http://wordpress.org/latest.tar.gz`

`sudo tar xzvf latest.tar.gz sudo rm -rf latest.tar.gz`

`cp wordpress/wp-config-sample.ph wordpress/w-config.php`

`cp -R wordpress /var/www/html/`

7. Configure SELinux Policies**

>Copy Below Code

`sudo chown -R apache:apache /var/www/html/wordpress`

`sudo chon -t httpd_sys_rw_content_t /var/www/html/wordpress sudo setsebool -P httpd_can_network_connect=1`


### **Step 4: Install MySQL on your DB Server EC2**

>Copy Below Code

`sudo yum update`

`sudo yum install mysql-server`

+ Verify that the service is up and running by using `sudo systemctl status mysqld`, if it is not running, restart the service and enable it so it will be running even after reboot:

>Run the Below Code

`sudo systemctl restart mysqld` 

`sudo systemctl enable mysqld`

### **Step 5: Configure DB to work with WordPress**

>Copy Below Code

`sudo mysql`

>CREATE DATABASE wordpress;

>CREATE USER 'myuser'@'<Web-Server-Private-IP-Address>' IDENTIFIED BY 'mypass';

>GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';

>FLUSH PRIVILEGES;

>SHOW DATABASES;

>exit

### **Step 6: Configure WordPress to connect to remote database.**

+ Hint: Do not forget to open MySQL port 3306 on DB Server EC2. For extra security, you shall allow access to the DB server ONLY from your Web Server's IP address, so in the Inbound Rule configuration specify source as /32

1. Install MySQL client and test that you can connect from your Web Server to your DB server by using mysql-client
Copy Below Code

`sudo yum install mysql`

`sudo mysql -u admin -p -h <DB-Server-Private-IP-address>`

2. Verify if you can successfully execute SHOW DATABASES; command and see a list of existing databases.

3. Change permissions and configuration so Apache could use Word Press:

4. Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation's IP)

5. Try to access from your browser the link to your WordPress http://<Web-Server-Public-IP-Address>/wordpress/

![Alt text](<Images/Screenshot 2024-01-04 at 7.06.38 AM.png>)

![Alt text](<Images/Screenshot 2024-01-04 at 8.12.38 AM.png>)

![Alt text](<Images/Screenshot 2024-01-04 at 9.29.15 AM.png>)

![Alt text](<Images/Screenshot 2024-01-04 at 10.16.55 AM.png>)

![Alt text](<Images/Screenshot 2024-01-04 at 10.17.07 AM.png>)

![Alt text](<Images/Screenshot 2024-01-04 at 10.17.25 AM.png>)

![Alt text](<Images/Screenshot 2024-01-04 at 10.17.32 AM.png>)

![Alt text](<Images/Screenshot 2024-01-04 at 11.55.21 AM.png>)

![Alt text](<Images/Screenshot 2024-01-04 at 5.02.44 PM.png>)

# DONE