# Host a Dynamic Ecommerce Website on AWS

In this project, i demonstrated how to deploy an ecommerce website on aws using a reference architecture. AWS services used in this project includes; VPC, Security group, Ec2, Nat gateways, RDS, Application Loadbalancer, Route 53, Autoscaling Group, Certificate Manager, S3 and more.


## Project Reference Architecture

![1 _LAMP_Stack_Project_Reference_Architecture](https://user-images.githubusercontent.com/115881685/225284346-905bb80b-ad99-4b5c-9f28-832aa455dde3.jpg)



# Create VPC with Public and Private Subnets
To kick off the project, we will build a custom vpc, see vpc reference architecture below which is three(3) tiered designed.

In a 3 tier vpc reference architecture, the infrastructure is divided into 3 tiers, the first tier contains a public subnet which holds a nat gateway, bastion host and loadbalancer. In the second tier, we have a private subnet which hold the webservers(ec2 instances), the third tier contains a private subnet, which holds the data base. All this subnets are replicated accross multiple availability zones to provide high avalaibilty and fault tolerence. Then finally we will create an internet gateway and route table, this will provide resourses in the vpc with internet access.


### VPC Reference Architecture
![2 _VPC_Reference_Architecture](https://user-images.githubusercontent.com/115881685/225285355-25409fca-777c-4784-b8de-23e4dcb70191.jpg)

To create the VPC, go to the AWS console a type vpc in the search box and select vpc. In the VPC dashboard, select VPC, then click create VPC.
Name the VPC "Dev VPC". In the IPv4 cidr block space, type "10.0.0.0/16". Leave everything as default and scroll down and click create vpc.

![image](https://user-images.githubusercontent.com/115881685/225288822-88cf185b-e21f-495c-b72d-84fd55696d3d.png)
![image](https://user-images.githubusercontent.com/115881685/225289117-2781efec-f4eb-42b6-b70b-faac8863321d.png)
![image](https://user-images.githubusercontent.com/115881685/225289229-2cb26b84-533e-49b6-acf3-fbe68687aad5.png)

![image](https://user-images.githubusercontent.com/115881685/225289405-5f49df39-ef2e-4fbc-b1bc-1f88f8385f59.png)

#### Next step is to eneble DNS hostname in the vpc
To do that, select "action" and then edit "vpc setting". In the next page that opens, check the enable "DNS hostname" box and scroll down and hit save button.

![image](https://user-images.githubusercontent.com/115881685/225291345-c31135a1-fa3a-4846-9bac-3a3e72cf1ff7.png)
![image](https://user-images.githubusercontent.com/115881685/225291408-914debf2-26c2-4844-b03a-d971f6255a57.png)


### create Internet Gateway

On the vpc dashboard, click on internet gateway and hit create internet gateway. Name it "Dev Internet Gateway", then scroll down and hit create internet gateway.

![image](https://user-images.githubusercontent.com/115881685/225293323-2a957e58-e968-4d79-ae90-c4aa55883d03.png)
![image](https://user-images.githubusercontent.com/115881685/225293451-87a74411-1ddd-4cc6-b793-d6b1a23f1f51.png)

Next step is to attach the internet gateway to the vpc that was created, to do that, click on "attach to a vpc" or  "action".
Under "available vpcs", select the Dev VPC that was created, and the scroll down and click "attach internet gateway".

![image](https://user-images.githubusercontent.com/115881685/225295224-23de14b1-fbe9-4e38-9a31-386cfd680595.png)
![image](https://user-images.githubusercontent.com/115881685/225294860-ccab48fd-65a2-45d0-b749-0eb458b2ec7f.png)
![image](https://user-images.githubusercontent.com/115881685/225295395-8e663aa2-607e-4d47-a3f3-c472ce689cc1.png)


### Create Subnets.
Next step is to create a subnets in the first and second availability zones according to the vpc refference architecture. To do this, select "subnet" in the vpc dashboard and click "create subnets". Under "vpc id", select our "Dev VPC" and the scroll down to "subnet name", according to our reference architecture, it should be called "public subnet AZ1". Under "availability zone", select "us-east-1a". Under IPv4 Cidr block, type in "10.0.0.0/24". scroll down and click "create subnets"

![image](https://user-images.githubusercontent.com/115881685/225298356-e7b1d44f-c588-4aa2-bdb7-778ef1f33487.png)
![image](https://user-images.githubusercontent.com/115881685/225298547-6dd77db0-f912-458d-b68a-78a1f043fbaa.png)
![image](https://user-images.githubusercontent.com/115881685/225298768-26513a48-edc6-4206-89a9-b9fa9bf3bc96.png)

For our second subnet, follow the same steps as the first one, under "subnet name", call it "public subnet AZ2", for availability zones, select "us-east-1b", Under IPv4 Cidr bock, type, "10.0.1.0/24 and then scroll down and hit create subnets.
To see the two subnets, filter by the vpc created.

![image](https://user-images.githubusercontent.com/115881685/225300423-b79e295f-951a-4718-94b2-99b14af0aec3.png)
![image](https://user-images.githubusercontent.com/115881685/225300692-99f0dfa5-06fb-413b-9638-dcb44e307e12.png)
![image](https://user-images.githubusercontent.com/115881685/225303279-3a1661a4-9914-4dd0-b20f-242012fffa96.png)


#### enable auto-assign ip for the subnet
To do this, select the first subnet and click "action", click "edit subnets settings", under "auto-assign ip setting", enable "auto-assign public ipv4 address", then scroll down and click save.

![image](https://user-images.githubusercontent.com/115881685/225302758-e31a1a10-5c75-4230-8eed-78cf372e3e8c.png)
![image](https://user-images.githubusercontent.com/115881685/225303472-a8dd6b08-754d-477c-867e-11634ddb6e32.png)
![image](https://user-images.githubusercontent.com/115881685/225303577-5a4327c8-3303-4aac-b542-284dda1f7538.png)

Repeat the same step for the second subnet.

### Create route table.
To create route table, select routetable on the left hand side of the vpc dashboard and click "create route table".
Note there is always one main route table created along with the vpc creation process.
Under "name", call it "public-route-table", under vpc, select our "Dev-vpc", then scroll down and hit "create route table"

![image](https://user-images.githubusercontent.com/115881685/225306384-ebaf6e45-99bd-4c06-accd-874b7b500bee.png)
![image](https://user-images.githubusercontent.com/115881685/225306515-d4eb6358-d01e-4ede-94b0-5b5e8dba17ea.png)
![image](https://user-images.githubusercontent.com/115881685/225306742-0b7be1cc-7606-4d66-91a3-c5b25f779c11.png)


### Next step Add public route to the route table
The route aalows the public route table to route traffic to the internet.

To do this, make sure you are on the route tab, then click "edit routes", then click "add route", under "destination", type in "0.0.0.0/0" in the box. After that, under "target", select "internet gateway", then click "save changes"

![image](https://user-images.githubusercontent.com/115881685/225329518-05d1b16d-fd84-4578-be76-6189b6b54ef5.png)
![image](https://user-images.githubusercontent.com/115881685/225329742-4ea562f6-2661-4ccc-be49-e861979caa8a.png)
![image](https://user-images.githubusercontent.com/115881685/225330142-26489a07-0cb6-491e-9e42-773197630259.png)

### Next step, associate the two public subnet to the route table

To this, select the "subnet association" tab, then click "edit subnet association", once in the next page, select the two subnet that you see and hit "save aasociation".

![image](https://user-images.githubusercontent.com/115881685/225331798-87967bff-4cd3-4cb9-8fb7-e47fa36a7d62.png)
![image](https://user-images.githubusercontent.com/115881685/225332075-38c30c18-7c55-4f46-8135-003cacf7df4f.png)
![image](https://user-images.githubusercontent.com/115881685/225332570-0c542af2-1657-46e4-abc7-30d7d674852e.png)


### Create private subnet.
Following our vpc refference architecture, we will create 4 private subnets. to this select "subnets" in the left hand side of your vpc dashboard, click "create subnets". make sure you filter by the Dev vpc created earlier, so that you are only seeing the two public subnets that was previously created.

under "VPC", select our Dev vpc, under "name", type "private app subnet AZ1" select us-east-1 under "availability zone" under "cidr block", type, "10.0.2.0/24", then scroll down and hit create subnet.

![image](https://user-images.githubusercontent.com/115881685/225336899-87c94012-6e9c-495b-9186-e2b8c13b4705.png)

Do the same for the remaining three(3) subnets by following the VPC reference architecture, see screenshots below.

![image](https://user-images.githubusercontent.com/115881685/225337804-b2d5a862-93cf-4564-bec7-2be55bf6586b.png)
![image](https://user-images.githubusercontent.com/115881685/225338006-eb5f4d33-bc77-4f40-9fe4-47b30b0dc6e7.png)
![image](https://user-images.githubusercontent.com/115881685/225338298-617b20a0-ff8f-428b-bfed-924548624210.png)

TO see all the subnets(6), filter by the dev vpc.

![image](https://user-images.githubusercontent.com/115881685/225338851-abcf4ab3-9eab-44a0-87e3-dbeb3a2ead74.png)

Following our overall lamp stack project reference architecture, next step is to create nate gateways.


## Create NAT Gateways in the Public Subnets

### Nat Gateway reference architecture.
![3 _Nat_Gateway_Reference_Architecture](https://user-images.githubusercontent.com/115881685/225374326-b7a41ca3-8712-451a-bc46-1c26d1d66e7a.jpg)


Following the architecture, we are creating two nat gateaways, one in public subnet az1 and the second in public subnet az2.
To do this, select "natgateway" in the vpc dashboard and click "create nat gateways", give it the name "nat gatway az1", select "public subnet az1" and scroll down, under "elastic ip allocation id", click "allocate elastic ip", scroll down and hit "create nat gateway".

![image](https://user-images.githubusercontent.com/115881685/225372601-b6a479c1-6d66-421a-a35e-1a1e6b4b784e.png)
![image](https://user-images.githubusercontent.com/115881685/225372921-d2b2c6e9-0596-47c1-9b2d-e1d0e079fb96.png)
![image](https://user-images.githubusercontent.com/115881685/225373593-2af0442a-c9ae-45f1-bc35-0ede9efc593d.png)



### Next step create a route table.
Like before simply select "route table" in the left side of your vpc dashboard and hit "create route table". Give the route table the name, "private route table az1" and then select our "Dev vpc", scroll down and click "create route table".

![image](https://user-images.githubusercontent.com/115881685/225377151-ad9a36dc-45c8-4c5f-98a0-e97f94ecd803.png)
![image](https://user-images.githubusercontent.com/115881685/225377295-f636c8fb-a960-4da6-bf3c-181e9673a505.png)


Next we add route to the the private route table az1 to route traffic to the internet through the nat gateway in the public subnet az1

To this, like before, select the "route" tab and click "edit route", click "add route", under "destination", type "0.0.0.0/0 and click it. under "local" select our nat gateway az1 in the public subnet and hit "save changes".

![image](https://user-images.githubusercontent.com/115881685/225380230-059c6834-010a-4f62-9ccc-81b0cce57714.png)
![image](https://user-images.githubusercontent.com/115881685/225380436-af00a519-5a53-4be9-a92a-78ff2dd2c774.png)


Next associte this route table with "private app subnet az1" and "private data subnet az1" following the nat gateway reference architecture.

Like before select the "subnet associations" tab and then click "edit subnet associations". Under "available subnets", select private app subnet az1 and private data subnet az1 respectively and click "save associations".


![image](https://user-images.githubusercontent.com/115881685/225382241-b3f03d95-1928-41aa-aa15-49f54f509916.png)
![image](https://user-images.githubusercontent.com/115881685/225382429-9e6f4e36-8478-48a1-912f-a152930057f7.png)
![image](https://user-images.githubusercontent.com/115881685/225382633-47769ae7-b642-4324-bd30-c07cf479f93f.png)


Next we create the second nat gateway in the public subnet az2. see screenshots below

![image](https://user-images.githubusercontent.com/115881685/225389066-a8e358d0-270a-4927-9cc1-b4a78015bbbc.png)
![image](https://user-images.githubusercontent.com/115881685/225390059-71b9088b-51db-47d0-9f39-22c1666696ab.png)

Create another route table and call it private route table az2, see screenshots below.

![image](https://user-images.githubusercontent.com/115881685/225391210-0825109a-9d26-4038-a346-83734b5e80f3.png)
![image](https://user-images.githubusercontent.com/115881685/225391399-8ba84f7f-6cdb-4b76-b516-0333014baf6e.png)


Next step add a route to the private route table az2 to route traffic to the internet through the nat gateway in the public subnet az2.
Follow the steps we use in creating the previous one.

![image](https://user-images.githubusercontent.com/115881685/225393218-27bf887d-1dc0-4cf4-b01c-0517f05eb33d.png)
![image](https://user-images.githubusercontent.com/115881685/225393596-38c303e7-29be-4c11-b73f-be64b1de114c.png)


Next step associate this route table with private app subnet az2 and private data subnet az2, like before, follow the previous steps.

![image](https://user-images.githubusercontent.com/115881685/225395523-7a667ab4-6a5d-479a-ade1-feaf2e5c08a4.png)
![image](https://user-images.githubusercontent.com/115881685/225396128-642a2d75-e721-428e-933f-14c7fa96ddd4.png)


The next major block of this project is security group.

## Create Security Groups

#### security group reference architecture for our project

![4 WordPress_SG](https://user-images.githubusercontent.com/115881685/225399771-2094091e-5512-497b-8193-10771573cd0c.jpg)


To create security group, select "security group" in the vpc dashboard, filter by the dev vpc, in there you will see a default security group, click "create security group", name it "alb security group", select our dev vpc under vpc. in "inbound rules", click "add rule". The first rule will be under port 80 which is http, search for it and select it. "source" will be 0.0.0.0/0 which means traffic from anywhere, select it.  The second rule will be https which is on port 443, search for it and select it. Also the "source" will be still be "0.0.0.0/0". scroll down and click "create security group".


![image](https://user-images.githubusercontent.com/115881685/225405267-f7316e3c-db72-440d-8d85-e84352b97bdf.png)
![image](https://user-images.githubusercontent.com/115881685/225405482-f0410487-8a7e-4413-b8e9-73b947dbdea9.png)
![image](https://user-images.githubusercontent.com/115881685/225448688-d5cf17f9-e5f6-43e3-8397-87d1943c4217.png)


Next we create SSH Security group, using the security group architecture by follwing the same steps.

![image](https://user-images.githubusercontent.com/115881685/225449342-07cc831a-9f7a-4228-b220-8dc36d104a2f.png)
![image](https://user-images.githubusercontent.com/115881685/225449697-02b1f566-ec37-4084-9198-21622f9b74fc.png)

Use the same steps in creating the the web server security group, database security group and efs security group using the securerity group reference architecture for guide on the diffrerent ports.

When you are done, filter the security groups by your dev vpc to see the six(6) security groups including the default security group.

![image](https://user-images.githubusercontent.com/115881685/225451348-e9f05f62-f340-4066-9424-443771a02a4b.png)



The next major component in our over all lamp stack project is creating rds instance in the private data subnets.



## Create the RDS Instance

#### RDS database architecture

![5 WordPress_RDS](https://user-images.githubusercontent.com/115881685/225452479-89a0ecc7-1f12-43f7-8405-3aac322110ff.jpg)



To create rds database in the management console, type "rds" and select rds under services in the search box.

Once in the rds dashboard, lets create "subnet groups" first, so select it and click "create db subnet group". Name it database subnets, use the same name for "description", then select the dev vpc under "vpc", scroll down, under "add subnet", select us-east-1a and us-east-1b, then under "subnets", select the private data subnets with cidr block "10.0.4.0/24 and "10.0.5.0/24 then scroll down and click "create".


![image](https://user-images.githubusercontent.com/115881685/225455872-133fceef-7006-4248-9cb8-77ae7f01c10d.png)
![image](https://user-images.githubusercontent.com/115881685/225456089-4cc6ecec-164f-40a2-a56e-4fa6968aa2b6.png)
![image](https://user-images.githubusercontent.com/115881685/225456354-10c15e06-b8c3-48ab-9308-98f1cdaa5e01.png)
![image](https://user-images.githubusercontent.com/115881685/225456541-37add466-d2f8-4189-9691-1c9d25aab3b7.png)



Next create database, select "database" and click "create database". Under "choose a database creation method", select "standard create", under "engine type" choose "mysql", under "version" select the latest myssql 5.7, scroll down under "template" select "dev/test", scroll down, under "db instance identifier" type in the name "dev-rds-db", under "credential settings", type in your username and password and repete the password. scroll down, under "db instance class", select "bustable classes" and the toggle previous instance classes. scroll down, under connectivity(vpc), select the dev vpc, under "subnet group", select the subnet group we just created. scroll down, under "vpc security group" select "choose existing", in the search box delete the default and select "database security group", under "availability zone" select "us-east-1b". scroll down and expand "additional configuration", under database name, type in "applicationdb", scroll all the way down and hit "create database". It usually takes some minutes before the rds instance is created.


![image](https://user-images.githubusercontent.com/115881685/225460888-592a6bed-ab41-410a-8d0a-1c73a24d59bf.png)
![image](https://user-images.githubusercontent.com/115881685/225460961-d5f40577-e59b-4794-8f6e-b9a30dfd93e1.png)
![image](https://user-images.githubusercontent.com/115881685/225461045-5b8ea733-c8df-456c-b423-6e3980d1b693.png)
![image](https://user-images.githubusercontent.com/115881685/225461166-14821c81-e314-468e-b58f-72918e3e9382.png)
![image](https://user-images.githubusercontent.com/115881685/225461378-508d031f-af0e-4ac1-b851-a7305f9ad0a3.png)
![image](https://user-images.githubusercontent.com/115881685/225461557-508fd323-a466-44ba-b350-1b13cf72c52e.png)
![image](https://user-images.githubusercontent.com/115881685/225461683-b09ec678-4208-480f-a289-3cc173bc36b1.png)
![image](https://user-images.githubusercontent.com/115881685/225461752-58c61dbf-a0b0-4d8f-a9ea-a9a39ca24284.png)
![image](https://user-images.githubusercontent.com/115881685/225461877-0e2b9ec4-c004-40f3-acbf-06530334a511.png)




## Create S3 Buckets

Next we will upload the web files for this project into AWS management console via an S3 bucket, so go ahead and create a bucket.

Login in to your aws account, and search for S3, once there, click on "create bucket". We will need two buckets, one for the webfiles and another for dummyfiles. Give the first bucket a name, and follow the screenshot below.


![image](https://user-images.githubusercontent.com/115881685/226164914-b09276f5-b051-48ee-b036-938ed2cda677.png)

![image](https://user-images.githubusercontent.com/115881685/226164961-389c912e-0cc0-41a2-8c5c-0c68b6e5206c.png)

![image](https://user-images.githubusercontent.com/115881685/226165006-758459cd-5c10-4340-b29a-41286e0f63d4.png)

![image](https://user-images.githubusercontent.com/115881685/226165041-d43fbd21-a9e8-4c5c-91df-f7f4b37c0305.png)


Once done with creating the bucket, click on it and upload your webfiles from the folder you downloaded it on your computer.


![image](https://user-images.githubusercontent.com/115881685/226165222-74d5fdd6-99b3-426c-88d6-2f740c068fcf.png)

![image](https://user-images.githubusercontent.com/115881685/226165366-081b2231-b976-4064-8f53-b5ec06bac0fa.png)

![image](https://user-images.githubusercontent.com/115881685/226165280-73f99623-5cd1-427e-823f-aca2b4ca56aa.png)

![image](https://user-images.githubusercontent.com/115881685/226165514-2c8fcd13-53c4-4482-9d95-e9e56bf86b62.png)



Follw the same process above and create the second bucket for uploading the dummyfiles needed by our website.


![image](https://user-images.githubusercontent.com/115881685/226165676-44707946-a961-447c-8a0c-b32eb800c0c5.png)
![image](https://user-images.githubusercontent.com/115881685/226165696-9981a8a9-8040-4d5d-be56-6bff3901752c.png)
![image](https://user-images.githubusercontent.com/115881685/226165735-58ad81d0-cf1b-4f22-884c-9a640add87f1.png)


We are done creating and uploading our webfiles into two seperate buckets. 



## Create an IAM Role with S3 Policy.
The IAM role will allow our instance to download the web and dummy files in the S3 bucket we just created.

To do this, go to the IAM dashboard in the console and select "Roles" and click "create role". Follow the instructions in the screenshots below.


![image](https://user-images.githubusercontent.com/115881685/226166182-a1f5ccff-6f25-429e-946c-a8148be96f1d.png)

![image](https://user-images.githubusercontent.com/115881685/226166505-f9ec9f8b-d40e-447f-a384-75fd621e1604.png)


![image](https://user-images.githubusercontent.com/115881685/226166301-6140779d-04c6-4e12-937c-b1fefe5a5918.png)
![image](https://user-images.githubusercontent.com/115881685/226166384-8282243f-e011-4e64-ac73-1a4a1215ad38.png)
![image](https://user-images.githubusercontent.com/115881685/226166772-3ad010dc-d79a-4600-9b7d-7ac3e9c01757.png)
![image](https://user-images.githubusercontent.com/115881685/226166812-d624831b-b5e5-4681-aa5e-cdb9111255bb.png)





## Deploy eCommerce Website.

The next step is to setup a server(instance) that we will use to deploy the ecommerce website. To do this go to the EC2 dashboard and launch an instance, follow the screenshot below.



![image](https://user-images.githubusercontent.com/115881685/225553562-700ea842-c89b-4c18-9629-ca9db31931c7.png)
![image](https://user-images.githubusercontent.com/115881685/225553694-d049cc90-d92f-425f-8d5d-318b0edf6aca.png)
![image](https://user-images.githubusercontent.com/115881685/225554107-4c97de97-c32c-44b3-9787-fa3436524ec8.png)

![image](https://user-images.githubusercontent.com/115881685/226168078-f7f9320b-fc80-473e-adc3-f67c1c1b0c67.png)
![image](https://user-images.githubusercontent.com/115881685/226168108-9ea087ab-3eda-4c85-b356-05b976da3a42.png)

![image](https://user-images.githubusercontent.com/115881685/226168225-50032bf5-f0f0-418f-9857-43c5effc7359.png)


### Next SSH into the server.

Copy the public ipv4 address and open putty then ssh into it.


![image](https://user-images.githubusercontent.com/115881685/226168451-b8425c7d-f52f-4e0f-9625-c0b369ffd9d3.png)

![image](https://user-images.githubusercontent.com/115881685/226168513-f7cd056d-3a44-4872-acc0-539d7b78b618.png)


Next we will run the command below to install the website on the ec2 instance. The commands each have detailed information as to what they do.
Ensure to change the name of the s3 bucket in the sixth(6) to the name of your webfiles bucket.



```
#1. update ec2 instance
sudo su
sudo yum update -y


#2. install apache 
sudo yum install -y httpd httpd-tools mod_ssl
sudo systemctl enable httpd 
sudo systemctl start httpd


#3. install php 7.4
sudo amazon-linux-extras enable php7.4
sudo yum clean metadata
sudo yum install php php-common php-pear -y
sudo yum install php-{cgi,curl,mbstring,gd,mysqlnd,gettext,json,xml,fpm,intl,zip} -y


#4. install mysql5.7
sudo rpm -Uvh https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
sudo yum install mysql-community-server -y
sudo systemctl enable mysqld
sudo systemctl start mysqld


#5. set permissions
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;


#6. download the FleetCart zip from s3 to the html derectory on the ec2 instance
sudo aws s3 sync s3://georgenal86-fleet-cart-webfiles /var/www/html


#7. unzip the FleetCart zip folder
cd /var/www/html
sudo unzip FleetCart.zip


#8. move all the files and folder from the FleetCart directory to the html directory
sudo mv FleetCart/* /var/www/html



#9. move all the hidden files from the FleetCart diretory to the html directory
sudo mv FleetCart/.DS_Store /var/www/html
sudo mv FleetCart/.editorconfig /var/www/html
sudo mv FleetCart/.env /var/www/html
sudo mv FleetCart/.env.example /var/www/html
sudo mv FleetCart/.eslintignore /var/www/html
sudo mv FleetCart/.eslintrc /var/www/html
sudo mv FleetCart/.gitignore /var/www/html
sudo mv FleetCart/.htaccess /var/www/html
sudo mv FleetCart/.npmrc /var/www/html
sudo mv FleetCart/.php_cs /var/www/html
sudo mv FleetCart/.rtlcssrc /var/www/html


#10. delete the FleetCart and FleetCart.zip folder
sudo rm -rf FleetCart FleetCart.zip


#11. enable mod_rewrite on ec2 linux, add apache to group, and restart server
sudo sed -i '/<Directory "\/var\/www\/html">/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/httpd/conf/httpd.conf
chown apache:apache -R /var/www/html 
sudo service httpd restart
```



To the Access the website, copy the ipv4 address of the setup server and paste it in a new tab in your web browser, enter it,


![image](https://user-images.githubusercontent.com/115881685/226174978-7f6208a3-6536-4be0-beee-3873bfbd85ec.png)



So there you go, we can now access our website, to finish installing the website, scroll down and click "continue".



![image](https://user-images.githubusercontent.com/115881685/226175240-7cc5693d-5d94-48e7-88cf-59494de9e36b.png)



On the Configuration page that opens, enter your rds database end point url in the "host field", then enter your rds "db username", db "password" and "database name" respectively.



![image](https://user-images.githubusercontent.com/115881685/226175619-7177f1ac-5462-4246-8564-edfc8365e0e5.png)



Then scroll down and enter your, personal details and scroll down again.



![image](https://user-images.githubusercontent.com/115881685/226175776-f7928189-9a91-4eaa-8440-9e3cb34c7a87.png)




Finally enter your "store details" and click "install".



![image](https://user-images.githubusercontent.com/115881685/226175862-12006288-6ad5-47be-838f-b8351a28e94a.png)

![image](https://user-images.githubusercontent.com/115881685/226176039-c77d3bac-d2aa-4bcf-87e8-49eee7cc9d01.png)


Fantastic !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!, we have successfully install the website, click "go to your shop"



![image](https://user-images.githubusercontent.com/115881685/226176091-beabc13c-9d0b-4d9b-af9f-f9644418fe13.png)



And this is the ecommerce store, we have successfully install the fleetcart store, and currently the site is empty because the web developer has given us a blank template so that we can customize the website how we want, next we are going to upload?import all the dummy data that the site needs to make it presentable.





## Import the Dummy Data for the Website

To import the dummy data for the website, you will need an application called "mysql workbench", so if you dont have, go and download and install it on your system.



![image](https://user-images.githubusercontent.com/115881685/226176952-b3a302f0-1bc5-4e42-a51e-b68ef7e52dbd.png)




Next step is to create a pem file, go to the ec2 dashboard and on the left side, select "key pair" and click "create key pair". Follow the instructions in the screenshots below.





![image](https://user-images.githubusercontent.com/115881685/226177025-25f134d4-ff06-48fc-9f97-4aa9a7c44b77.png)





Next we will launch a ec2 instance in the public subnet that can be sshed into, follow the previous steps, see screenshots below.



![image](https://user-images.githubusercontent.com/115881685/226177583-254121c0-a0df-48fd-b8c7-b2ca4f71a6c9.png)




 The next thing to do is to create a security group that we will add to the rds instance, to allow us to connect to it from the dummy server we just created.
 
 
 


![image](https://user-images.githubusercontent.com/115881685/226177870-7d1ff426-496a-4d1d-a7b0-e7bbc602ab33.png)

![image](https://user-images.githubusercontent.com/115881685/226177901-936a8901-4568-4feb-8494-3bce4c1fc08b.png)




Next add the security group to the rds instance to allow the dummy server to connect it. Open the Rds dashboard.



![image](https://user-images.githubusercontent.com/115881685/226178156-a7bc0fe3-9b1e-45fa-a5e4-5d11a98e22eb.png)

![image](https://user-images.githubusercontent.com/115881685/226178194-15af827b-617d-4004-99f9-30d80d1d9406.png)

![image](https://user-images.githubusercontent.com/115881685/226178252-9d3e6d61-f09f-48ea-8e8e-15fb3026fb4a.png)




Next step is to import the SQL file of our website into the RDS database. To do this open MySQL Workbench, select "database" click "connect to database". Note "ssh host name" refers to the dns of the public ipv4 of the dummy server, "ssh key file" refers to the pem file that was created and downloaded, "mysql hostname" is the end point url of our rds database, and "username" and "password" refers to the rds database username and password respectively.




![image](https://user-images.githubusercontent.com/115881685/226179007-8aca4a4a-3b59-4799-bfe3-5629e2d85b7e.png)



![image](https://user-images.githubusercontent.com/115881685/226179245-fcdae584-23d5-4b22-a7f4-61bf37b664e4.png)




then click ok, ignore the warning message and hit ok again.




![image](https://user-images.githubusercontent.com/115881685/226180545-4936191c-7c58-4e4d-9058-270ac17d8e7f.png)



Finally we have connected to the RDS database, we can tell by looking at the name of the rds database, "applicationdb" at the top left hand corner.


Next we will import our SQL file from our computer into the RDS database. click on "administration" tab and follow the screenshots.



![image](https://user-images.githubusercontent.com/115881685/226180768-7abe8cdc-36ec-4bef-a68c-b44a1eebb1f6.png)

![image](https://user-images.githubusercontent.com/115881685/226181015-d2a3193d-dd56-41d7-a69d-ca63c2edf037.png)




Give it some time to finish importing.





![image](https://user-images.githubusercontent.com/115881685/226181131-25d32dea-176a-47cf-b0da-ab0e92ec1447.png)





Hurray !!!!!!!!!!!!!!!!!!!!!!! the importation is complete. All our data has now been imported into the rds database.



Next step is to delete all the resourses we created for the importation, things like, dummy key, dummy server, dummy security group. so go aheard and delete them. Also delete the dummy security group associated with the RDS database.




### Next step SSH into the setup server.

Copy the the public ipv4 address of the setup server and ssh into it, then run the command below that will enable us to add the dummy data into our website.


```
sudo su
sudo aws s3 sync s3://georgenal86-fleetcart-dummy-data /home/ec2-user
sudo unzip dummy.zip
sudo mv dummy/* /var/www/html/public
sudo mv -f dummy/.DS_Store /var/www/html/public
sudo rm -rf /var/www/html/storage/framework/cache/data/cache
sudo rm -rf dummy dummy.zip
chown apache:apache -R /var/www/html 
sudo service httpd restart
```




Once done with running the command, still copy the public ipv4 address of your setup server and paste in a new tab in your browser, press enter.




![image](https://user-images.githubusercontent.com/115881685/226182412-f8cbe70a-69c2-4172-b92b-69b95ced1ea4.png)

![image](https://user-images.githubusercontent.com/115881685/226182532-dab729f1-9651-4369-90ac-e50e4cfb09d0.png)

![image](https://user-images.githubusercontent.com/115881685/226182572-a857c58c-3b8f-4db8-be77-f8cb09532516.png)






Yes all our dummy data have been successfully added to our website and its no longer empty.



Now that we have installed and configured our website, we will use the setup server(ec2 instance) we installed our website on to create an AMI. Then we can use the AMI to launch new EC2 instances with our website already configured on them.



## Create an Amazon Machine Image (AMI)

![image](https://user-images.githubusercontent.com/115881685/226191655-83c80af4-fe9b-4432-84b2-3009b03597e1.png)

![image](https://user-images.githubusercontent.com/115881685/226191690-c756189d-25c6-4c13-a424-080b52e40113.png)

![image](https://user-images.githubusercontent.com/115881685/226191725-cd36939e-9beb-47c5-bccb-d70b07490ecf.png)

![image](https://user-images.githubusercontent.com/115881685/226191774-9cf17881-c4f8-4b39-8ff8-633d4afa6c19.png)




When create an AMI a snapshot is als created, click on the snapshot tab to see it.

Now we can use the AMI to launch new instances with our website already installed on them.



## Create an Application Load Balancer(ALB)

Next step is to create application loadbalancer, which helps to route traffic to instances in the private app subnets.

Before creating the alb, first we will launch EC2 instances in each of the private app subnet with the help of the AMI we just created. Click on Launch instance in the EC2 dashboard and follow the steps below.



![image](https://user-images.githubusercontent.com/115881685/226192609-c51b9778-ee10-4d79-95a7-669e969422ed.png)

![image](https://user-images.githubusercontent.com/115881685/226192659-a7c5572a-df84-48d8-9957-313f4d68d29e.png)

![image](https://user-images.githubusercontent.com/115881685/226192701-f8b973e1-7622-4eba-a023-61d63e4eba4b.png)

![image](https://user-images.githubusercontent.com/115881685/226192893-45f42f9f-22fb-46c2-802f-57c29ed813b1.png)



Follow the same step to launch the second ec2 instance in the private app subnet az2.

Once done go to your ec2 dashboard to see your instances



![image](https://user-images.githubusercontent.com/115881685/225570488-9a55e742-b4eb-475f-8bd9-ee3d7c41147a.png)



### Create target group
Next step is to create target group and place the two instances in it, this will allow the application loadbalancer to route traffic to the intances.

Select target groups in the left side of the ec2 management console and click create "target group". see screenshots below for instructions.


![image](https://user-images.githubusercontent.com/115881685/225576333-7743d14e-b60b-45a0-ab2e-180f0f3bdde0.png)

![image](https://user-images.githubusercontent.com/115881685/225575828-df05340b-efae-4b9d-9bd0-230b129734d2.png)

![image](https://user-images.githubusercontent.com/115881685/225576061-5a34adcf-2872-45e7-b045-6de783858209.png)

![image](https://user-images.githubusercontent.com/115881685/225576714-edea1191-d52d-4b3b-a2bb-58aed88c652c.png)




We can now go on and create the application loadbalancer, see screenshots below for instructions.



![image](https://user-images.githubusercontent.com/115881685/225577953-3448d42c-efd9-4b55-8452-331e052a63a3.png)

![image](https://user-images.githubusercontent.com/115881685/225578179-ac11782e-63fe-4ecc-a34a-71a718a718a8.png)

![image](https://user-images.githubusercontent.com/115881685/225578484-081ec3f2-aaef-4efc-a90f-28a58ef5af62.png)

![image](https://user-images.githubusercontent.com/115881685/225578915-fb862d6f-a469-49d3-98c2-91a9c1e2bce4.png)

![image](https://user-images.githubusercontent.com/115881685/225579182-528e7d85-53b4-40e0-812a-d9f0465a47b4.png)

![image](https://user-images.githubusercontent.com/115881685/225579387-b8db40a4-11ed-4329-8fe4-577e58d05b15.png)
 
 
Then click create loadbalancer.



![image](https://user-images.githubusercontent.com/115881685/226195203-46f31f22-0fee-4bac-829a-6801d8900164.png)



We have successfully created the loadbalancer, wait for the status to change to active, then copy the the dns of the loadbancer and paste it in your browser, press enter.


![image](https://user-images.githubusercontent.com/115881685/226195239-3d6ea163-2fa7-447a-aa93-99850867e707.png)

![image](https://user-images.githubusercontent.com/115881685/226195261-1ee0b49e-ccce-4950-b29d-d075f5bd00ca.png)




We can now acceass our webiste using the dns name of the application loadbalancer.


Next step, terminate the setup server.



## Create a Record Set in Route 53
Next step is to create a record set in route 53 to access the website with our domain.

Go to the console and search route 53, select it under services, once in the route 53 dashboard, click on "hosted zone" and select your domain, on next page click "create record", under "record name", type in "www" then toggle on the "alias, scroll down and select the drop down, select "alias to application and classic loadbalancer", then select the "us-east-1", next select the drop down and select the application loadbalancer that was created. Then click "create record".




![image](https://user-images.githubusercontent.com/115881685/225618256-e6816b45-2ed8-4990-b4a2-fa3e10f51f86.png)

![image](https://user-images.githubusercontent.com/115881685/225618394-0ad70fc4-557a-4adb-a806-d81b06d594bc.png)

![image](https://user-images.githubusercontent.com/115881685/225618618-3c94e027-1d20-4c05-b1d2-b3f5c2ce237c.png)


Now the record set has been created, to access our website, copy the record name and paste it in your browser press enter.



![image](https://user-images.githubusercontent.com/115881685/226195956-e713e0f7-770b-42ee-9f0d-978258bf3c4c.png)


![image](https://user-images.githubusercontent.com/115881685/226195664-560c5818-6d81-4c52-b2f8-4679c0895f5e.png)



We can now access the website with our domain name. If you notice the website not loading properly, its because our domain has changed, we need to go into the website configuration files and update it, which we will do later.





## Register for an SSL Certificate in AWS Certificate Manager
The certificate manager encrypts all communication between the web browser and our web servers.

To create a free certificate, go to the certificate manager dashboard in the console, and follow the steps in the screenshots below.



![image](https://user-images.githubusercontent.com/115881685/225624926-a8d7dcc0-ddc3-46f1-ad79-bf67022a1160.png)
![image](https://user-images.githubusercontent.com/115881685/225625050-ec772b81-046b-4dc6-b128-305b4ed946c3.png)
![image](https://user-images.githubusercontent.com/115881685/225625247-4664ac7f-8eef-4b08-8179-8fc02062fb62.png)
![image](https://user-images.githubusercontent.com/115881685/225625381-6ed19d1a-97bd-4cc7-a81c-02f43fd2431c.png)


![image](https://user-images.githubusercontent.com/115881685/225627483-e9ea68cd-2229-4683-a73b-7006608cc87d.png)



We have successfully requested for a certificate, the status is pending because it needs to be validated in route 53. To do that click "create a record" in the page and follow the instructions in the screenshots.




![image](https://user-images.githubusercontent.com/115881685/225627634-44d5198c-13d5-4b20-9014-961d668ee4af.png)
![image](https://user-images.githubusercontent.com/115881685/225627831-5bce3113-e095-4ef6-bfb2-270ff830e4ad.png)
![image](https://user-images.githubusercontent.com/115881685/225630927-a8737485-0f82-4c1d-8cc7-283215b394f2.png)



If you now check the status, it has been "issued", and the status of our two name name is now "success"



## Create a HTTPS Listener
Next we will use the ssl certificate to secure all communication with our website by creating https listener in loadbalancer dashboard.

Follow the instructions in the screenshots.



![image](https://user-images.githubusercontent.com/115881685/226197176-b284a3c0-ae59-4406-9c0e-09a49e07dccc.png)

![image](https://user-images.githubusercontent.com/115881685/226197301-315f31f3-e5dc-457d-a34c-50fb9ccfc560.png)

![image](https://user-images.githubusercontent.com/115881685/226197376-bdf4ee55-3030-4105-abdc-7e5261732dfa.png)

![image](https://user-images.githubusercontent.com/115881685/226197705-0b2fb689-0f90-46f7-bf0f-ab94da3a732d.png)



The HTTPS listener has been created, next thing is to modify our HTTP Listener to redirect traffic to HTTPS. See screenshots below for instruction.


![image](https://user-images.githubusercontent.com/115881685/226197947-5794ff97-7bbc-485b-adca-3103ac22165f.png)

![image](https://user-images.githubusercontent.com/115881685/226198051-62a8e5b9-7b15-42b4-b37e-2d323e998574.png)

![image](https://user-images.githubusercontent.com/115881685/226198081-d6da80d9-1b16-4e28-8672-1dd605a16c3b.png)





Now you can check to see if you can access the website. Go to your browser and type in your domain name starting with "https//www.domain name".



![image](https://user-images.githubusercontent.com/115881685/226198255-1ed7ca7d-4864-43c9-bfe8-03dd3fefbb0f.png)




Yes, not only can we access the website, it now has the lock symbol which means all communication between our website and web browser is now secured. What we just did is also called "security in transit".




## SSH into an EC2 Instance in the Private Subnet

Next step is to ssh into the the instance in the private subnet, but to that we first need to launch an instance in the public subnet, this instance is called bastion host, we will ssh into it, and from there we can now ssh into the instance in the private subnet.

Follow the exact same step as before in launching instance in the public subnet.



![image](https://user-images.githubusercontent.com/115881685/226199010-6ce9e24b-9061-444c-b56e-1977dfb379fc.png)




Now copy the public ipv4 of the server and ssh into it.


![image](https://user-images.githubusercontent.com/115881685/226199147-de7fce88-2fb2-407e-81a6-00cf545d6d27.png)


Once you have ssh into the bastion host which is in the public subnet, from here we easily ssh into any instance in the private subnet.

To do this, simply type, "ssh ec2-user@private ipv4 address" of any of webserver in the private subnet.




![image](https://user-images.githubusercontent.com/115881685/226199475-206ce0ce-41c4-4e14-a61b-df2ba4c5bd5d.png)




![image](https://user-images.githubusercontent.com/115881685/226199551-c8b8d3bd-04a5-4e22-9c69-2fe904de9339.png)





We have successfully ssh into the instance in the private subnet, we can tell we are in the ec2 instance in the private subnet, because the ip address here, is the same as the webserver az1 in the ec2 dashboard





## Update the ENV File

IF you observer our website is not loading properly, this is because we need to update the domain name setting of our website.

To this, first terminate the Webserver AZ2 in the EC2 dashboard. Then SSH into Webserver AZ1 in the private subnet. Once you are in the type "sudo su" to be logged in as the root user.




![image](https://user-images.githubusercontent.com/115881685/226200701-8b4df8c6-f762-4e0e-93eb-15ddebc56687.png)




Then change our directory to the HTML directory by typing "cd /var/www/html" and press enter, next type in "nano .env" and enter. Env is the file we want to edit.




![image](https://user-images.githubusercontent.com/115881685/226201120-726d1890-b2ed-4a3f-814b-05ace4fb18f5.png)



![image](https://user-images.githubusercontent.com/115881685/226201648-2f93bbf6-78d2-4ad8-9a78-9b3fb60cdeea.png)



Once in edit the App URL by deleting the you see there and replacing it with the domain name of your website. When done save it and exit.





![image](https://user-images.githubusercontent.com/115881685/226203192-115ead02-1371-4e76-a2d0-897ed9b3e81c.png)




We have successfully updated the app url of the env file of our website, next restart the server by typing in "service httpd restart" and enter.





![image](https://user-images.githubusercontent.com/115881685/226203678-80fc5cc1-ab2f-4a96-aff1-c8ff5b931216.png)




We have successfully restart our apache webserver.



Then go to the website and click "refresh button"





![image](https://user-images.githubusercontent.com/115881685/226203804-006cd922-5be4-4be6-a4ac-52368b81c9cf.png)

![image](https://user-images.githubusercontent.com/115881685/226203902-3c5c59f9-0735-41eb-b182-fc7d739c06db.png)





There you go our website is now loading properly.





## Create Another AMI

Next, we will create a new AMI because we updated the configuration files of the website. Follow the same process as before.





![image](https://user-images.githubusercontent.com/115881685/226204219-3819ef18-a37b-4283-bc25-f1aaf838d02a.png)

![image](https://user-images.githubusercontent.com/115881685/226204242-cc4233f3-dd5b-468c-9d57-f1cc367e2ec6.png)

![image](https://user-images.githubusercontent.com/115881685/226204284-4d88e533-bad1-47e5-9b7b-cd170f0c2794.png)

![image](https://user-images.githubusercontent.com/115881685/226212842-19d6e588-4d31-4c0d-91fc-aeda63f276f8.png)


![image](https://user-images.githubusercontent.com/115881685/226204312-79c94abe-58c7-411a-9504-ef4f392f49fc.png)



Since we have updated the AMI, we can go ahead and delete the first AMI as well as its snapshot.





## Create Auto Scaling Group


 We need to create an autoscaling group to dynamically create and scale the webservers in the private app subnets.
 
 Before creating the autoscaling groups first terminate the web server AZ1 that was manually created.
 
 
 
 ![image](https://user-images.githubusercontent.com/115881685/226212974-4bb05c57-9b31-4651-be48-1bd2712432de.png)



 
Now go ahead, Follow the steps in the screenshots below, first we will create launch template.
The launch template contains configuration about our instance that autoscaling group will use to launch new instances.





![image](https://user-images.githubusercontent.com/115881685/226213030-0396ebcd-725d-4c60-947b-49ede96ca863.png)


![image](https://user-images.githubusercontent.com/115881685/226213068-ddf8fba5-b1f6-4a8d-b6d3-b629ccf86bc5.png)


![image](https://user-images.githubusercontent.com/115881685/226213114-9579bcbc-4dd5-4fbc-8ca3-27ea80b0b9e4.png)

![image](https://user-images.githubusercontent.com/115881685/226213147-f1cdb91e-b07d-4bda-a96b-5ec598658882.png)





The Launch template has been created, go on create the Auto-Scaling group.




![image](https://user-images.githubusercontent.com/115881685/225646762-c799ddd5-5e2c-4f79-93cc-01ecb897f33e.png)

![image](https://user-images.githubusercontent.com/115881685/225646962-554c6591-9901-4025-adbc-bdace4164a2a.png)

![image](https://user-images.githubusercontent.com/115881685/225647149-fc377b11-eb47-4736-87af-5355595b3646.png)

![image](https://user-images.githubusercontent.com/115881685/225647941-6cdc619b-fbdb-4c9e-9568-30e764850679.png)

![image](https://user-images.githubusercontent.com/115881685/225648143-eaa00795-4a6d-4215-af0c-070a82a14bd5.png)

![image](https://user-images.githubusercontent.com/115881685/225648301-9e4334e4-b9c7-4d3f-a36d-31d6d3d8234e.png)

![image](https://user-images.githubusercontent.com/115881685/225648588-2f364f9e-77cf-4ae0-be43-00198dc3e31d.png)

![image](https://user-images.githubusercontent.com/115881685/225648873-f5d665fd-9240-40ee-a112-1d32f9fee0e4.png)

![image](https://user-images.githubusercontent.com/115881685/225649073-c6babd13-c36e-481c-842d-03b074253bb5.png)

![image](https://user-images.githubusercontent.com/115881685/225649326-172d9019-0ac9-4ac7-9d0f-d6cd36b6e79d.png)


![image](https://user-images.githubusercontent.com/115881685/226214103-c18b6815-d947-4642-bf0a-fb94cd445c09.png)



Now go to the ec2 dashboard to see the two new instances created by our autoscaling group.





![image](https://user-images.githubusercontent.com/115881685/226213877-fddbad43-21c2-4743-b145-75e6d07dfa9f.png)





There you go two new webservers have been spun up by the Auto Scaling Group we just created, you can tell by their names because thats what we gave the in the ASG creation process, further more, under status, they are initializing.

To still know if we can access our website, type in your domain name in a browser and press enter.




![image](https://user-images.githubusercontent.com/115881685/226214073-8d376711-4729-43e1-ae6b-18f2e60c899b.png)


![image](https://user-images.githubusercontent.com/115881685/226214138-98fddaf7-b529-4f01-8c8a-3bf60cf5a7b5.png)




And there you go i can access my website and everything is working properly.

A big congratulations to you if made it to the end of this project.




### Pls note, delete all the resources/services created for this project to avoid charges.










