# Lab: Docker Compose with Secrets & Volumes

This lab demonstrates a 3-tier architecture using **WordPress**, **MySQL**, and **phpMyAdmin**. It focuses on using **Docker Compose**, **Custom Bridge Networks**, **Bind Mounts** for database persistence, and **Docker Secrets** (via files) for security.

### Prerequisites

Ensure your Ubuntu system has Docker and Docker Compose installed:

```bash
sudo apt-get update
sudo apt-get install docker-compose -y

```

---

### 1. Project Directory Setup

First, we create the folder structure to hold our configuration, database data, and secrets.

```bash
mkdir -p ~/compose_lab/secrets
mkdir -p ~/compose_lab/mysql_data
cd ~/compose_lab

```

### 2. Create Password Secrets

Create the text files that store our database credentials. **Note:** These are created as files, not directories.

```bash
echo "**aa12345" > ~/compose_lab/secrets/root_pass.txt
echo "wordpress" > ~/compose_lab/secrets/wp_pass.txt

```

### 3. Create the Docker Compose File

Create a file named `docker-compose.yaml` in the `~/compose_lab` directory:
`nano docker-compose.yaml`

**Paste the following configuration:**

```yaml
version: '3.8'

services:
  db:
    image: mysql:5.7
    container_name: mysql_container
    restart: always
    networks:
      my_network:
        ipv4_address: 172.18.0.2
    volumes:
      - ./mysql_data:/var/lib/mysql
      - ./secrets/root_pass.txt:/run/secrets/root_pass
      - ./secrets/wp_pass.txt:/run/secrets/wp_pass
    environment:
      MYSQL_ROOT_PASSWORD_FILE: /run/secrets/root_pass
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD_FILE: /run/secrets/wp_pass

  wordpress:
    image: wordpress:latest
    container_name: wordpress_container
    restart: always
    ports:
      - "8080:80"
    networks:
      - my_network
    volumes:
      - wordpress_data:/var/www/html
      - ./secrets/wp_pass.txt:/run/secrets/wp_pass
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD_FILE: /run/secrets/wp_pass
      WORDPRESS_DB_NAME: wordpress
    depends_on:
      - db

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: phpmyadmin_container
    restart: always
    ports:
      - "8081:80"
    networks:
      - my_network
    environment:
      PMA_HOST: db
    depends_on:
      - db

networks:
  my_network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.18.0.0/16

volumes:
  wordpress_data:

```

---

### 4. Deploy the Infrastructure

Run the following command to pull images and start the containers in detached mode:

```bash
sudo docker-compose up -d

```

### 5. Verification Steps

Verify that all containers are running and the secrets are correctly mounted:

1. **Check Container Status:**
```bash
sudo docker ps

```


2. **Test Secret Access inside WordPress:**
```bash
sudo docker exec -it wordpress_container cat /run/secrets/wp_pass

```



### 6. Access the Sites

Once everything is up, open your browser and go to:

* **WordPress:** `http://<YOUR_VM_IP>:8080`
* **phpMyAdmin:** `http://<YOUR_VM_IP>:8081` (Login with user: `root` and password: `**aa12345`)

---

### Cleanup

To stop the lab and remove all containers and networks created by this file:

```bash
sudo docker-compose down -v

```
