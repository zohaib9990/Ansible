# How We Deploy a High-Performance Java-Based Application for Enhanced User Experience 🚀

“Explore how EIT Crafted a Robust and Secure Application Deployment for Enhanced User Engagement.”
---
## The Problem ⚠️
In an increasingly digital world 🌐, our client faced challenges with their existing application deployment processes. The traditional deployment was slow ⏳, prone to errors ❌, and lacked security measures 🔒, resulting in frequent downtimes 🛑. They needed a secure, scalable, and efficient Continuous Integration and Continuous Deployment (CI/CD) pipeline to streamline their operations 🔄. Additionally, the client struggled with response times and overall application performance, which frustrated users and hindered engagement. Inconsistent application performance could also negatively impact user experience 😤, leading to lost opportunities for user interaction and support 💔.

## Technology Overview 🛠️

- **Version Control 🐙**: Code is managed in a centralized GitHub repository where developers commit changes, ensuring robust version control and collaboration across teams 🤝.

- **CI/CD ♾️**: Jenkins serves as the CI/CD tool for automating the deployment process. It monitors the GitHub repository for commits, automatically triggering the CI/CD pipeline when new code is detected. This accelerates development cycles by reducing manual processes ⏩.

- **Build and Package 📦**: Jenkins utilizes Maven to build and package the application into a deployable format (e.g., JAR). This guarantees consistent builds from the source code and simplifies deployments. 

- **Dependency Check 🛡️**: Integrated Maven with Dependency-Check to scan project dependencies for vulnerabilities, identifying and mitigating security risks early in development.

- **Configuration Management ⚙️**: Ansible automates the configuration and setup of Docker containers, ensuring consistent environments across various stages of deployment. 

- **Containerization 🐳**: Docker is used to containerize the application, encapsulating the app and its dependencies into portable containers. This eliminates issues such as "works on my machine" and provides consistent runtime environments. 

- **Code Compilation and Testing 🔄**: Maven compiles the application and executes automated tests to verify functionality, ensuring that new changes do not introduce bugs 🐞. 

- **Code Analysis 🧐**: Jenkins integrates with SonarQube to perform code quality and security analysis, maintaining high standards for reliability and maintainability. 

- **Container Image Scanning 🔍**: Trivy scans Docker images for known vulnerabilities before deployment, ensuring that containers are secure and free from critical vulnerabilities.

- **Container Orchestration 🚢**: Jenkins deploys the containerized application to a Kubernetes cluster, leveraging Kubernetes for orchestrating, scaling, and maintaining the high availability of the application.


## Phase 1: Initial Setup

### Step 1: Launch EC2 (Ubuntu 24.04)
1. **Provision an EC2 instance on AWS** with Ubuntu 24.04.
2. **Launch an Ubuntu T2 Large Instance**:
   - Create a new key pair or use an existing one.
   - Enable HTTP and HTTPS in the Security Group.
3. **Connect to the instance via SSH.**

---

### Step 2: Clone the Code
1. **Fork the repository to your GitHub account.**
2. **Update all packages**:
   ```bash
   # Update all installed packages to the latest version
   sudo apt-get update && sudo apt-get upgrade -y
   ```
3. **Clone your application repository**:
   ```bash
   # Replace the URL with your GitHub repository URL and clone the repository from GitHub
   git clone <your-repository-url>
   ```

## Phase 2: Install Docker, SonarQube, Trivy, and Jenkins

### Step 1: Install Docker
1. **Set up Docker**:
   ```bash
   # Install Docker
   sudo apt-get install docker.io -y
   
   # Add current user to the Docker group to manage Docker as a non-root user
   sudo usermod -aG docker $USER
   
   # Apply Docker group changes immediately
   newgrp docker
   
   # Adjust permissions on the Docker socket
   sudo chmod 777 /var/run/docker.sock
   ```

---

### Step 2: Install SonarQube and Trivy

#### SonarQube
1. **Run SonarQube in a container**:
   ```bash
   # Run SonarQube in a Docker container and expose it on port 9000
   docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
   ```
2. **Access SonarQube**:
   - Open port **9000** in the EC2 Security Group.
   - Access via `http://<public-IP>:9000` (default credentials: `admin/admin`).

