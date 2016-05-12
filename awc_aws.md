First, lets go over some common services/abbreviations that we use with AWS:

* **EC2** (Elastic Compute Cloud) - Instances, the virtual machines that run our clients' sites
* **RDS** (Relational Database Service) - Virtual machines that only host database services, in our case, MySQL.
* **S3** (Simple Storage Service) - A really big harddrive.  Used to store assets, uploads, anything content that is not in database or code base.
* **Route 53** - DNS service
* **SES** (Simple Email Service) - AWS's email service, currently not in use.
* **IAM** (Identity & Access Management) - User accounts, privileges, and keys.  
* **VPC** (Virtual Private Cloud) - Essentially VPNs that contain all of our instances, used primarily for sercurity.  We currently only have one setup, but may set one up per client in the future.
* **Cloudwatch** - Service that monitors logs and sends alerts based on set metrics
* **Elastic IPs** - A static IP address assigned to an instance.
* **Security Groups** - A dumb down firewall that allows us to open ports for incoming/ougoing traffic
* **User** - Users are accounts that access AWS services. These are AWC employees, clients, services that use the AWS API, etc
* **Roles** - Similar to a user, we apply a role to an EC2, and then assign privileges and whatnot 

# EC2
Lets create a single EC2 instance for a client that will host multiple domains/projects.

## Create New EC2 instance
* Log into AWS, click on EC2.  Click on **Launch Instance**
* Step 1: Select the Ubuntu Server 14.04 64bit AMI
* Step 2: Select an Instance type.  The t2.micro will be plenty for most small clients. Click on **Next: Configure Instance Details**
* Step 3: Only items that require attention are: Network - use **AWC**, Subnet - select **us-east-1b**, IAM role - select **AWC**
Shutdown behavior - be sure **Stop** is selected.  Click on **Next: Add Storage**.
* Step 4: By default, 8gb will be setup as the partition, which should be enough for all clients, but adjust if needed.
* Step 5: Add a tag for this instance - Key: **client**, Value: **Client abbreviation**. Click **Next: Configure Security Group**
* Step 6: Select the client's security group if it already exists.  If one doesn't, we need to create a new security group. **Click Review and Launch**.
* Step 7: If all looks correct, click on **Launch**.
* Key pairs: If one exists already for client, select that one.  If not, select Create a new key pair, and name it the client's abbreviation.  Download the Key pair and store it Sparrow.  Do note that this is the only time you can download this key.  If it is lost, the instance is trashed. Click on **Launch Instances**.
* The instance we just created will take a few minutes to launch.  There's a couple things to do in the meantime: Go to the Instances dashboard.  Find the instance you just launch, and rename it the client abbreviation followed by an appended number (ie: AWC-001)
* Assign an Elastic IP to your new instance.  Click on Elastic IPs from the left menu.  Click on **Allowcate New Address**, then **Yes**.  Select the new address in the list once its assigned, and click **Associate Address**.  Find the instance you just created and click **Associate**.  This will be the public IP address of this instance.
* By now, your instance should be done.  You can view it's progress by clicking on Instances on the left.  When it's Status Checks says 2/2 checks passed, you're ready to login.

### Setup EC2 instance
* Log into the instance by using the key provided by AWS upon launch and it's public IP address:
`ssh -i key.pem ubuntu@1.2.3.4`
You may have to chmod the key to 600 if ssh from the terminal:
`chmod 600 key.pem`
* Install git:
`sudo apt-get install git`
* Clone the **system** repo:
`git clone https://deployallwebcafe:vB75U9UB@bitbucket.org/allwebcafe/awc_system system`
* Run the LAMP setup script:
`sudo ./system/lamp_ubuntu_14.sh`
You will be asked for a MySQL root user/pass:  
**MySQL:**  Use the client's user/pass that is documented in Sparrow (Most likely, MySQL server will not be used on this instance, but needs to be installed to connect to the RDS db).   
**Postfix Configuration:** Select Internet Site, use defualt EC2 name (should already be populated).
* The script will take a few minutes to run.  In the end, it will ask for a reboot. The reboot should only take a minute.  SSH back in.


