- Setup reverse proxy [[Servers/Reverse Proxy/Traefik]]
	- Setup Traefik unix user and login in
		- ```sh
		  sudo adduser traefik
		  sudo su traefik && cd
		  ```
			- Reccomendations:
				- Set a strong password
	- Download latest traefik version
		- ```sh
		  mkdir -m 700 prod
		  cd prod
		  wget "https://github.com/traefik/traefik/releases/download/v3.3.4/traefik_v3.3.4_linux_amd64.tar.gz"
		  tar -zxvf traefik_v3.3.4_linux_amd64.tar.gz
		  rm traefik_v3.3.4_linux_amd64.tar.gz
		  rm CHANGELOG.md
		  rm LICENSE.md
		  chmod 700 traefik
		  ```
		- Useful tip: If you are reading this and a newer Traefik version is avaliable just change the v3.3.4 and traefik_v3.3.4 parts to desired version
	- Change to sudoer user (or root if the case)
	- Create config files for traefik
		- ```sh
		   mkdir -m 700 /etc/traefik
		   touch /etc/traefik/acme.json
		   touch /etc/traefik/traefik.yaml
		   touch /etc/traefik/dynamic.yaml
		   chown -R traefik:traefik /etc/traefik
		   chmod 600 -R /etc/traefik
		  ```
	- Create the Linux service to always run traefik on startup
		- ```sh
		   touch /etc/systemd/system/traefik.service
		  ```
		- Paste this in the file
			- ```service
			  [Unit]
			  Description=Traefik reverse proxy
			  Documentation=https://doc.traefik.io/traefik/
			  After=network-online.target
			  AssertFileIsExecutable=/home/traefik/prod/traefik
			  AssertPathExists=/etc/traefik/traefik.yaml
			  
			  [Service]
			  # Run traefik as its own user (create new user with: useradd -r -s /bin/false -U -M traefik)
			  User=traefik
			  AmbientCapabilities=CAP_NET_BIND_SERVICE
			  
			  # configure service behavior
			  Type=simple
			  ExecStart=/home/traefik/prod/traefik --configFile=/etc/traefik/traefik.yaml
			  Restart=always
			  WatchdogSec=2s
			  
			  # lock down system access
			  # prohibit any operating system and configuration modification
			  ProtectSystem=strict
			  # create separate, new (and empty) /tmp and /var/tmp filesystems
			  PrivateTmp=true
			  # turns off access to physical devices (/dev/...)
			  PrivateDevices=true
			  # make kernel settings (procfs and sysfs) read-only
			  ProtectKernelTunables=true
			  # make cgroups /sys/fs/cgroup read-only
			  ProtectControlGroups=true
			  
			  # allow writing of acme.json
			  ReadWritePaths=/etc/traefik/acme.json
			  # depending on log and entrypoint configuration, you may need to allow writing to other paths, too
			  
			  # limit number of processes in this unit
			  #LimitNPROC=1
			  
			  [Install]
			  WantedBy=multi-user.target
			  ```
		- Now you modify /etc/traefik/traefik.yaml
			- ```yaml
			  entryPoints:
			    websecure:
			      address: ":443"
			      forwardedHeaders:
			        trustedIPs:
			          - "127.0.0.1/32"
			          - "192.168.x.x"
			  certificatesResolvers:
			    myresolver:
			      acme:
			        preferredChain: 'ISRG Root X2'
			        keyType: 'EC384'
			        storage: /etc/traefik/acme.json
			        dnsChallenge:
			          provider: provider-name
			          delayBeforeCheck: 0
			  
			  providers:
			    file:
			      filename: /etc/traefik/dynamic.yaml
			      watch: true
			  
			  ```
		- Now make service changes avaliable for systemd daemon and add the enviroment variables for your dns provider
			- ```sh
			  systemctl edit traefik.service 
			  ```
				- ```[Service]
				  [Service]
				  Environment="API_KEY=1234"
				  Environment="OTHER_ENV="
				  ```
				- ```sh
				  sudo systemctl daemon-reload
				  ```
		- Now turn on Traefik service
			- Create Traefik apparmor profie
				- ```bash
				  touch /etc/apparmor.d/home.traefik.prod.traefik && nano /etc/apparmor.d/home.traefik.prod.traefik
				  ```
				- ```bash
				  aa-enforce /etc/apparmor.d/home.traefik.prod.traefik
				  ```
			- In second console start and enable traefik service
				- ```sh
				   systemctl enable --now traefik.service
				  ```
			- Then in first terminal scan the traefik request and allow all traefik requests
	- Install Nextcloud
		- You can install Nextcloud in the way you prefer, for didactical pruposes i will use the snap package
			- ```sh
			  sudo snap install nextcloud
			  sudo snap set nextcloud ports.http=81
			  sudo snap set nextcloud ports.https=444
			  sudo snap set nextcloud php.memory-limit=1024M
			  sudo snap set nextcloud http.compression=true
			  sudo nextcloud.occ config:system:set session.cookie_secure --value="true"
			  ```
			-
	- Make traefik handle nextcloud requests
		- nano /etc/traefik/dynamic.yaml
		- ```yaml
		  http:
		    middlewares:
		      hsts-header:
		        headers:
		          customResponseHeaders:
		            Strict-Transport-Security: "max-age=15552000; includeSubDomains; preload"
		      nextcloud-caldav-redirect:
		        redirectRegex:
		          regex: "^https://subdomain.domain.com/.well-known/caldav"
		          replacement: "https://subdomain.domain.com/remote.php/dav/"
		          permanent: true
		      nextcloud-carddav-redirect:
		        redirectRegex:
		          regex: "^https://subdomain.domain.com/.well-known/carddav"
		          replacement: "https://subdomain.domain.com/remote.php/dav/"
		          permanent: true
		      nextcloud-webfinger-redirect:
		        redirectRegex:
		          regex: "^https://subdomain.domain.com/.well-known/webfinger"
		          replacement: "https://subdomain.domain.com/index.php/.well-known/webfinger"
		          permanent: true
		      nextcloud-nodeinfo-redirect:
		        redirectRegex:
		          regex: "^https://subdomain.domain.com/.well-known/webfinger"
		          replacement: "https://subdomain.domain.com/index.php/.well-known/nodeinfo"
		          permanent: true
		  
		    routers:
		      nextcloud:
		        middlewares:
		          - hsts-header
		          - nextcloud-caldav-redirect
		          - nextcloud-carddav-redirect
		          - nextcloud-webfinger-redirect
		          - nextcloud-nodeinfo-redirect
		        rule: "Host(`subdomain.domain.com`)"
		        service: nextcloud-service
		        tls:
		          certResolver: myresolver
		        entryPoints:
		          - websecure
		          - web
		    services:
		      nextcloud-service:
		        loadBalancer:
		          servers:
		            - url: "http://localhost:81"
		  ```
		- Install nextcloud
			- ```sh
			  sudo nextcloud.occ config:system:set overwriteprotocol --value="https"
			  sudo nextcloud.occ config:system:set overwritehost --value="subdomain.domain.com"
			  sudo nextcloud.occ config:system:set default_phone_region --value="MX"
			  sudo nextcloud.occ config:system:set default_language --value="en"
			  sudo nextcloud.occ config:system:set default_locale --value="es_MX"
			  sudo nextcloud.occ config:system:set trusted_proxies 0 --value="your.machine.ip.value"
			  sudo nextcloud.occ config:system:set overwrite.cli.url --value="https://subdomain.domain.com"
			  
			  ```
	- Install mariadb
		- ```sh
		   apt-get install apt-transport-https curl
		   mkdir -p /etc/apt/keyrings
		   curl -o /etc/apt/keyrings/mariadb-keyring.pgp 'https://mariadb.org/mariadb_release_signing_key.pgp'
		  ```
		-
		- Now paste in `/etc/apt/sources.list.d/mariadb.sources`
			- ```source
			  # MariaDB 11.4 repository list - created 2025-03-15 04:42 UTC
			  # https://mariadb.org/download/
			  X-Repolib-Name: MariaDB
			  Types: deb
			  # deb.mariadb.org is a dynamic mirror if your preferred mirror goes offline. See https://mariadb.org/mirrorbits/ for details.
			  # URIs: https://deb.mariadb.org/11.4/debian
			  URIs: https://mirrors.accretive-networks.net/mariadb/repo/11.4/debian
			  Suites: bookworm
			  Components: main
			  Signed-By: /etc/apt/keyrings/mariadb-keyring.pgp
			  
			  ```
		- Now Install mariadb, to avoid a weird crash between mariadb and mysql used in nextcloud snap you musto stop nextcloud while mariadb is installing
			- ```sh
			  snap stop nextcloud
			  apt-get update
			  apt-get install mariadb-server
			  snap start nextcloud
			  ```
		-
		- Now make traefik handle the mariadb requests. Add to entrypoints in `/etc/traefik/traefik.yaml`
			- ```yaml
			    database-entrypoint:
			      address: ":0000"
			  ```
			-
		- And redirect the requests to mariadb `/etc/traefik/dynamic.yaml`
			- ```yaml
			  tcp:
			    routers:
			      mdb:
			        rule: "HostSNI(`subdomain.domain.com`)"
			        service: mariadb-service
			        entryPoints:
			          - database-entrypoint
			        tls:
			          certResolver: myresolver
			    services:
			      mariadb-service:
			        loadBalancer:
			          servers:
			            - address: "127.0.0.1:3306"
			  
			  ```
		- fg
	- Install gitea
		- Create mariadb user
			- ```sh
			  mariadb -u root
			  ```
			- ```sql
			  CREATE DATABASE giteadb CHARACTER SET 'utf8mb4' COLLATE 'utf8mb4_bin';
			  CREATE USER 'gitea'@'example.gitea' IDENTIFIED BY 'gitea';
			  GRANT ALL PRIVILEGES ON giteadb.* TO 'gitea'@'example.gitea';
			  FLUSH PRIVILEGES;
			   
			  ```
			- If mariadb is in same host check the error and change the user host
		- Install git
			- ```sh
			  apt update; apt install git
			  ```
		- Create gitea user
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
		- Create required folders
			- ```sh
			  mkdir -p /var/lib/gitea/{custom,data,log}
			  chown -R git:git /var/lib/gitea/
			  chmod -R 750 /var/lib/gitea/
			  mkdir /etc/gitea
			  chown root:git /etc/gitea
			  chmod 770 /etc/gitea
			  ```
		- Download gitea binary and set the working folders
			- ```sh
			  wget https://dl.gitea.com/gitea/1.23.5/gitea-1.23.5-linux-amd64
			  mv gitea-1.23.5-linux-amd64 gitea
			  chmod 700 gitea
			  export GITEA_WORK_DIR=/var/lib/gitea/
			  cp gitea /usr/local/bin/gitea
			  chown git:git /usr/local/bin/gitea
			  ```
			- If you want to install a newer or older version just change the desired version in the url.
		- Create the gitea linux service in `/etc/systemd/system/gitea.service`
			- ```sh
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
			- Add this in `/etc/apparmor.d/usr.local.bin.gitea`
				- ```apparmor
				  abi <abi/3.0>,
				  
				  include <tunables/global>
				  
				  /usr/local/bin/gitea {
				    include <abstractions/apache2-common>
				    include <abstractions/base>
				    include <abstractions/dbus-session-strict>
				  
				    /etc/passwd r,
				    /proc/sys/net/core/somaxconn r,
				    /sys/kernel/mm/transparent_hugepage/hpage_pmd_size r,
				    /usr/bin/git mrix,
				    /usr/local/bin/gitea mr,
				    owner /etc/gitea/app.ini rw,
				    owner /home/git/** rw,
				    owner /proc/*/cpuset r,
				    owner /var/lib/gitea/** rwlk,
				  }
				  ```
			- Then turn on profile
				- ```sh
				  aa-enforce /etc/apparmor.d/usr.local.bin.gitea
				  ```
			- Then start gitea service
				- ```sh
				  systemctl enable --now gitea.service
				  ```
		- Now make traefik handle the requests
			- Add to `/etc/traefik/dynamic.yaml`
				- ```yaml
				    routers:
				      gitea:
				        rule: "Host(`sub.domain.com`)"
				        service: gitea-service
				        tls:
				          certResolver: myresolver
				        entryPoints:
				          - websecure
				  ```
		-
	- Dump traefik certs generated
		- With root user download the script to dump traefik certs into files
			- ```sh
			  mkdir -m 700 certs-dumping
			  cd certs-dumping/
			  wget "https://github.com/ldez/traefik-certs-dumper/releases/download/v2.9.3/traefik-certs-dumper_v2.9.3_linux_amd64.tar.gz"
			  tar -zxvf traefik-certs-dumper_v2.9.3_linux_amd64.tar.gz
			  rm traefik-certs-dumper_v2.9.3_linux_amd64.tar.gz LICENSE
			  chmod 700 traefik-certs-dumper
			  ```
			- Now create an script to execute when some cert change
				- ```sh
				  touch when-cert-changes.sh
				  chmod 700 when-cert-changes.sh
				  ```
				- Add to the file
				- ```sh
				  #!/bin/bash
				  cp ~/certs-dumping/dump/private/letsencrypt.pem /var/snap/nextcloud/current/
				  cp ~/certs-dumping/dump/subdomain.domain.com/certificate.pem /var/snap/nextcloud/current
				  cp ~/certs-dumping/dump/subdomain.domain.com/privatekey.pem /var/snap/nextcloud/current
				  sudo nextcloud.enable-https custom -s /var/snap/nextcloud/current/cert.pem /var/snap/nextcloud/current/privkey.pem /var/snap/nextcloud/current/chain.pem
				  ```