#### Trivy
1. **Install dependencies**:
   ```bash
   # Install required packages for downloading and verifying Trivy
   sudo apt-get install wget apt-transport-https gnupg lsb-release -y
   ```
2. **Install Trivy**:
   ```bash
   # Download and add Trivy's GPG key
   wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
   
   # Add Trivy's repository to the sources list
   echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
   
   # Update package list and install Trivy
   sudo apt-get update && sudo apt-get install trivy -y
   ```

---

### Step 3: Install Jenkins for Automation
1. **Install Java**:
   ```bash
   # Update package list
   sudo apt update -y
   
   # Fetch and install Adoptium GPG key for Java
   wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | sudo tee /etc/apt/keyrings/adoptium.asc
   
   # Add Adoptium's repository to the sources list
   echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print $2}' /etc/os-release) main" | sudo tee /etc/apt/sources.list.d/adoptium.list
   
   # Update package list again to include the new Java source
   sudo apt update -y
   
   # Install Temurin 17 JDK (Java Development Kit)
   sudo apt install temurin-17-jdk -y
   ```

2. **Install Jenkins**:
   ```bash
   # Fetch and install Jenkins' GPG key
   curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
   
   # Add Jenkins repository to the sources list
   echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
   
   # Update package list to include Jenkins' repository
   sudo apt-get update -y
   
   # Install Jenkins
   sudo apt-get install jenkins -y
   
   # Start Jenkins service
   sudo systemctl start jenkins
   
   # Enable Jenkins to start at boot
   sudo systemctl enable jenkins
   ```

3. **Change Jenkins Port from 8080 to 8090**:
   ```bash
   sudo systemctl stop jenkins
   sudo systemctl status jenkins
   cd /etc/default
   sudo vi jenkins   # Change HTTP_PORT=8090 and save and exit
   cd /lib/systemd/system
   sudo vi jenkins.service  # Change Environment="Jenkins_port=8090", save and exit
   sudo systemctl daemon-reload
   sudo systemctl restart jenkins
   sudo systemctl status jenkins
   ```

4. **Access Jenkins**:
   - Open Inbound Ports **8080** and **8090** (for Jenkins), **9000** (for SonarQube) in the EC2 Security Group.
   - Access Jenkins via `http://<EC2 Public IP>:8090`.
   - Retrieve the Jenkins initial admin password:
     ```bash
     sudo cat /var/lib/jenkins/secrets/initialAdminPassword
     ```

5. **Complete Jenkins Setup**:
   - Unlock Jenkins using the administrative password.
   - Install the suggested plugins.
   - Create a user and click "Save and Continue."
   - Complete setup on the Jenkins Getting Started screen.

---

### Step 4: Install kubectl
1. **Install kubectl**:
   ```bash
   # Install curl for downloading files
   sudo apt install curl -y
   
   # Download the latest stable version of kubectl
   curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
   
   # Install kubectl
   sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
   
   # Check the installed version of kubectl
   kubectl version --client
   ```
---

### Step 5: Install Ansible
1. **Add the Ansible Repository**:
   ```bash
   sudo apt-get update
   sudo apt install software-properties-common
   sudo add-apt-repository --yes --update ppa:ansible/ansible
   sudo apt install python3
   ```

2. **Install Ansible**:
   ```bash
   sudo apt install ansible -y
   sudo apt install ansible-core -y
   ansible --version
   ```

3. **Create an Inventory File in Ansible**:
   ```bash
   cd /etc/ansible
   sudo nano hosts
   ```
   - Add the public IP of Jenkins under a group, for example:
   ```bash
   [local]
   <Jenkins_Public_IP>
   ```

## Phase 3: Jenkins and SonarQube Configuration

### Step 1: Install Necessary Plugins in Jenkins
1. **Navigate to the Plugins Section**:
   - Go to **Manage Jenkins** → **Manage Plugins** → **Available Plugins**.

2. **Install the following plugins**:
   - Eclipse Temurin Installer
   - SonarQube Scanner
   - Docker
   - Docker Commons
   - Docker Pipeline
   - Docker API
   - Docker Build Step
   - OWASP Dependency-Check
   - Kubernetes
   - Kubernetes CLI
   - Kubernetes Client API
   - Kubernetes Pipeline DevOps Steps
   - Kubernetes Credentials
   - Kubernetes Credentials Provider
   - Ansible
   - Email Extension Template

