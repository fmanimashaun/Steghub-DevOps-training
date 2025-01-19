# Ansible Refactoring & Static Assignments(imports & roles)

## Table of Contents
1. [Introduction](#introduction)
2. [Project Overview](#project-overview)
3. [Architecture](#architecture)
4. [Prerequisites](#prerequisites)
5. [AWS setup overview](#aws-setup-overview)
6. [Self-Study](#self-study)
	 - [Reusing Ansible Artifacts](#reusing-ansible-artifacts)
7. [Implementation](#implementation)
8. [Testing and Validation](#testing-and-validation)
9. [Future Improvements](#future-improvements)
10. [References](#references)

## Introduction

This project builds upon the [Ansible Configuration Management](../Ansible_configuration_management/README.md) - Jenkins workflow by enhancing the automation of configuration management and application deployment. By integrating Ansible and Jenkins, the project leverages Jenkins as a bastion host to execute Ansible playbooks seamlessly across target servers. The focus of the project is to refactor Ansible code for improved maintainability, adopt static assignments, and utilize imports and roles to enhance code reusability.



## Project Overview
The objective of the project is to refactor the existing Ansible codebase for better organization, readability, and extensibility. This involves breaking down monolithic playbooks into smaller, reusable components through the use of imports and roles. The key improvements include:

- **Enhanced Jenkins Workflow**: Introducing a new Jenkins job to streamline artifact storage and reduce space consumption.
- **Code Refactoring**: Splitting large playbooks into modular components for easier maintenance and reuse.
- **Static Assignments**: Organizing playbooks and roles into structured directories for better clarity.
- **Reusable Roles**: Creating dedicated Ansible roles for common tasks like configuring web servers and deploying applications.

By adopting these practices, the project aligns with the DevOps philosophy of iterative improvement, promoting better collaboration, efficiency, and scalability.

## Architecture

The architecture of the project comprises the following key components:

1. **Jenkins-Ansible Server**:
	- Acts as a bastion host.
	- Hosts Jenkins for CI/CD automation.
	- Executes Ansible playbooks on target servers.

2. **Target Servers**:
	- Various environments (e.g., dev, stage, uat, prod).
	- Configured and managed using Ansible.

3. **Ansible Codebase**:
	- Organized into structured directories:
		- static-assignments: Contains reusable playbooks like common.yml.
		- roles: Contains modular roles for specific tasks.
	- Centralized entry point (site.yml) for playbook execution.

4. **Artifact Management**:
	- Artifacts generated during Jenkins builds are stored in a dedicated directory (/home/ubuntu/ansible-config-artifact).
	- Managed using the Copy Artifact plugin in Jenkins.


## Prerequisites
To successfully set up your local development environment for this project, follow the steps below.

**1. Clone the `ansible-config-mgt` Repository**

Start by cloning the `ansible-config-mgt` repository from GitHub:

```bash
git clone https://github.com/fmanimashaun/ansible-config-mgt.git
```

Navigate into the cloned repository:

```bash
cd ansible-config-mgt
```

Open the project in Visual Studio Code:

```bash
code .
```

**2. Set Up the Python Virtual Environment**

Open the terminal inside Visual Studio Code (`Ctrl + \`` or **Terminal > New Terminal**).

Create a Python virtual environment within your project folder by running:

```bash
python -m venv .venv
```

This will create a `.venv` directory in your project folder containing the isolated Python environment.

>NB: Ensure you are using the latest python (v3.13)

**3. Activate the Virtual Environment**

Once the virtual environment is created, activate it by running:

- **On Windows:**

  ```bash
  .venv\Scripts\activate
  ```

- **On macOS/Linux:**

  ```bash
  source .venv/bin/activate
  ```

The terminal should now indicate that the virtual environment is active, for example:

```
(.venv) your-folder-name $
```

**4. Install Ansible in the Virtual Environment**

With the virtual environment activated, install Ansible by running the following command:

```bash
pip install ansible
```

in case you needed to update your `ansible-lint` module, run:

```bash
pip install --upgrade ansible-lint
```

This will install Ansible and its dependencies within the isolated virtual environment.

**5. Verify the Installation**

After installing Ansible, confirm the installation by running:

```bash
ansible --version
```

This should output the installed version of Ansible, confirming that the installation was successful.

**6. Install Ansible Extension for VS Code (Optional)**

For enhanced Ansible development in Visual Studio Code, install the **Ansible extension**. This extension provides syntax highlighting, linting, and other features specific to Ansible playbooks.

You can find the extension in the [VS Code marketplace](https://marketplace.visualstudio.com/items?itemName=vscoss.vscode-ansible).



## Self-Study

### Reusing Ansible Artifacts

Efficiently managing and organizing Ansible automation requires breaking down tasks into smaller, reusable files. The following concepts are central to mastering this approach:

**Creating Reusable Components**
- **Variables Files**: Dedicated files containing only variables for consistent use.
- **Task Files**: Modular files that focus solely on specific tasks.
- **Roles**: Comprehensive sets of tasks, variables, defaults, and handlers organized into structured directories for specific functionalities.

**Dynamic and Static Reuse**
- **Dynamic Reuse (Includes)**:
	- Processed at runtime, enabling tasks to adapt based on earlier playbook results.
	- Includes modules such as include_role and include_tasks.
	- Ideal for scenarios requiring loops or variable input.

- **Static Reuse (Imports)**:
	- Pre-processed during playbook parsing, ensuring predictable execution order.
	- Incorporates modules like import_role and import_tasks.
	- Suitable for workflows requiring strict task sequencing.

**When to Use Roles**
- Simplify complex playbooks by encapsulating logic into roles.
- Store defaults, handlers, and tasks in separate directories for clarity and reusability.
- Share and version control roles easily using platforms like Ansible Galaxy.

**Key Considerations**
- **Task and Handler Reuse**: Use `include_tasks` or `import_tasks` for repetitive actions (e.g., restarting services).
- **Performance Impact**: Dynamic reuse may increase runtime complexity, while static reuse enhances performance by reducing processing overhead.
- **Avoid Mixing Approaches**: For consistency, use either dynamic or static reuse within a single playbook.


## Implementation

### Step 1: Set Up Jenkins Server
1. Provision an EC2 instance for Jenkins (t2.micro, Ubuntu 24.04 LTS).
2. Configure security groups to allow necessary traffic (SSH on port 22, HTTP on port 8080).

![Security group](images/sg-jenkins.png)

### Step 2: Configure Jenkins for CI/CD

1. Update the system and install the default JDK:

   ```bash
   sudo apt update
   sudo apt install default-jdk-headless
   ```

2. Install Jenkins using the official Jenkins repository:

   ```bash
   sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
     https://pkg.jenkins.io/debian/jenkins.io-2023.key
   echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
     https://pkg.jenkins.io/debian binary/ | sudo tee \
     /etc/apt/sources.list.d/jenkins.list > /dev/null
   ```

   ```bash
   sudo apt-get update
   sudo apt-get install jenkins
   ```

3. Start Jenkins and enable it to run on boot:

   ```bash
   sudo systemctl start jenkins
   sudo systemctl enable jenkins
   ```

4. Check the status of Jenkins to ensure it's running:

   ```bash
   sudo systemctl status jenkins
   ```

5. Access Jenkins through your browser by navigating to `http://<your-server-ip>:8080` and complete the initial setup.

6. During the initial setup, you'll be asked for the initial admin password. You can retrieve it with:

   ```bash
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```

7. Follow the on-screen instructions to complete the installation, including installing recommended plugins and creating the first admin user.

![jenkins first page](images/jenkins-first-page.png)
![jenkins installation](images/jenkins-web-installation-1.png)
![jenkins installation](images/jenkins-web-installation-2.png)
![jenkins installation](images/jenkins-instance-config.png)
![jenkins installation](images/jenkins-final-installation.png)
![jenkins installation](images/setup-admin-user.png)
![Jenkins dashboard](images/jenkins-dashboard.png)

### Step 3: Setup initial Jenkin job flow
1. Add a Webhook in GitHub

   - Open the GitHub repository for your project: [https://github.com/fmanimashaun/ansible-config-mgt](https://github.com/fmanimashaun/ansible-config-mgt).
   - Go to **Settings** > **Webhooks**.
   - Click **Add webhook** and configure it as follows:
     - **Payload URL**: Enter your Jenkins webhook URL:
       
       ```
       http://<your-server-ip>:8080/github-webhook/
       ```

     - **Content type**: Select `application/json`.
     - **Which events would you like to trigger this webhook?**: Choose **Just the push event**.
   - Click **Add webhook** to save.

This webhook will notify Jenkins of any push events in the GitHub repository, triggering an automated build.

![github webhook](images/github-webhook.png)

2. Create a New Jenkins Job

   - Go to your Jenkins Dashboard at `http://<your-server-ip>:8080`.
   - Click **New Item**.
   - Name the job `ansible`, select **Freestyle project**, and click **OK**.

   ![create jenkins job](images/create-job.png)

3. Configure Source Code Management

   - In the job configuration page, scroll to the **Source Code Management** section.
   - Select **Git** and enter the GitHub repository URL:

     `https://github.com/fmanimashaun/ansible-config-mgt.git`

   - In the **Credentials** dropdown, add your GitHub credentials (username and password or a personal access token)
   ![githib link](images/repo-link-1.png)
   ![githib link](images/repo-link-2.png)

   - save and then test the configuration by clicking **build now** in job page
   ![github test](images/github-test.png)

4. Configure Build Trigger

   - In the job page, click on **configure** 
   - Scroll down to the **Build Triggers** section.
   - Select **GitHub hook trigger for GITScm polling**. This will ensure Jenkins triggers a build every time a change is pushed to GitHub.
   - Scroll to the **Post-build Actions** section.
   - Click **Add post-build action** and select **Archive the artifacts**.
   - In the **Files to archive** field, enter `**`, which will archive all files generated during the build.
   - Click **Save** to finalize the configuration.
   - To test the setup, push a change to the GitHub repository, which should trigger an automatic build in Jenkins.
   - Check the **Build History** in Jenkins to confirm that the build triggered correctly. You should also see the archived files in the build’s **Artifacts** section.

   ![github trigger](images/github-trigger.png)

### Step 4: Jenkins Job Enhancement

1. Connect to the jenkins-server via ssh and run the following commands:



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
9. **Video demonstration**:
   A simple demonstration of the whole project from setup to running the ansible playbook
   URL: [Video Demonstration Link](https://youtu.be/Ng4j6ldrf7Q)

