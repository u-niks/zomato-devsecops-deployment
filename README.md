## 🚀 **DevOps Project: Zomato Clone App Deployment**

In this **DevOps project**, I demonstrate how to deploy a **Zomato Clone App** using a modern **DevSecOps CI/CD workflow** with Jenkins, SonarQube, Docker, Amazon EKS, Kubernetes, Prometheus, and Grafana.

The project covers application build, code quality checks, container image scanning, Docker image publishing, Kubernetes deployment, and end-to-end monitoring for both **VM infrastructure** and **Kubernetes workloads**.

---

## 🛠️ Tools & Services Used:

1. **GitHub** ![GitHub](https://img.shields.io/badge/GitHub-181717?style=flat-square&logo=github&logoColor=white)
2. **Jenkins** ![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=flat-square&logo=jenkins&logoColor=white)
3. **SonarQube** ![SonarQube](https://img.shields.io/badge/SonarQube-4E9BCD?style=flat-square&logo=sonarqube&logoColor=white)
4. **Docker** ![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)
5. **Kubernetes** ![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=flat-square&logo=kubernetes&logoColor=white)
6. **Prometheus** ![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=flat-square&logo=prometheus&logoColor=white)
7. **Grafana** ![Grafana](https://img.shields.io/badge/Grafana-F46800?style=flat-square&logo=grafana&logoColor=white)
8. **ArgoCD** ![ArgoCD](https://img.shields.io/badge/ArgoCD-EF7B4D?style=flat-square&logo=argo&logoColor=white)
9. **OWASP** ![OWASP](https://img.shields.io/badge/OWASP-000000?style=flat-square&logo=owasp&logoColor=white)
10. **Trivy** ![Trivy](https://img.shields.io/badge/Trivy-00979D?style=flat-square&logo=trivy&logoColor=white)

---

### 📌 **Project Stages**

- **Stage 1** — Build, scan, and deploy the app as a Docker container
- **Stage 2** — Deploy the app to an Amazon EKS Kubernetes cluster
- **Stage 3** — Configure SonarQube quality checks with Jenkins integration
- **Stage 4** — Create a monitoring server using Prometheus and Grafana
- **Stage 5** — Monitor EC2 servers, Jenkins, and Kubernetes workloads

---


### 📂 GitHub Repo Link: [**Zomato Clone DevOps Project**](#)

---

## Connect with me on LinkedIn: [![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=flat-square&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/nikhil-jadhav-ab6b0021a/)

---

## 📚 **Table of Contents**

- [Lab: Install Jenkins Controller on EC2](#lab-install-jenkins-controller-on-ec2)
- [Lab: Configure SSH-based Jenkins Agent on EC2](#lab-configure-ssh-based-jenkins-agent-on-ec2)
- [Lab: Install SonarQube on EC2](#lab-install-sonarqube-on-ec2)
- [Lab: Configure SonarQube for Jenkins](#lab-configure-sonarqube-for-jenkins)
- [Lab: Install AWS CLI, Docker Engine, Trivy, and Jenkins Plugins](#lab-install-aws-cli-docker-engine-trivy-and-jenkins-plugins)
- [Lab: Create Monitoring Server with Prometheus and Grafana](#lab-create-monitoring-server-with-prometheus-and-grafana)
- [Lab: Install Node Exporter on Linux Servers](#lab-install-node-exporter-on-linux-servers)
- [Lab: Monitor Jenkins Using Prometheus and Grafana](#lab-monitor-jenkins-using-prometheus-and-grafana)
- [Lab: Install kubectl on Linux](#lab-install-kubectl-on-linux)
- [Lab: Install eksctl on Linux](#lab-install-eksctl-on-linux)
- [Lab: Create Amazon EKS Cluster and Node Group](#lab-create-amazon-eks-cluster-and-node-group)
- [Lab: Monitor Kubernetes Using kube-prometheus-stack](#lab-monitor-kubernetes-using-kube-prometheus-stack)
- [What You Should Not Do](#what-you-should-not-do)
- [Conclusion](#conclusion)
- [References](#references)

---

# **Lab: Install Jenkins Controller on EC2**

This VM acts only as the **Jenkins controller**. All build workloads should run on Jenkins agents.

## **1. Create the EC2 instance**

- **AMI:** Ubuntu 24.04 LTS
- **Instance type:** c7i-flex.large or similar
- **Root volume:** 20 GB or more

**Security Group — Jenkins Controller**

| Port | Purpose | Source |
|---|---|---|
| 22 | SSH | Your IP |
| 8080 | Jenkins UI | Your IP |

## **2. Connect to the instance**

```bash
ssh -i <your-key>.pem ubuntu@<jenkins-controller-public-ip>
```

Secure the private key:

```bash
chmod 600 ~/.ssh/<your-private-key>
chown $(whoami):$(whoami) ~/.ssh/<your-private-key>
```

Set hostname and timezone:

```bash
sudo hostnamectl set-hostname jenkins-controller
exec bash

sudo timedatectl set-timezone Asia/Kolkata
timedatectl status
```

## **3. Install Java 21**

```bash
sudo apt update
sudo apt install -y openjdk-21-jdk
java -version
javac -version
```

## **4. Install Jenkins**

```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key

echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/" | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install -y jenkins
```

Set Jenkins JVM timezone:

```bash
sudo mkdir -p /etc/systemd/system/jenkins.service.d
cat <<'EOF' | sudo tee /etc/systemd/system/jenkins.service.d/override.conf > /dev/null
[Service]
Environment="JAVA_OPTS=-Duser.timezone=Asia/Kolkata"
EOF
```

## **5. Start and enable Jenkins**

```bash
sudo systemctl daemon-reload
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```

## **6. Access Jenkins UI**

Open:

```text
http://<jenkins-controller-public-ip>:8080
```

Get the initial password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Then:

- Install suggested plugins
- Create the Jenkins admin user
- Log in to the Jenkins dashboard

---

# **Lab: Configure SSH-based Jenkins Agent on EC2**

The Jenkins agent executes build jobs and contains tools such as Docker, Trivy, AWS CLI, kubectl, and eksctl.

## **1. Create Jenkins Agent EC2 instance**

- **AMI:** Ubuntu 24.04 LTS
- **Instance type:** c7i-flex.large or similar
- **Root volume:** 30 GB or more

**Security Group — Jenkins Agent**

| Port | Purpose | Source |
|---|---|---|
| 22 | SSH from Jenkins controller | Jenkins controller private IP |

## **2. Connect to the Jenkins Agent**

```bash
ssh -i <your-key>.pem ubuntu@<jenkins-agent-public-ip>
```

Set hostname and timezone:

```bash
sudo hostnamectl set-hostname jenkins-agent
exec bash

sudo timedatectl set-timezone Asia/Kolkata
timedatectl status
```

## **3. Install Java 21**

```bash
sudo apt update
sudo apt install -y openjdk-21-jdk
java -version
javac -version
```

## **4. Create Jenkins user on Agent**

```bash
sudo useradd -m -s /bin/bash jenkins
```

---

# **Lab: Add Jenkins Agent to Jenkins Controller**

## **1. Generate SSH key on Jenkins Controller**

Run on the **Jenkins controller**:

```bash
sudo su - jenkins
ssh-keygen -t ed25519 -f /var/lib/jenkins/.ssh/jenkins-agent-key -C "jenkins-agent-access"
cat /var/lib/jenkins/.ssh/jenkins-agent-key.pub
```

## **2. Copy public key to Jenkins Agent**

Run on the **Jenkins agent**:

```bash
sudo su - jenkins
mkdir -p ~/.ssh
vim ~/.ssh/authorized_keys
```

Paste the public key, then set permissions:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

## **3. Test SSH from Controller to Agent**

Run on the controller as the `jenkins` user:

```bash
sudo su - jenkins
ssh -i /var/lib/jenkins/.ssh/jenkins-agent-key jenkins@<jenkins-agent-private-ip> hostname
```

Expected output:

```text
jenkins-agent
```

## **4. Configure Agent in Jenkins UI**

Navigate to:

```text
Jenkins Dashboard → Manage Jenkins → Nodes → New Node
```

Use the following configuration:

- **Node name:** jenkins-agent
- **Type:** Permanent Agent
- **Remote root directory:** `/home/jenkins`
- **Labels:** `docker-maven-trivy`
- **Launch method:** Launch agents via SSH
- **Host:** `<jenkins-agent-private-ip>`
- **Credentials:** SSH Username with private key
- **Username:** `jenkins`
- **Private key:** `/var/lib/jenkins/.ssh/jenkins-agent-key` content
- **Host Key Verification Strategy:** Known hosts file Verification Strategy

Populate Jenkins known hosts:

```bash
ssh-keyscan <jenkins-agent-private-ip> | sudo -u jenkins tee -a /var/lib/jenkins/.ssh/known_hosts > /dev/null
```

Click **Save**, then **Launch agent**.

---

# **Lab: Install SonarQube on EC2**

SonarQube is used for static code analysis, code smells, bugs, vulnerabilities, duplication checks, quality profiles, and quality gates.

For this lab, SonarQube and PostgreSQL are installed on the same VM.

## **1. Create SonarQube EC2 instance**

- **AMI:** Ubuntu 22.04 LTS or Ubuntu 24.04 LTS
- **Instance type:** c7i-flex.large or higher
- **Root volume:** 20 GB or more

**Security Group — SonarQube**

| Port | Purpose | Source |
|---|---|---|
| 22 | SSH | Your IP |
| 9000 | SonarQube UI | Your IP and Jenkins controller/agent |

## **2. Connect to the instance**

```bash
ssh -i <your-key>.pem ubuntu@<sonarqube-public-ip>
```

Set hostname and timezone:

```bash
sudo hostnamectl set-hostname sonarqube
exec bash

sudo timedatectl set-timezone Asia/Kolkata
timedatectl status
```

## **3. Install Java 21**

```bash
sudo apt-get update
sudo apt-get install -y openjdk-21-jre
java -version
```

## **4. Install PostgreSQL and create SonarQube database**

```bash
sudo apt install -y postgresql postgresql-contrib
sudo systemctl enable postgresql
sudo systemctl start postgresql
```

Open PostgreSQL shell:

```bash
sudo -u postgres psql
```

Run:

```sql
CREATE ROLE sonar WITH LOGIN ENCRYPTED PASSWORD 'StrongPasswordHere';
CREATE DATABASE sonarqube OWNER sonar;
GRANT ALL PRIVILEGES ON DATABASE sonarqube TO sonar;
\q
```

Verify:

```bash
sudo -u postgres psql -c "\du"
sudo -u postgres psql -c "\l"
PGPASSWORD='StrongPasswordHere' psql -U sonar -h localhost -d sonarqube -c "\dt"
```

## **5. Tune kernel settings**

```bash
echo 'vm.max_map_count=524288' | sudo tee -a /etc/sysctl.d/99-sonarqube.conf
echo 'fs.file-max=131072' | sudo tee -a /etc/sysctl.d/99-sonarqube.conf
sudo sysctl --system
```

Set limits for the `sonar` user:

```bash
echo 'sonar   -   nofile   131072' | sudo tee /etc/security/limits.d/99-sonarqube.conf
echo 'sonar   -   nproc    8192'   | sudo tee -a /etc/security/limits.d/99-sonarqube.conf
```

## **6. Create SonarQube user**

```bash
sudo useradd -m -d /opt/sonarqube -s /bin/bash sonar
```

## **7. Download and install SonarQube**

> Check the official SonarQube download page for the latest LTA or current release before using the URL below.

```bash
cd /tmp
curl -L -o sonarqube.zip "https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-26.6.0.123539.zip"

sudo apt-get update
sudo apt-get install -y unzip
unzip sonarqube.zip

sudo mv sonarqube-26.6.0.123539 /opt/sonarqube-current
sudo chown -R sonar:sonar /opt/sonarqube-current
```

## **8. Configure SonarQube database connection**

```bash
sudo -u sonar vim /opt/sonarqube-current/conf/sonar.properties
```

Set:

```properties
sonar.jdbc.username=sonar
sonar.jdbc.password=StrongPasswordHere
sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube
```

## **9. Create systemd service for SonarQube**

```bash
sudo vi /etc/systemd/system/sonarqube.service
```

Paste:

```ini
[Unit]
Description=SonarQube service
After=network.target postgresql.service

[Service]
Type=forking
User=sonar
Group=sonar
ExecStart=/opt/sonarqube-current/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube-current/bin/linux-x86-64/sonar.sh stop
Restart=on-failure
LimitNOFILE=131072
LimitNPROC=8192

[Install]
WantedBy=multi-user.target
```

Start SonarQube:

```bash
sudo systemctl daemon-reload
sudo systemctl enable sonarqube
sudo systemctl start sonarqube
sudo systemctl status sonarqube
```

Check logs if required:

```bash
sudo tail -n 200 /opt/sonarqube-current/logs/sonar.log
sudo tail -n 200 /opt/sonarqube-current/logs/web.log
sudo tail -n 200 /opt/sonarqube-current/logs/es.log
```

## **10. Access SonarQube UI**

Open:

```text
http://<sonarqube-public-ip>:9000
```

Default credentials:

```text
Username: admin
Password: admin
```

You will be asked to change the password after first login.

---

# **Lab: Configure SonarQube for Jenkins**

This section connects SonarQube with Jenkins so Jenkins can run code analysis and wait for the SonarQube quality gate result.

## **1. Create a SonarQube user for Jenkins**

Log in to SonarQube as an administrator.

Navigate to:

```text
Administration → Security → Users → Create User
```

Create a dedicated CI user:

| Field | Value |
|---|---|
| Login | `jenkins` |
| Name | `Jenkins CI` |
| Email | `jenkins@example.com` |
| Password | Use a strong password |

## **2. Add Jenkins user to the administrators group**

Navigate to:

```text
Administration → Security → Groups
```

Open the group:

```text
sonar-administrators
```

Add the `jenkins` user to this group.

> For production, prefer least-privilege permissions instead of full administrator access. For a lab, administrator access is acceptable for initial setup.

## **3. Create SonarQube token for Jenkins**

Log in as the `jenkins` SonarQube user, then navigate to:

```text
My Account → Security → Generate Tokens
```

Create a token:

| Field | Value |
|---|---|
| Token name | `jenkins-token` |
| Type | User token |
| Expiration | Set as per your organization policy |

Copy the token immediately. It will be shown only once.

## **4. Add SonarQube token in Jenkins credentials**

In Jenkins, navigate to:

```text
Manage Jenkins → Credentials → System → Global credentials → Add Credentials
```

Use:

| Field | Value |
|---|---|
| Kind | Secret text |
| Secret | `<sonarqube-token>` |
| ID | `sonarqube-token` |
| Description | `SonarQube token for Jenkins` |

## **5. Configure SonarQube server in Jenkins**

Navigate to:

```text
Manage Jenkins → System → SonarQube servers
```

Add:

| Field | Value |
|---|---|
| Name | `sonarqube-server` |
| Server URL | `http://<sonarqube-private-ip>:9000` |
| Server authentication token | `sonarqube-token` |

Save the configuration.

## **6. Create SonarQube webhook for Jenkins controller**

In SonarQube, navigate to:

```text
Administration → Configuration → Webhooks → Create
```

Create webhook:

| Field | Value |
|---|---|
| Name | `jenkins-webhook` |
| URL | `http://<jenkins-controller-public-or-private-ip>:8080/sonarqube-webhook/` |
| Secret | Optional, but recommended for production |

> The trailing slash `/` at the end of `/sonarqube-webhook/` is required for Jenkins quality gate callbacks.

## **7. Example Jenkins pipeline SonarQube quality gate stage**

```groovy
stage('Build and SonarQube Analysis') {
    steps {
        withSonarQubeEnv('sonarqube-server') {
            sh 'mvn clean verify sonar:sonar'
        }
    }
}

stage('Quality Gate') {
    steps {
        timeout(time: 1, unit: 'HOURS') {
            waitForQualityGate abortPipeline: true
        }
    }
}
```

---

# **Lab: Install AWS CLI, Docker Engine, Trivy, and Jenkins Plugins**

Run these steps on the **Jenkins agent**.

## **1. Install AWS CLI v2**

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip
sudo apt install -y unzip
unzip awscliv2.zip
sudo ./aws/install

aws --version
sudo -u jenkins aws --version
```

## **2. Install Docker Engine**

```bash
sudo apt remove docker docker-engine docker.io containerd runc -y
sudo apt update
sudo apt install -y ca-certificates curl gnupg

sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo usermod -aG docker jenkins
sudo usermod -aG docker ubuntu

docker --version
sudo -u jenkins docker --version
```

> Reconnect the Jenkins agent after adding the `jenkins` user to the `docker` group.

## **3. Install Trivy**

```bash
sudo apt-get install -y wget
wget https://github.com/aquasecurity/trivy/releases/download/v0.71.2/trivy_0.71.2_Linux-64bit.deb
sudo dpkg -i trivy_0.71.2_Linux-64bit.deb
trivy --version
```

## **4. Install Docker Scout**

```bash
docker login -u <docker-username>

sudo su
curl -sSfL https://raw.githubusercontent.com/docker/scout-cli/main/install.sh | sh -s -- -b /usr/local/bin
exit

docker scout version
```

> Avoid using `chmod 777 /var/run/docker.sock` in production. Add trusted users to the `docker` group instead.

## **5. Install required Jenkins plugins**

Navigate to:

```text
Manage Jenkins → Plugins → Available Plugins
```

Install:

- Pipeline: Stage View
- Eclipse Temurin Installer
- SonarQube Scanner
- Docker
- Docker Commons
- Docker Pipeline
- Docker API
- Docker Build Step
- Email Extension Template
- OWASP Dependency-Check
- NodeJS
- Prometheus Metrics

Restart Jenkins after plugin installation if prompted.

---

# **Lab: Create Monitoring Server with Prometheus and Grafana**

This server collects metrics from EC2 instances, Jenkins, and optionally federated Kubernetes Prometheus.

## **1. Create Monitoring EC2 instance**

- **AMI:** Ubuntu 24.04 LTS
- **Instance type:** t3.medium or higher
- **Root volume:** 20 GB or more

**Security Group — Monitoring Server**

| Port | Purpose | Source |
|---|---|---|
| 22 | SSH | Your IP |
| 9090 | Prometheus UI | Your IP / internal network |
| 3000 | Grafana UI | Your IP / internal network |
| 9100 | Node Exporter | Prometheus server only |

## **2. Create Prometheus system user**

Use lowercase `prometheus` consistently for the Linux user and paths.

```bash
sudo useradd --system --no-create-home --shell /bin/false prometheus
```

## **3. Download Prometheus**

```bash
cd /tmp
wget https://github.com/prometheus/prometheus/releases/download/v3.12.0/prometheus-3.12.0.linux-amd64.tar.gz

tar -xvf prometheus-3.12.0.linux-amd64.tar.gz
cd prometheus-3.12.0.linux-amd64
```

## **4. Move binaries and create directories**

```bash
sudo mv prometheus promtool /usr/local/bin/
sudo mkdir -p /etc/prometheus
sudo mkdir -p /var/lib/prometheus
```

## **5. Copy Prometheus configuration**

```bash
sudo cp prometheus.yml /etc/prometheus/
```

## **6. Set permissions**

```bash
sudo chown -R prometheus:prometheus /etc/prometheus
sudo chown -R prometheus:prometheus /var/lib/prometheus
sudo chown prometheus:prometheus /usr/local/bin/prometheus /usr/local/bin/promtool
```

## **7. Create Prometheus systemd service**

```bash
sudo nano /etc/systemd/system/prometheus.service
```

Paste:

```ini
[Unit]
Description=Prometheus
After=network.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus \
  --web.enable-lifecycle
Restart=always

[Install]
WantedBy=multi-user.target
```

## **8. Start Prometheus**

```bash
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus
sudo systemctl status prometheus
```

Open:

```text
http://<monitoring-server-public-ip>:9090
```

---

# **Lab: Install Node Exporter on Linux Servers**

Install Node Exporter on each Linux host you want to monitor, such as Jenkins controller, Jenkins agent, SonarQube server, and monitoring server.

## **1. Create Node Exporter user**

```bash
sudo useradd --system --no-create-home --shell /usr/sbin/nologin node_exporter
```

## **2. Download Node Exporter**

```bash
cd /tmp
wget https://github.com/prometheus/node_exporter/releases/download/v1.11.1/node_exporter-1.11.1.linux-amd64.tar.gz
```

## **3. Extract archive**

```bash
tar -xvf node_exporter-1.11.1.linux-amd64.tar.gz
cd node_exporter-1.11.1.linux-amd64/
```

## **4. Move binary**

```bash
sudo mv node_exporter /usr/local/bin/
```

## **5. Set ownership**

```bash
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

## **6. Create Node Exporter service**

```bash
sudo vim /etc/systemd/system/node_exporter.service
```

Paste:

```ini
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter
Restart=always

[Install]
WantedBy=multi-user.target
```

## **7. Start Node Exporter**

```bash
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
sudo systemctl status node_exporter
```

Verify metrics:

```bash
curl http://localhost:9100/metrics | head
```

## **8. Add Node Exporter targets in Prometheus**

Edit Prometheus configuration on the monitoring server:

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Example:

```yaml
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node_exporter"
    static_configs:
      - targets:
          - "<monitoring-server-private-ip>:9100"
          - "<jenkins-controller-private-ip>:9100"
          - "<jenkins-agent-private-ip>:9100"
          - "<sonarqube-private-ip>:9100"
```

Validate and restart:

```bash
promtool check config /etc/prometheus/prometheus.yml
sudo systemctl restart prometheus
```

---

# **Lab: Install Grafana**

Run these steps on the **monitoring server**.

## **1. Install prerequisites**

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https wget gnupg
```

## **2. Import Grafana GPG key**

```bash
sudo mkdir -p /etc/apt/keyrings
sudo wget -O /etc/apt/keyrings/grafana.asc https://apt.grafana.com/gpg-full.key
sudo chmod 644 /etc/apt/keyrings/grafana.asc
```

## **3. Add Grafana stable repository**

```bash
echo "deb [signed-by=/etc/apt/keyrings/grafana.asc] https://apt.grafana.com stable main" | \
  sudo tee /etc/apt/sources.list.d/grafana.list
```

> Do not add the beta repository unless you specifically want beta packages.

## **4. Install Grafana OSS**

```bash
sudo apt-get update
sudo apt-get install -y grafana
```

## **5. Start Grafana server**

```bash
sudo systemctl daemon-reload
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
sudo systemctl status grafana-server
```

Open:

```text
http://<monitoring-server-public-ip>:3000
```

Default login:

```text
Username: admin
Password: admin
```

Change the password after first login.

## **6. Add Prometheus data source**

In Grafana UI:

```text
Connections → Data Sources → Add data source → Prometheus
```

Use:

```text
URL: http://<monitoring-server-private-ip>:9090
```

Click **Save & Test**.

## **7. Import dashboards**

Go to:

```text
Dashboards → New → Import
```

Import these dashboard IDs:

| Dashboard ID | Purpose |
|---|---|
| 1860 | Node Exporter Full |
| 9964 | Jenkins Performance and Health Overview |

---

# **Lab: Monitor Jenkins Using Prometheus and Grafana**

## **1. Install Jenkins Prometheus Metrics plugin**

In Jenkins:

```text
Manage Jenkins → Plugins → Available Plugins
```

Install:

```text
Prometheus Metrics
```

Restart Jenkins if required.

## **2. Verify Jenkins metrics endpoint**

Open in browser:

```text
http://<jenkins-controller-ip>:8080/prometheus
```

You should see Prometheus metrics output.

## **3. Add Jenkins target in Prometheus**

Edit Prometheus config:

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Add:

```yaml
  - job_name: "jenkins"
    metrics_path: "/prometheus"
    static_configs:
      - targets:
          - "<jenkins-controller-private-ip>:8080"
```

Validate and restart:

```bash
promtool check config /etc/prometheus/prometheus.yml
sudo systemctl restart prometheus
```

## **4. Import Jenkins Grafana dashboard**

In Grafana:

```text
Dashboards → New → Import → Dashboard ID: 9964
```

Select the Prometheus data source and click **Import**.

---

# **Lab: Install kubectl on Linux**

Run on the Jenkins agent or any machine from where you will manage Kubernetes.

## **1. Download latest kubectl release**

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

## **2. Validate binary checksum**

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
```

Expected output:

```text
kubectl: OK
```

## **3. Install kubectl**

```bash
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

## **4. Verify installation**

```bash
kubectl version --client
kubectl version --client --output=yaml
```

---

# **Lab: Install eksctl on Linux**

## **1. Download eksctl**

```bash
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH

curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
```

## **2. Verify checksum**

```bash
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | \
  grep $PLATFORM | sha256sum --check
```

## **3. Install eksctl**

```bash
tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
sudo install -m 0755 /tmp/eksctl /usr/local/bin && rm /tmp/eksctl
```

## **4. Verify installation**

```bash
eksctl version
```

---

# **Lab: Create Amazon EKS Cluster and Node Group**

> EKS cluster creation normally takes **20–25 minutes**.

For consistency, this README uses:

```text
Cluster name: aws-cluster
Region: us-east-1
Node group: demo-ng-public1
SSH key name: demo
```

## **1. Create EKS cluster**

```bash
eksctl create cluster \
  --name aws-cluster \
  --region us-east-1 \
  --without-nodegroup
```

Verify cluster:

```bash
eksctl get cluster --region us-east-1
kubectl get svc
```

You can also verify stack creation in:

```text
AWS Console → CloudFormation
```

## **2. Associate IAM OIDC provider**

```bash
eksctl utils associate-iam-oidc-provider \
  --region us-east-1 \
  --cluster aws-cluster \
  --approve
```

## **3. Create managed node group in public subnets**

```bash
eksctl create nodegroup --cluster=aws-cluster \
  --region=us-east-1 \
  --name=demo-ng-public1 \
  --node-type=m5.large \
  --nodes=2 \
  --nodes-min=2 \
  --nodes-max=4 \
  --node-volume-size=80 \
  --ssh-access \
  --ssh-public-key=demo \
  --managed \
  --asg-access \
  --external-dns-access \
  --full-ecr-access \
  --appmesh-access \
  --alb-ingress-access
```

## **4. Verify nodes**

```bash
kubectl get nodes
```

---

# **Lab: Monitor Kubernetes Using kube-prometheus-stack**

This setup deploys Prometheus, Grafana, Alertmanager, Node Exporter, kube-state-metrics, and Prometheus Operator inside Kubernetes using Helm.

## **1. Verify cluster access**

Run on the Jenkins agent or admin machine:

```bash
kubectl get nodes
```

## **2. Install Helm**

Use the official Helm install script.

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4
chmod 700 get_helm.sh
./get_helm.sh
helm version
```

> If your environment requires Helm 3 specifically, use `get-helm-3` instead of `get-helm-4`.

## **3. Add Prometheus Community Helm repository**

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

## **4. Create monitoring namespace**

```bash
kubectl create namespace monitoring
```

If the namespace already exists, use:

```bash
kubectl get namespace monitoring
```

## **5. Install kube-prometheus-stack**

```bash
helm install kps prometheus-community/kube-prometheus-stack --namespace monitoring
```

## **6. Verify installation**

```bash
kubectl get pods -n monitoring
kubectl get svc -n monitoring
```

## **7. Expose Kubernetes Prometheus**

Check the Prometheus service name:

```bash
kubectl get svc -n monitoring | grep prometheus
```

Patch the Prometheus service to LoadBalancer:

```bash
kubectl patch svc kps-kube-prometheus-stack-prometheus \
  -n monitoring \
  -p '{"spec":{"type":"LoadBalancer"}}'
```

Get external endpoint:

```bash
kubectl get svc -n monitoring
```

Example:

```text
EXTERNAL-IP = a1b2c3.amazonaws.com
```

## **8. Connect Kubernetes Prometheus to external Grafana**

In external Grafana running on the monitoring EC2:

```text
Connections → Data Sources → Add data source → Prometheus
```

Add:

```text
Name: k8s-prometheus
URL: http://<external-ip-or-dns>:9090
```

Click **Save & Test**.

## **9. Import Kubernetes dashboards**

In Grafana:

```text
Dashboards → New → Import
```

Import:

| Dashboard ID | Purpose |
|---|---|
| 315 | Kubernetes cluster monitoring via Prometheus |
| 6417 | Kubernetes cluster / container summary metrics |

## **10. Optional: Federate Kubernetes Prometheus into central Prometheus**

On the monitoring EC2, edit:

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Add:

```yaml
  - job_name: "kubernetes-federation"
    metrics_path: "/federate"
    params:
      'match[]':
        - '{job=~".+"}'
    static_configs:
      - targets:
          - "<k8s-prometheus-dns-or-ip>:9090"
```

Validate and restart:

```bash
promtool check config /etc/prometheus/prometheus.yml
sudo systemctl restart prometheus
```

---

# **What You Should Not Do**

❌ **Do not manually configure Kubernetes node public IPs with `nodeIP:9100`.**

Kubernetes nodes are dynamic. Nodes can be replaced, scaled, or recreated, so hardcoding node IPs is not reliable.

❌ **Do not copy EC2 public IPs for EKS worker nodes into Prometheus.**

Use Kubernetes-native discovery through kube-prometheus-stack, ServiceMonitor, PodMonitor, or Prometheus Operator.

❌ **Do not rely only on standalone Node Exporter for Kubernetes monitoring.**

Node Exporter gives Linux host metrics, but Kubernetes monitoring also needs metrics from kube-state-metrics, kubelet/cAdvisor, API server, scheduler, controller manager, and workload-level metrics.

❌ **Do not expose Prometheus, Grafana, Jenkins, or SonarQube openly to the internet in production.**

Use HTTPS, private networking, security groups, VPN, authentication, and least-privilege access.

❌ **Do not use `chmod 777` on Docker socket in production.**

Use the `docker` group carefully and only for trusted users because Docker access is effectively root-level access.

---

# **Conclusion**

This project demonstrates a practical end-to-end DevSecOps workflow for deploying a Zomato Clone application using Jenkins, SonarQube, Docker, Amazon EKS, Prometheus, and Grafana.

By completing this setup, you will have:

- A Jenkins controller and SSH-based Jenkins agent
- SonarQube integrated with Jenkins using token authentication and webhook callbacks
- Docker, Trivy, Docker Scout, AWS CLI, kubectl, and eksctl on the Jenkins agent
- An Amazon EKS cluster with managed node groups
- A Prometheus and Grafana monitoring server
- Node Exporter-based EC2 monitoring
- Jenkins metrics monitoring
- Kubernetes monitoring using kube-prometheus-stack
- Grafana dashboards for infrastructure, Jenkins, and Kubernetes visibility

This setup is suitable for learning, demos, and portfolio projects. For production, add HTTPS, IAM hardening, private networking, backup strategy, alerting, persistent volumes, secret management, and infrastructure as code.

---

# **References**

- Jenkins Linux installation documentation
- SonarQube Server documentation
- SonarQube Scanner for Jenkins documentation
- Prometheus download documentation
- Node Exporter GitHub releases
- Grafana installation documentation
- Kubernetes kubectl installation documentation
- eksctl installation documentation
- Helm installation documentation
- prometheus-community kube-prometheus-stack Helm chart
- Grafana dashboard marketplace