## New Client Site
Lets create a new site on an EC2 instance!

* Log into the EC2 instance using it's key and public IP:
`ssh -i key.pem ubuntu@1.2.3.4`
* If the client user does not exist yet, run the new_client.sh script:
`sudo ./system/new_client.sh`
Username/Password should match what's in Sparrow. 
* Run the new_site script:
`sudo ./system/new_site_apache.sh`
* Restart Apache. To test, you can add an entry to your hostfile, pointing the domain to the EC2 ipaddress.


## Staging & Deploying
Our staging server is Awcbits. We can setup and stage sites manually, using FTP, or setup auto staging and push button deployment.

### New Stage User
* Log into Awcbits using AWC's key:
`ssh -i awc.pem ubuntu@54.165.92.162`
* Run the new_client script:
`sudo ./system/new_client.sh`
Be sure the user matches the user on the live host

### New Stage Site
* Log into Awcbits using AWC's key:
`ssh -i awc.pem ubuntu@54.165.92.162`
* Run the new_site_apache script to create a new client:
`sudo ./system/new_site_apache.sh`
The site will be something like **site.awcbits.com** This script will ask if you'd like to setup staging and deploying.  If you setup deploying, be sure to have a live EC2 running with the client's user setup.

##Network Interfaces

Elastic Network Interfaces (ENI): You can think of these as virtual network cards. We can add these on the fly to any EC2 instance.

For now, the only reason we may need to add another ENI or private IP is to add a second or more SSL to the EC2.

If you have maxed out the number of IPs allowed on avaliable ENIs, youll have to create and attach another.  By default, every EC2 will have one ENI: eth0.

###Private IP Addresses

Private IP addresses are assigned inside a VPC. These addresses are used internally to communicate to AWS routings.  

**Note:** You can put multiple private IPs on a single ENI, but there are different limits on ENIs and how many private IPs can be used per instance depending on the size of the instance.  You can get a general limit here: http://technochrati.com/ec2-network-interface-number/

###Create ENI

* On the EC2 Dashboard, click on **Network Interfaces**, then **Create Network Interfaces**
* Select the subnet **us-east-1b**
* Select the appropiate security group for the EC2 you are attaching this to. Click **Yes, Create**

###Attach ENI

* Click on **Instances**, select on the instance, **Actions**, **Networking**, **Attach Network Interface**
* Select the interface you just created, then **Attach**

###Add a Private IP

* Select the instance, then **Actions**, **Networking**, **Manage Private IP Addresses**.
* Under the network inteface, click on **Assign new IP**, then **Yes, Update**. This will refresh the model with a Private IP in place. You will need this IP later in the SSL/Apache config.
* You can now setup an Elastic IP to this Private IP.  You'll need to know the ENI ID, then go to **Elastic IPs**, **Allocate New Address**, then **Associate Address**.  Paste or find the ENI ID in **Network Interface**, then select the private IP address you created, and **Associate**

**Note:** These private IPs are dynamic.  If the system restarts, and the EC2 instance is not configured to set the IP address statically, they will change.

###Network/Apache Conf
We have to setup additional private IPs as static in the EC2's network config file.

**Note: Be careful editing the network config.  You can very easily brick the machine**

* SSH into the EC2 instance, and open **/etc/network/interfaces**. By default, the file should look like this (comments removed):
`
auto lo
iface lo inet loopback  
source /etc/network/interfaces.d/*.cfg
`

* Add the following to add a static private IP to a ENI:
`
iface eth0 inet static  
	address 172.31.53.147/20
`
eth0 will be the device, and 172.31.53.147/20 will be the private IP/mask that AWS gave you.  Use the same mask that the ENI has; it is the /number after the IP address.
* You can now either reboot the machine, or manually add the IP to the current running network config.  Replace the IP and eth0 if needed:
`
ip addr add 172.31.53.147/20 dev eth0
`
* To verify, run the following command and you can see the current IPs associated with the device:
`
ip addr list dev eth0
`

