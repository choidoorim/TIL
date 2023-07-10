# Docker - MySQL 실행
1. `> docker compose up -d` 실행
```
# docker-compose.yaml
version: '3.7'
services:
	db:
		image: mysql:8.0
		container_name: mysql-practice
		restart: always
		environment:
			MYSQL_ROOT_PASSWORD: 'root-password'
		command: --character-set-server=utf8mb4
			--collation-server=utf8mb4_unicode_ci
		ports:
			- '3306:3306'
```
2. `> docker exec -it mysql-practice /bin/bash`  실행
3. `bash-x.x# mysql -u root -p`  실행