---

### Step 2: Configure Global Tools in Jenkins
1. **Navigate to Global Tool Configuration**:
   - Go to **Manage Jenkins** → **Global Tool Configuration**.

2. **Configure Java (JDK 17)**:
   - Click on **Add JDK**.
   - Name: `jdk17`.
   - **Install automatically** → **Install from adoptium.net**.
   - Version: `jdk-17.0.8.1+1`.

3. **Configure Maven**:
   - Click on **Add Maven**.
   - Name: `maven3`.
   - Version: `3.6.0`.

4. **Configure Docker**:
   - Click on **Add Docker**.
   - Name: `docker`.
   - **Install automatically** → **Download from docker.com**.
   - Version: `latest`.

5. **Configure OWASP Dependency-Check**:
   - Click on **Add Dependency-Check**.
   - Name: `DP-Check`.
   - **Install automatically** → **Install from github.com**.
   - Version: `dependency-check 8.4.0`.

6. **Configure Ansible**:
   - Click on **Add Ansible**.
   - Name: `ansible`.
   - Command: `which ansible` (run on the instance).
   - Install directory: `/usr/bin/`.

7. **Configure SonarQube Scanner**:
   - Click on **Add SonarQube Scanner**.
   - Name: `sonar-scanner`.
   - **Install automatically** → **Install from Maven Central**.
   - Version: `SonarQube Scanner 5.0.1.3006`.

8. **Save all configurations**:
   - Click on **Apply** and **Save** to save all the tool configurations.

---

### Step 3: Configure Tools and Credential

1. **Set up Quality Gate in SonarQube**:
   - Go to your **SonarQube Server**.
   - In the SonarQube Dashboard, navigate to **Administration** → **Configuration** → **Webhooks**.
   - Click on **Create**:
     - Name: `Jenkins`.
     - URL: `http://<jenkins-public-ip>:8090/sonarqube-webhook/` (replace with your Jenkins public IP).
     - Click on **Create**.

2. **Generate and Configure SonarQube Token**:
   - In the SonarQube Dashboard, navigate to **Administration** → **Security** → **Users** → **Tokens**.
   - Click on **Update Token** → Name the token → Click on **Generate Token**.
   - Copy the generated token.

3. **Add SonarQube Token to Jenkins**:
   - Go to the **Jenkins Dashboard** → **Manage Jenkins** → **Credentials** → **System** → **Global credentials (unrestricted)**.
   - Add Credentials:
     - **Kind**: Secret Text.
     - **Secret**: Paste the token copied from SonarQube.
     - **ID**: `Sonar-token`.
     - **Description**: `Sonar-token`.
     - Click on **Create**.

4. **Add DockerHub Credentials to Jenkins**:
   - In **Global Credentials**, add credentials:
     - **Kind**: Username and Password.
     - **Username**: Your DockerHub username.
     - **Password**: Your DockerHub password.
     - **ID**: `docker`.
     - **Description**: `docker`.
     - Click on **Create**.

5. **Add SSH Credentials to Invoke Ansible with Jenkins**:
   - Add Credentials:
     - **Kind**: SSH Username with private key.
     - **ID**: `SSH`.
     - **Description**: `SSH`.
     - **Username**: `ubuntu`.
     - **Private Key**: Enter directly and paste your PEM file for the key.
     - Click on **Create**.
6. **Add Mail Credentials to Jenkins**:
   - In **Global Credentials**, add credentials:
     - **Kind**: Username and Password.
     - **Username**: Enter your email address
     - **Password**: Input your email's generated password (e.g., an app password).
     - **ID**: `mail`.
     - **Description**: `mail`.
     - Click on **Create**.

---

### Step 4: Configure SonarQube in Jenkins
1. **Navigate to the Dashboard**:
   - Go to **Dashboard** → **Manage Jenkins** → **System**.

2. **Find SonarQube Servers** → **SonarQube Installations**.
   - Click on **Add SonarQube**:
     - **Name**: `sonar-server`.
     - **Server URL**: `http://<ipaddress>:9000/` (replace with your SonarQube server IP).
     - **Server Authentication Token**: `Sonar-token`.

