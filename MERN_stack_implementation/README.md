# Implementing MERN Stack on AWS EC2 with Self-Hosted MongoDB: My Learning Journey

This README documents my personal experience setting up a MERN (MongoDB, Express, React, Node.js) stack application on an AWS EC2 instance for production, using a self-hosted MongoDB database. I'll share insights, challenges, and lessons learned throughout the process.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Installing MongoDB](#installing-mongodb)
3. [Configuring MongoDB](#configuring-mongodb)
4. [Installing Node.js](#installing-nodejs)
5. [Setting Up PM2 Process Manager](#setting-up-pm2-process-manager)
6. [Installing and Configuring Nginx](#installing-and-configuring-nginx)
7. [Deploying the MERN Application](#deploying-the-mern-application)
8. [Final Steps and Reflections](#final-steps-and-reflections)

## Prerequisites

Before starting this journey, I ensured I had:
- An AWS EC2 instance running Ubuntu
- Basic knowledge of Linux commands
- A MERN stack application ready for deployment

>Personal Note: Setting up the EC2 instance was straightforward, but I recommend thoroughly understanding AWS security groups and network settings before proceeding. see my [previous video](https://youtu.be/5K0mHssuN-I) on how to set it up

>Ensure the ports 5000 and 27017 are open

## Installing MongoDB

Installing MongoDB was more involved than I initially thought. Here's what I did:

1. Imported the public key for the MongoDB package:

```bash
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg \
   --dearmor
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

>Personal Note: The installation process was smooth, but I found the [official MongoDB documentation](https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-ubuntu/#std-label-install-mdb-community-ubuntu) incredibly helpful. I'd recommend keeping it open in a separate tab for reference.

## Configuring MongoDB

Configuring MongoDB for production use was crucial. Here's what I learned:

1. Configuring for remote access (optional):

```bash
sudo nano /etc/mongod.conf
```

Changed the `bindIp` setting:

```yaml
net:
  bindIp: 0.0.0.0
```

>Insight: While this allows remote connections, it's essential to understand the security implications. In a real production environment, I'd probably limit this to specific IP addresses. For me, I left the default configuration as there was no intention ti communnicate with the mongodb remotely

2. Setting up a MongoDB user:

```bash
mongosh
```

```mongosh
use sample_todo_app
db.createUser({
  user: "sampleTodoUser",
  pwd: "Password.1",
  roles: [{ role: "readWrite", db: "sample_todo_app" }]
})
```

>Personal Note: Creating a specific user for the application, rather than using the root user, is a best practice I'm glad I followed. Note you can set whatever password for your application, Password.1 is for demonstration purpose

3. Enabled authentication:

```bash
sudo nano /etc/mongod.conf
```

Added:

```yaml
security:
  authorization: enabled
```

>Insight: This step is crucial for security. Without it, anyone could potentially access your database.

The resulting connection string:

```
mongodb://sampleTodoUser:Password.1@localhost:27017/sample_todo_app?authSource=sample_todo_app
```

![mongodb shell](images/mongodb-shell.png)

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

PM2 was new to me, but I quickly realized its importance:

```bash
sudo npm install pm2 -g
```

Insight: PM2's ability to keep the application running after system reboots is invaluable for maintaining uptime.

## Installing and Configuring Nginx

Setting up Nginx as a reverse proxy was an enlightening experience:

1. Installed Nginx:

```bash
sudo apt update
sudo apt install nginx -y
```

2. Created a custom Nginx configuration:

```bash
sudo nano /etc/nginx/sites-available/sample_todo_app
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

>Personal Note: Understanding how Nginx works as a reverse proxy was a significant learning point for me. It's not just about routing traffic, but also about potential performance improvements and additional security.

3. Activated the new configuration and deactivated the default:

```bash
sudo ln -s /etc/nginx/sites-available/sample_todo_app /etc/nginx/sites-enabled/
sudo nginx -t
sudo unlink /etc/nginx/sites-enabled/default
sudo systemctl restart nginx
```

>Insight: The symlink approach for managing Nginx configurations is elegant and makes it easy to enable/disable sites.

## Deploying the MERN Application

Finally, the moment of truth - deploying the actual application:

1. Cloned the repository:

```bash
git clone https://github.com/fmanimashaun/sample-todo-app.git
```

2. Set up the frontend:

```bash
cd sample-todo-app/client
sudo npm i && sudo npm run build
```

>Personal Note: Deploying MERN to prooduction without using any external services was quite interesting

3. Set up the backend:

```bash
cd ../backend
sudo npm i
```

4. Created the `.env` file inside the backend folder:

```bash
sudo nano .env
```

Content:

```
PORT=5001
DB=mongodb://sampleTodoUser:Password.1@localhost:27017/sample_todo_app?authSource=sample_todo_app
NODE_ENV=production
```

>Insight: Using environment variables is crucial for maintaining security and flexibility across different deployment environments.

5. Started the application with PM2:

```bash
pm2 start index.js --name mern-app
```

6. To keep the app running, and ensure automatically restarting in case of crash or system reboot

```bash
pm2 startup

sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u <linux user> --hp /home/<linux user>

pm2 save
```

>In my case, `ubuntu` is the `linux user` for the ec2 instance.


![pm2 status](images/terminal-log.png)

>Personal Note: Make sure you are insid the backend folder before running the command above

## Final Steps and Reflections

1. Checked the process status:

```bash
pm2 logs mern-app
```

2. Accessed the application at `http://<aws ec2 instance public ip>`.
![Final output](images/final-output.png)

3. Testing the backend api with postman
![Backend API testing for post](images/backend-api-post.png)
![Backend API testing for get](images/backend-api-get.png)
![Backend API testing for delete](images/backend-api-delete.png)

Personal Reflections:
- This process taught me a lot about full-stack deployment in a cloud environment.
- The importance of security at every step cannot be overstated.
- I now appreciate the complexity of setting up a production environment and the tools that help manage it.
- In the future, I'd like to explore adding HTTPS, implementing proper logging, and setting up automated backups.

Challenges I Faced:
- Initially, I ran into issues with MongoDB authentication. Careful reading of the documentation helped resolve this.
- Understanding the interaction between Nginx and my Node.js application took some time to grasp fully.
- Remembering to use `sudo` for certain commands and not for others (like when running PM2) was a frequent stumbling point.

Key Takeaways:
- Always refer to official documentation.
- The order of operations matters, especially when setting up interdependent services.
- Patience is crucial - not everything works on the first try, and that's okay!
- Understanding each component's role in the stack is as important as knowing how to set them up.

This journey has been incredibly educational, and I'm excited to continue learning and improving my deployment skills!