# Setting up AWS Instance and Building a complete CI/CD Pipeline 

## Table of Contents

- [Project Description](#project-description)

- [Technologies Used](#technologies-used)

- [Aws Steps](#aws-steps)

- [CI CD Steps](#ci-cd-steps)

- [Installation](#installation)

- [Usage](#usage)

- [How to Contribute](#how-to-contribute)

- [Tests](#test)

- [Questions](#questions)

- [References](#references)

- [License](#license)


## Project Description

- Configured AWS CLI

- Created VPC 

- Created EC2 instance 

- SSH into the server and installed docker on it 

- Set up Completed CI/CD pipeline 
  
  * Added docker-compose for deployment 
  
  * Configured acces from browser (ec2 security group)

  * Configured automatic triggering of multi-branch pipeline 

## Technologies Used 

* AWS 

* Jenkins

* Nexus

* Docker 

* Linux 

* Git 

* Docker hub 

* Node 

## Aws Steps 

Step 1: Create IAM user 

[IAM user creation](/images/Aws/01_creating_iam_user.png)

Step 2: Create group you want this user to be in 

[Group policy1](/images/Aws/02_added_correct_policy.png)
[Group policy2](/images/Aws/02_devops_policies.png)
[Group permissions](/images/Aws/02_devops_group_with_permissions.png)
[Final page](/images/Aws/03_final_setup.png)

Step 3: Log into AWS CLI temporary as user 

     export AWS_ACCESS_KEY_ID=*****************
     export AWS_SECRET_ACCESS_KEY=*************

[Logged in AWS CLI](/images/Aws/04_logging_into_AWS_temporary_as_user.png)

Step 4: Create VPC 

     aws ec2 create-vpc --cidr-block 10.0.0.0/16

[VPC](/images/Aws/05_created_vpc.png)

Step 5: Create a subnet for vpc 

     aws ec2 create-subnet --vpc-id vpc-090d737dd5391344a --cidr-block 10.0.1.0/24

[Subnet](/images/Aws/06_created_subnet_for_vpc.png)

Step 6: Create Security group 

     aws ec2 create-security-group --group-name devops-sg --description "devops" --vpc-id vpc-090d737dd5391344a

[Security Group](/images/Aws/07_created_security_group.png)

Step 7: Create internet gateway 

     aws ec2 create-internet-gateway

[Internet Gateway](/images/Aws/08_creating_internet_gateway.png)

Step 8: Attach internet gateway to vpc 

     aws ec2 attach-internet-gateway --vpc-id vpc-090d737dd5391344a --internet-gateway-id 09c96042fb0297e79

[Attached IG to vpc](/images/Aws/09_attaching_internet_gateway_to_vpc.png)

Step 9: Create route table 

     aws ec2 create-route-table --vpc-id vpc-090d737dd5391344a

[Route table](/images/Aws/10_create_route_table.png)

Step 10: Create a route

      aws ec2 create-route --route-table-id rtb-02c4d1582bc5a706f --destination-cidr-block 0.0.0.0/0 --gateway-id igw-09c96042fb0297e79

[route](/images/Aws/11_creating_a_route.png)

Step 11: Associate the route table with the subnet created

     aws ec2 associate-route-table --route-table-id rtb-02c4d1582bc5a706f --subnet-id subnet-0adba31976b134a0f

[associated route table with subnet](/images/Aws/12_associating_route_table_with_subnet.png)

Step 12: Open port 22 fo ssh access in security group

[Opened port 22](/images/Aws/13_opening_port_22_for_sg.png)

Step 13: Create key pair 

     aws ec2 create-key-pair --key-name --query 'KeyMaterial' --output text > devops_key.pem

[Key pair](/images/Aws/14_creating_key_pair.png)

Step 14: Start instance 

     aws ec2 run-instances --image-id ami-0648ea225c13e0729 --count 1 --instance-type t2.micro --key-name devops-key --security-group-ids sg-08025ec20b8660a5b --subnet-id subnet-0adba31976b134a0f

[Instance](/images/Aws/15_starting_instance.png)

Step 15: Give key user only read permissions 

     chmod 400 .ssh/devops-key.pem 

note: I carried out some changes at the end because AWS didn't randomly generate public ip address for me. These changes are available below 

[Changes 1](/images/Aws/17_.png)
[Change 2](/images/Aws/17_enabling_auto_assign_public_ipv4.png)

Step 16: ssh into server 

     ssh -i .ssh/MykpCli.pem ec2-user@13.40.167.66

[ssh into ec2 instance](/images/Aws/18_ssh_into_ec2_instance.png)


# CI CD Steps 

Step 1: Download ssh plugin and add key into multibranch credential 

[key in m credentials](/images/CICD/01_adding_key_into_jenkins_multibranch_credentials.png)

Step 2: Install docker compose on ec2 server 

[Installed docker compose](/images/CICD/02_install_docker_compose_on_ec2_instance.png)

Step 3: Create docker compose file that will run Nodejs application 

[Docker compose](/images/CICD/03_create_your_docker_compose_file.png)

Step 4: Insert deploy stage in Jenkinsfile so you could copy docker compose file and a script to the ec2 instance and run nodejs application as a container

[deploy stage](/images/CICD/04_insert_deploy_stage_in_jenkins_file.png)

Step 5: Create a script that runs the docker compose files and sets environmental variale on server for dynamic versioning 

[script](/images/CICD/05_insert_a_script_that_run_docker_compose.png)

Step 6: Configure security group to allow ssh access to jenkins ip 

[ssh access to jenkins ip](/images/CICD/06_allowing_ssh_access_to_jenkins_server.png)

Step 7: Check Pipeline 

[Successful Pipeline](/images/CICD/07_successful_pipeline.png)
[Image in dockerhub](/images/CICD/08_image_in_docker_hub.png)

Step 8: Configure port 3000 to allow any ip address to gain access to application

[port 3000](/images/CICD/09_opening_port_3000_to_allow_application_be_accessible_through_browser.png)

[Application on Browser](/images/CICD/10_application_on_browser.png)

Step 9: Adding conditional to only allow main branch to build and deploy application

[Expressions](/images/CICD/11_only_main_builds_and_pushes.png)
[Successful Pipeline 2](/images/CICD/12_successful_pipeline.png)

Step 10: Download webhook trigger plugin on jenkins and configure it on jenkins to enable auto triggered pipeline jobs 

[Webhook jenkins](/images/CICD/13_webhook_configuration_on_jenkins.png)

Step 11: Configure webhook in gitlab setting to enable auto trigerred pipeline jobs after every push 

[Webhook gitlab](/images/CICD/14_webhook_configuration_on_gitlab.png)

Note: I have built a project specifically for auto triggering jobs, if you want more information there is a link to the gitlab repo below 

-->  https://gitlab.com/solomoncode26/trigger-ci-pipeline-automatically-on-every-change.git  <--

## Installation

     DOCKER_CONFIG${DOCKER_CONFIG:-$HOME/.docker}
     mkdir -p $DOCKER_CONFIG/cli-plugins
     curl -SL https://github.com/docker/compose/releases/download/v2.12.2/docker-compose-linux-x86_64 -o $DOCKER_CONFIG/cli-plugins/docker-compose

## Usage 

Run $ node server.js

## How to Contribute

1. Clone the repo using $ git clone git@gitlab.com:omacodes98/nodejs-complete-ci-cd-pipeline.git

2. Create a new branch $ git checkout -b your name 

3. Make Changes and test 

4. Submit a pull request with description for review

## Tests

Test were ran using mvn test.

## Questions

Feel free to contact me for further questions via: 

Gitlab: https://gitlab.com/solomoncode26

Email: soburabari@gmail.com

## References

This project: https://gitlab.com/solomoncode26/nodejs-complete-ci-cd-pipeline

Auto Triggering jobs: https://gitlab.com/omacodes98/trigger-ci-pipeline-automatically-on-every-change.git

## License

The MIT License 

For more informaation you can click the link below:

https://opensource.org/licenses/MIT

Copyright (c) 2022 Solomon Ntor.

## License