An example **/etc/network/interfaces** file with multiple private IPs assigned to multiple ENIs:

    auto lo  
    iface lo inet loopback  

    iface eth0 inet static  
            address 172.31.49.87/20  
    iface eth0 inet static  
            address 172.31.49.88/20  

    iface eth1 inet static  
            address 172.31.49.89/20  
    iface eth1 inet static  
            address 172.31.49.90/20  

    source /etc/network/interfaces.d/*.cfg


##SSLs
### Apache conf
To setup an SSL in Apache, you'll have to add the following to the site's .conf file:

    <IfModule mod_ssl.c>
        <VirtualHost IPADDRESS:443>
            DocumentRoot /home/USER/sites/DOMAIN
            <Directory /home/USER/sites/DOMAIN>
                Allowoverride ALL
                Options +Indexes
            </Directory>
            ServerName DOMAIN:443

            <IfModule mpm_itk_module>
                AssignUserId USER USER
            </IfModule>

            ErrorLog /var/log/apache2/DOMAIN/error.log
            CustomLog /var/log/apache2/DOMAIN/access.log common

            SSLEngine on
            SSLCertificateFile /etc/apache2/ssl/CLIENTABBR/2015-SITENAME.crt
            SSLCertificateKeyFile /etc/apache2/ssl/CLIENTABBR/2015-SITENAME.key
            SSLCertificateChainFile /etc/apache2/ssl/CLIENTABBR/2015-SITENAME-ca.crt

        </VirtualHost>
    </IfModule>

Replace the following:  
**IPADDRESS** - Internal address that the Elastic IP is attached to (ex: 172.31.49.87)  
**USER** - Client's username (ex: awcuser)  
**CLIENTABBR** - Client's abbreviation (ex: awc)  
**DOMAIN** - The site's domain.  Include the subdomain if the SSL includes the subdomain (ex: allwebcafe.com)  
**SITENAME** - Name of the site (ex: allwebcafe). Be sure to change the year to the year the cert expires.

If this is not the only site with a SSL on this EC2, you'll have to add the IPADDRESS to the :80 VirtualHost config for this site (in the same conf file that you added the IfModule).

### IPs
For each SSL on an EC2 instance, we need a unique private IP and elastic IP. **If this EC2 will only have one site with an SSL**, more IPs/Interfaces will not be needed.

You will need to create a ENI and/or add a private IP to an ENI, and use that private IP in the site's apache config:

* Edit the site you are configuring the SSL for.  Here, the Virutal Hosts are setup to accept all IPs on port 80 and 443, like so:
`
<VirtualHost *:80>  
<VirtualHost *:443>
`
Replace the * with the private IP address:
`
<VirtualHost 172.31.53.147:80>  
<VirtualHost 172.31.53.147:443>
`
* Save the sites config file, run the apache config test to ensure no errors:
`apachectl configtest`
* Restart Apache
`service apache2 restart`
* You can quickly test the IP and SSL setup by adding the new public/Elastic IP to your host file



# Security Groups
Every EC2 and RDS instance must be associated with a security group.  Each client will have it's own security group.  By default, each security group will look something like this:

(IMG)

An RDS instance can only be accessed from within it's security group.  

## Create a New Security Group
You can either create a new group manually from the EC2 service page, or on Step 5 of Creating a new EC2 instance.  Skip the first bullet if you are doing the latter:

* Go to EC2 service page, click on Security Groups on the left menu, then click on **Create Security Group**
* Security group name will be the client abbr, ex: AWC. Description: 'AWCs group'. VPC is the client's VPC, for now, AWC.  
* Add the following inbound rules: (SSH, TCP, 22, Anywhere 0.0.0.0/0), (HTTP, TCP, 80, Anywhere 0.0.0.0/0), (HTTPS, TCP, 443, Anywhere 0.0.0.0/0). Click Save.
* Find the new group, and edit the name to the client's abbr.
* We need to add one more rule for MySQL, but we need to know the new security group's ID.  Now that it's created, copy the ID, click on the **Inbound** tab and click **Edit**.  Add Rule, and add: (MYSQL, TCP, 3306, Custom IP sg-xxxxxxxx). 

You now have a new security group to apply to all of the client's EC2 and RDS instances.


# RDS
We use AWS's RDS service to create database instances for MySQL.  All RDS instances have a login **studio** with root privileges (found in AWS). Each client has their own user with only SELECT, INSERT, UPDATE, and DELETE privileges.

All RDS instances also has a **rdsadmin** user associated with it.  This is AWS's account used for updates/mainenance/etc and should not be touched.

## Create a RDS instance
* Go to the RDS service page, click **Launch a DB Instance**
* Select MySQL
* Select No, we will not be using Multi AZ deployment, click **Next Step**
* Specify DB Details: **DB Instance Class** - Select the appropriate instance size. Most likely, db.t2.micro will be enough. **Multi-AZ Deployment** - No.  **Allocated Storage** - DB size, adjust accordingly.
* Settings: **DB Instance Identifier** - Name of the instance, client's abbr with an appended '-db' (ex: 'awc-db').  **Master Username and Password** - Root user/pass, use **studio** credentials found in Lockdown. Click **Next Step**
* Network & Security: **VPC** - Default VPC, **Publicly Accessible** - No, **Availability Zone** - us-east-1b, **VPC Security Group** - Select the clients security group
* Database Options - You can leave this blank.  There is a script in system that will create databases and give appropriate privileges for clients.
* Backup - Set Backup Window to 7:00 UTC (2am EST).
* Maintenance - Set Maintenance Window to 8:00 UTC (3am EST). Click **Launch DB Instance**



## Create a Client DB
* Login to AWCbits if setting up a stage DB, or client's EC2 instance:
`ssh -i awc.key ubuntu@54.88.4.11`
* Run the new_db script:
`sudo ./system/new_db.sh`
You can get the DB host url from RDS -> Instances, clicking on the instance, and copying the Endpoint url
* All databases on **awc-db** and **awcbits-db** should be prefixed with the client's abbr (ex: awc_blog).
* All databases should have a user named as the database that has select/update/create/delete privileges.  (ex: awc_blog will be setup with user awc_blog).


## Backup
### Snapshots
AWS will keep 7 days (or whatever was setup) worth of RDS snaphots, just in case the RDS server goes up in flames.

### Hourly Cron Script
Each live database has a script running that backs up each database to S3 every hour. Anyone who has a database on **awc-db** will automatically be setup to backup. In the case of **awc-db**, this script is setup on AWC-001.

On S3, all db backups are stored in **awc-dbbackups**. In this bucket, each RDS server has its own folder, followed by the database name, then the gz of the sql dump.  You can download a copy of a sql dump by browsing through S3.

Any client with their own RDS server will need to be setup. To setup this script on a EC2 instance for a new RDS server:

* In S3, create a new folder for the database in the awc-dbbackups bucket.
* Determine a primary machine that will run the backup script.  If the client is in an auto scale group, be sure to use the primary instance and not include the cronjob in the AMI
* As ubuntu, issue these commands:
`sudo mkdir -p /opt/dbbackups/RDSNAME  
sudo cp ~/system/cron/dbbackup.sh /opt/dbbackups  
sudo chmod +x /opt/dbbackups/dbbackup.sh  
sudo chmod 600 /opt/dbbackups`

* Mount the S3 bucket/folder to the /opt/dbbackups/RDSNAME folder by adding a line in /etc/fstab:
`s3fs#awc-dbbackups:/RDSNAME /opt/dbbackups/RDSNAME fuse rw,_netdev,allow_other,use_cache=/s3fscache 0 0`
* Mount the folder:
`sudo mount -a`
* Add the following line to root's crontab:
`sudo crontab -e  
0 * * * * /opt/dbbackups/dbbackup.sh > /dev/null 2>&1`



# IAM
IAM is AWS's user management service.  Each client will have it's own user, and that user will need access to whatever server they maybe using.

## Roles
If the AWS account does not have a role, you need to create one.  This will give EC2's access to S3 and Cloudwatch services
* Find the IAM/Identity & Access Management page, click on **Roles** on the left menu  
* Click on **Create New Role**  
* Name your role.  Most like, if this is the accounts first role, you're probably creating it for most EC2's/general purpose.  You can name it whatever, but usually it will be the owner of the account, ex: 'awc', 'jsp', 'brandshare'. Click **Next Step**  
* Click **Select** for Amazon EC2.  
* Find and select the following: **AmazonS3FullAccess**, **CloudWatchFullAccess**.  Click **Next Step**  
* Click **Create Role**  

## Create Client User
* Find the IAM/Identity & Access Management page.
* Click on Users on the left menu, click on Create New Users
* Enter the client's abbr, leave Generate an access key checked, and click Create
* For each user you just created, AWS will list Access Key IDs and Secret keys.  Store these in Sparrow.

## AWC Employees
Every AWC employee that needs access to AWS should have their own user account. Lets create one:

* Find the IAM/Identity & Access Management page.
* Click on Users on the left menu, click on **Create New Users**
* Fill out all the users you want to create (Stick with first initial, last name like our logins for everything else), uncheck Generate an access key, click **Create**
* Click on the new user, scroll down to **Sign-In Credentials**, click **Manage Password**.  Add their password, click **Apply**

A group called **AWC-Devs** exist with admin privileges.  An account for each Allwebcafe developer is created and added to this group.

* Click on **Add User to Groups** and add them to **AWC-Devs**

Login page for users: https://720318307502.signin.aws.amazon.com/console  
*NOTE: When logging in for the first time, be sure you are in region **U.S. East**.  Change this by clicking on the Region dropdown at the top of the page.

# S3
Each client will have it's own bucket.  In our case, a bucket is a container with all of the client's domains and folders/mount points.

## Creating a bucket
* Go to the S3 service page.  Click on Create Bucket
* Bucket Name: awc-Client's abbreviation (ex: awc-jsp), Region: US Standard
* Open the bucket, create a folder for the client's domain (ex: awc-jsp/epp.com)
* In the domain folder, you can create any other necessary folder you need, such as uploads, assets, logs, etc.  These will be used to mount to the EC2 instance

## Mounting
Let's mount a folder on S3 to a folder on EC2:

* SSH to client's EC2 instance
* **If setting up a mount for the first time on this instance:** Add s3fs access and secret keys to /etc/passwd-s3fs by running the following command:
`echo AWS_ACCESS_KEY_ID:AWS_SECRET_ACCESS_KEY > /etc/passwd-s3fs`
Be sure to replace AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY with s3fs's keys found in Sparrow.
* Add the following to **/etc/fstab**:
`s3fs#BUCKETNAME:/FOLDER ~/sites/site.com/uploads fuse    rw,_netdev,allow_other,use_cache=/s3fscache 0 0`
Replace BUCKETNAME and FOLDER with what you did in S3.  Replace the uploads directory with the directory you want pointing the mount to.  
* To mount the drive, run:
`sudo mount -a`

