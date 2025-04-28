public:: true

- ## Install Java 21 runtime environment
  id:: 680be69d-c4f1-4a6d-883d-a9bd3e4e03d3
	- By default Debian use an stable and old version of Java in their repository, so to ensure we use a open source and newer Java version will use the Adoptium repo.
	  id:: 680be6fa-1991-42aa-a9e9-e04da6b0b7f6
	- https://adoptium.net/installation/linux/
	  id:: f6aec238-b49c-4fde-9255-f97d1b766cc4
	- ### Use Adoptium packages
	  id:: 44388c2d-b2db-485e-9ea6-f95f4f2516ed
		- id:: 66898cf2-7e9d-4feb-9f91-cdbf45f785bb
		  ```bash
		  apt install -y wget apt-transport-https gpg
		  wget -qO - https://packages.adoptium.net/artifactory/api/gpg/key/public | gpg --dearmor | tee /etc/apt/trusted.gpg.d/adoptium.gpg > /dev/null
		  echo "deb https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
		  apt update
		  apt install temurin-21-jre
		  java --version
		  ```