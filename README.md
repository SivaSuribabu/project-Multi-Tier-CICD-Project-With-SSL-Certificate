# project-Multi-Tier-CICD-Project-With-SSL-Certificate

**High-level flow**

1.Create a central VM (Ubuntu 24.04 LTS) to run Terraform & tools.

2.Install AWS CLI, configure aws configure with access key + region (ap-south-1 in video).

3.Install Terraform and use the repo’s Terraform EKS module (eks terraform folder). terraform init → terraform plan → terraform apply -auto-approve.

4.Spin up three more EC2s (Docker containers used on these) for: Nexus, SonarQube, Jenkins (Jenkins uses a larger instance).

5.Setup Nexus & SonarQube via Docker containers (pull images, map ports). Retrieve initial passwords inside container (cat admin.password).

6.Install Java and Jenkins; install recommended plugins + extra plugins (Sonar, Maven, Docker, Kubernetes plugin, config file provider, pipeline Maven integration, SonarQube Scanner plugin).

7.Install Docker and Trivy on Jenkins host (Trivy used to scan file system and images).

8.Create a Jenkins pipeline with stages: Checkout → Compile → Tests (or skip with -DskipTests=true) → Trivy FS scan → SonarQube analysis → Build (mvn package) → Publish artifact to Nexus (configure settings.xml with Nexus repos and credentials) → Docker build → Trivy image scan → Push image to Docker Hub → Deploy to EKS (kubectl).

9.Setup RBAC in cluster for Jenkins: create namespace (web-apps), create serviceaccount, Role (verbs) and RoleBinding, then create a secret token for Jenkins to authenticate to cluster. Configure Kubernetes credentials in Jenkins (API endpoint, token, cluster name, namespace).

10.Deploy MySQL Kubernetes manifest (Deployment + Service) and app Deployment that uses the MySQL service name in datasource URL. Verify pods and services.

11.Map custom domain to ALB: add a CNAME (www) pointing to the load balancer DNS (use A record if you have a static IP).

12.Use Cloudflare (free tier) to add domain, change nameservers in GoDaddy to Cloudflare’s, enable “Always use HTTPS” (edge certificates). After propagation, site shows HTTPS / secure
