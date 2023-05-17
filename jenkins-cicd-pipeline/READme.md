## ***This document has the steps that I took to setup a Jenkins Pipelines for a frontend and backend application:***
---

## Overview:

CI pipeline was build using the using following tools on an EC2 instance:
	Docker
	Npm
	Maven
	Trivy
	Cosign
	Kubectl
	Gcloud
	Helm
	Sonar-Scanner

CD pipeline was build using the following Setup:
	Kubernetes cluster on Gcloud
	Sonarqube 
	Kyverno

---
## Source Code:
The source code is located at the following GitHub repository:https://github.com/devsecops23/

---
	
## Steps:
- Create a new EC2 instance " jenkins" on AWS using keypair & the following configurations: Memory 15GB, OS Ubuntu, Processor t2.micro
Start the EC2 instance***

- SSH to EC2 from local computer
---
	SSH -i <pemfile> ubuntu@<ipaddress of EC2> 
---	
- Login as root
---

    sudo su - root 

- Install jdk 11
---
	sudo apt-get update && apt-get install openjdk-11-jre

- Check java version
---
	java -version

- Install jenkins: 
---
	curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
	/usr/share/keyrings/jenkins-keyring.asc > /dev/null
		
---
	echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
	https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
	/etc/apt/sources.list.d/jenkins.list > /dev/null

---
	sudo apt-get update
---
	sudo apt-get install jenkins

- Verify jenkins is installed
---
	jenkins --version
---
- Go to EC2 instance > Security > Security groups > edit inbound traffic > - Add rule rules and allow TCP 8080 > All traffic

- Log on to Jenkins web server
---
	http://<ec2-instance-public-ip>:8080
---
- Get password from EC2
---
	cat /var/lib/jenkins/secrets/initialAdminPassword
---

- Click "Install suggested plugins" and wait till all plugins are installed. This will take some time!!
Once done log in to jenkins again 
Go to Manage Jenkins > Manage plugins > Advance tab >search for "Docker pipeline"
Install without restart
Restart jenkins server
	http://<ec2-instance-public-ip>:8080/restart. Wait for jenkins to restart!!

### Install Docker 
- Go back to the EC2 instance and sure you are logged in as root
---
	sudo apt-get update
---
	sudo apt-get install ca-certificates curl gnupg
---
	sudo install -m 0755 -d /etc/apt/keyrings
---
	curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
---
	sudo chmod a+r /etc/apt/keyrings/docker.gpg
---

	echo \
	"deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
	"$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
	sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
---

  	sudo apt-get update
---
	sudo apt-get install docker-ce docker-ce-cli containerd.io 
---
	docker-buildx-plugin docker-compose-plugin
---

- Verify that the Docker Engine installation is successful by running the hello-world image
---
	sudo docker run hello-world

Add jenkins to docker
---
	adduser jenkins docker
---
	adduser jenkins ubuntu
---
	systemctl restart docker
---
- Login as jenkins 
---
	su - jenkins
---
	Docker login 
  _Make sure to log in as jenkins user.  Create a docker Id if you don't have one: https://hub.docker.com_

- Install npm and maven as root
---
 	sudo apt-get update && apt-get install -y npm maven
---
### Install Trivy as root user

	 wget "https://github.com/aquasecurity/trivy/releases/download/v0.41.0/trivy_0.41.0_Linux-64bit.tar.gz"
	 tar -xvf trivy_0.41.0_Linux-64bit.tar.gz
	 mv trivy /usr/local/bin/
	 
- Test Trivy installation
---
	trivy --version

- Install Cosign as root 
Go to https://github.com/sigstore/cosign/releases > Assets > cosign-linux_amd64 > get link

	wget "https://github.com/sigstore/cosign/releases/download/v2.0.2/cosign-linux-amd64"
	mv cosign-linux-amd64 cosign
	chmod 755 cosign
	mv cosign /usr/local/bin
	cosign generate-key-pair --output-key-prefix jenkins

- Verify that jenkins.pub and jenkins.key is created
---
	ls jenkins.key jenkins.pub 
---
- Install Kubectl logged as root

	curl -LO "https://dl.k8s.io/release/v1.27.1/bin/linux/amd64/kubectl"
	chmod 755 kubectl
---
	mv kubectl  /usr/local/bin

- ls to see kubectl binary
---
	ls kubectl 

- Install Gcloud CLI as root
---
	sudo apt-get install apt-transport-https ca-certificates gnupg
---

	echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
---
	curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg add -
---
	sudo apt-get update && sudo apt-get install google-cloud-cli
---
- Install gcloud-auth-plugin
---
	sudo apt-get install google-cloud-sdk-gke-gcloud-auth-plugin
- First switch to jenkins user before next step
---
	su - jenkins
---
	whoami
---
	gcloud init
---

- Folow instructions as displayed on screen and log into gcloud account and copy authorization code & paste
- Go to https://console.cloud.google.com/ and select a project
- It will ask you to select a region.
- Go to google console -> Kubernetes Engine -> Get the location -> us-east1
- In the  cluster list -> ... -> select Action -> connect and copy comand
- Run the command in EC2 instance
- gcloud container clusters get-credentials jenkins-cluster --region us-east1 --project data-centaur-385400
- You should see a message " kubeconfig entry generated for jenkins-cluster."
- Run kubectl get pods but it will return no resource
---

- Install Helm as root
---
 	logout
---
	wget "https://get.helm.sh/helm-v3.11.3-linux-amd64.tar.gz"
---
	tar -xzvf helm-v3.11.3-linux-amd64.tar.gz
---
	cd linux-amd64
---
	chmod 775 helm
---
	mv helm /usr/local/bin
---
- Test helm
---
	helm
---
 	logout 
---
	apt-get install unzip
---
-  Install Sonar-scanner as root user
---
	su - jenkins
---
	mkdir sonar-scanner
---
	wget “https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.8.0.2856-linux.zip

	ls /root/sonar-scanner-4.8.0.2856/linux/bin/sonar-scanner
---
- Install dependencies. For example for frontend-project with java-script
---
	 apt-get update && apt-get install node.js 
---






