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
	  CREATE USER 'gitea'@'%' IDENTIFIED BY 'gitea';
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
	- Create service file `/etc/systemd/system/gitea.service`
	- ```sh
	  touch /etc/systemd/system/gitea.service
	  nano /etc/systemd/system/gitea.service
	  systemctl daemon-reload
	  ```
	- And paste
	- ```
	  [Unit]
	  Description=Gitea (Git with a cup of tea)
	  After=network.target
	  Wants=mariadb.service
	  After=mariadb.service
	  
	  [Service]
	  # Uncomment the next line if you have repos with lots of files and get a HTTP 500 error because of that
	  # LimitNOFILE=524288:524288
	  RestartSec=2s
	  Type=simple
	  User=git
	  Group=git
	  WorkingDirectory=/var/lib/gitea/
	  # If using Unix socket: tells systemd to create the /run/gitea folder, which will contain the gitea.sock file
	  # (manually creating /run/gitea doesn't work, because it would not persist across reboots)
	  #RuntimeDirectory=gitea
	  ExecStart=/usr/local/bin/gitea web --config /etc/gitea/app.ini
	  Restart=always
	  Environment=USER=git HOME=/home/git GITEA_WORK_DIR=/var/lib/gitea
	  [Install]
	  WantedBy=multi-user.target
	  ```
- ### Create AppArmor Gitea profile
	- Create profile `/etc/apparmor.d/usr.local.bin.gitea`
		- ```sh
		  touch /etc/apparmor.d/usr.local.bin.gitea
		  nano /etc/apparmor.d/usr.local.bin.gitea
		  ```
		- And paste
		- ```apparmor
		  abi <abi/3.0>,
		  
		  include <tunables/global>
		  
		  /usr/local/bin/gitea {
		    include <abstractions/apache2-common>
		    include <abstractions/base>
		    include <abstractions/dbus-session-strict>
		    include <abstractions/gio-open>
		  
		    /etc/mime.types r,
		    /etc/passwd r,
		    /proc/sys/net/core/somaxconn r,
		    /sys/kernel/mm/transparent_hugepage/hpage_pmd_size r,
		    /usr/bin/git mrix,
		    /usr/local/bin/gitea mr,
		    /usr/share/git-core/templates/ r,
		    owner /etc/gitea/app.ini rw,
		    owner /home/git/** rw,
		    owner /proc/*/cpuset r,
		    owner /var/lib/gitea/** rwlk,
		  
		  }
		  
		  ```
		- And enforce AppArmor profile
			- ```sh
			  aa-enforce /etc/apparmor.d/usr.local.bin.gitea
			  ```
- ### Enable and start Gitea Service
	- ```sh
	  sudo systemctl enable gitea --now
	  ```
- Add Gitea to Traefik dynamic config `/etc/traefik/dynamic.yaml`
	- ```yaml
	    routers:
	      gitea:
	        rule: "Host(`sub.domain.com`)"
	        service: gitea-service
	        tls:
	          certResolver: myresolver
	        entryPoints:
	          - websecure
	    services:
	      gitea-service:
	        loadBalancer:
	          servers:
	            - url: http://127.0.0.1:3000
	  ```
- ### Fill the Gitea installation form
- ### Now change the permissions of config files to only allow git UNIX user to modify
	- ```sh
	  chmod 750 /etc/gitea
	  chmod 640 /etc/gitea/app.ini
	  ```