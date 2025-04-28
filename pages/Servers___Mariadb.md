public:: true

- Mariadb is a open source database
- ## Install Mariadb
  id:: 680fb9ee-ad2e-4ca3-b0b0-e7f24c09f493
	- Given Debian stable does not provide the latest versions of most software we must set the Mariadb source to download the proper version
	- Firs we must generate the proper repo info in the mariadb page https://mariadb.org/download/?t=repo-config
	- ### Import gpg keys
	- ```sh
	  sudo apt-get install apt-transport-https curl
	  sudo mkdir -p /etc/apt/keyrings
	  sudo curl -o /etc/apt/keyrings/mariadb-keyring.pgp 'https://mariadb.org/mariadb_release_signing_key.pgp'
	  ```
	- ### Add Mariadb repo to sources `/etc/apt/sources.list.d/mariadb.sources`
	- ```sh
	  touch /etc/apt/sources.list.d/mariadb.sources
	  ```
	- ### Paste source
	- ```source
	  # MariaDB 11.8 repository list - created 2025-04-28 17:27 UTC
	  # https://mariadb.org/download/
	  X-Repolib-Name: MariaDB
	  Types: deb
	  # deb.mariadb.org is a dynamic mirror if your preferred mirror goes offline. See https://mariadb.org/mirrorbits/ for details.
	  # URIs: https://deb.mariadb.org/11.rc/debian
	  URIs: https://mirrors.accretive-networks.net/mariadb/repo/11.8/debian
	  Suites: bookworm
	  Components: main
	  Signed-By: /etc/apt/keyrings/mariadb-keyring.pgp
	  ```
	- ### Install mariadb
	- ```sh
	  sudo apt-get update
	  sudo apt-get install mariadb-server
	  ```