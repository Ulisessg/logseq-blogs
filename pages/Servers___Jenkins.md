## Jenkins installation and basic configuration [[Servers/Services]]
	- This installation and configuration allows you to create Jenkins Workflow using Nodejs
	- ### Install Jenkins https://www.jenkins.io/doc/book/installing/linux/#debianubuntu
		- ```bash
		  wget -O /usr/share/keyrings/jenkins-keyring.asc \
		    https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
		  echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
		    https://pkg.jenkins.io/debian-stable binary/ | tee \
		    /etc/apt/sources.list.d/jenkins.list > /dev/null
		  apt-get update
		  apt-get install jenkins
		  ```
	- ### Change Jenkins port (optional)
		- ```bash
		  systemctl edit jenkins.service
		  ```
		- ```service
		  [Service]
		  Environment="JENKINS_PORT=8081"
		  OtherEnv=""
		  ```
		- ```bash
		  systemctl stop jenkins && systemctl start jenkins
		  ```
	- ### Add Jenkins to Traefik dynamic file
		- ```bash
		  nano /etc/traefik/dynamic.yaml
		  ```
			- ```yaml
			  http:
			    routers:
			      jenkins:
			        rule: "Host(`subdomain.domain.com`)"
			        service: jenkins-service
			        tls:
			          certResolver: myresolver
			        entryPoints:
			          - websecure
			    services:
			      jenkins-service:
			        loadBalancer:
			          servers:
			            - url: "http://127.0.0.1:8081"
			  ```
	- ### Configure Jenkins
		- #### Unlock Jenkins
			- Copy the initial password from `/var/lib/jenkins/secrets/initialAdminPassword`
				- ```bash
				  cat /var/lib/jenkins/secrets/initialAdminPassword
				  ```
		- #### Install suggested plugins
		- #### Create admin user
		- #### Disallow the Build-In node setting number of executor to 0
			- In current Jenkins version this can be configured in `https://subdomain.domain.com/computer/(built-in)/configure`
			- This is a good recommendation to avoid security risks
		- #### [Install Nodejs plugin](https://plugins.jenkins.io/nodejs/)
		- #### Create a new Jenkins Agent
			- This step applies if you want to make the agent in different machine or in same machine
			- **THIS METHOD IS USING BARE METAL, IF YOU ARE USING DOCKER REFER TO OFFICIAL DOCS TO MORE INFO**
			- Create a new user to serve as agent node
			  logseq.order-list-type:: number
				- logseq.order-list-type:: number
				  ```bash
				  adduser cool-jenkins-agent-name
				  ```
				- Login to that user and create an ssh keys pair (this will be used to launch Jenkins agent using ssh)
				  logseq.order-list-type:: number
					- logseq.order-list-type:: number
					  ```bash
					  su cool-jenkins-agent-name
					  cd
					  # Remember to use a strong password to avoid ssh key being stoled and used without permission
					  ssh-keygen -t ed25519 -C "your_email@example.com"
					  ```
					- Create `~/.ssh/authorized_keys` file
					  logseq.order-list-type:: number
						- logseq.order-list-type:: numbertouch .ssh/
						  ```bash
						  touch ~/.ssh/authorized_keys
						  ```
					- Copy public key to authorized_keys file
					  logseq.order-list-type:: number
					- Restart SSH service (Using root user)
					  logseq.order-list-type:: number
						- logseq.order-list-type:: number
						  ```bash
						  systemctl reload ssh
						  ```
			- Go to Manage Nodes config `https://subdomain.domain.com/manage/computer/`
			  logseq.order-list-type:: number
			- Do click in create new Node
			  logseq.order-list-type:: number
			- Give new node a name and set it as permanent agent
			  logseq.order-list-type:: number
			- Set the number of executors following the rules:
			  logseq.order-list-type:: number
				- Number of executors never surpass the number of processor cores, bare metal or virtual
				  logseq.order-list-type:: number
			- Set the remote directory using the unix user home path, technically any folder with rwx permissions for user could be used but using home directory is more easy
			  logseq.order-list-type:: number
				- logseq.order-list-type:: number
				  ```
				  /home/cool-jenkins-agent-name/agent
				  ```
			- [Set the labels](https://plugins.jenkins.io/nodelabelparameter/)
			  logseq.order-list-type:: number
			- In launch method set `Launch agents via SSH`
			  logseq.order-list-type:: number
				- Add the host of the machine, if you are creating agent in same host use the machine IP address
				  logseq.order-list-type:: number
				- In credentials click on `+ Add`
				  logseq.order-list-type:: number
					- In kind select `SSH username with private key`
					  logseq.order-list-type:: number
						- Give and meaningful id
						  logseq.order-list-type:: number
						- Description is optional
						  logseq.order-list-type:: number
						- Paste the Jenkins agent UNIX username `cool-jenkins-agent-name` and select the option `Threat username as secret`
						  logseq.order-list-type:: number
						- Press `Private Key` > `Enter directly`
						  logseq.order-list-type:: number
							- Paste the private key generated in step 1
							  logseq.order-list-type:: number
							- Paste the phrase of private key
							  logseq.order-list-type:: number
				- Select the created credential
				  logseq.order-list-type:: number
				- In `Host Key Verification Strategy` Select `Manually trusted key verification strategy` and `Require manual verification of initial connection`
				  logseq.order-list-type:: number
				- In availability set the proper config for your case, if don't have an idea leave it in default
				  logseq.order-list-type:: number
				- Click on save and go to Noide configuratio to trust remote ssh
				  logseq.order-list-type:: number
	- **NOW YOU ARE READY TO CREATE JENKINS WORKFLOWS WITH NODE**