public:: true

- Gitea is an open source code hosting, similar to GitHub. [[Servers/Services]]
- ### ((680fb9ee-ad2e-4ca3-b0b0-e7f24c09f493))
- ### Login in root mariadb user
	- ```sh
	  mysql -u root
	  ```
- ### Create gitea user in MariaDB
	- https://docs.gitea.com/installation/database-prep
	- ```sh
	  CREATE DATABASE giteadb CHARACTER SET 'utf8mb4' COLLATE 'utf8mb4_bin';
	  SET old_passwords=0;
	  CREATE USER 'gitea'@'%' IDENTIFIED BY 'gitea' REQUIRE SSL;
	  GRANT ALL PRIVILEGES ON giteadb.* TO 'gitea';
	  FLUSH PRIVILEGES;
	  ```
	- Since MariaDB 11.4 TLS is activaded by default using a random self generated TLS, you can specify your TLS following the official tutorial https://mariadb.com/kb/en/securing-connections-for-client-and-server/
- ### Download amd64 binary
	- ```sh
	  wget -O gitea https://dl.gitea.com/gitea/1.23.7/gitea-1.23.7-linux-amd64
	  chmod +x gitea
	  ```
- ### Verify git is installed
	- ```sh
	  git --version
	  ```
- ### Create Gitea UNIX user
	- You can name it as you want
	- ```sh
	  adduser \
	     --system \
	     --shell /bin/bash \
	     --gecos 'Git Version Control' \
	     --group \
	     --disabled-password \
	     --home /home/git \
	     git
	  ```
- ### Create folder structure}
	- ```sh
	  mkdir -p /var/lib/gitea/{custom,data,log}
	  chown -R git:git /var/lib/gitea/
	  chmod -R 750 /var/lib/gitea/
	  mkdir /etc/gitea
	  chown root:git /etc/gitea
	  chmod 770 /etc/gitea
	  ```
- ### Copy Gitea to global binary location
	- ```sh
	  cp gitea /usr/local/bin/gitea
	  ```
- ### Create Gitea Linux service
	- https://docs.gitea.com/installation/linux-service
	- ```sh
	  ```