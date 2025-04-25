## Install Nextcloud [[Servers/Services]]
	- Even if snaps are controversial, as system admin I don't like to manage so much things and Nextcloud tends to be hard to maintain, snaps provide automatic updates and all dependencies are installed in isolated environment (such as Docker) so installed dependencies are never affected and could have newer or older versions.
	- ### Install Snap daemon
		- ```sh
		  apt install snapd
		  snap install snapd
		  snap install nextcloud
		  ```
		- In rare situations you will need to restart your server, most of the time no.
	- ### Modify Nextcloud config
		- ```sh
		  snap set nextcloud ports.http=81
		  snap set nextcloud ports.https=444
		  snap set nextcloud php.memory-limit=1024M
		  snap set nextcloud http.compression=true
		  nextcloud.occ config:system:set session.cookie_secure --value="true"
		  nextcloud.occ config:system:set overwriteprotocol --value="https"
		  nextcloud.occ config:system:set overwritehost --value="subdomain.domain.com"
		  nextcloud.occ config:system:set default_phone_region --value="MX"
		  nextcloud.occ config:system:set default_language --value="en"
		  nextcloud.occ config:system:set default_locale --value="es_MX"
		  nextcloud.occ config:system:set trusted_proxies 0 --value="your.machine.ip.value"
		  nextcloud.occ config:system:set overwrite.cli.url --value="https://subdomain.domain.com"
		  
		  ```
		- Explanation:
		- Changes default HTTP port, this is required because Traefik is handling por 80 requests, even if Nextcloud could handle requests in por 81, the firewall will block those request to force Traefik as unique requests handler.
		  logseq.order-list-type:: number
		- Change HTTPS port, same reason as change port 80.
		  logseq.order-list-type:: number
		- Increase the allowed memory consumption if PHP, useful to faster and robust resources request. You can use as much as you want, also is useful if you have troubles with images not showing in fronted.
		  logseq.order-list-type:: number
		- Enables HTTP compression, send user compressed resources reducing network load.
		  logseq.order-list-type:: number
		- Use secure protocol to handle cookies.
		  logseq.order-list-type:: number
		- Enforce HTTPS protocol usage, if request come from HTTP will be upgraded internally in snap
		  logseq.order-list-type:: number
		- Sets explicitly the domain used.
		  logseq.order-list-type:: number
		- Sets default phone region, if you live in
		  logseq.order-list-type:: number
	- ### Add Nextcloud to Traefik dynamic file [[Servers/Traefik]]
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