# Cloudwatch
Cloudwatch is the monitoring service we use.  It is broken down to a couple parts:

## Logging
Each EC2 instance is setup to send all apache, mail, and syslogs to Cloudwatch. This means we can monitor these errors without logging into the EC2 machines, and set up alarms for specific errors.

## Alarms
Cloudwatch can send out emails based one certain metrics on EC2 instances and error messages.  Each EC2 instance should have the following being monitored: Disk space usage, Memory usage, CPU usage.  

**CPU Usage Percentage**  

* On the EC2 dashboard, select the EC2 instance->Monitoring->Create Alarm
* Send a notification to: studio-alerts (studio@allwebcafe.com)
* Whenever: **Average** of **CPU Utilization** Is **>= 50**  Percent For at least **1** consecutive periods of **5 minutes**.
* Name of alarm: (Name of instance)-CPU-Utilization (ex: AWCbits-CPU-Utilization)
* Finish with **Create Alarm**

**Disk Space**  

* On the Cloudwatch Dashboard, click on Linux System on the left, then select the Instance with the Metric Name DiskSpaceUtilization, click on Create Alarm
* Name: (Instance name)-DiskSpaceUtilization (ex: AWCbits-DiskSpaceUtilization)
* Description: (Instance Name) disk space used percentage
* Whenever DiskSpaceUtilization is **>= 75** for **2** consecutive periods (Why 2 and not 1?  Just in case we are doing backups and are moving a bunch of files around)
* Whenever this alarm: **State is ALARM** Send notification to: **studio-alerts** Email list: **studio@allwebcafe.com** (Should already be populated)
* Click Create Alarm

