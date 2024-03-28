# NO non-sense Installing Docker on an Azure Linux VM running Jenkins

# Part 2

This article shows how to install [Docker](https://docs.docker.com/engine/install/ubuntu) on an Ubuntu Linux VM for developing, packaging and shipping your containerized apps.

## What is Docker?

>> Docker is a software platform that allows you to build, test, and deploy applications quickly. 
>> Docker packages software into standardized units called containers that have everything the software needs to run including libraries, system tools, code, and runtime. 
>> Using Docker, you can quickly deploy and scale applications into any environment and you are convinced that your code will run. Platform Agnostic.
>> Docker provides developers and admins a highly reliable, low-cost way to build, ship, and run distributed applications at any scale.

## Why use Docker?

Docker makes it really easy to install, build, test, ship and run software without worrying about its setup or dependencies and libraries.

![image](https://github.com/mfkhan267/jenkins_on_azure2024/assets/77663612/08687f1c-0c10-4901-b418-2248dcfb78f7)


## 1. Install Docker Engine using the convenience script

Docker provides a convenience script at https://get.docker.com/ to install Docker into development environments non-interactively. 
We shall use the scripted installation method for convenience.

// bash command

    curl -fsSL https://get.docker.com -o get-docker.sh
    sudo sh get-docker.sh

Note: In case you get permission denied while trying to connect to the Docker daemon socket Error: /var/run/docker.sock: connect: permission denied. Since a Jenkins job runs as service by the name 'jenkins', you will need to add the 'jenkins' user to the docker group. This will allow you to run Docker with non-root privileges

// bash command

    sudo usermod -aG docker jenkins
    echo "jenkins  ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/jenkins

Restart the VM so that the group membership is re-evaluated.

// bash command

    sudo reboot
    sudo service docker status

Let us run Docker Version to confirm the installation.

// bash command

    docker --version

![image](https://github.com/mfkhan267/my_jenkins_app/assets/77663612/31060bec-20e2-4e81-b6db-8f4408b74653)

There you have it. You have successfully installed the docker engine. It is time to get started with docker containers and images.

## Next steps

[Part 3 - Coming Soon]
