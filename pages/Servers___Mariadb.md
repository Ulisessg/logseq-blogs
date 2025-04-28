- Mariadb is a open source database
- ## Install Mariadb
	- Given Debian stable does not provide the latest versions of most software we must set the Mariadb source to download the proper version
	- Firs we must generate the proper repo info in the mariadb page https://mariadb.org/download/?t=repo-config
	- ### Import gpg keys
	- ```sh
	  sudo apt-get install apt-transport-https curl
	  sudo mkdir -p /etc/apt/keyrings
	  sudo curl -o /etc/apt/keyrings/mariadb-keyring.pgp 'https://mariadb.org/mariadb_release_signing_key.pgp'
	  ```
	- ### Add Mariadb repo to sources
	-