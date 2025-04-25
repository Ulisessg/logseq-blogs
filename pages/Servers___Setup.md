public:: true

- ## Resources
- [Traefik Linux service](https://github.com/traefik/traefik/blob/master/contrib/systemd/traefik.service)
- [IOMMU](https://lenovopress.lenovo.com/lp1467.pdf)
- [IOMMU by kernel changes](https://www.kernel.org/doc/Documentation/Intel-IOMMU.txt)
- [Spectre vulnerability](https://spectreattack.com/spectre.pdf)
- [/dev/mem](https://man7.org/linux/man-pages/man4/mem.4.html)
- [Linux lockdown](https://www.muylinux.com/2019/10/02/lockdown-es-el-nuevo-modulo-de-seguridad-que-permitira-bloquear-partes-del-kernel-en-linux-5-4/)
- [Kernel parameters - Arch wiki](https://wiki.archlinux.org/title/Kernel_parameters)
- [AppArmor - Arch wiki](https://wiki.archlinux.org/title/AppArmor)
- [AppArmor - Ubuntu docs](https://documentation.ubuntu.com/server/how-to/security/apparmor/index.html)
- [Linux kernel params - Reference](https://docs.kernel.org/admin-guide/kernel-parameters.html)
- [Mariadb server repositories](https://mariadb.org/download/?t=repo-config&d=Debian+12+%22Bookworm%22&v=11.4&r_m=accretive)
- [Fix install error of mariadb while Nextcloud snap is running](https://askubuntu.com/questions/1523326/cant-install-mariadb-on-ubuntu-24-04)
- [Gitea linux service](https://docs.gitea.com/installation/linux-service)
- [Gitea installation from binary](https://docs.gitea.com/installation/install-from-binary)
- [Gitea Linux service reference](https://github.com/go-gitea/gitea/blob/release/v1.23/contrib/systemd/gitea.service)
- {{video https://www.youtube.com/watch?v=irQqa_D9Rik}}
- [How to install redmine - Video tutorial](video https://www.youtube.com/watch?v=irQqa_D9Rik)
- [Install ruby](https://guides.rubyonrails.org/install_ruby_on_rails.html#install-ruby-on-ubuntu)
-
-
-
- ## During server OS installation
	- ### Encrypt your hard drive [[Servers/Hardening]]
		- Is so recommended when you install any OS encrypt your data to avoid being stole if someone access physically your hard drive. Debian (and any Linux distro) allow you to use Luks to encrypt data.
		- [What is LUKS](https://es.wikipedia.org/wiki/LUKS)
		- [How to setup LUKS encryption](https://www.youtube.com/watch?v=GEl2S5MI-WU)
		- [LUKS Faq](https://gitlab.com/cryptsetup/cryptsetup/-/wikis/FrequentlyAskedQuestions)
- ## After OS installation
	- ### Verify all Debian sources use HTTPS protocol [[Servers/Hardening]]
		- Even if debian packages have package intregity checks is a good idea to use encrypted comunications all the time
		- https://www.debian.org/doc/manuals/securing-debian-manual/deb-pack-sign.en.html
		- Open with sudo permissions `/etc/apt/sources.list` file and verify *URI* uses https protocol, if folder `/etc/apt/sources.list.d/` also have files do the same
	- ### Upgrade all packages and their repos
		- ```sh
		  sudo apt update && sudo apt upgrade
		  ```
	- ### Use SSH key to remote connection [[Servers/Hardening]]
		- Using a ssh key to remote login gives you an extra layer of security, if you use password login and someone stole your password that person will be able to access your server, but if you use ssh keys even if someone stole your password they will no be able to login into your account.
		- Execute this command, replace the fake email with yours or leave it in blank
			- ```sh
			  ssh-keygen -t ed25519 -C "your_email@example.com"
			  ```
			- When prompt asks you a password **use a secure password**, if someone stole your private ssh key they will not be able to use it without your password.
		- Add the ssh key to your ssh agent, if you use a custom name for ssh key remember to use it in the command.
			- ```sh
			  eval "$(ssh-agent -s)"
			  ssh-add ~/.ssh/id_ed25519
			  ```
		- Copy public key in `~/.ssh/authorized_keys` in user in server.
			- In your local machine.
				- ```sh
				  ssh-copy-id -i ~/.ssh/id_ed25519 user@hostname.example.com
				  ```
		- Disable password authentication
			- In the server login as root user or use sudo to modify `/etc/ssh/sshd_config` file, change two values
			- ```
			  PasswordAuthentication no
			  PermitRootLogin no
			  ```
			- Then reset ssh and sshd services
				- ```sh
				  systemctl restart ssh
				  systemctl restart sshd
				  ```
		- https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent
	- ### Use a firewall [[Servers/Hardening]]
		- Check if you have installed "Uncomplicated Firewall" (ufw).
			- ```sh
			  ufw status numbered
			  ```
			- If you need to install use next command
				- ```sh
				  apt install ufw
				  ```
		- Now enable 22, 80 and 443 ports
			- Port 22 is used for ssh connections
			- Port 80 is used for http request **(don't worry we will convert http requests in https in further posts)**
			- Port 443 is used for https connections
			- ```sh
			   sudo ufw allow 22
			   sudo ufw allow 80
			   sudo ufw allow 443
			   sudo ufw enable
			  ```
	- ### Use AppArmor as security module [[Servers/Hardening]]
		- I have two strong reasons to use Apparmor instead Selinux
			- Is more easy to do maintenance and debug.
			  logseq.order-list-type:: number
			- I have no experience with Selinux, I'm learning about and eventually will learn so for now let's use Apparmor
			  logseq.order-list-type:: number
		- As root verify if Apparmor is enabled. I'm very confident will be enabled in fresh install of Debian 12 but is a good idea check first
			- ```sh
			  aa-enabled
			  ```
		- Then install some useful utilities
			- ```sh
			  apt install apparmor-profiles apparmor-utils auditd
			  ```
			- Apparmor profiles package gives you predefined and tested profiles.
			- Apparmor utils gives you utils as aa-logprof to check which programs are requesting permission over a file or resource, the impact of give the permission or the ability to ignore it.
			- Auditd is a dependence to aa-logprof to correct behavior
	- https://wiki.debian.org/AppArmor/HowToUse
	- ### Improve kernel security adding parameter to the kernel on boot [[Servers/Hardening]]
		- You can modify Linux kernel behavior and do some vulnerabilities mitigation such as CPU attacks.
		- As root or with sudo command open file `/etc/default/grub`
		- Search the param "GRUB_CMDLINE_LINUX" and add
			- quiet splash apparmor=1 security=apparmor mem_encryption=on lockdown=confidentiality mitigations=auto,nosmt intel_iommu=on iommu.strict=1 mem_sleep_default=s2idle
			- Explanation:
				- quiet: Disable most log messages when kernel is booting, this is helpful to avoid someone so curious know what happens with your server during boot.
				- splash: Makes more pretty the loading screen.
				- mitigations: Add mitigation mechanisms for know CPU vulnerabilities such as [spectre](https://es.wikipedia.org/wiki/Spectre_(vulnerabilidad)). You can specify which mitigations you want but this approach gives you an scalable way to protect against most vulnerabilities.
				- intel_iommu: Allows you to use your hardware in virtual machines, if you have AMD CPU change intel for amd, some CPU are not compatible with this technology.
				- mem_encryption: Some intel and AMD processors allows you to encrypt RAM memory to avoid being read in plain text.
				- lockdown: Security module to avoid any user reads confidential kernel info such as [phisical RAM memory](https://man7.org/linux/man-pages/man4/mem.4.html). Useful if you have no mem_encryption option, if you have mem_encryptio provides an extra security layer.
				- mem_sleep_default: Disable Suspend To Ram mode, if pc goes to suspend mode could save the current state in different places, if that place is ram someone can read the current state if have physical access to server (more dangerous if your CPU don't support ram encryption and you don't have confidentiality mode in lockdown parameter), most servers (or maybe all) never goes to suspend mode, this is more useful in personal computers or laptops.
				- https://www.linux-kvm.org/page/How_to_assign_devices_with_VT-d_in_KVM
				- https://docs.kernel.org/admin-guide/kernel-parameters.html
			- Update grub config
				- ```sh
				  update-grub
				  reboot
				  ```
	- ### Upgrade server firmware [[Servers/Hardening]]
		- You can upgrade the firmware of some device in Linux using fwupd tool. This tool also is useful to verify machine hardware security, as advice you **NEVER** will have all security checks, the best approach is have as many as you can. Also this is not a 100% reliable check, even if you have almost all checks you can be hacked from other vectors (like don't upgrade your software, weak passwords, insecure connections, etc).
			- If you are using `secure boot` and I hope it, you must use a Machine Owner Key to sing the installed firmware.
				- ```sh
				  apt install dkms
				  dkms generate_mok
				  # This will prompt you to write a password
				  mokutil --import /var/lib/dkms/mok.pub
				  mokutil --list-new
				  reboot
				  ```
				- After reboot you will have to enroll MOK using the password.
			- Verify fwupd is already installed and check hardware security
				- ```sh
				  fwupdmgr security
				  ```
			- If not installed use
				- ```sh
				  apt install fwupd
				  ```
			- Check for updates in firmware and install it
				- ```sh
				  fwupdmgr get-upgrades
				  fwupdmgr upgrade
				  ```
		- https://wiki.debian.org/SecureBoot
		- https://wiki.debian.org/SecureBoot#MOK_-_Machine_Owner_Key