# Steghub-DevOps-training

This repository documents project-based learning tasks for the Steghub DevOps Engineering training program.

## Projects:
---
### 1. [Web Stack Implementation (LAMP)](/Webstack_implementation_lamp/README.md)

#### Overview:
This project involves deploying a **LAMP stack**—Linux, Apache, MySQL, and PHP—on an **Ubuntu 22** server hosted on **AWS EC2**. This stack enables dynamic web application hosting by leveraging **Apache** as the web server, **MySQL** as the database, and **PHP** for server-side scripting. Apache was selected for its robust community support and straightforward configuration.

The deployment process includes setting up an EC2 instance, installing and configuring each component, and configuring Apache with virtual hosts to allow multiple websites to be hosted on a single server. As a test, a virtual host was set up for `fidara.com.ng` to demonstrate serving dynamic content via PHP. This setup provides an environment ready for scalable applications, making it an excellent foundational project for cloud-based web hosting and management.

#### [Video Demonstration Link](https://youtu.be/5K0mHssuN-I)

---
### 2. [Web Stack Implementation (LEMP)](/Webstack_implementation_lemp/README.md)

#### Overview:
This project involves deploying a **LEMP stack**—Linux, Nginx, MySQL, and PHP—on an **Ubuntu 24.04 LTS** instance hosted on **AWS EC2**. This stack enables dynamic web application hosting by leveraging **Nginx** as the web server, **MySQL** as the database, and **PHP** for scripting. Nginx is chosen for its efficient handling of concurrent requests, making it suitable for high-traffic environments and reducing latency.

The deployment process covers setting up the EC2 instance, installing and configuring each component, and testing the setup with a sample PHP application connected to a MySQL database. This configuration is optimal for applications requiring scalability, secure data handling, and efficient load management, and is well-suited to environments where high performance and dynamic content delivery are essential.

#### [Video Demonstration Link](https://youtu.be/WTpdTachEmY)

---
### 3. [MERN Stack Implementation](/MERN_stack_implementation/README.md)

#### Overview:
This project focuses on deploying a **MERN stack**—MongoDB, Express, React, Node.js—on an **AWS EC2 instance** with a self-hosted MongoDB database. This stack supports full-stack application hosting with a custom-configured backend database on a production server, ensuring flexibility and control over data management.

The deployment process includes setting up MongoDB directly on the EC2 instance, configuring it for secure access, and using **Node.js** for server-side processing. **PM2** is implemented as a process manager to keep the application running smoothly, even after crashes or system reboots. Additionally, **Nginx** is configured as a reverse proxy, directing incoming traffic to the Node.js server for optimized handling.

This configuration offers a stable and scalable environment, ideal for production deployments where security, uptime, and efficient request handling are critical. The self-hosted setup is particularly valuable for those needing full control over their stack in cloud-based environments.

#### [Video Demonstration Link](https://youtu.be/pfCMVZymwdY)

---
### 4. [MEAN Stack Implementation](/MEAN_stack_implementation/README.md)

#### Overview:
This project focuses on deploying a **MEAN stack**—MongoDB, Express, Angular, Node.js—on an **AWS EC2 instance** with a self-hosted MongoDB database. This stack allows for full-stack application hosting with direct backend control, ideal for production environments where data privacy and autonomy are prioritized.

The deployment process includes setting up MongoDB on the EC2 instance and configuring it for secure access. **Node.js** is installed to manage server-side logic, while **PM2** is used to ensure continuous uptime by restarting the application automatically upon failures or system reboots. **Nginx** is configured as a reverse proxy, managing and routing traffic to the Node.js server, enhancing efficiency and security.

This setup creates a stable, scalable environment suitable for production MEAN applications, allowing seamless handling of dynamic requests and offering reliable uptime with simplified process management on AWS.

#### [Video Demonstration Link](https://youtu.be/3Ii6A5701IY)

---
### 5. [Client-Server Architecture with MySQL](/Client-Server_implementation/README.md)

#### Overview:
This project focuses on deploying a **client-server architecture** using **MySQL** on two **AWS EC2 instances**. This architecture allows secure, remote MySQL database access with one instance acting as the MySQL server and the other as the client, highlighting the configuration of AWS security groups and MySQL’s remote access features.

The deployment process includes setting up MySQL on the `mysql-server` instance, securing the root account, and configuring the server to allow connections from the `mysql-client` instance. The setup also involves configuring AWS security groups to permit traffic over port 3306 exclusively between these instances, ensuring controlled access and enhanced security.

This configuration provides a stable, scalable environment for client-server architectures on AWS, offering valuable insights into secure remote database access and network management with AWS infrastructure.

#### [Video Demonstration Link](https://youtu.be/Ttx4QE7_QRI)

---
### 6. [Web Solution with Wordpress](/Web_solution_with_wordpress/README.md)

#### Overview:
This project involves setting up **WordPress** on an **AWS EC2 instance** running **Red Hat Enterprise Linux (RHEL) 9.4** with multiple **EBS volumes** configured using **Logical Volume Manager (LVM)** for efficient storage management. The architecture consists of two EC2 instances:

1. **web-server**: Hosts the WordPress application, configured with Apache, PHP, and LVM to manage application data and logs.
2. **db-server**: Runs the MySQL database, also configured with LVM for scalable storage.

