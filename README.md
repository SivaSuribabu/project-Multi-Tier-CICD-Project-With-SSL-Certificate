# "project-Multi-Tier-CICD-Project-With-SSL-Certificate"

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




**Key files / places mentioned**

1.app properties: src/main/resources/application.properties — database credentials & datasource URL (service name) used by app.

2.Terraform folder: eks terraform inside the GitHub repo (variables for region, AZs, private key name).

3.Dockerfile (if missing create it) — example uses JDK 17 base, copies target/*.jar and exposes 8080.

4.Jenkins settings.xml (managed file) — contains Nexus repo URLs + credentials (server id = repo name).

5.Kubernetes manifests: mysql-deployment.yaml, ds.yaml (app), svc.yaml, and RBAC files: svc.yaml, role.yaml, bind.yaml, sec.yaml.


**Most important commands**

**AWS configure:**

aws configure


**Terraform:**

terraform init
terraform plan
terraform apply -auto-approve


**Clone repo:**

git clone <repo-url>
cd <repo>/eks-terraform


**Create EKS kubeconfig:**

aws eks --region <region> update-kubeconfig --name <cluster-name>


**Kubectl apply:**

kubectl apply -f <manifest>.yaml -n web-apps


**To view pods/services:**

kubectl get pods -n web-apps
kubectl get svc -n web-apps
kubectl logs <pod> -n web-apps -f


**Docker run (Nexus example):**

sudo docker run -d --name nexus3 -p 8081:8081 sonatype/nexus3


**Access Jenkins initial admin password:**

sudo cat /var/lib/jenkins/secrets/initialAdminPassword


**Skip tests in Maven:**

mvn package -DskipTests=true


**Trivy:**

trivy fs --format template --template "@contrib/html.tpl" -o fs-report.html .
trivy image <image-name>


**Common pitfalls & troubleshooting**

1.Region / AZ mismatch in Terraform variables — set region and AZs correctly.

2.Private key name must match the keypair name in AWS (set in Terraform variables).

3.Missing Dockerfile in repo caused Docker build failure — add/copy Dockerfile into repo.

4.Tools missing on Jenkins host (kubectl, trivy) — install them on Jenkins VM, otherwise pipeline will fail.

5.Test failures: use -DskipTests=true for demo; for production fix tests instead of skipping.

6.Name server propagation: Cloudflare nameserver changes can take minutes → wait and verify with DNS checker / nslookup.

7.Service connection: app should connect to DB using Kubernetes service name, not pod IP.
