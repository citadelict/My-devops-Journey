### AWS-CLOUD-SOLUTION-FOR-2-COMPANY-WEBSITES-USING-A-REVERSE-PROXY-TECHNOLOGY
------
#### General Overview

You will create a secure infrastructure within AWS VPC (Virtual Private Cloud) for a fictional company named Citatech. This infrastructure will support Citatech's main business website, which uses the WordPress Content Management System (CMS), as well as a Tooling Website (PHP/MySQL) for their DevOps team. To enhance security and performance, the company has decided to implement NGINX reverse proxy technology. The project prioritizes cost-efficiency, security, and scalability. The goal is to develop an architecture that ensures both the WordPress and Tooling websites are resilient to web server failures, can handle increased traffic, and maintain reasonable costs.

![](./images/aws.png)

### Requirements
There are few requirements that must be met before you begin:

1. Properly configure your AWS account and Organization Unit 
    * Create an AWS Master account. (Also known as Root Account)
    * Within the Root account, create a sub-account and name it DevOps. (You will need another email address to complete this)

![](./images/1.png)

![](./images/2.png)

![](./images/3.png)

   * Login to the newly created AWS account using the new email address.
     
2. Create a domain name for your company. I used [Hostinger](https://www.hostinger.com/).
3. Create a hosted zone in AWS, and map it to your domain. to do this ,
   * On your AWS console search bar, type `route 53` and click on it, > select `create hosted zones`

![](./images/4.png)    

   * Set your already purchased domain name, and set the type o public hosted zone, click on `create hosted zone`
   * copy the namservers and go back to your domain management , and add the nameservers, allow it sometime to propagate, usually between few hours to a day

 ### SET UP A VIRTUAL PRIVATE CLOUD (VPC)

 --------
1. Create a VPC

   ![](./images/5.png)

2. Create the subnets as shown in the Architecture

   P.S :  Use this website to get the CIDR blocks easily. [ipinfo](https://ipinfo.io/ips).

  ![](./images/6.png)

3. Create a route table and associate it with public subnets. to do this ,
     * on the side vpc dashboard side medu, click on `route tables` > `create route table`
     * Associate Public subnet to the Public route table. Click the tab `Actions` >  `edit subnet associations` - select the subnets and click `save associations`

  ![](./images/7.png)
  
  ![](./images/8.png)
 
4. Create a route table and associate it with private subnets. to do this
     * Repeat same process as step 3 above

  ![](./images/9.png)

  ![](./images/10.png)

  ![](./images/11.png)

5. Create internet gateway as shown in the architecture.and attach it to the VPC

  ![](./images/12.png)

  Next, attach the internet gateway to the VPC we created earlier.
  On the same page click `Attach to VPC`, select the vpc and click `attach internet gateway`

  ![](./images/13.png)
  

6. Edit a route in public route table, and associate it with the Internet Gateway. (This is what allows a public subnet to be accessible from the Internet).To achieve this,
      * Select the public route table we created, the click on `actions` > `edit route`
  
   
  ![](./images/14.png)

7. Create Elastic IP to configured with the NAT gateway. The NAT gateway enables connection from the public subnet to private subnet and it needs a static ip to make this         happen.`VPC` > `Elastic IP addresses` > `Allocate Elastic IP address` - add a name tag and click on `allocate`

  ![](./images/15.png)

8. Create a Nat Gateway and assign the Elastic IPs
    Click on `VPC` > `NAT gateways` > `Create NAT gateway`

   - Select a Public Subnet
   - Connection Type: Public
   - Allocate Elastic IP
  
9. Update the Private route table - add allow anywhere ip and associate it the NAT gateway.

   ![](./images/16.png)

10. Create a Security Group for the following

   - `Nginx Servers`: Access to Nginx should only be allowed from a Application Load balancer (ALB). At this point, we have not created a load balancer, therefore we will          update the rules later. For now, just create it and put some dummy records as a place holder.
   - `Bastion Servers`: Access to the Bastion servers should be allowed only from workstations that need to SSH into the bastion servers. Hence, you can use your workstation       public IP address. To get this information, simply go to your terminal and type curl www.canhazip.com
   - `Application Load Balancer`: ALB will be available from the Internet
   - `Webservers`: Access to Webservers should only be allowed from the `Nginx` servers. Since we do not have the servers created yet, just put some dummy records as a place       holder, we will update it later.
   - `Data Layer`: Access to the Data layer, which is comprised of `Amazon Relational Database Service (RDS)` and `Amazon Elastic File System (EFS)` must be carefully               desinged – only `webservers` should be able to connect to `RDS`, while `Nginx` and `Webservers` will have access to `EFS` Mountpoint.

   ![](./images/18.png)


### TLS Certificates From Amazon Certificate Manager (ACM)

   You will need TLS certificates to handle secured connectivity to your Application Load Balancers (ALB). to do this, 
   navigate to `amazon certificate manager (acm)` > `request certificate` and fill in rquired details

   ![](./images/19.png)

   ![](./images/20.png)

   ![](./images/21.png)

### Configure EFS
---

   * Create a new EFS File system. Create an EFS mount target per AZ in the VPC, associate it with both subnets dedicated for data layer. Associate the Security groups             created earlier for data layer.

   ![](./images/22.png)

   ![](./images/23.png)

   ![](./images/24.png)

   * Create 2 access points - For each of the website (wordpress and tooling) so that the files do not overwrite each other when we moun

   ![](./images/25.png)

   ![](./images/26.png)

   ![](./images/27.png)

   ![](./images/28.png)

### Configure RDS
-----
#### Pre-requisite:

   * Create a KMS Key

   ![](./images/29.png)

   ![](./images/30.png)

   ![](./images/31.png)

   ![](./images/32.png)

   * To ensure that yout databases are highly available and also have failover support in case one availability zone fails, we will configure a multi-AZ set up of RDS MySQL       database instance. In our case, since we are only using 2 AZs, we can only failover to one, but the same concept applies to 3 Availability Zones.

   * To configure RDS, follow steps below:

   
   1. Create a subnet group and add 2 private subnets (data Layer)

      ![](./images/33.png)

      ![](./images/34.png)

      ![](./images/35.png)
      

   2. Create the DataBase by navigating to `amazon rds` > `databases` > `create database`

      ![](./images/36.png)

      ![](./images/37.png)

      ![](./images/38.png)

      ![](./images/39.png)

      ![](./images/40.png)

      ![](./images/41.png)


### Set Up Compute Resources for Bastion
-----
#### Provision the EC2 Instances for Bastion
Create an EC2 Instance based on Red Hat Enterprise Linux (AMI) (You can search for this ami, RHEL-8.7.0_HVM-20230215-x86_64-13-Hourly2-GP2
) per each Availability Zone in the same Region and same AZ where you created Nginx server
* Ensure that it has the following software installed
   * python
   * ntp  (we used `chrnony` instead, The is because the ntp package is not available in the repositories for our Enterprise Linux 9 system. Instead of ntp, chrony can be used, which is the default NTP implementation in newer versions of RHEL and its derivatives, including Enterprise Linux 9.)
   * net-tools 
   * vim
   * wget
   * telnet
   * epel-release
   * htop

We will use instance to create an ami for launching instances in Auto-scaling groups so all the installations will be done before creating the ami from the instance

### Bastion ami installation

   
    sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
    sudo yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-89.rpm 
    sudo yum install wget vim python3 telnet htop git mysql net-tools chrony -y 
    sudo systemctl start chronyd 
    sudo systemctl enable chronyd

#### Nginx ami installation

    sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm

    sudo yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm

    sudo yum install wget vim python3 telnet htop git mysql net-tools chrony -y

    sudo systemctl start chronyd

    sudo systemctl enable chronyd

### configure selinux policies for the nginx servers

    sudo setsebool -P httpd_can_network_connect=1
    sudo setsebool -P httpd_can_network_connect_db=1
    sudo setsebool -P httpd_execmem=1
    sudo setsebool -P httpd_use_nfs 1

### This section will install amazon efs utils for mounting the target on the Elastic file system

    git clone https://github.com/aws/efs-utils

    cd efs-utils

    sudo yum install -y make

    yum install -y rpm-build

    # openssl-devel is needed by amazon-efs-utils-2.0.4-1.el9.x86_64
    sudo yum install openssl-devel -y

    # Cargo command needs to be installed as it is necessary for building the Rust project included in the source.
    sudo yum install cargo -y

    make rpm 

    yum install -y  ./build/amazon-efs-utils*rpm

### Setting up self-signed certificate for the nginx instance

    sudo mkdir /etc/ssl/private

    sudo chmod 700 /etc/ssl/private

    openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/citatech.key -out /etc/ssl/certs/citatech.crt

    sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048

#### Webserver ami installation

    sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm

    sudo yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm

    sudo yum install wget vim python3 telnet htop git mysql net-tools chrony -y

    sudo systemctl start chronyd

    sudo systemctl enable chronyd


### configure selinux policies for the webservers

    sudo setsebool -P httpd_can_network_connect=1
    sudo setsebool -P httpd_can_network_connect_db=1
    sudo setsebool -P httpd_execmem=1
    sudo setsebool -P httpd_use_nfs 1

### This section will install amazon efs utils for mounting the target on the Elastic file system

    git clone https://github.com/aws/efs-utils

    cd efs-utils

    sudo yum install -y make

    yum install -y rpm-build

    # openssl-devel is needed by amazon-efs-utils-2.0.4-1.el9.x86_64
    sudo yum install openssl-devel -y

    # Cargo command needs to be installed as it is necessary for building the Rust project included in the source.
    sudo yum install cargo -y

    make rpm 

    yum install -y  ./build/amazon-efs-utils*rpm


### Setting up self-signed certificate for the apache webserver instance

    yum install -y mod_ssl

    openssl req -newkey rsa:2048 -nodes -keyout /etc/pki/tls/private/citatech.key -x509 -days 365 -out /etc/pki/tls/certs/citatech.crt

    vi /etc/httpd/conf.d/ssl.conf

#### Use these references to understand more

[IP ranges](https://ipinfo.io/ips)

[Nginx reverse proxy server](https://www.nginx.com/resources/glossary/reverse-proxy-server/)

[Understanding ec2 user data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html)

[Manually installing the Amazon EFS client](https://docs.aws.amazon.com/efs/latest/ug/installing-amazon-efs-utils.html#installing-other-distro)

[creating target groups for AWS Loadbalancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html)

[Self-Signed SSL Certificate for Apache](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-on-centos-8)

[Create a Self-Signed SSL Certificate for Nginx](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-on-centos-7)

### Create AMIs from the 3 instances

   * To create an AMI (Amazon Machine Image) from the 3 instances , you need to `go to the instance page` > `select the instance` > `actions` > `images and templates` > `create image`. Repeat same process for bastion, nginx and webservers

 ![](./images/42.png)

 ![](./images/43.png)

### Configure Load balancers and Target Groups

1. Create Target group for NGINX, tooling amd wordpress targets

   ![](./images/44.png)

   ![](./images/45.png)

   ![](./images/46.png)
   

3. Configure Application Load Balancer (ALB)

   External Application Load Balancer To Route Traffic To NGINX

   Nginx EC2 Instances will have configurations that accepts incoming traffic only from Load Balancers. No request should go directly to Nginx servers. With this kind of         setup, we will benefit from intelligent routing of requests from the ALB to Nginx servers across the 2 Availability Zones. We will also be able to offload SSL/TLS             certificates on the ALB instead of Nginx. Therefore, Nginx will be able to perform faster since it will not require extra compute resources to valifate certificates for       every request.

    Create an Internet facing ALB
    Ensure that it listens on HTTPS protocol (TCP port 443)
    Ensure the ALB is created within the appropriate VPC | AZ | Subnets
    Choose the Certificate from ACM
    Select Security Group
    Select Nginx Instances as the target group

      ![](./images/47.png)

      ![](./images/48.png)

      ![](./images/49.png)

      ![](./images/50.png)

      ![](./images/51.png)

      ![](./images/52.png)

      ![](./images/53.png)

    Application Load Balancer To Route Traffic To Webservers

   Since the webservers are configured for auto-scaling, there is going to be a problem if servers get dynamically scalled out or in. Nginx will not know about the new IP        addresses, or the ones that get removed. Hence, Nginx will not know where to direct the traffic.
   
   To solve this problem, we must use a load balancer. But this time, it will be an internal load balancer. Not Internet facing since the webservers are within a private         subnet, and we do not want direct access to them.

    Create an Internal ALB
    Ensure that it listens on HTTPS protocol (TCP port 443)
    Ensure the ALB is created within the appropriate VPC | AZ | Subnets
    Choose the Certificate from ACM
    Select Security Group
    Select webserver Instances as the target group

    ![](./images/54.png)

    ![](./images/55.png)

    ![](./images/56.png)

    ![](./images/57.png)

    ![](./images/58.png)


   The default target configured on the listener while creating the internal load balancer is to forward traffic to wordpress on port 443. Hence, we need to create a rule to route traffic to tooling as well.

    !. Select internal load balancer from the list of load balancers created:
        * Choose the load balancer where you want to add the rule.
    2. Listeners Tab:
        * Click on the Listeners tab.
        * Select the listener (HTTPS:443) and click Manage listener.
    3. Add Rules:
        * Click on Add rule.
    4. Configure the Rule:
        * Give the rule a name and click next.
        * Add a condition by selecting Host header.
        * Enter the hostnames for which you want to route traffic. (tooling.com and www.tooling.com).
        * Choose the appropriate target group for the hostname.
      
    ![](./images/59.png)

    ![](./images/60.png)

    ![](./images/61.png)

    ![](./images/62.png)

    ![](./images/63.png)

    ![](./images/64.png)

    ![](./images/65.png)

    ![](./images/66.png)

    ![](./images/67.png)


PREPARE LAUNCH TEMPLATE FOR NGINX (ONE PER SUBNET)

   1. Make use of the AMI to set up a launch template
   2. Ensure the Instances are launched into a public subnet.
   3. Assign appropriate security group.
   4. Configure [userdata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html) to update yum package repository and install nginx. Ensure to enable auto-            assign public IP in the Advance Network Configuration. you can access my repo on this project to get the userdata [repo](https://github.com/citadelict/citatech-project-config) and edit it with your details

   ![](./images/68.png)

   ![](./images/69.png)

   ![](./images/70.png)

   ![](./images/71.png)

 We need to update the reverse.conf file by updating proxy_pass value to the end point of the internal load balancer (DNS name) before using the userdata so as to clone the updated repository

   ![](./images/72.png)

Repeat the same setting for Bastion, the difference here is the userdata input, AMI and security group.

   
   ![](./images/73.png)

Wordpress Userdata

NB: Both Wordpress and Tooling make use of Webserver AMI.

Update the mount point to the file system, this should be done on access points for tooling and wordpress respectively., to get the mount point for each access points, 
* Go to `EFS` > `access points` and select wordpress, click on attach, and there you can seen the command to attach the access points mount points

  ![](./images/73.png)

  ![](./images/74.png)

  ![](./images/75.png)


The RDS end point is also needed. Go to `RDS` > `Databases` > `select the database you created earlier`

  ![](./images/76.png)

  ![](./images/77.png)


Tooling userdata ( Repeat same steps as you did for wordpress user-data for tooling, difference is, get the tooling access points mount point command instead)

  ![](./images/78.png)

All launch templates created.

  ![](./images/79.png)


### CONFIGURE AUTOSCALING FOR NGINX

   * Select the right launch template
   * Select the VPC
   * Select both public subnets
   * Enable Application Load Balancer for the AutoScalingGroup (ASG)
   * Select the target group you created before
   * Ensure that you have health checks for both EC2 and ALB
   * The desired capacity is 2
   * Minimum capacity is 2
   * Maximum capacity is 4
   * Set [scale out](https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-lifecycle.html) if CPU utilization reaches 90%
   * Ensure there is an [SNS](https://docs.aws.amazon.com/sns/latest/dg/welcome.html) topic to send scaling notifications

### Create Auto Scaling Group for Bastion

   ![](./images/80.png)

   ![](./images/81.png)

Access RDS through Bastion connection to craete database for wordpress and tooling.

Copy the RDS endpoint to be used as host

   ![](./images/82.png)

   ![](./images/83.png)


### Create Auto Scaling Group for Nginx

   ![](./images/84.png)

   ![](./images/85.png)

   ![](./images/86.png)

   ![](./images/87.png)

### Repeat the Nginx Auto Scaling Group steps above for Wordpress and Tooling with their right launch template

   ![](./images/88.png)

   ![](./images/89.png)


# Configuring DNS with Route53

Earlier in this project we registered a free domain with `Hostinger` and configured a hosted zone in [Route53](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/Welcome.html). But that is not all that needs to be done as far as `DNS` configuration is concerned.

We need to ensure that the main domain for the WordPress website can be reached, and the subdomain for Tooling website can also be reached using a browser.

Create other records such as [CNAME, alias and A records](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/route-53-concepts.html).

NOTE: You can use either CNAME or alias records to achieve the same thing. But alias record has better functionality because it is a faster to resolve DNS record, and can coexist with other records on that name. Read [here](https://support.dnsimple.com/articles/differences-between-a-cname-alias-url/#:~:text=The%20A%20record%20maps%20a,a%20name%20to%20another%20name.&text=The%20ALIAS%20record%20maps%20a,the%20HTTP%20301%20status%20code) to get to know more about the differences.

- Create an alias record for the root domain and direct its traffic to the ALB DNS name.
- Create an alias record for `tooling.citatech.online` and direct its traffic to the ALB DNS name.

  ![](./images/90.png)

  ![](./images/91.png)


### Test your websites.


__Now let us access our tooling website via a browser using our DNS name__ 

   
   ![](./images/92.png)

__Let's access our wordpress website__

   ![](./images/93.png)


We have sucessfully created a secured, scalable and cost-effective infrastructure to host 2 enterprise websites using various Cloud services from AWS. At this point, our infrastructure is ready to host real websites' load.




















