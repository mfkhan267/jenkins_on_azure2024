# NO non-sense Running Jenkins on an Azure Linux VM like a BOSS

# Part 1

This article shows you how to install [Jenkins](https://jenkins.io) on an Ubuntu Linux VM running in Azure Cloud.

Here are the quick steps:

> * Get yourself a free Azure Account if you do not have one already
> * Create a resource group
> * Create a Ubuntu Linux VM with the setup file also called the cloud init custom data
> * Open port 8080 in order to access Jenkins on the virtual machine (Yes Jenkins runs on port 8080)
> * Connect to the virtual machine via SSH and the PublicIP of the VM
> * Configure a sample Jenkins pipeline job
> * Build the sample Jenkins pipeline job

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/23fb922a-3770-4845-8642-f7b5663cc2a4)

## 1. Configure your environment

Azure subscription: If you don't have an Azure subscription, create a [Azure free account](https://azure.microsoft.com/en-in/free) before you begin.

## 2. Open Cloud Shell

If you already have a Cloud Shell session open, you can skip to the next section.

Browse to the [Azure Portal](https://portal.azure.com/)

If necessary, log in to your Azure subscription and change the Azure directory with the subscription that you would like to use.

Open Cloud Shell.

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/0d4f6a7e-600c-4154-8854-d7da074bf78d)

## 3. Create a virtual machine

Fetch your account details as shown below (we will need these further down the document as we proceed.

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/8325e956-29f2-488d-818f-35d5bed69ecd)

Create a test directory by a name of your choice. I called mine `jenkins267`.

Switch to the test directory created above.

Create a file named `cloud-init-jenkins.txt`.

Paste the following code into the new file:
  
    #cloud-init-config-Jenkins
    package_upgrade: true
    runcmd:
      - sudo apt-get update && sudo apt-get install fontconfig openjdk-17-jre -y
      - sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
      - echo 'deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/' | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
      - sudo apt-get update && sudo apt-get install jenkins -y
      - sudo systemctl start jenkins && sudo systemctl enable jenkins
    
Run [az group create](https://learn.microsoft.com/en-us/cli/azure/group?view=azure-cli-latest#az-group-create) command to create a resource group.

// azurecli command

    az group create --name jenkins267 --location eastus2

Run [az vm create](https://learn.microsoft.com/en-us/azure/virtual-machines/linux/quick-create-cli) to create a virtual machine. 

// azurecli command

    az vm create \
    --resource-group jenkins267 \
    --name jenkinsvm267 \
    --image Ubuntu2204 \
    --admin-username "azureuser" \
    --generate-ssh-keys \
    --public-ip-sku Standard \
    --custom-data cloud-init-jenkins.txt

OR (if you already have a ssh key pair that you would like to use) 
    
// azurecli command

    az vm create \
    --resource-group jenkins267 \
    --name jenkinsvm267 \
    --image Ubuntu2204 \
    --admin-username "azureuser" \
    --ssh-key-value ~/.ssh/id_rsa.pub \
    --public-ip-sku Standard \
    --custom-data cloud-init-jenkins.txt

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/65a754b1-cfd6-4de3-bca1-6bb1c1ce114f)
 
Run [az vm list](https://learn.microsoft.com/en-us/cli/azure/vm?view=azure-cli-latest#az-vm-list) command to verify the creation (and state) of the new virtual machine.

// azurecli command

    az vm list -d -o table --query "[?name=='jenkinsvm267']"

As Jenkins runs on port 8080, let us now run [az vm open-port](https://learn.microsoft.com/en-us/cli/azure/vm?view=azure-cli-latest#az-vm-open-port) command to open port 8080 on the new virtual machine. You also need to open additional ports later on if your container app is listening on other ports like 8080, 3000 and so on. 

Note: The Jenkins port may be configured as per your business requirement and security policies with the 'sudo systemctl edit jenkins' command.

// azurecli command

    az vm open-port \
    --resource-group jenkins267 \
    --name jenkinsvm267  \
    --port 8080 --priority 1010

## 4. Configure Jenkins

Run [az vm show](https://learn.microsoft.com/en-us/cli/azure/vm?view=azure-cli-latest#az-vm-show) command to get the public IP address for the sample virtual machine.

// azurecli command

    az vm show \
    --resource-group jenkins267 \
    --name jenkinsvm267 -d \
    --query [publicIps] \
    --output tsv

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/c3519e72-fce8-43c0-a194-f7c9c6fc3ee1)

You may retieve the IP address from the Azure Portal Console and selecting the Virtual Machine named `jenkinsvm267`

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/3273693f-9cbf-41d4-96f6-d0fb46c71bca)

Using the IP address retrieved in the previous step, SSH into the virtual machine. You'll need to confirm the connection request.

// azurecli command

    ssh azureuser@<ip_address>

Verify that Jenkins is running by getting the status of the Jenkins service.

// bash command

    service jenkins status

Get the autogenerated Jenkins password.

// bash command

    sudo cat /var/lib/jenkins/secrets/initialAdminPassword
    
![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/fa181795-bd11-427c-b0f8-15a8507a38b3)

Using the IP address, open the following URL in a browser: `http://<ip_address>:8080`

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/2f2891d8-7bf4-4e3c-a8eb-efbd7b4e6483)

Enter the password you retrieved earlier and select **Continue**.

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/d1fba404-9e0e-4add-9999-6cc4aa438ad5)

Select **Select Install suggested plug-ins**.

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/d40d9de6-6123-4f0b-baa5-2c0506630afb)

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/0a2f8287-5f07-47db-92db-718126b0adfc)

Enter the information for the first admin user and select **Save and Continue**. 
I highly recommend that you create the admin user by the same name as the admin user that you configured with the VM creation step.

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/ca282eac-b2ee-435f-a0f3-52f8d5ede78a)

On the **Instance Configuration** page, select **Save and Finish**.

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/ab5f2efd-fc56-4754-bf6a-d5c1d7e2a414)

Select **Start using Jenkins**.

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/1d83d9e5-2421-430b-9468-8d62f427c3aa)

## 5. Create your first job

On the Jenkins home page, select **Create a job**.

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/4218fa58-685f-48ee-b884-d7594ed0016d)

Enter a job name of `my-first-freestyle-hello-world`, select **Freestyle project**, and select **OK**.

Select the **Build** tab, then select **Add build step**

From the drop-down menu, select **Execute Shell**.

// bash command

    echo "Hello World"

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/9887b958-e5e8-4f41-9252-63b36e8cb922)

You may also select a **Pipeline project**, and select **OK** and try the sample Hello-World pipeline.

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/98307980-5e37-4f6b-a876-562f9153c181)

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/011aecfc-b897-4971-9c33-ed689f78f79e)

Scroll to the bottom of the page, and select **Save**.

## 6. Build the sample Freestyle or Pipeline Hello-World job

When the home page for your project displays, select **Build Now** to execute your Jenkins job.

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/cd6c4156-f9a3-4d57-8522-458a2645c6f7)

A graphic below the **Build History** heading indicates that the job is being executed or completed. Go to the job build and console output to see the results.

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/2a8c5461-c1eb-4d6d-9408-dd2480fe3aba)

Congratulations! You have successfully executed your first Jenkins Job. Your Jenkins server is now ready to build your projects in Azure!

## Troubleshooting

If you encounter any problems configuring Jenkins, refer to the [Jenkins installation page](https://www.jenkins.io/doc/book/installing/) for the latest instructions and known issues.

## Next steps

[Install Docker on Azure VM running Jenkins Part 2](./install_docker_on_linux_vm.md)
