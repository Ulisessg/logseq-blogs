- Keycloak is an Oauth provider, you can use it to add social media login in your apps, or use their own login service
- ### Installing keycloak
- {{embed ((680be69d-c4f1-4a6d-883d-a9bd3e4e03d3))}}
- {{embed ((680be6fa-1991-42aa-a9e9-e04da6b0b7f6))}}
- {{embed ((680fb9ee-ad2e-4ca3-b0b0-e7f24c09f493))}}
- {{embed ((680fb9f4-57c3-436d-b4b2-16b2a6d03c52))}}
- ### Create MariaDB user and database
	- ```sh
	  mysql -u root
	  ```
	- ```mysql
	  -- Create a user with a password
	  CREATE USER 'keycloak'@'%' IDENTIFIED BY 'ThestrongestPasswordInTheW0rldYeah';
	  
	  -- Create the database owned by the new user
	  CREATE DATABASE keycloak CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
	  
	  -- Grant all privileges on the database to the user
	  GRANT ALL PRIVILEGES ON keycloak.* TO 'keycloak'@'%';
	  
	  ```
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
- ### Create keycloak service
	- logseq.order-list-type:: number
	  ```bash
	  touch /etc/systemd/system/keycloak.service
	  nano /etc/systemd/system/keycloak.service
	  ```
	- logseq.order-list-type:: number
	  ```service
	  [Unit]
	  Description=Identity and session manager
	  After=syslog.target network.target
	  [Service]
	  Type=notify
	  User=keycloak
	  ExecStart=/home/keycloak/prod/bin/kc.sh start --optimized --http-enabled true --proxy-headers xforwarded --http-port=8180
	  [Install]
	  WantedBy=multi-user.target
	  ```
- ### Create AppArmor profile
	- logseq.order-list-type:: number
	  ```bash
	  touch /etc/apparmor.d/home.keycloak.prod.bin.kc.sh
	  nano /etc/apparmor.d/home.keycloak.prod.bin.kc.sh
	  ```
	- logseq.order-list-type:: number
	  ```service
	  abi <abi/3.0>,
	  
	  include <tunables/global>
	  
	  /home/keycloak/prod/bin/kc.sh {
	    include <abstractions/apache2-common>
	    include <abstractions/base>
	    include <abstractions/bash>
	    include <abstractions/dbus-session-strict>
	    include <abstractions/opencl-pocl>
	    include <abstractions/user-tmp>
	  
	    capability dac_override,
	    capability dac_read_search,
	  
	    /etc/nsswitch.conf r,
	    /etc/passwd r,
	    /etc/timezone r,
	    /home/keycloak/prod/bin/kc.sh r,
	    /proc/*/net/if_inet6 r,
	    /proc/cgroups r,
	    /proc/sys/net/core/somaxconn r,
	    /sys/devices/system/cpu/cpu0/microcode/version r,
	    /sys/fs/cgroup/system.slice/keycloak.service/memory.max r,
	    /sys/fs/cgroup/user.slice/user-0.slice/session-c3.scope/cpu.max r,
	    /sys/fs/cgroup/user.slice/user-0.slice/session-c3.scope/memory.max r,
	    /sys/kernel/mm/hugepages/ r,
	    /sys/kernel/mm/transparent_hugepage/enabled r,
	    /sys/kernel/mm/transparent_hugepage/hpage_pmd_size r,
	    /usr/bin/dash ix,
	    /usr/bin/dirname mrix,
	    /usr/bin/printf mrix,
	    /usr/bin/readlink mrix,
	    /usr/bin/sed mrix,
	    /usr/bin/uname mrix,
	    /usr/bin/xargs mrix,
	    /usr/lib/jvm/temurin-21-jre-amd64/bin/java mrix,
	    owner /home/** r,
	    owner /home/** w,
	    owner /proc/*/cgroup r,
	    owner /proc/*/cmdline r,
	    owner /proc/*/coredump_filter rw,
	    owner /proc/*/fd/ r,
	    owner /proc/*/mountinfo r,
	    owner /proc/*/stat r,
	  }
	  
	  ```
		- logseq.order-list-type:: number
	- Active profile
	  logseq.order-list-type:: number
		- logseq.order-list-type:: number
		  ```bash
		  aa-enforce /etc/apparmor.d/home.keycloak.prod.bin.kc.sh
		  ```