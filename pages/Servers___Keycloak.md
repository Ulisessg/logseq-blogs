- Keycloak is an Oauth provider, you can use it to add social media login in your apps, or use their own login service
- ### Installing keycloak
- {{embed ((680be69d-c4f1-4a6d-883d-a9bd3e4e03d3))}}
- {{embed ((680be6fa-1991-42aa-a9e9-e04da6b0b7f6))}}
- ### Create Keycloak UNIX user
	- ```sh
	  # Create keycloak UNIX user
	  adduser keycloak
	  su keycloak
	  ```
- ### Create folder structure and download Keycloak binaries
	- ```
	  cd && mkdir -m 700 prod && cd prod
	  # Download Keycloak binary, you can download a newer Keycloak vewrsion changing it in all commands
	  wget https://github.com/keycloak/keycloak/releases/download/26.2.1/keycloak-26.2.1.tar.gz
	  tar -zxvf keycloak-26.2.1.tar.gz
	  rm keycloak-26.2.1.tar.gz
	  cp -a ./keycloak-26.2.1/. .
	  rm -r keycloak-26.2.1/
	  nano conf/keycloak.conf
	  ```
	- And paste
	- ```.env
	  # Basic settings for running in production. Change accordingly before deploying the server.
	  # Database
	  # The database vendor.
	  db=mariadb
	  # The username of the database user.
	  db-username=keycloak
	  # The password of the database user.
	  db-password=
	  # HTTP
	  proxy=reencrypt
	  # Do not attach route to cookies and rely on the session affinity capabilities from reverse proxy
	  spi-sticky-session-encoder-infinispan-should-attach-route=false
	  # Hostname for the Keycloak server.
	  hostname=auth.anhomelab.online
	  ```
- ### Prepare Keycloak for production
	- ```sh
	  # This will create a temporal admin user, will prompt you to write the username and temp password
	  bin/kc.sh bootstrap-admin user
	  # Build the production ready keycloak, improves startup time
	  bin/kc.sh build
	  ```