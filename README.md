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

### - Creating S3 bucket

#### Command : 
    aws s3 mb s3://s3-name --profile profilename

### AWS CLI command to setup RDS(MariaDB 10.6)

### - Creating RDS 

#### Command : 
    aws rds create-db-instance --db-instance-identifier db-linuxhint --db-instance-class db.t2.micro --engine MariaDB --master-user-password 123456789 --master-username ottdemo --allocated-storage 20 --backup-retention-period 7 --profile OTT-demo


## AWS CLI commands to setup auto scaling groups



### i. Creating launch template

#### Command : 
    aws ec2 create-launch-template --launch-template-name ott-tempplate --version-description V1 --launch-template-data file://config.json --profile OTT-demo

##### config.json
      {
         "TagSpecifications": [{
            "ResourceType": "instance",
            "Tags": [{
               "Key": "purpose",
               "Value": "webserver"
            }]
         }],
         "ImageId": "ami-0f9fc25dd2506cf6d",
         "InstanceType": "t2.micro",
          "KeyName": "OTT",
          "SecurityGroupIds": ["sg-0d0e4f55c99ec9dc8"]
      }
### ii. Creating auto scaling group

#### Command : 
    aws autoscaling create-auto-scaling-group --auto-scaling-group-name ott-autoscale --launch-template LaunchTemplateId=lt-03dcba521360869c1,Version='$Latest' --min-size 1 --max-size 5 --desired-capacity 3 --vpc-zone-identifier "subnet-08cb168de3c6d9514" --profile OTT-demo
    
# Icareus OTT/Video Platform - Mikko (terraform script to install docker / make a container up)

### Prerequisites are : 
   * Create your project docker images
   * Push the image to your docker hub repository
   * Storing docker hub password in a file here (password.txt)

![image](https://user-images.githubusercontent.com/90750345/214501454-02e9f846-cac2-4328-a0d6-adb1bc2079a7.png)

   * Creating a script file here (container.sh)
   
   ![image](https://user-images.githubusercontent.com/90750345/214502271-01caa83e-1e54-478c-bbc4-acd481246177.png)
 
         #!/bin/bash
         cat ~/password.txt | sudo docker login --username name --password-stdin
         sudo docker pull username/imagename:version
         sudo docker run -d -p 8080:8080 username/imagename:version

   * eating a script with docker installation command (docker.sh)
   
   ![image](https://user-images.githubusercontent.com/90750345/214502498-4931c13f-c3e2-4724-ac64-71d33f8750a8.png)

   
    #!/bin/bash
    set -ex
    sudo yum update -y
    sudo amazon-linux-extras install docker -y
    sudo service docker start
    sudo usermod -a -G docker ec2-user
    sudo curl -L https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
    
   ## Create a .tf file with below contents
   
      resource "aws_instance" "docker" {
        ami           = "ami-0b5eea76982371e91"
        instance_type = "t2.micro"
        key_name = "OTT"
        security_groups = [ "OTT" ]
        user_data = file("docker.sh")

        root_block_device {
          volume_size = 8
        }

        tags = {
          Name = "Docker"
        }

      }

      resource "null_resource" "docker" {

      # ssh into the server
        connection {

          type  = "ssh"
          user  = "ec2-user"
          private_key = file ("~/terraform/aws/OTT.pem")
          host  =  aws_instance.docker.public_ip

        }


      #importing docker password into instance
        provisioner "file" {

          source = "password.txt"
          destination = "/home/ec2-user/password.txt"

        }

      #importing sh to login docker /pull/run the docker image 
        provisioner "file" {

          source = "container.sh"
          destination = "/home/ec2-user/container.sh"

        }

      # to run command inside the instance
        provisioner "remote-exec" { 
          inline = [      
            "sudo chmod +x /home/ec2-user/container.sh",
            "sh /home/ec2-user/container.sh",
          ]
        }

      #creating dependency on instance
        depends_on = [aws_instance.docker]

      }

### output in Instance 
 
![image](https://user-images.githubusercontent.com/90750345/214502857-eb469033-de0e-40b9-a745-c4cfd8ecc3f2.png)

---------------------------------------------------------------------------------




