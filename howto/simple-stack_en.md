# Creating JIN-KEI Simple Stack
![Architecture of JIN-KEI Simple Stack](./img/stack.png "Architecture of JIN-KEI Simple Stack")
Architecture

## With JIN-KEI, you can achieve:
- Amazon RDS leads to server fault tolerance;
- CloudFront leads to high-speed contents delivery;
- Amazon S3 leads to a cost reduction for media storage;

## Some prior arrangement
1. Set up EC2 instance with AMIMOTO AMI
2. Set up AWC-CLI on your PC/Mac
3. Configure AWC-CLI to control CloundFront from command line

## Commands
All of command we will use in this hands-on are listed on http://bit.ly/1Xio3cC

### Launch EC2 instance with AMIMOTO AMI
- On this hands-on, we'll migrate databases, so make sure you're working on develop environment.
- WooCommerce Powered by AMIMOTO (Apache HTTPD PHP7) on AWS Marketplace : https://aws.amazon.com/marketplace/pp/B01DAONMCK/
- AMI fee is free for 14 days (AWS infrastructure charges still apply);


Note: Try one instance of this product for 14 days. There will be no hourly software charges for that instance, but AWS infrastructure charges still apply. Free Trials will automatically convert to a paid hourly subscription upon expiration. Note that Free Trials are only applicable for hourly subscriptions, but you can opt to purchase an annual subscription at any time.


### Set up AWS CLI
Two ways to install AWS CLI: 

- For Mac user install it through package manager Homebrew:
http://brew.sh/index.html
- Following AWS User guide:
http://docs.aws.amazon.com/cli/latest/userguide/installing.html

#### Mac: Setting up with Homebrew
Copy below command then paste to Terminal.app
1. `/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`
2. `brew install wget`
3. `brew install awscli`

#### Windows: Download installer and use it
- Download the AWS CLI MSI installer for Windows (64-bit):https://s3.amazonaws.com/aws-cli/AWSCLI64.msi
- Download the AWS CLI MSI installer for Windows (32-bit):https://s3.amazonaws.com/aws-cli/AWSCLI32.msi

By default, files are extract into following directories:
- `C:\Program Files\Amazon\AWSCLI (64-bit)`
- `C:\Program Files (x86)\Amazon\AWSCLI (32-bit)`

### Set up IAM for AWS CLI
You need credentials to use AWS CLI, so create and get it from AWS console using IAM (AWS Identity and Access Management).

#### Open IAM page on AWS Console
https://console.aws.amazon.com/iam/home#home

#### Create a user
1. Click **[Users]** on left menu;
2. Click **[Create New Users]** to launch wizard;
3. Input **[amimoto-cli]** into **[Enter User Names]**, to define the user is for awscli;
4. Check **[Generate an access key for each user]**;
5. Create it;

#### Save credentials to your PC
Make sure you copied credentials into your PC (csv file), you will not able to get them again if you closed the window or tab.
If you lost them create another user and get another credentials.
- Access Key ID
- Secret Access Key


#### Initial set ups for AWS CLI
```
aws configure --profile awscli
```

##### Values
Input the following values in the interactive type:
- Credentials
- Default Region
- Output formant

```
AWS Access Key ID [None]: xxxxxxxxxx
AWS Secret Access Key [None]: xxxxxxxxxx
Default region name [None]: ap-northeast-1
Default output format [None]: json
```

### Some set ups to conduct CloudFront through AWC CLI
By default, AWS CLI is disabled the feature to control CloudFront . We'll enable it.
```
$ aws --profile amimoto-cli configure set preview.cloudfront true
## operation check 
$ aws --profile amimoto-cli cloudfront help

```

## Add CloudFront to AMIMOTO
Benefits to use CloudFront are: 
- You can use CDN server on around the world hosted by AWS;
- CDN reduces server loads and distribute contents high data transfer speeds;
- Server fault tolerance with customisable response error message;

### Commands for setting up
- Replace "ORIGIN URL" to the server domain name (or Public DNS) of AMIMOTO;
- Change **amimoto-cli** to your created profile, if necessary;

