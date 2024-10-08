# Project: DevOps Tooling Website Solution

## Table of Contents

1. [Introduction](#introduction)
   - [Project Overview](#project-overview)
   - [Self-Study: Storage Concepts](#self-study-storage-concepts)
2. [Prerequisites](#prerequisites)
3. [Architecture Overview](#architecture-overview)
4. [Setting Up AWS EC2 Instances](#setting-up-aws-ec2-instances)
5. [Configuring NFS Server](#configuring-nfs-server)
   - [EBS Setup](#ebs-setup)
   - [LVM Configuration](#lvm-configuration)
6. [Configuring Database Server](#configuring-database-server)
   - [EBS Setup](#ebs-setup-1)
   - [LVM Configuration](#lvm-configuration-1)
7. [AWS Security Group Configuration](#aws-security-group-configuration)
8. [Install and Configure MySQL Database Server](#install-and-configure-mysql-database-server)
9. [Configuring Web Servers](#configuring-web-servers)
   - [Installing NFS Client](#installing-nfs-client)
   - [Mounting NFS Shares](#mounting-nfs-shares)
   - [Installing Apache and PHP](#installing-apache-and-php)
10. [Deploying Tooling Website](#deploying-tooling-website)
11. [Final Steps and Reflections](#final-steps-and-reflections)

## Introduction

### Project Overview

This project is an extension of my [previous work](https://github.com/fmanimashaun/Steghub-DevOps-training/blob/main/Web_solution_with_wordpress), where I deployed a scalable WordPress application using a 3-tier architecture on AWS with EBS volumes and LVM for dynamic storage management. In this iteration,I'm enhancing the scalability and laying the foundation for a more robust DevOps tooling solution by implementing the following key changes:

1. **Centralized File Storage**: I'm introducing a Network File System (NFS) server to provide centralized, shared storage. This allows me to run multiple web servers efficiently, improving our application's scalability and performance.

2. **Multiple Web Servers**: Instead of a single web server, I'm now deploying multiple web servers that will connect to the centralized NFS storage. This setup enhances the ability to handle increased traffic and provides better fault tolerance.

3. **Dedicated Database Server**: We maintain a separate database server, continuing the practice of separating concerns in the architecture.

4. **DevOps Tooling Focus**: While the previous project centered on WordPress, this solution is geared towards hosting DevOps tools. This shift in focus prepares the infrastructure for hosting various DevOps-related applications and utilities.

5. **Enhanced Scalability**: By separating file storage from the web servers and using NFS, I'm creating a more flexible system that can easily scale by adding more web servers as needed.

6. **Continued Use of LVM**: I'm still leveraging Logical Volume Manager (LVM) for dynamic storage management, applying this to both the NFS server and database server for flexible disk space allocation.

The goal of this project is to create a scalable and maintainable infrastructure that can serve as a solid foundation for various DevOps tools and practices. By building on the [previous work](https://github.com/fmanimashaun/Steghub-DevOps-training/blob/main/Web_solution_with_wordpress) and introducing new elements like NFS, I'm moving towards a more robust, enterprise-ready solution that can adapt to growing demands and complexities in a DevOps environment.

### Self-Study: Storage Concepts

Below afre the concepts learnt during my self-study to have a general idea of how the varous technologies relates and theur use cases:

1. **Network Storage Types**:
   - **NAS (Network Attached Storage)**: File-level storage for easy file sharing in smaller environments, using protocols like NFS or SMB/CIFS.
   - **SAN (Storage Area Network)**: Block-level storage on a dedicated network, ideal for high-performance applications, using protocols like Fibre Channel or iSCSI.

2. **Data Storage Paradigms**:
   - **Block Storage**: Fixed-size data blocks with unique addresses, suitable for OS and databases, offering low-latency I/O (e.g., Amazon EBS).
   - **Object Storage**: Data stored as objects with metadata, highly scalable for unstructured data like media files (e.g., Amazon S3).
   - **Network File System (NFS)**: File-level protocol for remote file access, allowing shared access across multiple clients.

3. **AWS Cloud Storage Services**:
   - **Amazon EBS**: Block-level storage volumes for EC2 instances.
   - **Amazon S3**: Scalable object storage for various data types.
   - **Amazon EFS**: Managed NFS for EC2, providing scalable file storage.

These storage concepts form the foundation for understanding the infrastructure choices in our DevOps tooling project, particularly the use of NFS for shared storage across multiple web servers.

## Prerequisites

Before beginning this project, ensure you have the following:

1. **AWS Account**: An active AWS account with permissions to create and manage EC2 instances, EBS volumes, and security groups.


3. **Knowledge Base**:
   - Basic understanding of Linux command line
   - Familiarity with AWS services, particularly EC2 and EBS
   - Basic networking concepts

4. **Tools**:
   - SSH client (e.g., PuTTY for Windows or Terminal for MacOS/Linux)
   - Git installed on your local machine

5. **Security**:
   - Create or use an existing EC2 Key Pair for SSH access

6. **Browser**: A modern web browser for accessing the AWS Management Console

## Architecture Overview

Below is a diagram illustrating the architecture of our DevOps Tooling Website Solution:

<antArtifact identifier="devops-tooling-architecture" type="application/vnd.ant.mermaid" title="DevOps Tooling Website Solution Architecture">

```mermaid
graph TD
  Client((Client))
        
        Client <--> |Client traffic| WS1
        Client <--> |Client traffic| WS2
        Client <--> |Client traffic| WS3

    subgraph AWS Cloud
        subgraph NFS Server
            NFS[NFS Server t2.small]
            EBS1[EBS 15GB]
            EBS2[EBS 15GB]
            EBS3[EBS 15GB]
            NFS --- EBS1
            NFS --- EBS2
            NFS --- EBS3
        end
        
        subgraph Web Servers
            WS1[Web Server 1 t2.small]
            WS2[Web Server 2 t2.small]
            WS3[Web Server 3 t2.small]
        end
        
        subgraph DB Server
            DB[DB Server t2.micro]
            DBEBS1[EBS 10GB]
            DBEBS2[EBS 10GB]
            DBEBS3[EBS 10GB]
            DB --- DBEBS1
            DB --- DBEBS2
            DB --- DBEBS3
        end
        
        WS1 <--> |NFS Traffic| NFS
        WS2 <--> |NFS Traffic| NFS
        WS3 <--> |NFS Traffic| NFS
        
        WS1 <--> |DB Traffic| DB
        WS2 <--> |DB Traffic| DB
        WS3 <--> |DB Traffic| DB
    end
    
    style NFS fill:#f9f,stroke:#333,stroke-width:4px
    style WS1 fill:#bbf,stroke:#333,stroke-width:2px
    style WS2 fill:#bbf,stroke:#333,stroke-width:2px
    style WS3 fill:#bbf,stroke:#333,stroke-width:2px
    style DB fill:#bfb,stroke:#333,stroke-width:4px
```

This architecture provides:
- Scalability through multiple web servers
- Centralized file storage via NFS
- Separated database for better resource management

## Setting Up AWS EC2 Instances

Follow these steps to set up the required EC2 instances:

1. **Launch NFS Server**:
   - Navigate to EC2 dashboard in AWS Console
   - Click "Launch Instance"
   - Choose "Red Hat Enterprise Linux 9.4" AMI
   - Select t2.small instance type
   - Configure instance details (VPC, subnet, etc.)
   - Add 3 x 15GB EBS volumes
   - Configure security group (will be detailed later)
   - Review and launch with your EC2 key pair

2. **Launch Web Servers** (Repeat 3 times):
   - Follow similar steps as NFS server
   - Choose t2.small instance type
   - No additional EBS volumes required

3. **Launch Database Server**:
   - Follow similar steps as NFS server
   - Choose t2.micro instance type
   - Add 3 x 10GB EBS volumes

4. **Verify Instances**:
   - Ensure all instances are in the "running" state
   - Note down the private IP addresses of all instances
   - Confirm that all instances are in the same subnet (for this project)

5. **Tag Instances**:
   - Add descriptive tags to each instance (e.g., "NFS Server", "Web Server 1", "DB Server")
   - Consider adding a tag to indicate that this is a learning/development environment

6. **Configure Elastic IPs** (Optional):
   - Allocate and associate Elastic IPs to web servers if public access is required

> **Important Considerations:**
>- *Ensure all instances are in the same VPC and subnet for this project. Remember that while this simplifies our setup, it's **not a recommended practice for production environments**.*
>- *Use the same key pair for all instances for easier management*
>- *Double-check that the correct number and size of EBS volumes are attached to the NFS and DB servers*

> *For the purpose of this project and to simplify the setup, all instances are placed in the same subnet. However, it's crucial to understand that this configuration is not recommended for production environments due to security concerns. In a production setting, it's best practice to separate these components into different subnets (e.g., public subnet for web servers, private subnet for application servers, and another private subnet for databases) to enhance security through network segregation.*

![AWS instances](images/aws-instances.png)
![AWS EBS volumes](images/aws-ebs-volumes.png)

## Configuring NFS Serve

Connect to the nfs-server instance via the ssh terminal to have access to the system.

### EBS Setup

1. **Check Volume Visibility**:
   After SSH'ing into the instances, I listed the attached block devices using:

   ```bash
   lsblk
   ```
   The new volumes appeared as `/dev/xvdb`, `/dev/xvdc`, and `/dev/xvdd`.

   ![Showing EBS volumes attached](images/dev-blocks.png)

   > note down the name of the EBB volumes as shown on the output of the **lsblk** command

### LVM Configuration

1. **Install LVM Tools**:
   Since Red Hat Enterprise Linux 9.4 was being used, I ensured LVM was installed, I also installed nano (text editor)

   ```bash
   sudo dnf update
   sudo dnf install lvm2 nano
   ```

   you can check for available partition using **lvmdiskscan** command, however, since it is a fresh EBS volumes, there is no partition on it at the moment.

2. **Create partitions on EBS volumes**:

   Using the **gdisk** utitlity to create a single partition on each EBS block as follows:

   ```bash
   sudo gdisk /dev/xvdb
   ```

   This will launch the partition utility interface as shown below:

   ![Partition utility interface](images/partition-interface.png)

   use the following commands to create and save the partition table to disk:
   - **n** - To create a partition table, accept all defaults by pressing **Enter** key
   - **p** - Print partition table informatoon
   - **w** - Write changes to disk

   ![Partitioning outcome](images/partitioning.png)

   You do the same for the remaining EBS blocks using the commands below:
   - xvdc block:
   ```bash
   sudo gdisk /dev/xvdc
   ```

    - xvdd block:
   ```bash
   sudo gdisk /dev/xvdd
   ```

   confirm the partitions using **lsblk**
   ```bash
   lsblk
   ```
![Partition confirmation](images/confirmation%20of%20partitions.png)

> note down the partition names: **xvdb1**, **xvdc1** and **xvdd1**

3. **Create Physical Volumes**:
   Converted the three attached EBS block partitions into physical volumes (PVs):

   ```bash
   sudo pvcreate /dev/xvdb1 /dev/xvdc1 /dev/xvdd1
   ```

4. **Create Volume Group**:
   Next, Created a volume group (VG) to aggregate the PVs:

   ```bash
   sudo vgcreate webdata-vg /dev/xvdb1 /dev/xvdc1 /dev/xvdd1
   ```

5. **Create Logical Volumes**:
   Created logical volumes (LVs) for storing application data, logs and opt for jenkins. Here's how I did it:

   ```bash
   sudo lvcreate -n lv-apps -L 14G  webdata-vg
   sudo lvcreate -n lv-logs -L 14G  webdata-vg
   sudo lvcreate -n lv-opt -L 14G  webdata-vg
   lsblk
   ```

   ![Output of the LV creation](images/Logical-volumes.png)

   To verify the entire setup - view the VG, PV and LV, you can run:
   ```bash
   sudo vgdisplay -v
   ```

6. **Create File system and Mount points for the LVs**:
   Create o a file system on the Logival volumes by running the follwoing commands:

   ```bash
   sudo mkfs -t xfs /dev/webdata-vg/lv-apps
   sudo mkfs -t xfs /dev/webdata-vg/lv-logs
   sudo mkfs -t xfs /dev/webdata-vg/lv-opt
   ```

   ![File system creation](images/File-system-creation.png)

   - Create the following mount points:

   ```bash
   sudo mkdir /mnt/apps /mnt/logs /mnt/opt
   ```

   - Install the NFS server:
   ```bash
   sudo dnf update -y
   sudo dnf install nfs-utils -y
   sudo systemctl start nfs-server
   sudo systemctl enable nfs-server
   sudo systemctl status nfs-server
   ```

   - Set permission to the mount points:
   ```bash
   sudo chown -R nobody: /mnt/apps
   sudo chown -R nobody: /mnt/logs
   sudo chown -R nobody: /mnt/opt

   sudo chmod -R 777 /mnt/apps
   sudo chmod -R 777 /mnt/logs
   sudo chmod -R 777 /mnt/opt

   sudo systemctl restart nfs-server
   ```

   ![NFS server subnet CIDR](images/nfs-subnet-cidr.png)

   - Configure access to NFS for clients within same subnet (172.31.0.0/20)
   ```bash
   sudo nano /etc/exports
   ```

      paste the code below into the exports file:
      ```yml
      # exporting the nfs mount points: 
      /mnt/apps 172.31.0.0/20(rw,sync,no_all_squash,no_root_squash)
      /mnt/logs 172.31.0.0/20(rw,sync,no_all_squash,no_root_squash)
      /mnt/opt 172.31.0.0/20(rw,sync,no_all_squash,no_root_squash)
      ```
   ```bash
   sudo exportfs -arv
   ```
   ![NFS server export mount points](images/exports-mount-points.png)


   - Check the NFS server port:
   ```bash
   rpcinfo -p | grep nfs
   ```
   Add the following ports to the security group for the nfs-server on aws console:
      - TCP 111
      - TCP 2049
      - UDP 111
      - UDP 2049
   ![NFS server ports](images/nfs-ports.png)
   ![nfs-server security group](images/nfs-sg-config.png)