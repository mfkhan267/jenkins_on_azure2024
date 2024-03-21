# No Non-Sense Installing Docker on an Azure Linux VM running Jenkins

# Part 2

This article shows how to install [Docker](https://docs.docker.com/engine/install/ubuntu) on an Ubuntu Linux VM for developing, packaging and shipping your containerized apps.

## 1. Install Docker Engine using the convenience script

Docker provides a convenience script at https://get.docker.com/ to install Docker into development environments non-interactively. 
We shall use the scripted installation method for convenience.

// bash command

    curl -fsSL https://get.docker.com -o get-docker.sh
    sudo sh get-docker.sh

Note: In case you get permission denied while trying to connect to the Docker daemon socket Error: /var/run/docker.sock: connect: permission denied. Since a Jenkins job runs as service by the name 'jenkins', you will need to add the 'jenkins' user to the docker group. This will allow you to run Docker with non-root privileges

// bash command

    sudo usermod -aG docker jenkins
    sudo usermod -aG sudo jenkins
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