```
$ export origin_url='{ORIGIN URL}'; aws --profile amimoto-cli cloudfront create-distribution --cli-input-json "$(curl -l -s https://raw.githubusercontent.com/amimoto-ami/create-cf-dist-settings/master/source_dist_setting.sh?a | sh)"
```

#### Cannot find Public DNS on you EC2?
See
http://qiita.com/kasokai/items/4ea689ce9f206e78a523

### Wait for starting CloudFront
It takes 20-30 minutes to start up the CloudFront.
Set up S3 and RDS while CloudFront is progress on its starts.


## Add S3 to AMIMOTO
### Benefits to use S3
- S3 is the extremely low cost media storages;
- S3 has redundancy to improve server fault tolerance;
- S3 has no limits for number of files or sizes;

### Set up S3
1. Create IAM user
2. Create bucket
3. Configure bucket
4. Set up WordPress plugin

#### Create IAM user
##### Create IAM user
1. Click **[IAM]** in AWS Console;
2. Click **[Users]** on the left menu;
3. Click **[Create New Users]** to launch wizard;
4. Input **[amimoto-3]** into **[Enter User Names]**, to define the user is for S3;
5. Check **[Generate an access key for each user]**;
6. Create it;

##### Attach policy to created IAM user
1. Click **[Users]** on the left menu;
2. Click **[amimoto-3]**;
3. Click **[Permissions]**;
4. Click **[Attach Policy]** in **[Managed Policies]** area;
5. Choose **[AmazonS3FullAccess]** then click **[Attach Policy]**;


#### Create bucket
1. Click **[S3]** on AWS Console;
2. Click **[Create Bucket]**;
3. Enter specify name into **[Bucket Name]**;
note: Bucket name must be unique
4. Choose **[Region]**;
5. Click **[Create]** to create bucket;

#### Bucket settings
1. Choose bucket name which you created in the S3 list;
2. Click **[Properties]** on the right top of the window;
3. Click **[Static Website Hosting]**
4. Choose **[Enable website hosting]**
5. Enter **[index.html]** to **[Index Document]**
6. Click **[Save]**



## Add RDS to AMIMOTO
Benefits to use RDS are:
- You can use DB server with easy to set up and manage;
- Single click to replicate DB or changing its specifications;
- Various DB engines like MySQL/MariaDB/Amazon Aurora;

### Set up RDS
1. Access AWS Console
2. Click RDS icon
3. Click **[Launch DB Instance]** on **[Instance]**
4. Choose DB engine (MariaDB is recommended)
5. Amazon recommends its Aurora, a little expensive to use, you may ignore it;

### Initial set ups for RDS
You can set up on **[Specify DB Details]** page

#### Values of [Settings] column

| Item | Value |
| :-- | :-- |
| DB Instance Identifier | Name of DB instance |
| Master Username | Root user name of DB |
| Master Password | Root password of root user |
| Confirm Passoword | Confirm root password |

These are required values to connect the DB, you should save them all as text file.
You don't need to change other items, on this hands-on.

#### Set DB name on **[Configure Advanced Settings]**
Copy the value in [Database Name] which will be  sated as DataBase name.

##### Sample command to connect RDS
```
$ mysql -u {Master Username} -p {Master Password} {Database Name}
```

#### Create RDS
It takes a little time, you may drink some cup of coffee.

When you click **[Instance]**, you will find **[DB Instance]** same name as **[DB Instance Identifier]**.
Check the status column. If it is displayed as **[Availalble]**, you succeeded RDS creation.

#### Configure Security Group 
1. Let's configure security group
2. Click **[Edit Security Group]**
3. Click **[Inbound]** tab
4. Click **[Add Rule]**
5. Set **[Type]** as **[MySQL/Aurora]**
6. Choose **[Anywhere]** in **[Source]**
++ note: you can set AMIMOTO AMI's IP Address (xxx.xxxx.xxx.xxx/0) to **[Source]**.
7. Click **[Save]**


### Edit config file to connect RDS
#### SSH to AMIMOTO instance
Connect to your AMIMOTO instance through SSH
```
$ ssh -i /path/to/pem/{PEMFILENAME}.pem ec2-user@{INSTANCE_IP}
```

