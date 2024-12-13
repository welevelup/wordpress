# WordPress Local Environment Setup with Docker

This guide explains how to set up a local WordPress environment using Docker, retrieve the live site files and database via SSH, and restore the site locally.

---

## Prerequisites

1. [Docker](https://www.docker.com/products/docker-desktop) installed on your machine.
2. SSH access to your remote WordPress server.

---

## Step 1: Set Up the Local Docker Environment

1. **Create a Project Directory:**
   ```bash
   mkdir wordpress-docker
   cd wordpress-docker
   ```

2. **Create Required Files:**
   - `Dockerfile`
   - `docker-compose.yml`

3. **Write the `Dockerfile`:**
   ```dockerfile
   # Base image
   FROM wordpress:latest

   # Install additional PHP extensions if needed
   RUN apt-get update && apt-get install -y \
       libfreetype6-dev \
       libjpeg62-turbo-dev \
       libpng-dev \
       && docker-php-ext-configure gd --with-freetype --with-jpeg \
       && docker-php-ext-install gd

   # Set permissions (optional but useful)
   RUN chown -R www-data:www-data /var/www/html

   # Expose port 80
   EXPOSE 80
   ```

4. **Write the `docker-compose.yml`:**
   ```yaml
   version: '3.8'

   services:
     wordpress:
       build: .
       container_name: wordpress
       ports:
         - "8000:80"
       environment:
         WORDPRESS_DB_HOST: db:3306
         WORDPRESS_DB_USER: wordpress_user
         WORDPRESS_DB_PASSWORD: wordpress_password
         WORDPRESS_DB_NAME: wordpress_db
       volumes:
         - ./wordpress:/var/www/html

     db:
       image: mysql:5.7
       container_name: wordpress_db
       environment:
         MYSQL_ROOT_PASSWORD: root_password
         MYSQL_DATABASE: wordpress_db
         MYSQL_USER: wordpress_user
         MYSQL_PASSWORD: wordpress_password
       volumes:
         - db_data:/var/lib/mysql

   volumes:
     db_data:
   ```

5. **Start Docker Containers:**
   ```bash
   docker-compose up -d
   ```

6. Verify the environment by visiting `http://localhost:8000`.

---

## Step 2: Retrieve Files and Database from Remote Server via SSH

### **Retrieve WordPress Files**

1. **Copy `wp-content` folder:**
   ```bash
   scp -r user@your-remote-server:/path/to/wordpress/wp-content ./wordpress/wp-content
   ```

   Replace:
   - `user` with your SSH username.
   - `your-remote-server` with your server's domain or IP.
   - `/path/to/wordpress/wp-content` with the actual path to the `wp-content` folder on your remote server.

2. **Copy Additional Files (if needed):**
   ```bash
   scp user@your-remote-server:/path/to/wordpress/wp-config.php ./wordpress/wp-config.php
   ```

### **Retrieve the Database Backup**

1. **SSH into the remote server:**
   ```bash
   ssh user@your-remote-server
   ```

2. **Export the database using `mysqldump`:**
   ```bash
   mysqldump -u db_user -p db_name > wordpress-database-backup.sql
   ```

   Replace:
   - `db_user` with your database username.
   - `db_name` with your database name.

3. **Copy the database backup to your local machine:**
   ```bash
   scp user@your-remote-server:/path/to/wordpress-database-backup.sql ./wordpress-database-backup.sql
   ```

---

## Step 3: Restore WordPress Locally

### **Replace Files Locally**

Copy the `wp-content` folder and any additional files into the local project directory:
```bash
cp -r ./wordpress/wp-content/* ./wordpress/wp-content/
```

### **Import the Database**

1. **Import the SQL file into the MySQL Docker container:**
   ```bash
   docker exec -i wordpress_db mysql -u wordpress_user -pwordpress_password wordpress_db < ./wordpress-database-backup.sql
   ```

2. **Update Site URLs in the Database:**
   ```bash
   docker exec -it wordpress_db mysql -u wordpress_user -pwordpress_password wordpress_db
   ```

   Run the following SQL commands:
   ```sql
   UPDATE wp_options SET option_value = 'http://localhost:8000' WHERE option_name = 'siteurl';
   UPDATE wp_options SET option_value = 'http://localhost:8000' WHERE option_name = 'home';
   ```

3. Exit MySQL:
   ```bash
   exit
   ```

---

## Step 4: Verify and Troubleshoot

1. Open `http://localhost:8000` in your browser.
2. Log in to the WordPress admin dashboard.
3. **Fix Permalinks (if needed):**
   - Go to **Settings > Permalinks** in the dashboard.
   - Save your preferred structure to regenerate the `.htaccess` file.

---

## Step 5: Managing Docker Containers

- **Stop containers:**
  ```bash
  docker-compose down
  ```
- **Restart containers:**
  ```bash
  docker-compose up -d
  ```

---

This guide covers all steps for setting up a local WordPress environment, retrieving files and the database from a remote server, and restoring them locally using Docker.
