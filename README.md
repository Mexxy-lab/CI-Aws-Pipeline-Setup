## Continous integration project set up using AWS services, Code build, Code artifacts, Code pipeline, Code deploy, Sonarqube Cloud

- This project involves setting up of a continous integration pipeline for v-profile application written in Java. We would be using the below AWS web services for this project:

    - Code Commit 
    - Code Build
    - Code Artifacts
    - Code Pipeline
    - Code Deploy 
    - S3 storage
    
## Project Architecture 

![alt text](pictures/image.png)

## Project Steps 

1) Create a Code commit repository and configure it to connect from your local PC terminal. We would be using SSH connection

```bash
aws codecommit create-repository --repository-name pumej-vprofile-repo --repository-description "Repository for vprofile java project"
ssh-keygen -t rsa -b 4096 -C "pumej-codecommit" -f ~/.ssh/vpro-codecommit_rsa
cat ~/.ssh/vpro-codecommit_rsa.pub                                  | Cat the pub key and upload it under the SSH section of your IAM user. 
nano ~/.ssh/config                                                  | Create a ssh config file for codecommit and update file with the generated ssh key from console. 
ssh git-codecommit.us-east-1.amazonaws.com                          | Use this to confirm authentication to the repo. 
```

2) Create the Artifacts service using Code Artifacts service. 

    - Create a repository (	vprofile-maven-repo) in Artifacts console and make sure to select maven central store uder "Public upstream repositories". This repo would store the dependencies for our store / project.
    - Set a domain pathname and also the AWS account for it. Mine was vpropath. 
    - Choose option for AWS maganaged key. 
    - You can now view instructions on how to access the repository either pull or push to it. Use the instructions to update pom and settings file exactly. 
    - Update the setting.xml / Pom.xml files with new artifacts url, domain name and url in mirrors section. Use the exact instructions to update your files.  

```bash
export CODEARTIFACT_AUTH_TOKEN=`aws codeartifact get-authorization-token --domain vpropath --domain-owner 598189530267 --region us-east-1 --query authorizationToken --output text`
echo $CODEARTIFACT_AUTH_TOKEN                   | Copy the output and update it to aws parameter store 
``` 

3) Create or Set up Sonarqube cloud 

    - Create a sonarcloud account here -sonarcloud.io
    - Generate a token from my account section 
    - Create a project and assign a project key, just like you do in sonarqube server set up with jenkins. 
    - Update the parameters to Secret Manager parameter store 

        - HOST=https://sonarcloud.io
        - Organization=organization key in sonar cloud
        - Project=project-key
        - codeartifact-token=Auth token
        - LOGIN=sonartoken

    - We need these to build the build project next. 

4) Creating the Build project using Code Build service - vprofile-build

    - Select create project and fill out the details of your build job. 
    - This would also create a service role for the build job. Update this role to grant it access to the Parameter store and all other required permissions. 
    - Choose option to use a buildspec file, and specify the path or you can copy buildspec file content 
    - Run the build for both builds and confirm successful, you should see the dependencies been added to the codeartifacts repository. 

5) Create an SNS notification for the build process 

    - Go to SNS service and create a topic for the project - vprofile-pipeline-notification
    - Create a subscription under the topic - Protocol type should be email or SMS. 
    - We can now use the topic name to send notifications during build jobs. 

6) Create a pipeline job to connect every build

    - Go to code pipeline and select create Pipeline 