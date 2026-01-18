### Step 1: Create Lab Directory & Data Folders

Run these to set up the storage for the **MySQL Bind Mode** volume.

```bash
mkdir -p ~/my_lab/secrets
mkdir -p ~/my_lab/mysql_db_data
cd ~/my_lab

```

### Step 2: Create the Password Secrets

These commands create the actual text files (not folders) that Docker will use for authentication.

```bash
echo "SuperSecretRootPass" > ~/my_lab/secrets/root_pass.txt
echo "WordpressUserPass" > ~/my_lab/secrets/wp_pass.txt

```

### Step 3: Set Up the Docker Network

This creates the **MY_NETWORK** bridge with the specific subnet `172.18.0.0/16` required by your lab.

```bash
sudo docker network create --driver bridge --subnet 172.18.0.0/16 MY_NETWORK

```

### Step 4: Create the WordPress Docker Volume

This creates the managed volume specifically for the WordPress files.

```bash
sudo docker volume create wordpress_data

```

---

### Step 5: Run the MySQL Container

This command links the **Bind Mode** folder to the database and uses secrets for the passwords.

```bash
sudo docker run -d \
  --name mysql_container \
  --network MY_NETWORK \
  -v ~/my_lab/mysql_db_data:/var/lib/mysql \
  -v ~/my_lab/secrets/root_pass.txt:/run/secrets/root_pass \
  -v ~/my_lab/secrets/wp_pass.txt:/run/secrets/wp_pass \
  -e MYSQL_ROOT_PASSWORD_FILE=/run/secrets/root_pass \
  -e MYSQL_DATABASE=wordpress \
  -e MYSQL_USER=wordpress \
  -e MYSQL_PASSWORD_FILE=/run/secrets/wp_pass \
  mysql:5.7

```

### Step 6: Run the WordPress Container

This connects WordPress to the database using the internal network and the managed volume.

```bash
sudo docker run -d \
  --name wordpress_container \
  --network MY_NETWORK \
  -p 8080:80 \
  -v wordpress_data:/var/www/html \
  -v ~/my_lab/secrets/wp_pass.txt:/run/secrets/wp_pass \
  -e WORDPRESS_DB_HOST=mysql_container \
  -e WORDPRESS_DB_USER=wordpress \
  -e WORDPRESS_DB_PASSWORD_FILE=/run/secrets/wp_pass \
  -e WORDPRESS_DB_NAME=wordpress \
  wordpress:latest

```

### Step 7: Run the phpMyAdmin Container

This allows you to manage the database via your browser on port `8081`.

```bash
sudo docker run -d \
  --name phpmyadmin_container \
  --network MY_NETWORK \
  -p 8081:80 \
  -v ~/my_lab/secrets/root_pass.txt:/run/secrets/root_pass \
  -e PMA_HOST=mysql_container \
  -e MYSQL_ROOT_PASSWORD_FILE=/run/secrets/root_pass \
  phpmyadmin/phpmyadmin

```

---

### Step 8: Verify the Installation

Run this to see if all three containers are "Up":

```bash
sudo docker ps

```

You can now access 
- WordPress at example: `http://192.168.56.128:8080`
- phpMyAdmin at example:  `http://192.168.56.128:8081`
