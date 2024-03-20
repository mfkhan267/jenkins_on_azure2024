# No Non-Sense Getting Started with Jenkins on an Azure Linux VM

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

![image](https://github.com/mfkhan267/my_jenkins_app/assets/77663612/006dab74-e44e-48e3-9713-c077cf9ba352)

## 1. Configure your environment

Azure subscription: If you don't have an Azure subscription, create a [Azure free account](https://azure.microsoft.com/en-in/free) before you begin.

## 2. Open Cloud Shell

If you already have a Cloud Shell session open, you can skip to the next section.

Browse to the [Azure Portal](https://portal.azure.com/)

If necessary, log in to your Azure subscription and change the Azure directory and subscription that you would like to use.

Open Cloud Shell.

![image](https://github.com/mfkhan267/my_jenkins_app/assets/77663612/14b19209-2093-4a29-a5e6-df8093e4afa0)

## 3. Create a virtual machine

Fetch your account details as shown below (we will need these further down the document as we proceed.

![image](https://github.com/mfkhan267/my_jenkins_app/assets/77663612/735be17e-6c38-4a52-bade-44d10b8490ea)

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
      - sudo service jenkins restart
    
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

![image](https://github.com/mfkhan267/my_jenkins_app/assets/77663612/e935b17a-089d-4d76-a0fc-973e3b3179dd)
 
Run [az vm list](https://learn.microsoft.com/en-us/cli/azure/vm?view=azure-cli-latest#az-vm-list) command to verify the creation (and state) of the new virtual machine.

// azurecli command

    az vm list -d -o table --query "[?name=='jenkinsvm267']"

As Jenkins runs on port 8080, let us now run [az vm open-port](https://learn.microsoft.com/en-us/cli/azure/vm?view=azure-cli-latest#az-vm-open-port) command to open port 8080 on the new virtual machine. You also need to open additional ports later on if your container app is listening on other ports like 8080, 3000 and so on.

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

![image](https://github.com/mfkhan267/my_jenkins_app/assets/77663612/ce02a682-a07f-42f5-aaea-eeff49bee6e3)

You may retieve the IP address from the Azure Portal Console and selecting the Virtual Machine named `jenkinsvm267`

![image](https://github.com/mfkhan267/my_jenkins_app/assets/77663612/d1a43772-c1fd-4df4-be6a-43c63093551a)

    **Key points**:

    - The `--query` parameter limits the output to the public IP addresses for the virtual machine.

Using the IP address retrieved in the previous step, SSH into the virtual machine. You'll need to confirm the connection request.

// azurecli command

    ssh azureuser@<ip_address>

Verify that Jenkins is running by getting the status of the Jenkins service.

// bash command

    service jenkins status

Get the autogenerated Jenkins password.

// bash command

    sudo cat /var/lib/jenkins/secrets/initialAdminPassword
    
![image](https://github.com/mfkhan267/my_jenkins_app/assets/77663612/e0001f99-4302-4969-9687-0b8105fa2388)

Using the IP address, open the following URL in a browser: `http://<ip_address>:8080`

![image](https://github.com/mfkhan267/my_jenkins_app/assets/77663612/0eeb69aa-198d-49bf-bb12-14521b74f92f)

Enter the password you retrieved earlier and select **Continue**.

![image](https://github.com/mfkhan267/my_jenkins_app/assets/77663612/20495d31-5438-42a4-9f81-62fe73fb9da4)

Select **Select Install suggested plug-ins**.

![image](https://github.com/mfkhan267/my_jenkins_app/assets/77663612/1b528afd-7e76-4744-a7f9-113633cad0ff)

Enter the information for the first admin user and select **Save and Continue**. 
I highly recommend that you create the admin user by the same name as the admin user that you created with the VM creation step.

On the **Instance Configuration** page, select **Save and Finish**.

![image](https://github.com/mfkhan267/my_jenkins_app/assets/77663612/27e3628d-edf7-4dc4-a617-3ba76a25466e)

Select **Start using Jenkins**.

![image](https://github.com/mfkhan267/my_jenkins_app/assets/77663612/026900f5-3e33-4b30-8899-6c128085875f)

## 5. Create your first job

On the Jenkins home page, select **Create a job**.

![image](https://github.com/mfkhan267/my_jenkins_app/assets/77663612/9c49fc1e-3b89-47f8-8659-d7afe6773ae3)

Enter a job name of `my-first-freestyle-hello-world`, select **Freestyle project**, and select **OK**.

Select the **Build** tab, then select **Add build step**

From the drop-down menu, select **Execute Shell**.

// bash command

    echo "Hello World"

![image](https://github.com/mfkhan267/my_jenkins_app/assets/77663612/6406e84d-3bed-40cb-a5b1-cdc5f076755a)

You may also select a **Pipeline project**, and select **OK** and try the sample Hello-World pipeline.

![image](https://github.com/mfkhan267/my_jenkins_app/assets/77663612/5c7abcc7-655b-47c6-801d-f1c619e7ccb9)

![image](https://github.com/mfkhan267/my_jenkins_app/assets/77663612/db4d7c97-9904-428c-9616-5c9a898b5e7d)

Scroll to the bottom of the page, and select **Save**.

## 6. Build the sample Freestyle or Pipeline Hello-World job

When the home page for your project displays, select **Build Now** to execute your Jenkins job.

![image](https://github.com/mfkhan267/my_jenkins_app/assets/77663612/e58f8baf-c717-4361-aceb-9123dc83ffdc)

A graphic below the **Build History** heading indicates that the job is being executed or completed. Go to the job build and console output to see the results.

![image](https://github.com/mfkhan267/my_jenkins_app/assets/77663612/c44c7590-24c3-4028-b7db-deca786e78cb)

Congratulations! You have successfully executed your first Jenkins Job. Your Jenkins server is now ready to build your projects in Azure!

## Troubleshooting

If you encounter any problems configuring Jenkins, refer to the [Jenkins installation page](https://www.jenkins.io/doc/book/installing/) for the latest instructions and known issues.

## Next steps

[Install Docker on Azure VM running Jenkins](./install-docker-on-linux-vm.md)
