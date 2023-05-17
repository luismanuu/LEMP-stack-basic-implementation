# LEMP-stack-basic-implementation
## Pre-requisites

### Setting Up AWS EC2 Instance with Ubuntu Server

To complete this project, you will need an AWS account and a virtual server with Ubuntu Server OS.

### AWS Account Setup

1. Register a new AWS account.
2. Launch a new EC2 instance in your preferred region using the t2.micro family and Ubuntu Server 20.04 LTS (HVM).

**IMPORTANT:** Save your private key (.pem file) securely and **do not share it**.

### Connecting to the EC2 Instance

#### MAC/Linux Users

1. Open Terminal.
2. Use the same key downloaded from AWS; no need to convert it.
3. Change directory to the location of your PEM file (likely in the Downloads folder):
```cd ~/Downloads```
4. Change permissions for the private key file (.pem):
```sudo chmod 0400 <private-key-name>.pem```
5. Connect to the instance:
```ssh -i <private-key-name>.pem ubuntu@<Public-IP-address>```

#### AWS Free Tier Limits

- You can use 750 hours (31.25 days) of t2.micro server per month for the first 12 months **FOR FREE**.
- Stop your EC2 instance when not in use.
- By default, there is a soft limit of maximum 5 running instances at the same time.
- Note: Every time you stop and start your EC2 instance, you will have a new IP address. Update your SSH credentials accordingly.

## STEP 1 - INSTALLING THE NGINX WEB SERVER

Install Nginx using apt package manager:

```bash
sudo apt update
sudo apt install nginx
```

Check the status of the Nginx service:

```bash
sudo systemctl status nginx
```

Open TCP port 80 to allow incoming traffic:

```bash
# Add a rule to EC2 configuration
# to open inbound connection through port 80
```

Verify the installation locally:

```bash
# Access server via DNS name
curl http://localhost:80

# Access server via IP address
curl http://127.0.0.1:80
```

Access the server from the internet:

```bash
# Access using Public IP address
curl -s http://<Public-IP-Address>:80
```

If the server responds, it is correctly installed and accessible.
Also can check it on browser
![projje2-step1](https://user-images.githubusercontent.com/85270361/210116186-f5ec30cf-5fe3-410d-9c21-2f26b03c4815.PNG)

## STEP 2 - INSTALLING MYSQL

Install MySQL using apt package manager:

```bash
sudo apt install mysql-server
```

Log in to the MySQL console:

```bash
sudo mysql
```

Run a security script and set the root user password:

```bash
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';
exit
```

Run the interactive script to secure the installation:

```bash
sudo mysql_secure_installation
```

Answer the prompts to configure password validation and set a root password.

Test if you can log in to the MySQL console:

```bash
sudo mysql -p
```

To exit the MySQL console:

```bash
exit
```

For increased security, create dedicated user accounts for each database.

Note: PHP applications on MySQL 8 require mysql_native_password as the authentication method.

MySQL is now installed and secured. Next, we will install PHP.

## STEP 3 - INSTALLING PHP

Install PHP-FPM and PHP-MySQL packages:

```bash
sudo apt install php-fpm php-mysql
```

Confirm the installation by typing Y and pressing ENTER.

PHP components are now installed. Next, you will configure Nginx to use them.

## STEP 4 - CONFIGURING NGINX TO USE PHP PROCESSOR

Create a new server block configuration file for your domain:

```bash
sudo nano /etc/nginx/sites-available/projectLEMP
```

Paste the following configuration:

```nginx
server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;
    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

Save and close the file. Then, activate the configuration:

```bash
sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
```

Test the configuration for syntax errors:

```bash
sudo nginx -t
```

If no errors are reported, reload Nginx:

```bash
sudo systemctl reload nginx
```

Verify that your website is working by opening it in a browser using your server's IP address or public DNS name.

Your Nginx server is now configured to use PHP processing.

## STEP 5 - TESTING PHP WITH NGINX

Create a test PHP file named info.php in your document root:

```bash
sudo nano /var/www/projectLEMP/info.php
```

Add the following PHP code to the file:

```php
<?php
phpinfo();
```

Save and close the file. 

Access the page in your web browser using your server's domain name or public IP address, followed by /info.php:

```
http://server_domain_or_IP/info.php
```

You will see a page with detailed information about your PHP server.

After reviewing the information, remove the file for security reasons:

```bash
sudo rm /var/www/projectLEMP/info.php
```

Nginx can handle .php files by successfully executing the PHP code.

## STEP 6 - RETRIEVING DATA FROM MYSQL DATABASE WITH PHP (CONTINUED)

Create a test database and configure access:

1. Connect to the MySQL console using the root account:

   ```bash
   sudo mysql
   ```

2. Create a new database:

   ```mysql
   CREATE DATABASE `example_database`;
   ```

3. Create a new user with full privileges on the database:

   ```mysql
   CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
   GRANT ALL ON example_database.* TO 'example_user'@'%';
   ```

4. Exit the MySQL shell:

   ```mysql
   exit
   ```

5. Test the new user's permissions:

   ```bash
   mysql -u example_user -p
   SHOW DATABASES;
   ```

Create a test table and insert sample data:

```mysql
CREATE TABLE example_database.todo_list (
    item_id INT AUTO_INCREMENT,
    content VARCHAR(255),
    PRIMARY KEY(item_id)
);

INSERT INTO example_database.todo_list (content) VALUES ("My first important item");
```

Create a PHP script to retrieve and display the data:

1. Create a new PHP file:

   ```bash
   nano /var/www/projectLEMP/todo_list.php
   ```

2. Add the following PHP code to the file:

   ```php
   <?php
   $user = "example_user";
   $password = "password";
   $database = "example_database";
   $table = "todo_list";

   try {
     $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
     echo "<h2>TODO</h2><ol>";
     foreach($db->query("SELECT content FROM $table") as $row) {
       echo "<li>" . $row['content'] . "</li>";
     }
     echo "</ol>";
   } catch (PDOException $e) {
       print "Error!: " . $e->getMessage() . "<br/>";
       die();
   }
   ```

3. Save and close the file.

Access the page in your web browser using your server's domain name or public IP address, followed by `/todo_list.php`:

```
http://server_domain_or_IP/todo_list.php
```

You should see a page displaying the content from your test table.









