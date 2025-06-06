# PHP Project with Docker and MySQL

This project provides a ready-to-use Docker configuration for developing and running PHP applications with a MySQL database. It includes an Nginx web server, PHP-FPM, and a MySQL service, along with PhpMyAdmin for easy database management.

## Project Structure

```
myproject/
├── docker-compose.yml # Define Docker services (PHP, Nginx, MySQL, PhpMyAdmin)
├── php/
│ ├── Dockerfile # To customize PHP (install extensions, Composer, etc.)
│ └── php.ini # Custom PHP settings (upload_max_filesize, etc.)
└── src/ # This is where your PHP code goes (your application)
└── index.php # Initial example or entry point for your application
└── composer.json # If you use Composer for your PHP dependencies
```

## Prerequisites

* Docker Desktop installed on your system.

## How to Get Started

1. **Clone or download this project.**
2. **Place your existing PHP code in the `src/` directory.** If you are starting a new project, you can begin creating your PHP files within this folder.
3. **Set environment variables (optional):**
* You can modify the `docker-compose.yml` file to change MySQL passwords, exposed ports, or other settings according to your needs. Look for the `environment` sections in the `db` and `phpmyadmin` services.

## Usage

### 1. Build and Start the Containers

Open a terminal at the project root (where the `docker-compose.yml` file is located) and run the following command:

```bash
docker-compose up -d --build
```

