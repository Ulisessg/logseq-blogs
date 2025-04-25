## Install Java 21 runtime environment
	- By default Debian use an stable and old version of Java in their repository
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