**Memory Usage**  

* On the Cloudwatch Dashboard, click on Linux System on the left, then select the Instance with the Metric Name MemoryUtilization, click on Create Alarm
* Name: (Instance name)-MemoryUtilization (ex: AWCbits-MemoryUtilization)
* Description: (Instance Name) memory used percentage
* Whenever MemoryUtilization is **>= 75** for **1** consecutive periods
* Whenever this alarm: **State is ALARM** Send notification to: **studio-alerts** Email list: **studio@allwebcafe.com** (Should already be populated)
* Period: **1 Minute**
* Click Create Alarm

#ElastiCache
We can setup Redis or Memcache cache clusters.

To create a new cluster:

* In the AWS web console, find the ElastiCache Management dash.
* Click on **Launch Cache Cluster**
* Select the Redis engine, click **Next**
* Uncheck **Enable Replication**
* Enter **Cluster Name** (Client Abbr-cache ex: 'awc-cache')
* Select appropriate **Node Type** for the client (t2.small or t2.micro will be enough for most clients). Click **Next**.
* Select the **Availability Zone**: us-east-1b. Select the right security group for the cluster/client (most likely the AWC group).
* Select an appropriate **Maintenance Window** for the client (ex: 2am Sat nights). Disable SNS Notification, click **Next**.
* Verify all the settings, click **Launch Cache Cluster**.

