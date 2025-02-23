# Jenkins Pipeline for Java Applications using Maven, SonarQube, Argo CD, and Kubernetes

![flow1](https://github.com/user-attachments/assets/32c09791-ed37-4bc0-b35e-fcf201b1ca6b)


This project demonstrates how to set up an end-to-end Jenkins pipeline for a Java application. The pipeline integrates key DevOps tools such as SonarQube for code quality analysis, Argo CD for GitOps-based deployment, and Kubernetes for container orchestration.

## 🚀 Prerequisites

Ensure you have the following before proceeding:
- **Java application code** hosted in a Git repository (e.g., GitHub, GitLab, Bitbucket).
- **Jenkins server** installed and running.
- **Kubernetes cluster** (Minikube, AWS EKS, or any other Kubernetes setup).
- **Argo CD** installed for application deployment.

---

## 🛠 Step-by-Step Setup

### Step 1: Create an EC2 Instance (t2.large)
- Launch a new **t2.large** EC2 instance with your preferred OS.
- Open necessary ports:
  - **Port 22** (SSH access)
  - **Port 8080** (Jenkins)
  - **Port 9000** (SonarQube, if installed)

### Step 2: SSH into the EC2 Instance
```bash
cp jenkins.pem ~/.ssh/
chmod 400 ~/.ssh/jenkins.pem
ssh -i ~/.ssh/jenkins.pem ubuntu@<EC2_PUBLIC_IP>
```

### Step 3: Install Java and Jenkins
#### Install Java
```bash
sudo apt update
sudo apt install openjdk-17-jre
java -version
```

#### Install Jenkins
```bash
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

**Open Port 8080 in AWS Security Groups**
- Navigate to AWS EC2 > Security Groups.
- Add an **inbound rule** to allow TCP **port 8080**.

### Step 4: Access Jenkins
- Open Jenkins UI: `http://<EC2_PUBLIC_IP>:8080`
- Retrieve the Jenkins initial admin password:
  ```bash
  sudo cat /var/lib/jenkins/secrets/initialAdminPassword
  ```
- Complete the Jenkins setup wizard.

### Step 5: Install Required Jenkins Plugins
- Log in to Jenkins.
- Go to **Manage Jenkins > Manage Plugins**.
- Install the following plugins:
  - **Docker Pipeline**
  - **SonarQube Scanner**
- Restart Jenkins after installation.

### Step 6: Configure SonarQube on EC2
```bash
sudo apt install unzip
sudo adduser sonarqube
sudo su - sonarqube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.4.0.54424.zip
unzip sonarqube-9.4.0.54424.zip
sudo chmod -R 755 /home/sonarqube/sonarqube-9.4.0.54424
sudo chown -R sonarqube:sonarqube /home/sonarqube/sonarqube-9.4.0.54424
cd sonarqube-9.4.0.54424/bin/linux-x86-64/
./sonar.sh start
```

**Access SonarQube**: `http://<EC2_PUBLIC_IP>:9000`

### Step 7: Authenticate SonarQube in Jenkins
- Generate a **SonarQube token** under **My Account > Security > Tokens**.
- Add the token in Jenkins under **Manage Jenkins > Manage Credentials** as a **Secret text**.
- Similarly, add **DockerHub** and **GitHub** credentials.

### Step 8: Install Docker on EC2
```bash
sudo apt update
sudo apt install docker.io
sudo usermod -aG docker jenkins
sudo usermod -aG docker ubuntu
sudo systemctl restart docker
```
**Restart Jenkins**: `http://<EC2_PUBLIC_IP>:8080/restart`

### Step 9: Install Kubectl and Minikube
#### Install Kubectl
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```

#### Install Minikube
```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x minikube-linux-amd64
sudo mv minikube-linux-amd64 /usr/local/bin/minikube
minikube start --driver=docker
kubectl get nodes
```

### Step 10: Install Argo CD
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

**Access ArgoCD UI**
- Edit service type:
  ```bash
  kubectl edit svc argocd-server -n argocd
  ```
  Change `type: ClusterIP` to `type: NodePort`.
- Get the ArgoCD admin password:
  ```bash
  kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
  ```
- Access the UI: `minikube service argocd-server -n argocd --url`

### Step 11: Create & Run the Jenkins Pipeline
- In Jenkins, create a **new item**.
- Select **Pipeline** and configure:
  - Use Git for **repository URL**.
  - Select **main branch**.
  - Specify the **Jenkinsfile** script path.
- Run the pipeline and monitor the results in Jenkins.

### Step 12: Deploy with ArgoCD
- Create a **new application** in ArgoCD.
- Set the repository URL and deployment path.
- Deploy and verify:
  ```bash
  minikube service --url
  ```

---

## 🎯 Outcome
This Jenkins pipeline automates the **CI/CD process** for a Java application using:
- **Maven** for build & dependencies.
- **SonarQube** for code quality analysis.
- **Docker** for containerization.
- **Helm & Kubernetes** for deployment.
- **Argo CD** for continuous delivery.

By following this guide, you can efficiently manage Java application deployments using **GitOps** principles and modern DevOps tools.

---

## 📌 Future Enhancements
- Implement **Slack notifications** for pipeline status updates.
- Enable **automatic scaling** in Kubernetes.
- Configure **Helm charts** for different environments (dev, staging, prod).

🚀 **Happy DevOps-ing!** 🚀

#Jenkins #Maven #SonarQube #ArgoCD #Helm #Kubernetes #CI/CD #DevOps