3. **Click on Apply and Save**.
---
### Step 5: Configure Mail Server in Jenkins

1. **Generate an App-Specific Password in Gmail**:
   - Open your Gmail and click on your profile picture.
   - Navigate to **Manage Your Google Account** → **Security** tab (on the left side panel).
   - Ensure that **2-Step Verification** is enabled.
   - In the search bar, type **app passwords** and select **App Passwords**.
   - Enter your Gmail password to proceed.
   - You will be redirected to the **App Passwords** page.
   - Under **"Select App"**, type the name **Jenkins**.
   - Click **Create** and copy the generated app password.

2. **Navigate to the Jenkins Dashboard**:
   - Go to **Dashboard** → **Manage Jenkins** → **Configure System**.

3. **Configure E-mail Notification**:
   - Scroll down to the **E-mail Notification** section.
   - **SMTP Server**: `smtp.gmail.com`
   - Click on **Advanced**.
   - Enable **SMTP Authentication**.
   - **User Name**: `your-email@gmail.com` (replace with your Gmail address).
   - **Password**: Paste the app password you generated earlier.
   - Check **Use SSL**.
   - **SMTP Port**: `465`.
   - **Charset**: `UTF-8`.

4. **Configure Extended E-mail Notification**:
   - Find the **Extended E-mail Notification** section.
   - **SMTP Server**: `smtp.gmail.com`.
   - **SMTP Port**: `465`.
   - Click on **Advanced**.
   - **Credentials**: Select your Gmail credentials.
   - Enable **Use SSL**.
   - **Default Content Type**: `HTML (text/html)`.

5. **Set Default Triggers**:
   - Scroll to **Default Triggers**.
   - Check:
     - **Always**
     - **Failure - Any**

6. **Apply and Save**:
   - Click **Apply** and then **Save** to finalize the configuration.

## Phase 4: Kubernetes Setup

### **Step 1: Connect Your Machines**

1. **Check Ansible is installed on the Jenkins machine**.

2. **Pick Two Ubuntu 24.04 Instances:**
   - One for the **Kubernetes Master** and another for the **Worker**.

#### On Master Node:
1. Set the hostname:
   ```bash
   sudo hostnamectl set-hostname k8s-master
   exec bash
   ```

#### On Worker Node:
1. Set the hostname:
   ```bash
   sudo hostnamectl set-hostname k8s-worker
   exec bash
   ```
---
### **Step 2: Setup on Both Master and Worker Nodes**

Create a script to execute on both master and worker nodes to configure the Kubernetes environment.

```bash
#!/bin/bash

# Update and upgrade system
sudo apt update && sudo apt upgrade -y

# Disable swap
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# Load necessary kernel modules
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter

# Set kernel parameters for Kubernetes
sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

# Apply sysctl settings
sudo sysctl --system

# Install required packages
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates

# Define Kubernetes version and CRI-O version
KUBERNETES_VERSION=v1.31
CRIO_VERSION=v1.31

# Install dependencies for adding repositories
sudo apt update

# Add the Kubernetes repository
sudo curl -fsSL https://pkgs.k8s.io/core:/stable:/$KUBERNETES_VERSION/deb/Release.key |
    gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

sudo echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/$KUBERNETES_VERSION/deb/ /" |
    tee /etc/apt/sources.list.d/kubernetes.list

# Add the CRI-O repository
sudo curl -fsSL https://pkgs.k8s.io/addons:/cri-o:/stable:/$CRIO_VERSION/deb/Release.key |
    gpg --dearmor -o /etc/apt/keyrings/cri-o-apt-keyring.gpg

sudo echo "deb [signed-by=/etc/apt/keyrings/cri-o-apt-keyring.gpg] https://pkgs.k8s.io/addons:/cri-o:/stable:/$CRIO_VERSION/deb/ /" |
    tee /etc/apt/sources.list.d/cri-o.list

# Install the required packages
sudo apt update
sudo apt install -y cri-o kubelet kubeadm kubectl

# Start CRI-O
sudo systemctl start crio.service

# Install socat for networking
sudo apt install -y socat
```
---

### **Step 3: Kubernetes Master Node Initialization**