The cluster may take a few minutes to finish.  When done, it will create a single node. Selecting that node will give you it's endpoint that you can now use in the client's config settings.


#CloudFront CDN
Cloudfront is a CDN service that uses an S3 bucket to serve static files.

By default, the domain used to serve static assets is cloudfront.net.  This comes with HTTPS.  We can use our own subdomain, but we will have to supply a SSL.  

Before we create a distribution for a bucket, we must give public read access to it:

* Find the S3 Console, click on the bucket, click on Properties
* Select **Permissions**, click **Add bucket policy**.
* Enter the following, replace **BUCKETNAME** with the actual bucket name:

        {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Sid": "MakeItPublic",
                    "Effect": "Allow",
                    "Principal": "*",
                    "Action": "s3:GetObject",
                    "Resource": "arn:aws:s3:::BUCKETNAME/*"
                }
            ]
        }

* Click **Save**

To create a distribution for a S3 bucket:

* In the AWS web console, click on **Services**, find **CloudFront**.
* Click on **Create Distribution**
* **Origin Domain Name**: Select the bucket you're setting up.
* **Price Class**: Use Only US and Europe.  **Alternate Domain Names**: Enter any sub domain you want to use for the client (optional)
* Scan the rest of the settings.  Most defaults should work with all clients.  When done, click **Create Distribution**

Give it a few minutes, eventually the distribution will complete.  Click on it and you will find the domain given, something like: d1c7rmnlfdcwa.cloudfront.net.  You can now use this, both http and https, for a project.


#Auto Scaling
We can create an Auto Scaling group that will fire up as many instances as we want based on CPU usage a webserver maybe enduring, and then destroy instances down to a minimum number once CPU usage is back to normal.

What we need to create an autoscaling group:  
1. A running EC2  
2. An AMI  
3. A Load Balancer  

## AMI

