# Implementing MEAN Stack on AWS EC2 with Self-Hosted MongoDB: My Learning Journey

This README documents my personal experience setting up a MEAN (MongoDB, Express, Angular, Node.js) stack application on an AWS EC2 instance for production, using a self-hosted MongoDB database. I’ll share insights, challenges, and lessons learned throughout the process.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Installing MongoDB](#installing-mongodb)
3. [Configuring MongoDB](#configuring-mongodb)
4. [Installing Node.js](#installing-nodejs)
5. [Setting Up PM2 Process Manager](#setting-up-pm2-process-manager)
6. [Installing and Configuring Nginx](#installing-and-configuring-nginx)
7. [Deploying the MEAN Application](#deploying-the-mean-application)
8. [Final Steps and Reflections](#final-steps-and-reflections)

## Prerequisites

Before starting this journey, I ensured I had:
- An AWS EC2 instance running Ubuntu
- Basic knowledge of Linux commands
- A MEAN stack application ready for deployment

> **Personal Note:** Setting up the EC2 instance was straightforward, but I recommend thoroughly understanding AWS security groups and network settings before proceeding. See my [previous video](https://youtu.be/5K0mHssuN-I) on how to set it up.

> Ensure that ports 5000 and 27017 are open in the security group.

## Installing MongoDB

Installing MongoDB was more involved than I initially thought. Here's what I did:

1. Imported the public key for the MongoDB package:

    ```bash
    curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | \
       sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor
    ```

2. Created a MongoDB source list file:

    ```bash
    echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
    ```

3. Updated the package list and installed MongoDB:

    ```bash
    sudo apt-get update
    sudo apt-get install -y mongodb-org
    ```

4. Started MongoDB and enabled it to start on boot:

    ```bash
    sudo systemctl start mongod
    sudo systemctl enable mongod
    ```

5. Verified the installation:

    ```bash
    sudo systemctl status mongod
    ```

> **Personal Note:** The installation process was smooth, but I found the [official MongoDB documentation](https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-ubuntu/#std-label-install-mdb-community-ubuntu) incredibly helpful. I'd recommend keeping it open in a separate tab for reference.

## Configuring MongoDB

Configuring MongoDB for production use was crucial. Here’s what I learned:

1. **Configuring for remote access (optional):**

    ```bash
    sudo nano /etc/mongod.conf
    ```

    Changed the `bindIp` setting:

    ```yaml
    net:
      bindIp: 0.0.0.0
    ```

> **Insight:** While this allows remote connections, it’s essential to understand the security implications. In a real production environment, I would limit this to specific IP addresses. For my case, I kept the default configuration because there was no intention to communicate with MongoDB remotely.

2. **Secure the the Mongodb:**
    Start MongoDB without authentication enabled:
    ```bash
    mongosh
    ```
    Connect to MongoDB and create the admin user:
   ```mongosh
    use admin
    ```
    ```mongosh
    use admin

    db.createUser(
        {
            user: "root",
            pwd: "Password.1",
            roles: [ { role: "userAdminAnyDatabase", db: "admin" }, "readWriteAnyDatabase" ]
        }
    )

    exit
    ```
    Now, edit the MongoDB configuration file (usually /etc/mongod.conf on Linux systems) to enable authentication:
      ```bash
    sudo nano /etc/mongod.conf
    ```

    Changed the `security` setting:
    ```yaml
    security:
        authorization: enabled
    ```
    > **Insight:** Enabling authentication is crucial for security. Without it, anyone could potentially access your database.

    Restart the MongoDB service to apply the changes.
    ```bash
    sudo systemctl restart mongod
    ```
    Now you can connect to MongoDB with authentication:
    ```bash
    mongosh --authenticationDatabase "admin" -u "root" -p
    ```

    Once authenticated, you can remove test databases:
    ```mongosh
    use test
    db.dropDatabase()
    ```

3. **Setting up a MongoDB user:**

    ```bash
     mongosh --authenticationDatabase "admin" -u "root" -p
    ```

    ```mongosh
    use sample_book_register_app
    db.createUser({
      user: "bookRegisterUser",
      pwd: "Password.1",
      roles: [{ role: "readWrite", db: "sample_book_register_app" }]
    })
    ```

> **Personal Note:** Creating a specific user for the application, rather than using the root user, is a best practice I’m glad I followed. You can set your own password; `Password.1` is used here for demonstration purposes.

The resulting connection string:

```
mongodb://bookRegisterUser:Password.1@localhost:27017/sample_book_register_app?authSource=sample_book_register_app
```

![MongoDB Shell](images/mongodb-shell.png)

## Installing Node.js

Installing Node.js was straightforward:

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo bash -
sudo apt-get install -y nodejs
```

Verified the installation:

```bash
node -v
npm -v
```

## Setting Up PM2 Process Manager

PM2 was new to me, but I quickly realized its importance for production:

```bash
sudo npm install pm2 -g
```

> **Insight:** PM2’s ability to keep the application running after system reboots is invaluable for maintaining uptime.

## Installing and Configuring Nginx

Setting up Nginx as a reverse proxy was an enlightening experience:

1. Installed Nginx:

    ```bash
    sudo apt update
    sudo apt install nginx -y
    ```

2. Created a custom Nginx configuration:

    ```bash
    sudo nano /etc/nginx/sites-available/sample_book_register_app
    ```

Configuration:

```nginx
server {
    listen 80;
    server_name _;
    location / {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'Upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

> **Personal Note:** Understanding how Nginx works as a reverse proxy was a major learning point. It’s not just about routing traffic but also potential performance improvements and additional security.

3. Activated the new configuration and deactivated the default:

    ```bash
    sudo ln -s /etc/nginx/sites-available/sample_book_register_app /etc/nginx/sites-enabled/
    sudo nginx -t
    sudo unlink /etc/nginx/sites-enabled/default
    sudo systemctl restart nginx
    ```

> **Insight:** The symlink approach for managing Nginx configurations is elegant and makes it easy to enable or disable sites.

## Deploying the MEAN Application

Finally, the moment of truth—deploying the application:

1. Install the Angular-cli:

    ```bash
    sudo npm install -g @angular/cli
    ```

2. Cloned the repository:

    ```bash
    git clone https://github.com/fmanimashaun/sample-book-register-app.git
    ```

3. Set up the frontend:

    ```bash
    cd sample-todo-app/client
    sudo npm i && sudo npm run build
    ```

> **Personal Note:** Deploying a MEAN app to production without external services was quite interesting.

4. Set up the backend:

    ```bash
    cd ../backend
    sudo npm i
    ```

5. Created the `.env` file inside the backend folder:

    ```bash
    sudo nano .env
    ```

Content:

```
PORT=5000
DB=mongodb://bookRegisterUser:Password.1@localhost:27017/sample_book_register_app?authSource=sample_book_register_app
NODE_ENV=production
```

> **Insight:** Using environment variables is crucial for maintaining security and flexibility across different deployment environments.

6. Started the application with PM2:

    ```bash
    pm2 start index.js --name mean-app
    ```

7. Kept the app running and ensured it restarted automatically after crashes or system reboots:

    ```bash
    pm2 startup
    sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u <linux user> --hp /home/<linux user>
    pm2 save
    ```

In my case, `ubuntu` is the `linux user` for the EC2 instance.

![PM2 Status](images/terminal-log.png)

> **Personal Note:** Make sure you are inside the backend folder before running the command above.

## Final Steps and Reflections

1. Checked the process status:

    ```bash
    pm2 logs mean-app
    ```

2. Accessed the application at `http://<aws-ec2-instance-public-ip>`. 

    ![Final Output](images/final-output.png)

3. Tested the backend API using Postman:

    ![Backend API Testing](images/backend-api-post.png)
    ![Backend API Get](images/backend-api-get.png)
    ![Backend API Delete](images/backend-api-delete.png)

### Personal Reflections

- This process taught me a lot about full-stack deployment in a cloud environment.
- The importance of security at every step cannot be overstated.
- I now appreciate the complexity of setting up a production environment and the tools that help manage it.
- In the future, I’d like to explore adding HTTPS, implementing proper logging, and setting up automated backups.

### Challenges I Faced

- I initially ran into issues with MongoDB authentication. Careful reading of the documentation helped resolve this.
- Understanding the interaction between Nginx