public:: true

- ## Setup Traefik Reverse Proxy in Debian with Apparmor [[Servers/Reverse Proxy]]
	- Create Unix user. This will no prompt you to set a password or allows you to login.
	  logseq.order-list-type:: number
		- logseq.order-list-type:: number
		  ```bash
		  useradd -r -s /bin/false -U -M traefik
		  ```
	- Download Traefik binary.
	  logseq.order-list-type:: number
		- logseq.order-list-type:: number
		  ```bash
		  wget "https://github.com/traefik/traefik/releases/download/v3.3.6/traefik_v3.3.6_linux_amd64.tar.gz"
		  tar -zxvf traefik_v3.3.6_linux_amd64.tar.gz
		  rm traefik_v3.3.6_linux_amd64.tar.gz
		  rm CHANGELOG.md
		  rm LICENSE.md
		  chmod 700 traefik
		  mv traefik /usr/bin
		  chown traefik:traefik /usr/bin/traefik
		  ```
		- Tip: If a newer Traefik version is released just change desired version in url, tar and rm commands
		  logseq.order-list-type:: number
	- Create Traefik config files and directory.
	  logseq.order-list-type:: number
		- logseq.order-list-type:: number
		  ```bash
		   mkdir -m 700 /etc/traefik
		   touch /etc/traefik/acme.json
		   chmod 600 /etc/traefik/acme.json
		   touch /etc/traefik/traefik.yaml
		   chmod 600 /etc/traefik/traefik.yaml
		   touch /etc/traefik/dynamic.yaml
		   chmod 600 /etc/traefik/dynamic.yaml
		   chown -R traefik:traefik /etc/traefik
		  ```
	- Configure Traefik entrypoints in `/etc/traefik/traefik.yaml`.
	  logseq.order-list-type:: number
		- logseq.order-list-type:: number
		  ```yaml
		  entryPoints:
		    websecure:
		      http3:
		        advertisedPort: 443
		      address: ":443"
		      forwardedHeaders:
		        trustedIPs:
		          - "127.0.0.1/32"
		          - "192.168.x.x"
		          - "::1"
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
	- Create Traefik Linux service.
	  logseq.order-list-type:: number
		- logseq.order-list-type:: number
		  ```bash
		  touch /etc/systemd/system/traefik.service
		  ```
		- And paste.
		  logseq.order-list-type:: number
			- logseq.order-list-type:: number
			  ```service
			  [Unit]
			  Description=Traefik reverse proxy
			  Documentation=https://doc.traefik.io/traefik/
			  After=network-online.target
			  AssertFileIsExecutable=/usr/bin/traefik
			  AssertPathExists=/etc/traefik/traefik.yaml
			  
			  [Service]
			  # Run traefik as its own user (create new user with: useradd -r -s /bin/false -U -M traefik)
			  User=traefik
			  AmbientCapabilities=CAP_NET_BIND_SERVICE
			  
			  # configure service behavior
			  Type=notify
			  ExecStart=/usr/bin/traefik --configFile=/etc/traefik/traefik.yaml
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
	- Now configure your DNS provider API keys and other environment variables
	  logseq.order-list-type:: number
		- logseq.order-list-type:: number
		  ```bash
		  systemctl edit traefik.service 
		  ```
			- logseq.order-list-type:: number
			  ```service
			  [Service]
			  Environment="API_KEY=1234"
			  Environment="OTHER_ENV="
			  ```
	- Now generate AppArmor Traefik profile.
	  logseq.order-list-type:: number
		- logseq.order-list-type:: number
		  ```bash
		  touch /etc/apparmor.d/usr.bin.traefik
		  ```
		- Now paste the rules.
		  logseq.order-list-type:: number
			- logseq.order-list-type:: number
			  ```
			  abi <abi/3.0>,
			  
			  include <tunables/global>
			  
			  /usr/bin/traefik {
			    include <abstractions/apache2-common>
			    include <abstractions/base>
			  
			    /etc/resolv.conf r,
			    /proc/sys/net/core/somaxconn r,
			    /sys/kernel/mm/hugepages/ r,
			    /sys/kernel/mm/transparent_hugepage/hpage_pmd_size r,
			    /usr/bin/traefik mr,
			    owner /etc/traefik/acme.json rw,
			    owner /etc/traefik/dynamic.yaml r,
			    owner /etc/traefik/traefik.yaml r,
			  
			  }
			  
			  ```
		- And enable it.
		  logseq.order-list-type:: number
			- logseq.order-list-type:: number
			  ```bash
			  aa-enforce /etc/apparmor.d/usr.bin.traefik
			  ```
		- At last but not least, start and enable in startup Traefik service.
		  logseq.order-list-type:: number
			- logseq.order-list-type:: number
			  ```bash
			  systemctl enable --now traefik.service
			  ```
-