* Follow these (https://sparrow.allwebcafe.com/procedures/details/amazon-web-services/#toc1) directions (Create New, Setup, New Client Site) to create a basic EC2 instance and client setup.  Be sure to setup all sites necessary for this client/group.
* Deploy as normal - Ensure all necessary config files are in the client's config directory, awcbits is setup to deploy to this machine.
* Setup all S3 mounts as normal

Autoscaling works by creating new instances based off an AMI.  In an AMI, we need to ensure that it has the most up-to-date code.  We can't modify the code in an AMI, so we setup a git repo and rsync changes to the live web directory.

* cd into the /opt directory
* Do a git clone from the live git repo using the deploy user:
`sudo git clone https://deployallwebcafe:vB75U9UB@bitbucket.org/allwebcafe/REPO-NAME .`
* Give the repo directory ownership to the client user:
`sudo chown -R CLIENT:CLIENT REPO-NAME`
* Create a script in this directory (/opt/update.sh) that will update the git repo and rsync to the live web directory. You will need to exclude all config files (database, config, htaccess files, etc) and S3 mounts in the rsync file, similarly to the exclude files in deploy:
`git -C /opt/REPO-NAME pull
rsync -a --delete --exclude ".git*" --exclude "application/config/database.php" /opt/REPO-NAME/ /home/CLIENT/sites/CLIENTSITE/  
service apache2 start`
* Give this file execution permissions:
`sudo chmod +x /opt/update.sh`
* Add update.sh to crontab:
`sudo crontab -e`
`@reboot /opt/update.sh`
* Disable Apache on bootup.  We will start Apache when the web directory is all up to date:
`sudo update-rc.d -f apache2 disable`

Setup for the image is done.  Create the AMI by going to AWS Web Console, EC2 Dashboard, selecting the EC2 instance, click on Actions->Create Image. Name it the clients abbr, description is 'Abbr image', click **Create Image**.

## Load Balancer

* On the EC2 service page, click on Load Balancers on the left menu, then Create Load Balancer.
* Load Balancer name: Client's Abbr-LB 'ex: AWC-LB'. Add a HTTPS Load Balancer Protocol if this client uses SSL. Click Continue.
* If you added HTTPS, you will be asked to choose or upload an existing SSL.
* Under Configure Health Check, select Ping Protocol to TCP, Healthy Threshold: 2. Click Continue.
* Select existing Security Group.  If on the AWC account/using the AWC DB, you can select AWC.  Click Continue.
* Select the running EC2 instance you used to create the AMI.  Click Continue.
* Add the client key with the clients abbr.  Click Continue.
* If all looks well, click Create.
* Once created, AWS will give you a DNS name to use.  This will need to be setup in the domain's DNS as an A record.

## Auto Scaling Group

### Launch Configuration
We need to create a launch configuration if the client currently does not have one.  This will setup the AMI with EC2 settings

* On the EC2 service page, click on Auto Scaling Groups on the left menu, then Create Auto Scaling group, Create launch configuration
* Click on My AMIs on the left, select the AMI you created.
* Select the appropriate type of instance.  t2.micro or t2.small will be fine for most of our clients. Click on Next.
* Name: Client's Abbr-launch (ex: AWC-launch). Select client's IAM role (or AWC for default). Click Next.
* Adjust storage is needed, otherwise click Next.
* Select client's security group if they have one, otherwise select AWC. Click Review.
* If all looks well, click Create. Select the client's key pair, click Create again.

### Auto Scaling Group
* Group name: Client's Abbr-group (ex: AWC-group). Subnet: us-east-1b. Under Advanced Details, check 'Receive traffic from Elastic Load Balancer', and select the client's Load Balancer. Click Next.

Select Use scaling policies to adjust the capacity of this group.  These may be adjusted per client basis, but here is an example setup:

* Scale between **1** and **3** instances 
* Add an Increase Group Size.  Create a new alarm:
(Insert image here)
* Take the action: Add 1 instances, And then wait 300 seconds
* Add a Decrease Group Size. Create a new alarm:
(Insert image here)
* Take the action: Remove 1 instances, And then wait 300 seconds. Click Next.
* Send a notification to: studio-alerts (studio@allwebcafe.com). Click Next.
* Add a client key, use the client's abbr. Click Review.
* If all looks well, click Create.																																																												