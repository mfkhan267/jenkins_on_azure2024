# Deploy your containerized app to Azure App Service with Jenkins and the Azure CLI
# Part 3

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/23fb922a-3770-4845-8642-f7b5663cc2a4)

In this tutorial you will learn to dockerize your sample Java NodeJS application and deploy the same to Azure Web App, with the Azure CLI in a Jenkins Pipeline. 

Here are quick steps that we shall follow:

> * Create a Jenkins VM [Create a Jenkins Linux VM Part 1](./README.md)
> * Install Docker Engine on the Azure VM that is running Jenkins [Install Docker on a Linux VM Part 2](./install_docker_on_linux.md)
> * Run the Build manually to Deploy the new image build to your Azure Web App [Deploy Web App Part 3](./deploy_webapp.md)
> * Prepare your GitHub repository with the App Code
> * Create the new Automated CI-CD Jenkins pipeline to automatically execute the Build and Deploy tasks
> * Run the pipeline and verify the Web App is updated with the latest image build

# Prerequisites

> * Azure subscription: If you don't have an Azure subscription, create a free account before you begin.
> * Jenkins - Install Jenkins on a Linux VM
> * Azure CLI: Install Azure CLI (version 2.0.67 or higher) on the Jenkins server.
> * Sample Application Code can be found at my Github Repository [HERE](https://github.com/mfkhan267/jenkins_on_azure2024.git)

# Create your Fully Automated CI-CD Jenkins Pipeline

## Docker Build, Push and Run

Create a new GitHub repository with your application code. We will need to create a webhook for the GitHub repository that should be able to remotely trigger your builds in Jenkins everytime application changes are commited to the above application code repository.

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/d9537195-7f37-4353-8466-8c6d47668976)

We will define the Jenkins Pipeline with the Peipeline script from SCM method as shown below

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/8b8fcb66-97ba-4f2a-bfaa-da0a73c50b61)

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/24afb02b-d6c3-40f6-9a83-c0a3ef6d3ed2)

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/b2f5be21-3f09-453e-9aa6-d47f49c5ab3c)

This should now allow the repository webhook to remote trigger the Build Jobs for the above pipeline in Jenkins, whenever you commit changes to your application code repository.

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

## Next steps

[Create your Fully Automated CI-CD Jenkins Pipeline Part 4](./deploy_webapp_CICD.md)
