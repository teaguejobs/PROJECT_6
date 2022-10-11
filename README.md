# PROJECT_6
**WEB SOLUTION WITH WORDPRESS**

**LAUNCH AN EC2 INSTANCE THAT WILL SERVE AS “WEB SERVER**

1. Launch an EC2 instance (RedHat) that will serve as "Web Server". Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.

2. Attach all three volumes one by one to your Web Server EC2 instance,check the availability zone .

3. Use lsblk command to inspect what block devices are attached to the server. Notice names of your newly created devices ,Use df -h command to see all mounts and free space on your server
![web serv](./webserver%20disk.PNG)


4. Use gdisk to create partition [sudo gdisk /dev/xvdf], type n ,choose 1 enter, chooese 8e00,command p,then w].use thesame for xvdg and xvdh.
5. check wth lsblk to confrim

![gdisk](./webserver%20disk.PNG)

6. Install logical volume packet manager [sudo yum install lvm2 -y]
7. Create physical volume [sudo pvcreate dev/xvdf1 dev/xvdg1 xvdh1], check with sudo pvs

![physical vol](./physical%20volume.PNG)

8. Create the  volume group [sudo vg-webdata /dev/xvdf1 /dev/xvdg1 /dev/xvdh1]. Run sudo vgs to check

![vol group](./volume%20group.PNG)

9. Create logical volume [sudo lcreate -n app -lv -L 14G vg-webdata, sudo lcreate -n logs -lv -L 14G vg-webdata] , check with sudo lvs

![log vol](./logical%20volume.PNG)

10. Format the logical volume  to ext4 file format [sudo mkfs.ext4 /dev/vg-webdata/app-lv, sudo mkfs/dev/vg-webdata/logs-lv]

11. Create www/html directory [sudo mkdir -p /var/www/html]

12. Create /home/recovery/logs to store backup of log data [sudo mkdir -p /home/recovery/logs]

13. Mount /var/www/html on apps-lv logical volume [sudo mount /dev/webdata-vg/apps-lv /var/www/html/

] , check  with ls -l /var/www/html

14. Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)
[sudo rsync -av /var/log/. /home/recovery/logs/]

![backup](./backup(webserver)%20log.PNG)

15. Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 14 above is very
important) ,mount volume group and log group to var/log directory [sudo mount /dev/webdata-vg/logs-lv /var/log]

16. Restore log files back into /var/log directory [sudo rsync -av /home/recovery/logs/. /var/log] ,to check sudo ls -l var/log

17. Update /etc/fstab file so that the mount configuration will persist after restart of the server. [sudo vi /etc/fstab]
The UUID of the device will be used to update the /etc/fstab file;
[sudo blkid]
Update /etc/fstab in this format using your own UUID and rememeber to remove the leading and ending quotes.

18. Insert mounts for Wordpress (write this as a comment)
Add var/www/html ext4 defaults 0 0
var/log ext4 defaults 0 0

![fstab](./fstab%20add.PNG)

20. Check with sudo mount -a (make sure it doesnt return any error)
21. Reload the system [sudo systemctl daemon-reload]. check with df -h

![ver](./set%20up%20complete.PNG)

**Step 2 — Prepare the Database Server**

1. On our Ec2 instance (RedHat) we create volumes from our amazon dashboard.name them accordingly db1,db2,db3,then we attach the volumes accordingly
2. Connect into our instance and type lsblk to check 
3. Create our physical disk compartmnet [sudo gdisk /dev/xvdf choose n,then 1,then 8e00(type of partition),then p ,then w],
we create thesame for [sudo gdisk /dev/xvdg and sudo gdisk /dev/xvdh]

![gdisk](./database%201(gdisk).PNG)

4. Update the repository [sudo yum -y update].
5.install logical volume manager package [sudo yum install lvm2 -y]
6. Create physical volume [sudo pvcreate /dev/xvdg1 /dev/xvdh1 dev/xvdg1]check with sudo pvs

![phys vol](./database%202(physical%20vol).PNG)

7. Create volume group [sudo vgcreate vg-database /dev/xvdf1 /dev/xvdg1 /dev/xvdh1] , check with sudo vgs

![vol grop](./database%203(vol%20group).PNG
)
8. Create Logical volume [sudo lvcreate -n db-lv -L 20G vg-database] check with sudo lvs

![log vol](./database4(logical%20vol).PNG)
9. Ceate /db on the root [sudo mkdir /db]

![db](./database4%20(make%20root%20directory%20and%20file%20).PNG)

10. Convert to ext4 [sudo mkfs.ext4 /dev/vg-database/db-lv], check with sudo ls -l /db  (it should be empty)

11. Type sudo blkid then copu the UID variable ,exclude the " and copy to a notepad

