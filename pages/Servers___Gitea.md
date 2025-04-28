public:: true

- Gitea is an open source code hosting, similar to GitHub. [[Servers/Services]]
- ### ((680fb9ee-ad2e-4ca3-b0b0-e7f24c09f493))
- ### Login in root mariadb user
	- ```sh
	  mysql -u root
	  ```
- ### Create gitea user
	- ```sh
	  CREATE DATABASE giteadb CHARACTER SET 'utf8mb4' COLLATE 'utf8mb4_bin';
	  SET old_passwords=0;
	  CREATE USER 'gitea'@'%' IDENTIFIED BY 'gitea' REQUIRE SSL;
	  GRANT ALL PRIVILEGES ON giteadb.* TO 'gitea';
	  FLUSH PRIVILEGES;
	  ```
	-