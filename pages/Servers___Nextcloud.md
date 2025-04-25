## Install Nextcloud [[Servers/Services]]
	- Even if snaps are controversial, as system admin I don't like to manage so much things and Nextcloud tends to be hard to maintain, snaps provide automatic updates and all dependencies are installed in isolated environment (such as Docker) so installed dependencies are never affected and could have newer or older versions.
	- ### Install Snap daemon
		- ```sh
		  apt install snapd
		  snap install snapd
		  snap install nextcloud
		  ```
		- In rare situations you will need
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
- ```sh
   nextcloud.occ config:system:set overwriteprotocol --value="https"
   nextcloud.occ config:system:set overwritehost --value="subdomain.domain.com"
   nextcloud.occ config:system:set default_phone_region --value="MX"
   nextcloud.occ config:system:set default_language --value="en"
   nextcloud.occ config:system:set default_locale --value="es_MX"
   nextcloud.occ config:system:set trusted_proxies 0 --value="your.machine.ip.value"
   nextcloud.occ config:system:set overwrite.cli.url --value="https://subdomain.domain.com"
  
  ```