12. update fstab [sudo  vi /etc/fstab then paste it  like this UID = 3bcrt .. /db ext4 defaults 0 0]

![data](./database%206(paste%20uid).PNG)

13. Check with sudo mount -a (make sure it returns no error)

14. To restart the package [sudo systemctl daemon-reload],check with df -h  

![db5](./database5(mount%20on%20root%20db).PNG)


**Step 3 — Install WordPress on your Web Server EC2**
1. Update the repository [sudo yum -y update]

2. Install wget, Apache and it’s dependencies
[sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json]

3. Start Apache [sudo systemctl enable httpd
sudo systemctl start httpd]

4. To install PHP and it’s depemdencies

[sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1]

5. Restart Apache [sudo systemctl restart httpd]

6. Download wordpress and copy wordpress to var/www/html 
[ mkdir wordpress
  cd   wordpress
  sudo wget http://wordpress.org/latest.tar.gz
  sudo tar xzvf latest.tar.gz
  sudo rm -rf latest.tar.gz
  cp wordpress/wp-config-sample.php wordpress/wp-config.php
  cp -R wordpress /var/www/html/]

7. Configure SELinux Policies  
  [sudo chown -R apache:apache /var/www/html/wordpress
  sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
  sudo setsebool -P httpd_can_network_connect=1
  sudo setsbool -p httpd_can_connect_network_db1]

  **Step 4 — Install MySQL on your DB Server EC2**

1. sudo yum update

2. sudo yum install mysql-server-y

3. Verify that the service is up and running by using sudo systemctl status mysqld, if it is not running, restart the service and enable it so it will be running even after reboot:
  [sudo systemctl restart mysqld
  sudo systemctl enable mysqld]

  **Step 5 — Configure DB to work with WordPress**

1. on the database sever install [sud mysql_secure_installation]

2. sudo mysql , then enter passwors [sudo mysql -u root -p]

mysql > create database wordpress;
mysql> show databasees;

3. Set a User And Password (can replace % with private i.p of webserver)
> CREATE USER 'wordpress'@'%' IDENTIFIED WITH mysql_native_password BY 'wordpress';
> GRANT ALL PRIVILEGES ON *. *TO 'wordpress'@'%' WITH GRANT OPTION;
> FLUSH PRIVILEGES;

4. Set the bind address on the DATABASE SERVER [sudo vi /etc/my.cnf]
write the following [mysqld] bind address= 0.0.0.0 or private i.p of our webserver

5. Restart the serveice [sudo systemctl restart mysqld]

**Step 6 — Install MySQL on your WEB Server EC2**

1.sudo yum install mysql-server -y
2. start the mysqld on our webserver instance 
[sudo systemctl start mysqld]

3. Enable the service [sudo systemctl enable mysqld]

4. We need to configure our config.php file on our web server  [sudo vi wp-config.php]

5. Edit the DB_NAME,DB_USER,DB_PASSWORD,DB_HOST -> USE THE PRIVATE I.P OF THE DATABASE SERVER  OR 0.0.0.0

6. Restart the httpd (sudo systemctl restart httpd)

7. We need to disable the default apache page 
[sudo mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf/backup]

![apach](./apache%20confirmation.PNG)

8. Check if we can access our database server from our webserver 
[sudo mysql -h(ip address of database server) -u wordpress -p]

![mys](./mysql%20connect%20from%20webserver.PNG)


**Step 6 — Configure WordPress to connect to remote database.**
1. Open MySQL port 3306 on DB Server EC2. For extra security, you shall allow access to the DB server ONLY from your Web Server’s IP address, so in the Inbound Rule configuration specify source as /32.

2. Install MySQL client and test that you can connect from your Web Server to your DB server by using mysql-client
[sudo yum install mysql-client]
[sudo mysql -u admin -p -h <DB-Server-Private-IP-address>]

3. Verify if you can successfully execute SHOW DATABASES; command and see a list of existing databases.

4. Change permissions and configuration so Apache could use WordPress:
[sudo chown -R apache:apache /var/www/html/wordpress
  sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
  sudo setsebool -P httpd_can_network_connect=1
  sudo setsbool -p  httpd_can_connect_network_db1]

  5. Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation’s IP)

  6. Try to access from your browser the link to your WordPress http://<Web-Server-Public-IP-Address>/wordpress/

![word](./wordpress%20succesfuly%20installed.PNG)

![word2](./wordpress%20sucess2.PNG)

![word3](./wordpress%20success3.PNG)


  7. CONGRATULATIONS!
    we successfully configured Linux storage susbystem and have also deployed a full-scale Web Solution using WordPress CMS and MySQL RDBMS!


