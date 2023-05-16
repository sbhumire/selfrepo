This document has the steps that I took to setup a Jenkins Pipelines for a frontend and backend application:
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


Source Code:
The source code is located at the following GitHub repository:https://github.com/devsecops23/
	
***Steps:
Create a new EC2 instance " jenkins" on AWS using keypair & the following configurations: Memory 15GB, OS Ubuntu, Processor t2.micro
Start the EC2 instance
SSH to EC2 from local computer
	SSH -i <pemfile> ubuntu@<ipaddress of EC2>
Login as root : 
    sudo su - root 
Install jdk 11
	sudo apt-get update && apt-get install openjdk-11-jre
Check java version
	java -version
Install jenkins: 
		curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
		/usr/share/keyrings/jenkins-keyring.asc > /dev/null
		echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
		https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
		/etc/apt/sources.list.d/jenkins.list > /dev/null
		sudo apt-get update
		sudo apt-get install jenkins
Verify jenkins is installed
		jenkins --version

Go to EC2 instance > Security > Security groups > edit inbound traffic > Add rule rules and allow TCP 8080 > All traffic

Log on to Jenkins web server
	http://<ec2-instance-public-ip>:8080
Get password from EC2
	cat /var/lib/jenkins/secrets/initialAdminPassword
Click "Install suggested plugins" and wait till all plugins are installed. This will take some time!!
Once done log in to jenkins again 
Go to Manage Jenkins > Manage plugins > Advance tab >search for "Docker pipeline plugin"
Install the plugin
http://<ec2-instance-public-ip>:8080/restart. Wait for jenkins to restart!!

Install Docker 
Go back to the EC2 instance and sure you are logged in as root
	sudo apt-get update
	sudo apt-get install ca-certificates curl gnupg
	echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  sudo apt-get update
	sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

Verify that the Docker Engine installation is successful by running the hello-world image
	sudo docker run hello-world
Add jenkins to docker
	adduser jenkins docker
	adduser jenkins ubuntu
	systemctl restart docker
Login as jenkins & run
	sudo su -jenkins
	Docker login
Restart jenkins server
	http://<ec2-instance-public-ip>:8080/restart
Create dockerhub account

Install npm and maven as root
	apt-get update && apt-get install -y npm maven

Install Trivy
	 wget https://github.com/aquasecurity/trivy/releases/download/v0.41.0/trivy_0.41.0_Linux-64bit.tar.gz
	 tar –xzvf trivy.tar.gz
	 mv trivy /usr/local/bin/
Test Trivy installation
	Trivy

Login as jenkins user
	su - jenkins
Install Cosign
Go to https://github.com/sigstore/cosign/releases > Assets > cosign-linux_amd64 > get link
	weget "https://github.com/sigstore/cosign/releases/download/v2.0.2/cosign-linux-amd64"
	mv cosign-linux-amd64 cosign
	chmod 755 cosign
	mv cosign /usr/local/bin
	cosign generate-key-pair –output-key prefix jenkins 
	ls to make sure jenkins.pub and jenkins.key is created

Install Kubectl logged as root
	curl -LO https://dl.k8s.io/release/v1.27.1/bin/linux/amd64/kubectl

	echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
If valid, the output is: kubectl: OK
	ls to see kubectl binary
	chmod 755 kubectl
	mv kubectl /usr/local/bin
verify pods are created
	kubecl get pods to 

Install Gcloud CLI as root

	su - jenkins
	gcloud init

Install Helm as root
	wget "https://get.helm.sh/helm-v3.11.3-linux-amd64.tar.gz"
	tar -xzvf helm-v3.11.3-linux-amd64.tar.gz
	cd linux-amd64
	chmod 775 helm
	mv helm /usr/local/bin
Test helm
	helm

Install Sonar-scanner as jenkins user and make sure you are in the github folder that has code to scan for e.g. frontend-project
	mkdir sonar-scanner
	wget “https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.8.0.2856-linux.zip
	apt-get install unzip
	ls /root/sonar-scanner-4.8.0.2856/linux/bin/sonar-scanner
	 apt-get update && apt-get install node.js > for frontend-project