#### On Master Node:

1. Initialize the Kubernetes cluster:
   ```bash
   sudo kubeadm init
   ```

2. Configure `kubectl`:
   ```bash
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   ```

3. Install Calico CNI:
   ```bash
   kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml
   ```

---

### **Step 4: Kubernetes Worker Node Join**

#### On Worker Node:
1. Join the worker node to the master node:
   ```bash
   sudo kubeadm join <master-node-ip>:<master-node-port> --token <token> --discovery-token-ca-cert-hash <hash>
   ```

#### On Master Node:
1. Navigate to the directory containing the kubeconfig file:
   ```bash
   cd ~/.kube
   ```

2. Display the contents of the config file:
   ```bash
   cat config
   ```

3. Copy the contents of the config file and create a new file on your local system with a `.yaml` extension, then paste the copied content into this file.

4. Add the configuration file to Jenkins credentials:
   - Go to **Manage Jenkins** → **Credentials** → **System** → **Global credentials (unrestricted)**.
   - Add credentials:
     - **Kind**: Secret file.
     - **ID and Description**: `k8s`.
     - Choose the file you saved locally and click **Create**.

---

### **Step 5: Master-Slave Setup for Ansible and Kubernetes**

#### Configure SSH Key Exchange:
1. On the Jenkins instance, generate an SSH key pair:
   ```bash
   ssh-keygen
   ```

2. Change the directory to `.ssh` and copy the public key:
   ```bash
   cd .ssh
   cat id_rsa.pub
   ```

3. On the Kubernetes master node, paste the copied public key into `authorized_keys`:
   ```bash
   cd ~/.ssh
   sudo vi authorized_keys
   ```
   - Paste the public key into a new line and save the file.

4. Test the SSH connection from the Jenkins machine to the Kubernetes master:
   ```bash
   ssh ubuntu@<public-ip-k8s-master>
   ```

---

### **Step 6: Configure Ansible Hosts**

1. On the Jenkins instance, navigate to the Ansible directory:
   ```bash
   cd /etc/ansible
   ls
   sudo nano hosts
   ```

2. Add the following configuration to the `hosts` file:
   ```bash
   [k8s]   # You can name this section whatever you want
   <public-ip-of-k8s-master>
   ```
---

### **Step 7: Test Ansible Master-Slave Connection**

1. Use the following commands to test the Ansible master-slave connection:
   ```bash
   ansible -m ping k8s
   ansible -m ping all
   ```
---

### **Step 8: Configure Jenkins CI/CD Pipelines**

#### Create a CI/CD Pipeline to Automate Deployment:
1. In the Jenkins dashboard, click on **New Item**.
   - Enter an item name: `JavaApplication`.
   - Select **Pipeline** and click **Create**.

2. **Configure the Pipeline**:
   - Click on **Pipeline** and paste the following pipeline script:

```groovy
pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKER_CREDENTIALS = credentials 'docker'
    }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout SCM') {
            steps {
                git 'your-repository-url'
            }
        }
        stage('Maven Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }
        stage('Maven Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=JavaApplication \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=JavaApplication
                    '''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage ('Build war file') {
            steps {
                sh 'mvn clean install -DskipTests=true'
            }
        }
        stage("OWASP Dependency Check") {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --format XML ', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage('Install Docker Through Ansible') {
            steps {
                dir('Ansible') {
                    script {
                        ansiblePlaybook credentialsId: 'SSH', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/', playbook: 'docker.yaml',
                        extras: "--extra-vars 'DOCKER_USERNAME=${env.DOCKER_CREDENTIALS_USR} DOCKER_PASSWORD=${env.DOCKER_CREDENTIALS_PSW}'"
                    }
                }
            }
        }
        stage('k8s using ansible') {
            steps {
                dir('Ansible') {
                    script {
                        ansiblePlaybook credentialsId

: 'SSH', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/', playbook: 'kube.yaml'
                    }
                }
            }
        }
    }
}
```

3. Click **Apply** and **Save**.

4. Click **Build Now** to start the pipeline.

### Phase 5: CI and CD Pipeline

