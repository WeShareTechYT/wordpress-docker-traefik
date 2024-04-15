### 1. Setting up Permissions and Docker Network

Make sure to grant execute permission to the setup script and create a Docker network named "traefik_network".

```bash
chmod +x setup.sh
./setup.sh
docker network create traefik_network
```

### 2. Updating Your Data

Before proceeding, update the following details:

- Email: Must be the same on every container.
- Website URL: Specify the URL of your website.
- Passwords: Generate new passwords, ensuring consistency between the WordPress and MariaDB containers.

```yml
MYSQL_ROOT_PASSWORD: newpassword
WORDPRESS_DB_PASSWORD: newpassword
```

### 3. Usage
After updating your data, execute the following command to start the Containers:

```bash
docker-compose up -d
```

Then you can visit your site at https://yoursite.com and proceed with the WordPress installation.


### Adding a New Website
Follow these steps to add a new website:

1. Add SQL command to create_database.sql to create a new database:

```sql
CREATE DATABASE IF NOT EXISTS newwebsite;
```


2. Update docker-compose.yml

  2.1. Update Traefik Container:
  Add the following block to your Traefik container configuration:
  
  ```yml
  - "--certificatesresolvers.newwebsite_resolver.acme.tlschallenge=true"
  - "--certificatesresolvers.newwebsite_resolver.acme.email=name@yourmail.de"
  - "--certificatesresolvers.newwebsite_resolver.acme.storage=/letsencrypt/newwebsite.json"
  ```
  
  2.2. Add new Wordpress Container: 
  
  ```yml
  newwebsite:
    container_name: newwebsite
    image: wordpress:latest
    restart: always
    environment:
      WORDPRESS_DB_HOST: mariadb
      WORDPRESS_DB_USER: root
      WORDPRESS_DB_PASSWORD: 123
      WORDPRESS_DB_NAME: newwebsite
    networks:
      - traefik_network
    volumes:
      - "./data/newwebsite:/var/www/html"
      - "./uploads.ini:/usr/local/etc/php/conf.d/uploads.ini"
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.newwebsite.entrypoints=websecure"
      - "traefik.http.routers.newwebsite.rule=Host(`newwebsite.com`)"
      - "traefik.http.routers.newwebsite.tls=true"
      - "traefik.http.routers.newwebsite.tls.certresolver=newwebsite_resolver"
  ```

Ensure to replace placeholders such as newwebsite, newwebsite.com, and name@yourmail.de with actual values.
