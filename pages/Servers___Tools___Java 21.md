## Install Java 21 runtime environment
	- https://adoptium.net/installation/linux/
	- ### Use Adoptium packages
		- ```bash
		  apt install -y wget apt-transport-https gpg
		  wget -qO - https://packages.adoptium.net/artifactory/api/gpg/key/public | gpg --dearmor | tee /etc/apt/trusted.gpg.d/adoptium.gpg > /dev/null
		  echo "deb https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
		  apt update
		  apt install temurin-21-jre
		  java --version
		  ```
	- Install jenkins
		- ```bash
		  wget -O /usr/share/keyrings/jenkins-keyring.asc \
		    https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
		  echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
		    https://pkg.jenkins.io/debian-stable binary/ | tee \
		    /etc/apt/sources.list.d/jenkins.list > /dev/null
		  apt-get update
		  apt-get install jenkins
		  ```
	- Change Jenkins port (optional)
		- ```bash
		  systemctl edit jenkins.service
		  ```
		- ```service
		  [Service]
		  Environment="JENKINS_PORT=8081"
		  ```
		- ```bash
		  systemctl stop jenkins && systemctl start jenkins
		  ```
	- Add Jenkins to Traefik dynamic file
		- ```bash
		  nano /etc/traefik/dynamic.yaml
		  ```
			-