* `up`: Create and start the containers.
* `-d`: Run containers in the background (detached mode).
* `--build`: Build images before starting containers (useful if you've modified the PHP Dockerfile).

### 2. Access Your Application

Once the containers are running:

* **Your PHP application:** Open your browser and go to `http://localhost:8080` (or whatever port you configured for the web service in `docker-compose.yml`).
* **PhpMyAdmin:** Open your browser and go to `http://localhost:8081` (or whatever port you configured for `phpmyadmin`). Use the MySQL credentials defined in `docker-compose.yml` to log in. By default (if you haven't changed anything), the server is `db`, the user is `root`, and the password is the one you set in `MYSQL_ROOT_PASSWORD`.

### 3. Connect PHP to MySQL

In your PHP code in the `src/` directory, you can connect to the MySQL database using the following details:

* **Host:** `db` (this is the name of the MySQL service defined in `docker-compose.yml`)
* **Database name:** The one you defined in `MYSQL_DATABASE` in `docker-compose.yml` (for example, `mydatabase`).
* **User:** The one you defined in `MYSQL_USER` in `docker-compose.yml` (for example, `myuser`).
* **Password:** The one you defined in `MYSQL_PASSWORD` in `docker-compose.yml`.

**Example of connecting to a database with .env**

```
DB_SERVER=db
DB_USERNAME=my_db_username
DB_PASSWORD=root
DB_NAME=my_database
```

Place this code (or similar) in a `.env` file inside the `src/myproject` directory to test the connection to MySQL.

## Useful Docker Commands

* **View container logs:**
```bash
docker-compose logs -f web # Web container logs (Nginx/PHP)
docker-compose logs -f db # MySQL container logs
```
(Replace `web` or `db` with the name of the service you want to inspect.)

* **Run commands inside a container:**
This is useful for using Composer, running database migrations, or accessing the console of a PHP framework.
```bash
docker-compose exec web bash
```
This will give you a shell session inside the `web` container. Once inside, you can run commands like:
```bash
# composer install
# composer update
# exit (to exit the container)
```

* **Stop containers:**
```bash
docker-compose down
```

* **Stop and delete volumes (will delete MySQL data):**
**WARNING!** This command will permanently delete your MySQL database data if it is stored on a Docker-managed volume. ```bash
docker-compose down -v
```

* **Rebuild and start (if you've made changes to the Dockerfile or docker-compose.yml):**
```bash
docker-compose up -d --build
```

## Customization

* **PHP:** Modify `php/Dockerfile` to install additional PHP extensions (e.g., `gd`, `zip`, `intl`) or tools like Composer. Adjust `php/php.ini` for PHP-specific configurations (e.g., `upload_max_filesize`, `memory_limit`). After making changes, remember to rebuild the images with `docker-compose up -d --build`.
* **Nginx:** The Nginx configuration is located inside the PHP image (or you can build a custom configuration if needed). For most cases, the default configuration should work.

* **Nginx:** The Nginx configuration is located inside the PHP image (or you can build a custom configuration if needed). For most cases, the default configuration should work. * **MySQL:** You can change the MySQL version, passwords, and database names directly in the `docker-compose.yml` file.

## To develop with WordPress and be able to install plugins, follow this:
* Define FS_METHOD in wp-config.php (Most recommended for Docker)
* This is the cleanest and safest way for Docker environments. It tells WordPress to attempt to write directly to the filesystem, assuming the permissions are correct.
Edit your wp-config.php file: This file is located in the root of your WordPress installation (inside your src/ folder, which is mounted at /var/www/html/).
Add the following line somewhere in the file, preferably before the /* That's all, stop editing! Happy publishing. */ line:
```
define('FS_METHOD', 'direct');
```
* Save the wp-config.php file.
Ensure file permissions: After adding this line, you need to ensure that the user running Apache/PHP (usually www-data inside the container) has write permissions to the wp-content/plugins/ and wp-content/upgrade/ folders (and sometimes wp-content/uploads/ for other operations).
Inside the container (the preferred method for setting permissions if FS_METHOD is direct):
Enter the web container:

```
docker-compose exec web bash
```

* This will open a command line inside the web container. You'll see a different prompt, something like root@my_advanced_web_app:/var/www/html# or www-data@my_advanced_web_app:/var/www/html$.
Navigate to the WordPress root directory inside the container (if you're not already there):
The prompt should already tell you that you're in /var/www/html, which is where your src/ folder is mounted. If not, run:
```
cd /var/www/html
```
* Change the owner of the relevant folders to www-data:
Run the following commands one by one:

```
chown -R www-data:www-data wp-content/plugins
chown -R www-data:www-data wp-content/upgrade
chown -R www-data:www-data wp-content/themes
chown -R www-data:www-data wp-content/uploads
```
* If any of these folders don't already exist (for example, upgrade or uploads are sometimes created the first time they're needed), the command will simply do nothing for that specific folder, which is fine. The most important one for installing plugins is wp-content/plugins.

* chown changes the owner.
-R means recursive (applies the change to the folder and all its contents).
www-data:www-data sets the user www-data and the group www-data.
Adjust the folder permissions so that the owner (www-data) can write:
Run the following commands one by one:

```
chmod -R 755 wp-content/plugins
chmod -R 755 wp-content/upgrade
chmod -R 755 wp-content/themes
chmod -R 755 wp-content/uploads
```
* chmod changes the permissions.
-R is recursive.
755 means:
Owner (www-data): Read, Write, Execute (rwx)
Group (www-data): Read, Execute (r-x)
Other: Read, Execute (r-x)
For files within these folders, WordPress sometimes requires 644. If the above commands for directories aren't enough, you might need something more granular, but 755 for directories is usually the first step.
Exit the container:
Type exit and press Enter.
```
exit
```

# Managing System Packages in Docker Containers

How to install and uninstall operating system packages within a Docker container using a Debian or Ubuntu-based distribution (e.g., images like `php:apache`, `node`, `python`, `ubuntu`, `debian`, etc.).

There are two main approaches to package management:

1. Temporary Modifications: These are made within an already running container. They are useful for quick testing or one-off needs, but changes will be lost if the container is stopped and rebuilt from its original image.
2. Permanent Modifications: These are made by editing the Dockerfile that defines the container image. This is the recommended method to ensure changes persist across container restarts and rebuilds, and to maintain a reproducible environment.

---

## 1. Temporary Modifications (Inside a Running Container)

These changes only affect the current container instance.

### A. Accessing the Container Shell

To make changes, you first need to obtain an interactive shell inside the desired container.

* **If you are using Docker Compose** (and your service is called, for example, `my_service`):
```bash
docker-compose exec my_service bash
```

* **If you are using `docker run`** (and you know the container name or ID):
```bash
docker exec -it <container_name_or_id> bash
```

Once the command is executed, your prompt will change, indicating that you are inside the container shell (e.g., `root@container_id:/#`).

### B. Temporarily Installing Packages

From the container shell:

1. **Update the package list** of the repository (it's good practice to do this before installing):
```bash
apt-get update
```
or, more concisely:
```bash
apt update
```

2. **Install the desired package:**
Replace `package-name` with the actual name of the package you want to install (e.g., `nano`, `git`, `curl`, `vim`).
```bash
apt-get install -y package-name
```
or:
```bash
apt install -y package-name
```
The `-y` flag automatically answers "yes" to any confirmation prompts during the installation.

### C. Temporarily Uninstall Packages

From the container shell:

1. **Uninstall the package:**
```bash
apt-get remove package-name
```
or:
```bash
apt remove package-name
```

2. **To uninstall the package AND remove its configuration files:**
```bash
apt-get purge package-name
```
or:
```bash
apt purge package-name
```

3. **Optional: Clean up dependencies that are no longer needed:**
After uninstalling packages, some dependencies might be left behind. ```bash
apt-get autoremove
```
or:
```bash
apt autoremove
```

### D. Exiting the Container

When you're done installing or temporarily uninstalling packages, exit the container shell:

```bash
exit
```
Happy development!
