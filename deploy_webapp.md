# Deploy your containerized app to Azure App Service with Jenkins and the Azure CLI
# Part 3

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/23fb922a-3770-4845-8642-f7b5663cc2a4)

In this tutorial you will learn to dockerize your sample Java NodeJS application and deploy the same to Azure Web App, with the Azure CLI in a Jenkins Pipeline. 

Here are quick steps that we shall follow:

> * Create a Jenkins VM [Create a Jenkins Linux VM Part 1](./README.md)
> * Install Docker Engine on the Azure VM that is running Jenkins [Install Docker on a Linux VM Part 2](./install_docker_on_linux.md)
> * Create an Azure Service Principal
> * Create an Azure container registry
> * Create a Web App in Azure
> * Configure Jenkins to manage your Credentials
> * Create your Jenkins pipeline
> * Run the Build manually to Deploy the new image build to your Azure Web App
> * Prepare your GitHub repository with the App Code
> * Create the new Automated CI-CD Jenkins pipeline to automatically execute the Build and Deploy tasks
> * Run the pipeline and verify the Web App is updated with the latest image build

# Prerequisites

> * Azure subscription: If you don't have an Azure subscription, create a free account before you begin.
> * Jenkins - Install Jenkins on a Linux VM
> * Azure CLI: Install Azure CLI (version 2.0.67 or higher) on the Jenkins server.
> * Sample Application Code can be found at my Github Repository [HERE](https://github.com/mfkhan267/jenkins_on_azure2024.git)

# Create an Azure Service Principal

Automated tools like Jenkins, Terraform and others that use Azure services should always have restricted permissions to ensure that Azure resources are secure. Therefore, instead of having applications sign in as a fully privileged user, Azure offers service principals. An Azure service principal is an identity created for use with applications, hosted services, and automated tools. This identity is used to access Azure resources.

Let us now create a service principal with the `Contributor' role and scoped to my Tenant Subscription

      az ad sp create-for-rbac --name myappspn1 --role contributor --scopes /subscriptions/<Azure Tenant Subscription ID>

Output console should look like this:

      {
        "appId": "myAppId",
        "displayName": "myServicePrincipalName",
        "password": "myServicePrincipalPassword",
        "tenant": "myTentantId"
      }

Make a note of the above and keep them handy as you will need them later for managing credentials within Jenkins

# Create the Azure Container Registry

Azure Container Registry is a private registry service for building, storing, and managing container images and related artifacts. In this quickstart, we shall create an Azure container registry instance with the Azure portal. Then, use Docker commands to push a container image into the registry, and finally pull and run the image from the same registry.

Sign in to the Azure portal and create a container registry

Select Create a resource > Containers > Container Registry.

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/15ac0618-7cdc-4696-ac77-e5dba48dd802)

Create a new resource group in the East US 2 location named jenkinsRG, and a unique Registry name within Azure.

Accept default values for the remaining settings. Then select Review + create. After reviewing the settings, select Create.

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/886fcfe8-4a02-4b95-bc42-9ab71a9669a4)

Review the resource as below

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/09ceeac3-0834-4358-a0db-5d507a1820ef)

You may want to get the access keys to your Azure Container Registry as shown below (you will need this later on as we proceed further)

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/f12efb87-15ca-4590-b30d-5314ee2f7ef8)

# Create a Web App for Containers in Azure

App Service Web Apps lets you quickly build, deploy, and scale enterprise-grade web, mobile, and API apps running on any platform. Meet rigorous performance, scalability, security and compliance requirements while using a fully managed platform to perform infrastructure maintenance.

Let us now create ourselves a Web App for Containers

Sign in to the Azure portal and create a Web App for Containers.

Select Create a resource > Web App for Containers.

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/9adf3b84-1cb1-4f76-8174-1f736e6666d1)

Give your web app a unique App Name name within Azure.

Accept default values for the remaining settings. Then select Review + create. After reviewing the settings, select Create.

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/42957d0c-d64f-4083-a354-4d7350ff848d)

Once the Web App is created, you may review the settings as shown below.

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/60aa8dd8-2c15-4649-a88e-44556449e1a2)

You may now test your Web App by clicking on BROWSE to confirm that the Web App was deployed successfully.

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/050a566c-866f-419b-9eb8-9290e5cfe8ab)

# Configure Jenkins to manage your Credentials

Let us now create our credentials and secrets within Jenkins. This will allow the Jenkins Pipeline jobs to connect into our Azure Account and work with Azure resources like Web App and Azure Container Registry.

From the Jenkins Dashboard > Manage Jenkins > Credentials > System > Global credentials > Add credentials
   
## Add an Azure service principal inside the Jenkins Credential Manager

The following steps show how to manager your Azure credential with Jenkins:

1. Within the Jenkins dashboard, select **Credentials -> System ->**.

1. Select **Global credentials(unrestricted)**.

1. Select **Add Credentials** to add a Microsoft Azure service principal. Make sure that the credential kind is ***Username with password*** and enter the following items:

    * **Username**: Service principal `appId`
    * **Password**: Service principal `password`
    * **ID**: Credential identifier (such as `AzureServicePrincipal` OR `asp`)
  
![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/b7e51649-d6bf-450f-af0f-0d492b46dace)

Below are the Azure Container Registry credentials (Username and Password)

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/9e0b0a5e-3dc0-454c-a1a6-f2681ec5f450)

Below are the credentials (Username and Password) for the Azure Service Principal that you have created initially

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/66508068-cd11-4e89-ad8a-876c6352a545)

Below are the credentials (Secret Text) for the Azure Tenant ID

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/b2af9a0f-5263-4246-a906-4ccc79ff5e25)

# Ensure that your Azure VM has the NSG policy to allow inbound connections on the port that your application is listening on

Since my nodeJS application is listening on port 8080, you will need to open the VM Port accordingly

      az vm open-port \
      --resource-group jenkins267 \
      --name jenkinsvm267  \
      --port 8080 --priority 1010

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/dcac1982-06aa-4ece-a76a-5027549a4a58)

# Create your Jenkins pipeline (Manual Method)

## Docker Build, Push and Run

This is the pipeline code that we will use for this tutorial. It is also uploaded as a github repo HERE
     
      pipeline {
          agent any
          environment {
              AZURE_TENANT_ID = credentials('azure-tenant-id')
              ACR_REGISTRY = "acr267.azurecr.io"
              APP_REPO_NAME = "gsd"
          }
      
          stages {
              stage('git checkout') {
                  steps {
                    checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/mfkhan267/jenkins_on_azure2024.git']]])
                    sh 'pwd'
                    sh 'ls -ltr'
                    dir('app'){
                          sh 'pwd'
                              }
                          }
                                    }
              stage('build docker image'){
                  steps{
          	        dir('app'){
                          sh 'pwd'
                          sh 'docker build --force-rm -t ${ACR_REGISTRY}/${APP_REPO_NAME}:${BUILD_NUMBER} .'
                          sh 'docker image ls'
                              }
                  }
              }
              stage('push image'){
                  steps{
                      withCredentials([usernamePassword(credentialsId: 'acr', passwordVariable: 'password', usernameVariable: 'username')]) {
                      sh 'echo Pushing Image ${ACR_REGISTRY}/${APP_REPO_NAME}:${BUILD_NUMBER} to the ACR'
                      sh 'echo ${password} | docker login ${ACR_REGISTRY} --username ${username} --password-stdin'
                      sh 'docker push ${ACR_REGISTRY}/${APP_REPO_NAME}:${BUILD_NUMBER}'
                      }
                  }
              }
              stage('Install Azure CLI'){
                  steps{
                      sh '''
                      echo 'Installing Azure CLI'
                      cat /etc/os-release
                      hostname
                      hostnamectl
                      sudo apt-get update
                      curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
                      az --version
                      '''
                  }
              }
              stage('deploy web app'){
                  steps{
                      withCredentials([usernamePassword(credentialsId: 'asp', passwordVariable: 'password', usernameVariable: 'username')]) {
                      sh 'az login --service-principal -u ${username} -p ${password} --tenant ${AZURE_TENANT_ID}'
                      }
                      withCredentials([usernamePassword(credentialsId: 'acr', passwordVariable: 'password', usernameVariable: 'username')]) {
                      sh 'az webapp config container set --name tetris-webapp267 --resource-group jenkins267 --docker-custom-image-name ${ACR_REGISTRY}/${APP_REPO_NAME}:${BUILD_NUMBER} --docker-registry-server-url https://${ACR_REGISTRY} --docker-registry-server-user ${username} --docker-registry-server-password ${password}'
                      sh 'echo Successfully updated the tetris-webapp267 container app with the image version ${BUILD_NUMBER}'
                      }
                  }
              }
          }
          post {
              always {
                  echo 'Deleting all local images'
                  sh 'docker image prune -af'
                  sh 'docker logout'
                  sh 'az logout'
              }
          }
      }

Go to Jenkins >> Dashboard >> New Item >> Enter Job Name >> Select **Pipeline** >> Click **OK**

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/2319a7cc-ae8d-48d5-b1b5-f3ee6af05fda)

Paste the pipeline code into the Pipeline script box as shown below and click **Save**

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/2444d14c-0f2d-4298-a529-184f1e203eff)

# Deploy the new image build to your Azure Web App

On the Jenkins Pipeline Job page >> Click **Build Now**

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/9ac1a21e-72ef-40e2-9944-3502faa76101)

Wait until the Jenkins Pipeline build job is over. A graphic below the **Build History** heading indicates that the job is being executed or completed. Go to the job build and console output to see the results.

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/3654e0e8-dd51-4c37-b1c3-a0f85594ba83)

Congratulations! You have successfully **Deployed** your containerized app to your Azure Web App. Your Azure Web App should now be running your newly generated docker image with your App Code

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/a959bee1-0c3f-404f-b3e5-766d311ac295)

# Testing your Web App that should now be running your newly generated docker image with your App Code

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/168e4609-e4a7-4639-bf06-f55530b8501a)

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/f91973c4-bdf3-424c-90ba-8bac6cc6509e)

# Create your Jenkins pipeline (Fully Automated CI-CD Method)

## Docker Build, Push and Run

Create a new GitHub repository with your application code. We will need to create a webhook for the GitHub repository that should be able to remotely trigger your builds in Jenkins everytime application changes are commited to the above application code repository.

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/c7fb3930-8bb8-4c14-a098-5f65fa4e8e3c)

We will define the Jenkins Pipeline with the Peipeline script from SCM method as shown below

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/8b8fcb66-97ba-4f2a-bfaa-da0a73c50b61)

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/79fb0959-f78c-4f98-b603-4dad35fcc9cc)

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/ab82ba9c-661a-4443-849f-e80d3f13d375)

This should now allow the repository webhook to remote trigger the Build Jobs for the above pipeline in Jenkins, whenever you commit changes to your application code repository.
