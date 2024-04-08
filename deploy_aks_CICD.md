# Automatically deploy your containerized app to Azure Kubernetes Service with Jenkins CI-CD Pipeline using GitHub Webhook
# Part 5

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/23fb922a-3770-4845-8642-f7b5663cc2a4)

This is final part of the Jenkins CI-CD Pipeline series where we will be deploying our containerized app to Azure Kubernetes Service. You havent gone through the previous tutorials of this series, I highly recommend you to review them before this one for better continous understanding.

Here are quick steps that we shall follow:

> * Create a Jenkins VM [Create a Jenkins Linux VM Part 1](./README.md)
> * Install Docker Engine on the Azure VM that is running Jenkins [Install Docker on a Linux VM Part 2](./install_docker_on_linux.md)
> * Run the Build manually to Deploy the new image build to your Azure Web App [Deploy Web App Part 3](./deploy_webapp.md)
> * Prepare your GitHub repository with the App Code
> * Create the new Automated CI-CD Jenkins pipeline to automatically execute the Build and Deploy tasks
> * Run the pipeline and verify the Azure Kubernetes Service Cluster is updated with the latest image build

# Prerequisites

> * Azure subscription: If you don't have an Azure subscription, create a free account before you begin.
> * Jenkins - Install Jenkins on a Linux VM
> * Azure CLI: Install Azure CLI (version 2.0.67 or higher) on the Jenkins server.
> * Sample Application Code can be found at my Github Repository [HERE](https://github.com/mfkhan267/jenkins_on_azure2024.git)

# Create your Fully Automated CI-CD Jenkins Pipeline

## Docker Build, Push and Run

Create a new GitHub repository with your application code. We will need to create a webhook for the GitHub repository that should be able to remotely trigger your builds jobs in Jenkins everytime application changes are commited and pushed to the above application code repository.

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/d9537195-7f37-4353-8466-8c6d47668976)

We will define the Jenkins Pipeline with the Pipeline script from SCM method as shown below

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/b5daed29-e442-4a8d-afb8-3a6005083f8c)

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/2d44e9d1-3a0d-4cf1-90c9-5f0e62aa265f)

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/c9068614-a851-46dc-98b9-7b4effa251b5)

This should now allow the GitHub repository webhook to remote trigger the Build Jobs for the above pipeline in Jenkins, whenever you commit changes to your application code repository.

# Commit your application code changes and push the Application code repository on GitHub

The GitHub webhook should now trigger the Jenkins Pipeline Job and the Build Job should run automatically.

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/e9ce5161-084c-43fc-9ecc-0610fb4c04a9)

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/db0dd510-cb59-4a88-a270-fb6e5cac621d)

Wait until the Jenkins Pipeline build job is over. A graphic below the **Build History** heading indicates that the job is being executed or completed. Go to the job build and console output to see the results.

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/3654e0e8-dd51-4c37-b1c3-a0f85594ba83)

Congratulations! You have successfully **Deployed** your containerized app to your Azure Web App. Your Azure Web App should now be running your newly generated docker image with your App Code

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/a959bee1-0c3f-404f-b3e5-766d311ac295)

# Testing your Web App that should now be running your newly generated docker image with your App Code

Below is Application v13 running as a Deployment on the AKS CLuster (your build version may differ)

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/b5cc6c57-a124-4afb-9b70-f66746cefc66)

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/6debde58-3892-4198-9252-7c39e4dcd588)

Commit and push your changes to your application code on GitHub and the GitHub webhook should trigger the Jenkins Pipeline Build automatically.
The Jenkins Pipeline will automatically Build, Push and Deploy your containerized Application to the Azure Web App as part of your complete CI-CD Job.
The Application should now be running a new Build as your Web App (your build version differ)

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/949a003f-c4b3-4977-9247-aeee6c719a15)

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/d40cb5a4-fc22-4734-8a0f-a827d1a2f070)

## Next steps

[Create your Fully Automated CI-CD Jenkins Pipeline to Deploy your Application to an AKS Cluster Part 4](Coming Soon)
