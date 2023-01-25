# Icareus-OTT-Video-Platform-aws-terraform
   ### Connecting with IAM user  to awsCLI
   #### Command: 
      aws configure –profile profile-name
   
### Prerequisites are : 
* key (.pem file) 
* security group (if not present command are given below)

### 1. Creating a .pem file (Key)

#### Command : 
      aws ec2 create-key-pair --key-name keyname --query 'KeyMaterial' --output text > pem-file-name  --profile profile name

### 2. Creating a security group 

#### Command : 
      aws ec2 create-security-group --group-name security-group-name --description " OTT security group" --vpc-id vpc id  --profile profile name

### 3. Authorising ports to security group

#### Command : 
    aws ec2 authorize-security-group-ingress --group-name OTT  --protocol tcp --port 22 --cidr 0.0.0.0/0 --profile OTT-demo


 ### AWS CLI commands to setup AWS s3 bucket for storage

###  Creating S3 bucket

#### Command : 
    aws s3 mb s3://s3-name --profile profilename

### AWS CLI command to setup RDS(MariaDB 10.6)

###  Creating RDS 

#### Command : 
    aws rds create-db-instance --db-instance-identifier db-linuxhint --db-instance-class db.t2.micro --engine MariaDB --master-user-password 123456789 --master-username ottdemo --allocated-storage 20 --backup-retention-period 7 --profile OTT-demo







AWS CLI commands to setup auto scaling groups



i.    Creating launch template

Command : aws ec2 create-launch-template --launch-template-name ott-tempplate --version-description V1 --launch-template-data file://config.json --profile OTT-demo

config.json
