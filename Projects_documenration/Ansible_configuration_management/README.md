# Ansible Configuration Management

## Table of Contents
1. [Introduction](#introduction)
2. [Project Overview](#project-overview)
3. [Architecture](#architecture)
4. [Prerequisites](#prerequisites)
5. [AWS setup overview](#aws-setup-overview)
6. [Self-Study](#self-study)
	 - [SSL/TLS and Certificate Authorities (CAs)](#ssltls-and-certificate-authorities-cas)
	 - [Domains, DNS, and Common DNS Records](#domains-dns-and-common-dns-records)
	 - [Key Concepts for Nginx Load Balancing](#key-concepts-for-nginx-load-balancing)
7. [Implementation](#implementation)
8. [Testing and Validation](#testing-and-validation)
9. [Future Improvements](#future-improvements)
10. [References](#references)

## Introduction

This project extends the [DevOps Tooling Website Deployment with CI/CD](../DevOps_tooling_website_solution/README.md) - Jenkins workflow by incorporating Ansible to automate configuration management and application deployment. The Jenkins server is leveraged as a bastion host, allowing seamless execution of Ansible playbooks across target servers.

## Project Overview
This project builds upon the foundation laid by previous projects in the DevOps Tooling series, expanding their scope and introducing additional automation and configuration management capabilities.

1. [DevOps Tooling Website Solution](../DevOps_tooling_website_solution/README.md): The initial project in the series, focused on providing a complete DevOps solution for web application deployment, including the necessary tools and setup for a scalable, automated environment.

2. [DevOps Tooling Website Deployment with CI/CD - Jenkins](../Tooling_website_deployment_automation_with_continuous_integration-jenkins/README.md): This project added continuous integration and continuous deployment (CI/CD) capabilities to the website solution. It implemented Jenkins as the automation server, streamlining the deployment pipeline and enabling automated testing, building, and deployment of applications.

3. [Load Balancer Solution with Nginx and SSL/TLS](../Load_Balancer_solution_with_nginx_and_ssl_tls/README.md): Building on the previous solutions, this project introduced a load balancing solution using Nginx, ensuring high availability and fault tolerance for the application. Additionally, SSL/TLS encryption was implemented to enhance security and ensure safe communication between users and servers.

The current project further extends these efforts by incorporating Ansible for automated configuration management and application deployment. By leveraging the Jenkins server as a bastion host, Ansible playbooks can be executed seamlessly across target servers, ensuring consistent configurations and streamlined application updates. This integration aims to simplify the infrastructure management process and enhance the automation of the deployment pipeline.

## Architecture

```mermaid
graph TD
	Client((Client))
	GitHub[GitHub]

	Client <--> |HTTPS traffic| LB
	GitHub -.-> |Webhook trigger| Jenkins

	subgraph AWS_Cloud[AWS Cloud]
		subgraph LB_Layer[Load Balancer Layer]
			LB[Load Balancer t2.micro with SSL/TLS]
		end

		LB <--> |Distributed traffic| WS1
		LB <--> |Distributed traffic| WS2
		LB <--> |Distributed traffic| WS3

		subgraph Web_Layer[Web Server Layer]
			WS1[Web Server 1 t2.small]
			WS2[Web Server 2 t2.small]
			WS3[Web Server 3 t2.small]
		end

		subgraph NFS_Layer[NFS Server Layer]
			NFS[NFS Server t2.small]
			EBS1[EBS 15GB]
			EBS2[EBS 15GB]
			EBS3[EBS 15GB]
			NFS --- EBS1
			NFS --- EBS2
			NFS --- EBS3
		end

		subgraph DB_Layer[Database Layer]
			DB[DB Server t2.micro]
			DBEBS1[EBS 10GB]
			DBEBS2[EBS 10GB]
			DBEBS3[EBS 10GB]
			DB --- DBEBS1
			DB --- DBEBS2
			DB --- DBEBS3
		end

		subgraph CICD_Layer[CI/CD & Bastion Layer]
			Jenkins[Jenkins & ansible Server t2.micro]
			JenkinsEBS[EBS 8GB]
			Jenkins --- JenkinsEBS
		end

		WS1 <-.-> |NFS Traffic| NFS
		WS2 <-.-> |NFS Traffic| NFS
		WS3 <-.-> |NFS Traffic| NFS

		WS1 <-.-> |DB Traffic| DB
		WS2 <-.-> |DB Traffic| DB
		WS3 <-.-> |DB Traffic| DB

		Jenkins -.-> |Transfer built files| NFS
	end

	Jenkins <-.-> |Pull source code| GitHub

	style NFS fill:#f9f,stroke:#000,stroke-width:4px
	style WS1 fill:#bbf,stroke:#000,stroke-width:2px
	style WS2 fill:#bbf,stroke:#000,stroke-width:2px
	style WS3 fill:#bbf,stroke:#000,stroke-width:2px
	style DB fill:#bfb,stroke:#000,stroke-width:4px
	style LB fill:#ffa,stroke:#000,stroke-width:4px
	style Jenkins fill:#ff9900,stroke:#000,stroke-width:4px
	style GitHub fill:none,stroke:#000,stroke-width:2px
	style AWS_Cloud fill:#f0f0f0,stroke:#000,stroke-width:2px
	style LB_Layer fill:#ffe6cc,stroke:#d79b00,stroke-width:2px
	style Web_Layer fill:#dae8fc,stroke:#6c8ebf,stroke-width:2px
	style NFS_Layer fill:#fff2cc,stroke:#d6b656,stroke-width:2px
	style DB_Layer fill:#d5e8d4,stroke:#82b366,stroke-width:2px
	style CICD_Layer fill:#e1d5e7,stroke:#9673a6,stroke-width:2px

	classDef default fill:#fff,stroke:#000,stroke-width:2px;
	classDef plaintext fill:none,stroke:none,color:black;
	class Client,LB,NFS,WS1,WS2,WS3,DB,EBS1,EBS2,EBS3,DBEBS1,DBEBS2,DBEBS3,Jenkins,JenkinsEBS default;
	class AWS_Cloud,LB_Layer,Web_Layer,NFS_Layer,DB_Layer,CICD_Layer plaintext;
```

This architecture includes a client, GitHub for source control, Jenkins for CI/CD, and an AWS environment with the following components:
- A **Load Balancer Layer** using Nginx with SSL/TLS.
- **Web Server Layer** with three web servers handling application traffic.
- **NFS Server Layer** for shared storage across web servers.
- **Database Layer** to support data needs.
- **CI/CD & Ansible Layer** for automated builds and deployments using Jenkins and automated configuration management using Ansible.

## Prerequisites
Before starting, ensure you have:
1. Completed the [DevOps Tooling Website Solution](../DevOps_tooling_website_solution/README.md) setup, including:
	 - NFS Server
	 - Database Server
	 - Three Web Servers
2. Implemented the load balancer as described in the [Load Balancer Solution with Nginx](../Load_Balancer_solution_with_nginx_and_ssl_tls/README.md).
3. Set up CI/CD with Jenkins, following the [Jenkins Deployment Guide](../Tooling_website_deployment_automation_with_continuous_integration-jenkins/README.md).
4. A basic understanding of Linux and command-line interfaces.
5. Familiarity with Jenkins.
6. Access to an AWS account.
7. A GitHub account and repository for the project.

## AWS setup overview
1. EC2 Instances Overview

	![EC2 used](images/ec2-list.png)

2. Security Groups Overview

	![Security group list](images/sg-list.png)

3. Jenkins server Security Group Setup Overview

	![Security group Jenkins](images/jenkins-sg.png)


## Self-Study

### What is a Jump Server or Bastion Host?

A **Jump Server** (also called a **Bastion Host**) is a server that serves as an intermediary between a secure internal network and an external network. Its primary purpose is to provide secure access to servers in the internal network by acting as a controlled entry point. It is typically used to manage access to systems that are otherwise not directly accessible from outside the network.

**Importance**:
- **Security**: A bastion host provides an added layer of security by ensuring that only authenticated users can access the internal network.
- **Access Control**: It acts as a gatekeeper, controlling which users can access which servers and ensuring that the traffic is logged for auditing purposes.
- **Single Point of Entry**: Instead of allowing direct SSH access to each internal server, a bastion host consolidates access to a single server, simplifying the security management process.

### What is Ansible and Its Role as a Configuration Management Tool?

**Ansible** is an open-source automation tool used to automate configuration management, application deployment, and task orchestration. It is known for being simple to use, agentless (doesn’t require special software to be installed on managed nodes), and powerful in managing IT infrastructure.

**Key Features of Ansible**:
- **Configuration Management**: Allows you to define the desired state of your infrastructure (such as server configuration, applications installed, etc.) and automatically enforce that state.
- **Playbooks**: Ansible’s configuration files, written in YAML format, that define the tasks to be executed on target systems.
- **Idempotency**: Ansible ensures that running a playbook multiple times will not result in unintended changes or errors if the system is already in the desired state.

**Use Case in This Project**:
- In this project, Ansible is used to automate the configuration management and application deployment process, ensuring that servers are consistently configured and deployed without manual intervention.

### SSH-Agent and Remote SSH in Visual Studio Code

**SSH-Agent**:
An **SSH agent** is a program that holds your private SSH keys and manages authentication requests. It allows you to securely authenticate with remote systems without needing to repeatedly enter your SSH passphrase. The agent can be used to manage keys and handle the authentication for SSH connections.

**Using SSH-Agent**:
- After you load your private key into the SSH agent using the `ssh-add` command, you can make SSH connections to remote systems securely, without needing to provide your passphrase each time.
- The SSH agent improves workflow efficiency and security by managing key-based authentication.

**Remote SSH in Visual Studio Code**:
VS Code’s **Remote - SSH** extension allows developers to seamlessly work on remote machines over SSH directly from their local VS Code environment. It provides the following benefits:
- **Remote Development**: You can develop and debug code on a remote server, while using your local machine for editing and interacting with files.
- **Ease of Setup**: By using VS Code's remote capabilities, developers can quickly and easily connect to remote servers via SSH.
- **SSH Key Management**: When combined with an SSH agent, Remote - SSH in VS Code ensures that your private keys are securely stored and used for authenticating SSH connections, streamlining the connection process to remote environments.

**Use Case in This Project**:
- In this project, **Remote SSH in VS Code** is used to access and manage remote servers, where the SSH agent helps handle the authentication and makes it easy to configure and deploy the application using Ansible and Jenkins. 


## Implementation

### Step 1: Create the Repository

We began by creating a GitHub repository called **ansible-config-mgt**. Inside this repository, we created the following folder structure:

- `inventory/`
  - `dev.yml`
  - `prod.yml`
  - `staging.yml`
  - `uat.yml`
- `playbook/`
  - `common.yml`

### Step 2: Define Inventory Files

We defined the inventory for different environments in the `dev.yml` file, which contains the following groups and configuration:

**dev.yml**:

```yaml
[nfs]
172.31.11.175 ansible_ssh_user=ec2-user

[webs]
172.31.13.144 ansible_ssh_user=ec2-user
172.31.10.244 ansible_ssh_user=ec2-user
172.31.10.220 ansible_ssh_user=ec2-user

[db]
172.31.11.187 ansible_ssh_user=ec2-user

[lb]
172.31.9.89 ansible_ssh_user=ubuntu

[all:vars]
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

- **nfs**: Defines the NFS server.
- **webs**: Defines the web servers.
- **db**: Defines the database server.
- **lb**: Defines the load balancer.
- **all:vars**: Defines common SSH arguments for all hosts.

### Step 3: Create the Playbook

We created a simple Ansible playbook in **common.yml** to update the servers and install the latest version of Wireshark:

**common.yml**:

```yaml
---
- name: update web, nfs, and db servers
  hosts: nfs, webs, db
  become: yes
  tasks:
    - name: ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest

- name: update LB server
  hosts: lb
  become: yes
  tasks:
    - name: Update apt repo
      apt:
        update_cache: yes

    - name: ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest
```

- This playbook updates all servers in the `nfs`, `webs`, and `db` groups, ensuring Wireshark is installed at the latest version.
- The `lb` group uses the `apt` package manager to update the repository cache and install Wireshark.

### Step 4: Configure SSH Agent on Local Machine

1. Open **Visual Studio Code** and use the following commands to initialize the SSH agent on your local machine:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/path_to_the_private_key
ssh-add -l
```

2. After adding the private key, use **Remote SSH** in VS Code to connect to the remote host via SSH.
3. The first time you connect, VS Code will prompt for configuration in the `~/.ssh/config` file. Add the following configuration:

```bash
Host jenkins-ansible
  HostName <the jenkins-ansible dns url>
  User ubuntu
  IdentityFile <path to the private key on the local machine>
  ForwardAgent yes
  AddKeysToAgent yes
```

4. Save and connect. If everything is configured correctly, VS Code will establish an SSH connection to the `jenkins-ansible` server.

### Step 5: Ensure SSH Agent Forwarding Works on Remote Server

Once connected to the remote server, run the following command to check if the remote server has access to the local SSH agent:

```bash
ssh-add -l
```

If you receive an error, run the following command to set the SSH agent socket:

```bash
export SSH_AUTH_SOCK=$(ls -t /tmp/ssh-**/* | head -1)
```

Then, re-run the `ssh-add -l` command to ensure the private key is accessible on the remote server.

### Step 6: Clone the Repository and Run the Playbook

1. On the remote server, clone the **ansible-config-mgt** repository:

```bash
git clone https://github.com/fmanimashaun/ansible-config-mgt.git
cd ansible-config-mgt
```

2. Run the Ansible playbook using the following command:

```bash
ansible-playbook -i inventory/dev.yml playbook/common.yml
```

If everything is set up correctly, Ansible will run the playbook on the target servers as defined in the inventory, installing the latest version of Wireshark on the `nfs`, `webs`, `db`, and `lb` servers.

This process outlines the steps from creating the repository, setting up SSH agent forwarding, and configuring VS Code to running the Ansible playbook for updating servers across different environments.


## Future Improvements

While the current implementation provides a solid foundation for configuration management and automated deployment, there are several potential improvements that could be made to enhance the system's functionality, scalability, and security:

1. **Multi-Environment Configuration**: 
   - Implement more comprehensive environment-specific configurations (e.g., development, staging, production). This would involve separating sensitive configurations, such as database credentials or API keys, into secure vaults or using tools like **Ansible Vault** for encryption.

2. **Enhanced Playbook Automation**: 
   - Expand the playbooks to handle more complex tasks such as automated backups, server provisioning, or monitoring setup (e.g., integrating with tools like Prometheus or Nagios).

3. **Role-based Access Control (RBAC)**: 
   - Introduce role-based access control within Ansible to better manage permissions for various users running the playbooks. This could help control who can execute specific tasks or update particular groups of servers.

4. **Integration with Containerization Tools**: 
   - Integrate Ansible with containerization and orchestration platforms like **Docker** and **Kubernetes** for managing and deploying containerized applications, allowing for more scalable and flexible deployment pipelines.

5. **Improve Error Handling and Reporting**: 
   - Enhance the playbook's error handling capabilities to provide more detailed logs and failure reporting, improving the debugging process when things go wrong.

6. **Automated Testing of Configuration Changes**: 
   - Incorporate testing frameworks like **TestInfra** or **Molecule** into the playbook to automatically validate configuration changes and ensure that the infrastructure is in the desired state after every run.

7. **Cloud Integration**: 
   - Implement automation for cloud infrastructure management (e.g., using **AWS**, **Azure**, or **GCP**) to automate server provisioning and configuration based on cloud resources.

8. **Monitoring and Notifications**: 
   - Set up monitoring to track the health and performance of servers and provide real-time notifications (e.g., using **Slack** or **Email**) for any failed tasks or security vulnerabilities.

By implementing these improvements, the solution could be expanded to handle more complex use cases, improve operational efficiency, and offer better security and reliability for large-scale deployments.


## References

1. **Ansible Documentation**:  
   Ansible documentation provides comprehensive information on using Ansible for automation and configuration management.  
   URL: [https://docs.ansible.com/](https://docs.ansible.com/)

2. **Ansible Vault**:  
   Ansible Vault is a tool to encrypt secrets and sensitive information within playbooks.  
   URL: [https://docs.ansible.com/ansible/latest/user_guide/vault.html](https://docs.ansible.com/ansible/latest/user_guide/vault.html)

3. **SSH-Agent Documentation**:  
   A detailed guide on how to manage SSH keys and use the SSH agent for authentication.  
   URL: [https://www.ssh.com/ssh-agent/](https://www.ssh.com/ssh-agent/)

4. **Remote - SSH Extension for Visual Studio Code**:  
   This extension allows you to connect to remote servers using SSH from Visual Studio Code.  
   URL: [https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh)

5. **GitHub - Ansible Config Management Repository**:  
   The repository containing the implementation and resources for the Ansible configuration management project.  
   URL: [https://github.com/fmanimashaun/ansible-config-mgt](https://github.com/fmanimashaun/ansible-config-mgt)

6. **Ansible Best Practices**:  
   A guide on how to structure Ansible projects for maximum maintainability and scalability.  
   URL: [https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)

7. **Nginx Load Balancer Documentation**:  
   Official Nginx documentation for setting up and configuring load balancers.  
   URL: [https://nginx.org/en/docs/](https://nginx.org/en/docs/)

8. **Wireshark Installation Documentation**:  
   The official guide for installing Wireshark, a network protocol analyzer.  
   URL: [https://www.wireshark.org/docs/](https://www.wireshark.org/docs/)