1. **CI Pipeline**
   ```groovy
   pipeline {
       agent any
       tools {
           jdk 'jdk17'
           maven 'maven3'
       }
       environment {
           SCANNER_HOME = tool 'sonar-scanner'
           DOCKER_CREDENTIALS = credentials 'docker'
       }
       stages {
           stage('Clean Workspace') {
               steps {
                   cleanWs()
               }
           }
           stage('Checkout SCM') {
               steps {
                   git 'your-repository-url'
               }
           }
           stage('Maven Compile') {
               steps {
                   sh 'mvn clean compile'
               }
           }
           stage('Maven Test') {
               steps {
                   sh 'mvn test'
               }
           }
           stage('SonarQube Analysis') {
               steps {
                   withSonarQubeEnv('sonar-server') {
                       sh '''
                       $SCANNER_HOME/bin/sonar-scanner \
                       -Dsonar.projectName=JavaApplication \
                       -Dsonar.java.binaries=. \
                       -Dsonar.projectKey=JavaApplication
                       '''
                   }
               }
           }
           stage('Quality Gate') {
               steps {
                   script {
                       waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                   }
               }
           }
           stage('Build war file') {
               steps {
                   sh 'mvn clean install -DskipTests=true'
               }
           }
           stage("OWASP Dependency Check") {
               steps {
                   dependencyCheck additionalArguments: '--scan ./ --format XML ', odcInstallation: 'DP-Check'
                   dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
               }
           }
           stage('TRIVY FS SCAN') {
               steps {
                   sh "trivy fs . > trivy.txt"
               }
           }
           stage('Install Docker Through Ansible') {
               steps {
                   dir('Ansible') {
                       script {
                           ansiblePlaybook credentialsId: 'SSH', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/', playbook: 'docker.yaml',
                           extras: "--extra-vars 'DOCKER_USERNAME=${env.DOCKER_CREDENTIALS_USR} DOCKER_PASSWORD=${env.DOCKER_CREDENTIALS_PSW}'"
                       }
                   }
               }
           }
           stage('K8s Using Ansible') {
               steps {
                   dir('Ansible') {
                       script {
                           ansiblePlaybook credentialsId: 'SSH', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/', playbook: 'kube.yaml'
                       }
                   }
               }
           }
           stage("Trigger Deployment") {
               steps {
                   build job: 'CD-JavaApplication', wait: false
               }
           }
       }
       post {
           always {
               emailext(
                   attachLog: true,
                   subject: "'${currentBuild.result}'",
                   body: """
                       Project: ${env.JOB_NAME}<br/>
                       Build Number: ${env.BUILD_NUMBER}<br/>
                       URL: ${env.BUILD_URL}<br/>
                   """,
                   to: 'alert@exceptionalit.org',
                   attachmentsPattern: 'trivy.txt'
               )
           }
       }
   }
   ```

2. **CD Pipeline**
   ```groovy
   pipeline {
       agent any
       stages {
           stage('Clean Workspace') {
               steps {
                   cleanWs()  // Clean the workspace before starting
               }
           }
           stage('Checkout SCM') {
               steps {
                   git branch: 'legacy', url: 'your-repository-url'
               }
           }
           stage('Deploy to Container') {
               steps {
                   script {
                       // Stop and remove any existing container named 'chatbot'
                       sh '''
                       if [ "$(docker ps -q -f name=chatbot)" ]; then
                           docker stop chatbot
                           docker rm chatbot
                       fi
                       '''

                       // Pull the latest version of the Docker image
                       sh 'docker pull rizwanshaukat/chatbot:latest'

                       // Run the container with port mapping and in detached mode
                       sh 'docker run -d --name chatbot -p 3000:3000 rizwanshaukat/chatbot:latest'
                   }
               }
           }
           stage('Deploy to Kubernetes') {
               steps {
                   script {
                       withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                           sh 'kubectl apply -f k8s/chatbot-ui.yaml'
                       }
                   }
               }
           }
       }
       post {
           always {
               emailext(
                   attachLog: true,
                   subject: "'${currentBuild.result}'",
                   body: """
                       Project: ${env.JOB_NAME}<br/>
                       Build Number: ${env.BUILD_NUMBER}<br/>
                       URL: ${env.BUILD_URL}<br/>
                   """,
                   to: 'alert@exceptionalit.org'
               )
           }
       }
   }
   ```
