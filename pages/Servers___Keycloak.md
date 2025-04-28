- Keycloak is an Oauth provider, you can use it to add social media login in your apps, or use their own login service
- ### Installing keycloak
- {{embed ((680be69d-c4f1-4a6d-883d-a9bd3e4e03d3))}}
- {{embed ((680be6fa-1991-42aa-a9e9-e04da6b0b7f6))}}
- ### a
	- ```sh
	  # Create keycloak UNIX user
	  adduser keycloak
	  # Create required folder structure
	  su keycloak && cd && mkdir -m 700 prod && cd prod
	  # Download Keycloak binary, you can download a newer Keycloak vewrsion changing it in all commands
	  wget https://github.com/keycloak/keycloak/releases/download/26.1.4/keycloak-26.1.4.tar.gz
	  tar -zxvf keycloak-26.1.4.tar.gz
	  rm keycloak-26.1.4.tar.gz
	  cp -a ./keycloak-26.1.4/. .
	  rm -r keycloak-26.1.4/
	  nano conf/keycloak.conf
	  ```