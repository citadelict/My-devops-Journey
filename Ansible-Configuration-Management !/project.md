
# Ansible Configuration Management for EC2

## Overview

This guide details the process for deploying and managing EC2 instances using Ansible. It includes steps for Ansible installation, playbook creation, and server management automation.

## Installation: Setting Up Ansible on Your EC2 Instance

### Step 1: Preparing Your EC2 Instance

- **Rename your Jenkins EC2 instance to `Jenkins-Ansible`**:
  - **Purpose**: This specific naming helps in easily identifying the purpose of the instance, which is crucial when managing multiple servers.
  - **Action**: Update the `Name` tag on your EC2 instance dashboard.
    
  ![EC2 Instance List](https://github.com/citadelict/My-devops-Journey/blob/main/Ansible-Configuration-Management%20!/images/updated%20name%20tag%20jtoa.png)

### Step 2: Creating a GitHub Repository

- **Initiate a new repository named `ansible-config-mgt`**:
 


### Step 3: Installing Ansible

- **Install Ansible on your EC2 instance**:
  - **Purpose**: Ansible automates and simplifies application deployment, systems configuration, and other IT needs.

    ```bash
    sudo apt update && sudo apt upgrade -y
    sudo apt install ansible -y
    ```
  -  Ensure the installation was successful by checking the Ansible version.
    ```bash
    ansible --version
    ```
  ![Verify Ansible Installation](https://github.com/citadelict/My-devops-Journey/blob/main/Ansible-Configuration-Management%20!/images/installed%20ansible.png)

### Step 4: Configuring Jenkins

- **Configure Jenkins to build from the GitHub repository**:
  - **Purpose**: Automates the deployment process, ensuring that the latest configurations are always in use.
  - **Action**: Set up a new freestyle project in Jenkins and link it to the `ansible-config-mgt` GitHub repository.
 

- **Set up a GitHub webhook**:
  - **Purpose**: Triggers Jenkins builds automatically upon code commits, reducing the need for manual oversight.
  ![GitHub Webhook Configuration](https://github.com/citadelict/My-devops-Journey/blob/main/Ansible-Configuration-Management%20!/images/created%20webhook.png)

- **Configure SCM and build triggers**:
  - Under configure, select source code management and set it up to connect to github, set user name and password 
  ![SCM and Build Trigger Configuration](https://github.com/citadelict/My-devops-Journey/blob/main/Ansible-Configuration-Management%20!/images/scm.png)

  
  ![Build Trigger Setup](https://github.com/citadelict/My-devops-Journey/blob/main/Ansible-Configuration-Management%20!/images/build%20triggers.png)

- **Post-build actions**:
  - **Purpose**: Saves all necessary files and ensures that any deployment artifacts are stored properly.
    
- **Test the setup:**
- Make changes to the `README.md` file in the `main` branch.

- Ensure builds start automatically and Jenkins saves the files in:
  ![Console Output](https://github.com/citadelict/My-devops-Journey/blob/main/Ansible-Configuration-Management%20!/images/readme%20build%20auto.png)

         ls /var/lib/jenkins/jobs/ansible/builds/1/archive/
  
    ![Test](./images/readme.png)

## Preparing Your Development Environment

### Step 5: Using Visual Studio Code

- **Connect VSC to GitHub**:
  - **Commands**:
    ```bash
    git config --global user.email "your-github-email"
    git config --global user.name "Your-userName"
    ```

- **Clone the repository**:
  - **Purpose**: Allows you to work locally on the Ansible configurations.
  - **Command**:
    ```bash
    git clone <repository-url>
    ```
 

## Developing with Ansible

### Step 6: Ansible Playbook Development

- **Create a playbook directory structure**:
  - **Directories**: `playbooks` for storing playbooks and `inventory` for managing hosts.
  - **Purpose**: Organizes your Ansible project clearly, making it easier to manage.
  ![Create Playbook and Inventory](https://github.com/citadelict/My-devops-Journey/blob/main/Ansible-Configuration-Management%20!/images/created%20dir%20inv%2C%20playbooks.png)

### Step 7: Setting Up an Ansible Inventory

- **Configure inventory files**:
  - **Purpose**: Defines which hosts are managed by which playbooks, crucial for targeting the correct environments.
  - **Files**: `inventory/dev`, `inventory/staging`, `inventory/uat`, and `inventory/prod`.

      ![files](./images/files.png)

### Step 8: Running Your First Playbook    

  - Create a `common.yml`

   - **Edit `playbooks/common.yml` to include the following tasks:**

            - name: update web, nfs servers
          hosts: webservers, nfs, 
          become: yes
          tasks:
            - name: ensure wireshark is at the latest version
              yum:
                name: wireshark
                state: latest
           
    
            - name: update LB server and db server
              hosts: lb , db
              become: yes
              tasks:
                - name: Update apt repo
                  apt: 
                    update_cache: yes
            
                - name: ensure wireshark is at the latest version
                  apt:
                    name: wireshark
                    state: latest
            

        
Feel free to add additional tasks such as `creating a directory`, `changing the timezone`, or `running shell scripts`.

- **Here is the full `common.yml` playbook with all the additional tasks included:**
    ```yml
    ---
    - name: update web and nfs servers
      hosts: webservers, nfs
      become: yes
      tasks:
        - name: ensure wireshark is at the latest version
          yum:
            name: wireshark
            state: latest

        - name: create a directory
          file:
            path: /path/to/directory
            state: directory
            mode: '0755'

        - name: add a file into the directory
          copy:
            content: "This is a sample file content"
            dest: /path/to/directory/samplefile.txt
            mode: '0644'

        - name: set timezone to UTC
          timezone:
            name: UTC

        - name: run a shell script
          shell: |
            #!/bin/bash
            echo "This is a shell script"
            echo "Executed on $(date)" >> /path/to/directory/script_output.txt

    - name: update LB and db servers
      hosts: lb, db
      become: yes
      tasks:
        - name: Update apt repo
          apt:
            update_cache: yes

        - name: ensure wireshark is at the latest version
          apt:
            name: wireshark
            state: latest

        - name: create a directory
          file:
            path: /path/to/directory
            state: directory
            mode: '0755'

        - name: add a file into the directory
          copy:
            content: "This is a sample file content"
            dest: /path/to/directory/samplefile.txt
            mode: '0644'

        - name: set timezone to UTC
          timezone:
            name: UTC

        - name: run a shell script
          shell: |
            #!/bin/bash
            echo "This is a shell script"
            echo "Executed on $(date)" >> /path/to/directory/script_output.txt
    ```


 ### Step 9: Update GIT with the Latest Code

  - Commit your code changes to GitHub:

        git status
        git add .
        git commit -m "commit message"
        git push origin <branch-name>

![push](./images/gitpush.png)

   - Create a pull request and review the changes.

![pull request](./images/pull.png)

   - Act as a reviewer and ensure he code is okay

![review](./images/review-code.png)

   - merge the pull request to the master branch.


![merge](./images/merge.png)

   - pull the Latest Changes On Local Repository:
       - Checkout from the feature branch into the master branch

               git checkout main
               git pull origin main

  ![pull main](./images/pull-main.png)

### Step 10. Run Your first Ansible test

- **SSH into your Jenkins-Ansible instance Using the VSC `SSH-Agent`.**
  -  Configure SSH keys for Ansible:
        ```sh
        eval `ssh-agent -s`
        ssh-add <path-to-private-key>
        ssh-add -l
        ```
     ![Configure SSH Agent](https://github.com/citadelict/My-devops-Journey/blob/main/Ansible-Configuration-Management%20!/images/configure%20ssh.png)

  - SSH into your Jenkins-Ansible server using `ssh-agent`:
    ```sh
    ssh -A ubuntu@<public-ip>
    ```
    ![SSH Agent Connect](https://github.com/citadelict/My-devops-Journey/blob/main/Ansible-Configuration-Management%20!/images/ssh%20into%20ansible.png)


   - Navigate to `jenkins artifact archive directory for the latest build`
 
       ```sh
            #navigate to ansible build files
            cd /var/lib/jenkins/jobs/ansible/builds/1/archive/

   - Run the ansible test

             #run the playbook command
            ansible-playbook -i inventory/dev.yml playbooks/common.yml
            ```

  ![commons](https://github.com/citadelict/My-devops-Journey/blob/main/Ansible-Configuration-Management%20!/images/run-ansible-common.png)
  ![common2](https://github.com/citadelict/My-devops-Journey/blob/main/Ansible-Configuration-Management%20!/images/run-ansible-common2.png)


   - Verify that wire shark is running in each of the servers by running

               wireshark --version

     
### Webserver A
 ![output webserver](https://github.com/citadelict/My-devops-Journey/blob/main/Ansible-Configuration-Management%20!/images/webserver%20a.png)

### webserver B
 ![server2](https://github.com/citadelict/My-devops-Journey/blob/main/Ansible-Configuration-Management%20!/images/server2%20.png)

### DB server
 ![output db server](https://github.com/citadelict/My-devops-Journey/blob/main/Ansible-Configuration-Management%20!/images/db-server.png)

### NFS server
 ![nfs-server](https://github.com/citadelict/My-devops-Journey/blob/main/Ansible-Configuration-Management%20!/images/nfs-server.png)


## Optional Step: Repeat the Process
- Update your Ansible playbook with new tasks and repeat the full cycle of:
    > Checkout -> Change codes -> Commit -> PR -> Merge -> Build -> ansible-playbook

## Conclusion
congratulations, you have successfully created and automated routine tasks using ansible playbooks and running the builds through jenkins pipeline

---

  
         
     



  
