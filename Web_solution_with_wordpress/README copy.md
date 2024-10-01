## Installation of Apache, PHP 8.3, and PHP Extensions on RHEL 9.4

### Prerequisites

- **RHEL 9.4** on a server or EC2 instance with **root** or **sudo** privileges.
- **Apache (httpd)** as the web server.


### Step 1: Verify RHEL Version

Before starting, ensure you are running **RHEL 9.4**. This can be done with the following command:

```bash
cat /etc/redhat-release
```

Expected output:
```
Red Hat Enterprise Linux release 9.4 (Plow)
```

### Step 2: Install Apache (httpd)

WordPress requires a web server to handle HTTP requests, and **Apache** is the most commonly used web server for WordPress installations. 

1. Install **Apache** using the `yum` package manager:
   ```bash
   sudo yum install httpd
   ```

2. Start and enable Apache to ensure it runs on boot:
   ```bash
   sudo systemctl start httpd
   sudo systemctl enable httpd
   ```

3. Check that Apache is running:
   ```bash
   sudo systemctl status httpd
   ```


### Step 3: Enable Necessary Repositories

To install the latest PHP version, we need to enable additional repositories, which are not enabled by default on **RHEL 9**.

#### Install the EPEL Repository

**EPEL (Extra Packages for Enterprise Linux)** is a repository that contains additional software packages that are not provided in the default RHEL repository but are often needed for full functionality.

```bash
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
```

#### Install the Remi Repository

**Remi’s repository** is required to install **PHP 8.3**, as RHEL’s default repositories only provide PHP versions up to 8.2.

```bash
sudo yum install https://rpms.remirepo.net/enterprise/remi-release-9.rpm
```


### Step 4: Install PHP 8.3 and Extensions

1. Reset any previously installed PHP modules:
   ```bash
   sudo yum module reset php
   ```

2. Switch to **PHP 8.3** from the Remi repository:
   ```bash
   sudo yum module enable php:remi-8.3
   ```

3. Install **PHP 8.3** and the necessary extensions for WordPress:
   ```bash
   sudo yum install php php-opcache php-gd php-curl php-mysqlnd php-xml php-json php-mbstring php-intl php-soap php-zip
   ```

#### Explanation of Key PHP Extensions:

- **php-opcache**: Boosts performance by storing precompiled script bytecode in memory.
- **php-gd**: Provides image manipulation capabilities (needed for image uploads and manipulation in WordPress).
- **php-curl**: Allows external HTTP requests, used by WordPress to connect to other websites (e.g., for API calls).
- **php-mysqlnd**: Native MySQL driver for connecting WordPress to the database.
- **php-xml**: Handles XML parsing and writing.
- **php-json**: Allows WordPress to handle JSON data (used heavily in REST APIs).
- **php-mbstring**: Helps in handling multi-byte strings (essential for supporting various languages).
- **php-intl**: Adds support for internationalization features.
- **php-soap**: Adds SOAP protocol support.
- **php-zip**: Required for managing ZIP files (used for plugin/theme uploads and updates).


### Step 5: Configure Apache to Use PHP-FPM

PHP-FPM (FastCGI Process Manager) is recommended for handling PHP requests with Apache. It improves the performance of handling PHP scripts and is particularly useful in high-traffic scenarios.

1. Install **PHP-FPM** and the **mod_proxy_fcgi** Apache module:
   ```bash
   sudo yum install php-fpm mod_proxy_fcgi
   ```

2. Configure Apache to pass PHP requests to **PHP-FPM**:
   ```bash
   sudo nano /etc/httpd/conf.d/php-fpm.conf
   ```

   Add the following configuration:
   ```bash
   <FilesMatch \.php$>
       SetHandler "proxy:unix:/run/php-fpm/www.sock|fcgi://localhost/"
   </FilesMatch>
   ```

3. Start and enable **PHP-FPM**:
   ```bash
   sudo systemctl start php-fpm
   sudo systemctl enable php-fpm
   ```

4. Restart Apache to apply the changes:
   ```bash
   sudo systemctl restart httpd
   ```


### Step 6: Configure SELinux (Security-Enhanced Linux)

#### Why SELinux?

**SELinux** is a security module that enforces strict access control policies on your system, especially important for enterprise environments like Red Hat. It helps limit the damage that could be caused by compromised services, including the web server and PHP. By default, **SELinux** is set to **enforcing** mode on **RHEL**. This mode restricts many actions that Apache and PHP-FPM might need to function correctly.

#### Check SELinux Status

Verify that SELinux is enabled and in **enforcing** mode:
```bash
sestatus
```

Expected output:
```
SELinux status:                 enabled
Current mode:                   enforcing
```

#### Configure SELinux for PHP and Apache

To allow **Apache** and **PHP-FPM** to run without issues, you need to allow specific actions that would otherwise be restricted by SELinux.

1. Allow **Apache** to execute memory operations (needed by PHP’s OpCache):
   ```bash
   sudo setsebool -P httpd_execmem 1
   ```

2. Allow **Apache** to make network connections (required for external HTTP requests, for example, for connecting to APIs or downloading plugins/themes):
   ```bash
   sudo setsebool -P httpd_can_network_connect 1
   ```


### Step 7: Verify PHP Installation

After installation, check that PHP 8.3 is correctly installed and running.

1. Check the **PHP version**:
   ```bash
   php --version
   ```

   You should see output similar to:
   ```
   PHP 8.3.12 (cli) (built: Sep 26 2024 02:19:56) ( NTS )
   ```

2. List the installed **PHP modules**:
   ```bash
   php -m
   ```

   Ensure all necessary extensions for WordPress (like `curl`, `gd`, `mbstring`, etc.) are listed.


### Step 8: Test PHP Functionality

Create a **PHP info page** to verify that PHP is correctly served through Apache:

1. Create a test PHP file:
   ```bash
   sudo nano /var/www/html/info.php
   ```

2. Add the following code:
   ```php
   <?php
   phpinfo();
   ?>
   ```

3. Access this file via your web browser:
   ```bash
   http://your-server-ip/info.php
   ```

   If everything is configured correctly, a page displaying detailed PHP information should appear.


## Wordpress Installation

1. Install the wget package
   ```bash
   sudo yum install wget
   ```

2. Download the latest version of **WordPress**:
   ```bash
   wget https://wordpress.org/latest.tar.gz
   ```

3. Extract the WordPress archive:
   ```bash
   tar -xzvf latest.tar.gz
   ```

4. Move WordPress folder to your **Apache web root**:
   ```bash
   sudo mv wordpress/ /var/www/html/
   ```

5. Set the correct permissions for the **Apache** user:
   ```bash
   sudo chown -R apache:apache /var/www/html/wordpress
   sudo chmod -R 755 /var/www/html/wordpress
   ```

6. Restart Apache to apply the changes:
   ```bash
   sudo systemctl restart httpd
   ```
7. accessing `http://instance-public-ip/wordpress` from your browser to see the wordpress installation
 