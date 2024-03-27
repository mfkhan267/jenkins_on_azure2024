# DEPLOY-WEBAPPS-TO-AZURE-APP-SERVICE-USING-JENKINS

- Welcome to this comprehensive guide on deploying five web applications to Azure App Service using Jenkins. This Readme provides detailed instructions on the process, ensuring seamless and efficient deployment for all five applications.

    - Even each app does not have a Readme , the steps outlined here apply uniformly to all of them. 
    
    ![Blank diagram (1)](https://user-images.githubusercontent.com/113396342/218916484-db7e75ab-407d-4b3f-90b7-201871786329.png)

    
#

# Let's get started!
      
      
   APPS :
- Tetris-Game-App
- Apprun-realworld-example-app
- Crizmas-mvc-realworld-example-app
- React-redux-realworld-example-app
#

Prerequisites :
- An Azure account with an active subscription. <a href="https://azure.microsoft.com/en-us/free/?WT.mc_id=A261C142F" target="_blank">Create an account for free.</a> 
#

# Project Task and Steps:
1- Create a Azure Container Registry >>> <a href="https://github.com/hkaanturgut/DEPLOY-WEBAPPS-TO-AZURE-APP-SERVICE-USING-JENKINS/tree/main/terraspace%20codes/app/stacks/acr" target="_blank">ACR Terraspace Codes</a> 

![Screenshot 2023-02-14 at 9 00 31 PM](https://user-images.githubusercontent.com/113396342/218908353-53eb631b-d4c0-4f92-96c8-e5663679417b.png)

#

2- Provision a Jenkins Server , can use dind ( docker in docker container which contains Jenkins ) >> <a href="https://github.com/hkaanturgut/container-jenkins-with-nodejs?organization=hkaanturgut&organization=hkaanturgut" target="_blank">ACR Terraspace Codes</a> 

![Screenshot 2023-02-14 at 9 07 25 PM](https://user-images.githubusercontent.com/113396342/218909111-db4de5d2-a46a-4b66-8a7c-1f2cf0424f16.png)

- You know what to do ! Go ahead and set up your Jenkins 

![Screenshot 2023-02-14 at 9 08 24 PM](https://user-images.githubusercontent.com/113396342/218909324-886f597a-c122-4be2-b076-efaf2df1e2d9.png)

#

3- Once the Jenkins is all set up , install the needed plugins ( Azure app Service )

   - From Dashboard > Manage Jenkins > Manage Plugins > Available Plugins 
   
     - Once select the plugins , select install without restart
   
   ![Screenshot 2023-02-10 at 12 05 54 PM](https://user-images.githubusercontent.com/113396342/218909850-cfd095ba-6c0c-4cba-b0ca-d5ab361799a5.png)
   #
   
4- Credentials need to be added in order to have connection with ACR and our Azure account

   - From Dashboard > Manage Jenkins > Credentials > System > Global credentials > Add credentials
   
   ![Screenshot 2023-02-10 at 12 10 31 PM](https://user-images.githubusercontent.com/113396342/218910382-b6913906-e7f6-43a1-ba17-4e927a117fc1.png)

- Add the selected container registry credentials , in this project it is ACR.

![Screenshot 2023-02-10 at 12 10 56 PM](https://user-images.githubusercontent.com/113396342/218910527-eb03c2ca-a840-45db-a42c-64c6b7ec2a5e.png)

- In order to have connection with our Azure account , I created service principal and made the connection with Jenkins

![Screenshot 2023-02-10 at 12 14 43 PM](https://user-images.githubusercontent.com/113396342/218910777-00c73cc2-eac1-4871-a52f-1029264ba964.png)
#

## Here are the needed credentials to be able to keep going for the further steps

![Screenshot 2023-02-10 at 12 14 59 PM](https://user-images.githubusercontent.com/113396342/218910935-5c26dda0-7e69-43ea-8652-097e245705c1.png)
#

5- Let's create the pipeline.

![Screenshot 2023-02-10 at 12 15 39 PM](https://user-images.githubusercontent.com/113396342/218911109-6d34c9e1-b951-419d-beaf-6b4144e15079.png)
#

6- Starting our pipeline firstly by checking our code from the repository
     
        - pipeline{
    agent any
    stages{
        stage('git checkout'){
            steps{
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/hkaanturgut/Tetris-App-Jenkins.git']])
            }
         }
      }
    }

![Screenshot 2023-02-10 at 12 17 52 PM](https://user-images.githubusercontent.com/113396342/218911409-4a5a20ab-c775-4b22-8ea2-aa63cfb6343e.png)

  - Click Build Now and here is the succesfull step
  
![Screenshot 2023-02-10 at 12 20 28 PM](https://user-images.githubusercontent.com/113396342/218911715-07b2ddfb-3c9f-4f8e-a593-06906c23611a.png)
#

7- Build Docker image and push it to the ACR

        - stage('build docker image'){
            steps{
                sh 'docker build -t jenkinsacrkaan.azurecr.io/tetris .'
            }
        }
        stage('push image'){
            steps{
                withCredentials([usernamePassword(credentialsId: 'ACR', passwordVariable: 'password', usernameVariable: 'username')]) {
                sh 'docker login -u ${username} -p ${password} jenkinsacrkaan.azurecr.io'
                sh 'docker push jenkinsacrkaan.azurecr.io/tetris'
                }
            }
        }
        
![Screenshot 2023-02-10 at 12 22 28 PM](https://user-images.githubusercontent.com/113396342/218911871-2115090e-9485-4b5c-9ee7-6e07d8884fdc.png)

   - Click Build Now and the image is build and pushed to the ACR
   
   ![Screenshot 2023-02-10 at 12 25 59 PM](https://user-images.githubusercontent.com/113396342/218911955-84f31a16-3882-4e6b-bf2c-53a65bf81c2b.png)
   
   ![Screenshot 2023-02-10 at 12 26 16 PM](https://user-images.githubusercontent.com/113396342/218912046-c7442dcb-0da7-4825-b31b-a2d970ae7bb3.png)

#

8- Install Azure CLI into the pipeline

        - stage('install Azure CLI'){
            steps{
                sh '''
                apk add py3-pip
                apk add gcc musl-dev python3-dev libffi-dev openssl-dev cargo make
                pip install --upgrade pip
                pip install azure-cli
                '''
            }
        }

![Screenshot 2023-02-10 at 12 28 06 PM](https://user-images.githubusercontent.com/113396342/218912113-15f70626-9a53-4e05-9937-0958d6c71fbd.png)

   - Click Build Now and Azure CLI is installed into the pipeline
   
![Screenshot 2023-02-10 at 12 30 43 PM](https://user-images.githubusercontent.com/113396342/218912269-31caa05e-07d7-42c4-8770-2688711a3d9f.png)

#

## RELEASE TO AZURE WEB APPS

- Create a Azure Linux Web App for Containers >>> <a href="https://github.com/hkaanturgut/DEPLOY-WEBAPPS-TO-AZURE-APP-SERVICE-USING-JENKINS/tree/main/terraspace%20codes/app/stacks/tetris-game_linux_webapp" target="_blank">App Service Terraspace Codes</a> 

![Screenshot 2023-02-10 at 12 38 54 PM](https://user-images.githubusercontent.com/113396342/218912828-c08c57c6-d536-4c0f-a190-5b81d96db984.png)
#


### Deploy to Web App

         - stage('deploy web app'){
            steps{
                withCredentials([azureServicePrincipal('azureServicePrincipal')]) {
                sh 'az login --service-principal -u ${AZURE_CLIENT_ID} -p ${AZURE_CLIENT_SECRET} --tenant ${AZURE_TENANT_ID}'
                }
                withCredentials([usernamePassword(credentialsId: 'ACR', passwordVariable: 'password', usernameVariable: 'username')]) {
                sh 'az webapp config container set --name tetris-game-webapp --resource-group Tetris-App-RG --docker-custom-image-name jenkinsacrkaan.azurecr.io/tetris:latest --docker-registry-server-url https://jenkinsacrkaan.azurecr.io --docker-registry-server-user ${username} --docker-registry-server-password ${password}'
                }
            }
        }

![Screenshot 2023-02-10 at 12 46 20 PM](https://user-images.githubusercontent.com/113396342/218912979-41837393-ea3d-4fef-8901-fb55ade5c0b3.png)

  - Click Build Now and App deployed to Azure App Service
  
![Screenshot 2023-02-10 at 12 52 16 PM](https://user-images.githubusercontent.com/113396342/218913099-5ea5abba-6c0c-4999-a901-fd4c5dbfe932.png)
#

## Make sure to add WEBSITES_PORT setting from ;

    - Configuration > Application Settings > Add WEBSITES_PORT ( add your app's port ) > Save

![Screenshot 2023-02-10 at 12 53 02 PM](https://user-images.githubusercontent.com/113396342/218913442-d08e81fd-58cc-49e5-b2ca-ae25e0b17e9f.png)
#

# APP IS WORKING

![image](https://user-images.githubusercontent.com/113396342/218913540-1439882a-3208-43ef-9a03-fd9cea84dc96.png)

## Thank you for taking the time to read this blog.I hope that the information provided here proves to be useful in your efforts to efficiently deploy your applications. Best of luck.




   
