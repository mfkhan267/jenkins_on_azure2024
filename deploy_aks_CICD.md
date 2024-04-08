# Automatically deploy your containerized app to Azure Kubernetes Service with Jenkins CI-CD Pipeline using GitHub Webhook
# Part 5

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/23fb922a-3770-4845-8642-f7b5663cc2a4)

This is final part of the Jenkins CI-CD Pipeline series where we will be deploying our containerized app to Azure Kubernetes Service. You havent gone through the previous tutorials of this series, I highly recommend you to review them before this one for better continous understanding.

Here are quick steps that we shall follow:

> * Create a Jenkins VM [Create a Jenkins Linux VM Part 1](./README.md)
> * Install Docker Engine on the Azure VM that is running Jenkins [Install Docker on a Linux VM Part 2](./install_docker_on_linux.md)
> * Run the Build manually to Deploy the new image build to your Azure Web App [Deploy Web App Part 3](./deploy_webapp.md)
> * Prepare your GitHub repository with the Application Code.
> * Deploy a sample NodeJS Application to an AKS cluster.
> * Create a basic Jenkins project.
> * Set up credentials for Jenkins to interact with ACR.
> * Create a Jenkins build job and GitHub webhook for automated builds.
> * Test the CI/CD Jenkins pipeline to update the application in AKS based on GitHub code commits.

# Prerequisites

> * Azure subscription: If you don't have an Azure subscription, create a free account before you begin.
> * Jenkins - Install Jenkins on a Linux VM
> * Azure CLI: Install Azure CLI (version 2.0.67 or higher) on the Jenkins server.
> * Sample Application Code can be found at my Github Repository [HERE](https://github.com/mfkhan267/jenkins_on_azure2024.git)

# Create your Fully Automated CI/CD Jenkins Pipeline

## GitHub Repository with Sample Application Code

Create a new GitHub repository with your application code. Sample Application Code can be found at my Github Repository [HERE](https://github.com/mfkhan267/jenkins_on_azure2024.git)

## Docker Build, Push and Run

## GitHub Webhook

We will need to create a webhook for the GitHub repository that should be able to remotely trigger your builds jobs in Jenkins everytime application changes are commited and pushed to the above application code repository.

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/d9537195-7f37-4353-8466-8c6d47668976)

We will define the Jenkins Pipeline with the Pipeline script from SCM method as shown below

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/b5daed29-e442-4a8d-afb8-3a6005083f8c)

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/2d44e9d1-3a0d-4cf1-90c9-5f0e62aa265f)

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/c9068614-a851-46dc-98b9-7b4effa251b5)

This should now allow the GitHub repository webhook to remote trigger the Build Jobs for the above pipeline in Jenkins, whenever you commit changes to your application code repository.

# Deploy a sample NodeJS Application to an AKS cluster

Let us login to the Azure Portal >> Search >> Azure Kubernetes Servies >> Create Kubernetes Cluster with configurations as shown below

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/abdd88a1-149c-42ce-a71b-6cf83ba63bcb)

Click Add node pool >> Give the pool a name (nplinux OR npwindows) with User Mode and Node Size as D2s_v3 >> Add >> Review and Create

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/f7cccc1d-3f60-47eb-bb84-988f3e7230e2)

# Connecting to the AKS Cluster

Let us now launch the Azure Cloud Shell. The Cloud Shell has the kubectl pre-installed.

    az aks get-credentials --resource-group aks_RG267 --name aks_demo267

This command downloads credentials and configures the Kubernetes CLI to use them. To verify our connection with the AKS Cluster, let us run the following

    kubectl get nodes

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/e9d28347-20b6-4a48-85ab-d67dde424596)

Execute the below command to get the kubeconfig info, we will need to copy the entire content of the file to txt file that we will use to create a Jenkins Credentials with a secret file and name it "aks"

    cat ~/.kube/config

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/f0aa8915-872a-4e8d-870f-25d45cd42834)

How to verify integration between ACR and AKS Cluster?

az role assignment list --scope /subscriptions/<subscriptionID>/resourceGroups/<resourcegroupname>/providers/Microsoft.ContainerRegistry/registries/<acrname> -o table

az role assignment list --scope /subscriptions/c8105223-fff8-4acf-9281-4171ea50d6ac/resourceGroups/jenkins267/providers/Microsoft.ContainerRegistry/registries/acr267 -o table

Make sure that the output of the above command is not empty. If empty run the following command. For AKS to pull the Container images from the ACR, you will need to integrate your ACR with the AKS cluster using the az aks update command with the attach acr parameter and a valid value for acr-name or acr-resource-id. This should configure necessary permissions for AKS to access the ACR

    az aks update --resource-group <myResourceGroup> --name <myAKSCluster> --attach-acr <acr-resource-id>
 
    OR

    az aks update --resource-group <myResourceGroup> --name <myAKSCluster> --attach-acr <acr-name>

    az aks update --resource-group aks_RG267 --name aks_demo267 --attach-acr acr267

# Deploy a sample NodeJS Application to the AKS cluster

To deploy the application, you use a manifest file to create all the objects required to run the AKS Store application. A Kubernetes manifest file defines a cluster's desired state, such as which container images to run. The manifest includes the following Kubernetes deployments and services.

In the Cloud Shell, let us donwload the sample repository files

    git clone https://github.com/mfkhan267/jenkins_on_azure2024.git

    cd deployments

    kubectl apply -f svc-lb.yml 
    kubectl apply -f deploy-complete.yml

# Test the initial Application

Check for a public IP address for the web-deploy application in the load balancer service. 
Monitor progress using the kubectl get service command with the --watch argument.

    kubectl get service ps-lb

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/fb71848b-c583-4814-9dea-561ff5433cc9)

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/d0152de5-0817-4fac-b050-e4db85f0459e)

Once the EXTERNAL-IP address (public IP) is available for the load balancer service, open a web browser to the external IP address of your service to see the your sample NodeJS app is up and running.

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/6debde58-3892-4198-9252-7c39e4dcd588)

# Commit your application code changes and push the Application code repository on GitHub

The GitHub webhook should now trigger the Jenkins Pipeline Job and the Build Job should run automatically.

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/bf87a69c-4036-4d28-a9dd-d80089dac678)

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/99b53eb0-8d19-43ef-a405-f1fa39f4c059)

Wait until the Jenkins Pipeline build job is over. A graphic below the **Build History** heading indicates that the job is being executed or completed. Go to the job build and console output to see the results.

Congratulations! You have successfully **Deployed** your containerized app to your Azure Web App. Your Azure Web App should now be running your newly generated docker image with your App Code

# Testing your Web App that should now be running your newly generated docker image with your App Code

Below is Application v13 running as a Deployment on the AKS Cluster (your build version may differ)

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/b5cc6c57-a124-4afb-9b70-f66746cefc66)

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/6debde58-3892-4198-9252-7c39e4dcd588)

Commit and push your changes to your application code on GitHub and the GitHub webhook should trigger the Jenkins Pipeline Build automatically.
The Jenkins Pipeline will automatically Build, Push and Deploy your containerized Application to the Azure Kubernetes Service Cluster as part of your complete CI-CD Job. The Application should now be running the new Build v14 as the recently modified deployment on the AKS Cluster (your build version may differ)

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/949a003f-c4b3-4977-9247-aeee6c719a15)

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/d40cb5a4-fc22-4734-8a0f-a827d1a2f070)

## Next steps

[Create your Fully Automated CI-CD Jenkins Pipeline to Deploy your Application to an AKS Cluster Part 4](Coming Soon)