#### Edit local-config.php
```
$ cd /var/www/vhosts/{INSTANCE_ID}
$ vim local-config.php
```

#### you should edit file:
local-config.php (Line: 13-16)
```
if ( !$db_data ) {
         $db_data = array(
                'database' => '{Database Name}',
                'username' => '{Master Username}',
                'password' => '{Master Password}',
                'host'     => '{RDS_ENDPOINT}',
        );
}
```

##### If you're Vim user: 
| Command | mode |
| :-- | :--|
| [esc] | enter nomal mode|
| :set number | show line numbers |
| [shift]+[z]+[z] | save and exit |
| i | enter edit mode |

#### Check RDS connections
Settings and connections to RDS are succeeded, when you access your AMIMOTO site and got WordPress installation page.

#### Stop MySQL in EC2
```
$ vim /opt/local/amimoto.json
```
Add `["mysql": { "enabled": false },]` line.

##### Before the changes:
```
{
  "mod_php7" : { "enabled": true },
  "run_list" : [ "recipe[amimoto]" ]
}
```

##### After the changes:
```
{
  "mod_php7" : { "enabled": true },
  "mysql": { "enabled": false },
  "run_list" : [ "recipe[amimoto]" ]
}
```

##### Run provision command
```
$ sudo /opt/local/provision
$ sudo service mysql stop
```

#### Install plugin to WordPress
- Login to WordPress' Dashboard;
- Enable Nephila Clavata pluign on the plugin page;
- Fill in S3 settings to **[Nephila Clavata]** on **[Setting]**;

| Item Name | Value |
| :-- | :--|
| AWS Access Key | AWS Access key of your s3amimoto |
| AWS Secret Key | s3amimotoのAWS Secret Key |
| AWS Region | Choose specify region which you created S3 bucket on |
| S3 Bucket | Name of S3 bucket which you created |
| S3 URL | [Endpoint] of S3 Bucket |
| Storage Class | STANDARD |

## Last settings for connecting to CloudFront
Set up some utilities for CloudFront on WordPress

### Work flows
1. Create IAM user
2. Set up plugin for WordPress

#### IAM 
##### Create IAM user
1. Click **[IAM]** in AWS Console;
2. Click **[Users]** on the left menu;
3. Click **[Create New Users]** to launch wizard;
4. Input **[amimoto-cloudfront]** into **[Enter User Names]**, to define the user is for CloudFront;
5. Check **[Generate an access key for each user]**;
6. Create it;

##### Attach policies to IAM user
Attach policy to created IAM user
1. Click **[Users]** on the left menu;
2. Click **[amimoto-cloudfront]**;
3. Click **[Permissions]**;
4. Click **[Attach Policy]** in **[Managed Policies]** area;
5. Choose **[CloudFrontFullAccess]** then click **[Atattch Policy]**;

#### Set up plugin for WordPress
##### Set up Cache flush plugin: C3 CloudFront Clear Cache
1. Login to the Dashboard;
2. Enable **[C3 CloudFront Clear Cache]** on plugin page;
3. Fill in some value on **[C3 Settings ]** on **[Setting]** menu;

| Item Name | Value |
| :-- | :-- |
| CloudFront Distribution ID | input your CloudFront Distribution ID |
| AWS Access Key | AWS Access Key of amimoto-cloudfront |
| AWS Secret Key | AWS Secret Key of amimoto-cloudfront|

##### How to check CloudFront Distribution ID
![How to check CloudFront Distribution ID](./img/cf_distrib.png)

##### Plugin for apply changes on preview 
```
$ cd /var/www/vhost/{INSTANCE_ID}/wp-content
$ sudo mkdir mu-plugins && cd mu-plugins
$ sudo wget https://gist.githubusercontent.com/wokamoto/ecfd3a7ea9ef80ea1628/raw/02e4e011597c0969f0ff4dec48de539a89b96e4a/cloudfront-preview-fix.php
$ sudo chown nginx:nginx cloudfront-preview-fix.php
```