The deployment process includes attaching EBS volumes to both instances, creating and configuring LVM for dynamic volume management, installing and configuring MySQL on the db-server, and setting up Apache and PHP on the web-server for WordPress. AWS security groups were also configured to control access between the web-server and db-server securely.

This setup provides a scalable, production-ready environment, ideal for hosting WordPress with enhanced data management and secure database access on AWS infrastructure.

#### [Video Demonstration Link](https://youtu.be/RxvJB-i2eOw)

---
### 7. [DevOps Tooling Website Solution](/DevOps_tooling_website_solution/README.md)

#### Overview:
The project details the implementation of a **DevOps Tooling Website Solution** on AWS. It extends previous efforts by introducing **scalable multi-server architecture**, **centralized storage using an NFS server**, **dedicated MySQL database server**, and a **web application layer** supported by multiple Apache web servers with shared storage and load-balancing potential.

The setup involves:
1. **NFS Server Setup**: Three EBS volumes configured with LVM, providing shared storage for web servers to ensure uniform application state across instances.
2. **Database Server Configuration**: Dedicated MySQL server with three EBS volumes, also managed by LVM, to provide robust and isolated data storage.
3. **Web Server Configuration**: Three Apache servers with PHP extensions installed and configured to mount the shared NFS storage, supporting a cloned tooling website application.
4. **Application Deployment**: Deploying the tooling website using cloned files from GitHub, with database configurations pointed to the centralized MySQL server.

The infrastructure is managed through EC2 instances with properly assigned security groups for each server type, enforcing access control between the web, NFS, and database layers. **SELinux** configurations were maintained for added security.

#### [Video Demonstration Link](https://youtu.be/FwqMLh0AUJM)

---
### 8. [Load Balancer Solution with Apache](/Load_balancer_solution_with_apache/README.md)

#### Overview:
This project outlines the setup of an **Apache-based load balancer** to enhance the **DevOps Tooling Website Solution** with even traffic distribution across three web servers. By incorporating load balancing, the solution improves scalability and reliability, ensuring high availability and better resource utilization across the infrastructure.

**Key Components**:
1. **Apache Load Balancer**: Configured to manage and distribute incoming traffic to multiple web servers.
2. **Web Servers**: The application layer containing the DevOps Tooling website, served by three web servers sharing storage via NFS.
3. **NFS Server and Database Server**: NFS for centralized file storage and a MySQL database server to support application data, completing the 3-tier architecture.

**Apache Configuration Highlights**:
- **Load Balancing Algorithms**: Configured for `bytraffic` (default: `byrequests`), ensuring efficient load distribution based on current server traffic.
- **Session Persistence (Optional)**: To maintain user sessions across requests.
- **Optional Local DNS Setup**: Mapping server names to internal IPs simplifies the configuration and provides flexibility for potential IP changes.

**Testing & Validation**:
The setup is validated by accessing the application via the load balancer, checking log distribution, and performing failover tests. 

#### [Video Demonstration Link](https://youtu.be/mY70zQ54FMQ)

---
### 9. [DevOps Tooling Website Deployment with CI/CD - Jenkins](/Tooling_website_deployment_automation_with_continuous_integration-jenkins/README.md)

#### Overview:
This project extends the **DevOps Tooling Website Solution** and **Load Balancer Solution with Apache** by integrating a **CI/CD pipeline** using **Jenkins**. This pipeline enables automated builds, testing, and deployment for continuous delivery to our website. Leveraging Jenkins for CI/CD, this setup automates code integration from GitHub, triggers builds, and deploys updated files to the NFS server for centralized access across all web servers, streamlining the deployment process and improving site reliability and efficiency.

**Key Components**:
1. **CI/CD Pipeline with Jenkins**: Automates build and deployment steps for the DevOps tooling website.
2. **Jenkins Webhook**: Configures GitHub to trigger Jenkins builds upon code commits.
3. **NFS Server**: Hosts the updated website files, serving all web servers.
4. **Load Balancer**: Distributes user requests across web servers for high availability.
5. **GitHub Integration**: Enables source control and easy collaboration.

**Testing and Validation**:
The solution ensures continuous deployment by triggering builds from GitHub commits, verifying updated file synchronization across web servers via NFS.

#### [Video Demonstration Link](https://youtu.be/jkOIwwBbG3g)

---
### 10. [Load Balancer Solution with Nginx and SSL/TLS](/Load_Balancer_solution_with_nginx_and_ssl_tls/README.md)

#### Overview:
This project adds an Nginx-based load balancer with SSL/TLS to the DevOps Tooling Website infrastructure, replacing Apache. The setup includes configuring a secure HTTPS connection, associating an Elastic IP, and integrating a custom domain, making the infrastructure robust and secure.

**Key Components**:
1. **Nginx Load Balancer with SSL/TLS**: Ensures secure, encrypted connections and efficient traffic distribution across backend web servers.
2. **Domain and DNS Setup**: Configures DNS records for the custom domain and Elastic IP association, enabling a user-friendly URL.
3. **Certbot for SSL Management**: Uses Certbot to obtain and auto-renew SSL certificates from Let’s Encrypt.
4. **Updated CI/CD Pipeline**: Jenkins automation deploys updates to the website, verified by SSL-protected access via the load balancer.

**Testing and Validation**:
The setup is verified by checking HTTPS accessibility, testing CI/CD functionality, and confirming SSL auto-renewal.

#### [Video Demonstration Link](https://youtu.be/76VbOsroj5g)

---
