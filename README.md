Part 1: Prerequisites Installation (Windows)
Step 1: Install VS Code
Open browser and go to: https://code.visualstudio.com/
Click "Download for Windows"
Run the downloaded installer (VSCodeSetup.exe)
During installation:
make sure
 Check "Add to PATH"
Click "Next" → "Install"
Click "Finish" to launch VS Code
Verify Installation:

Step 2: Install Docker Desktop
Go to: https://www.docker.com/products/docker-desktop
Click "Download for Windows"
Run the installer (Docker Desktop Installer.exe)
During installation:
Check "Use WSL 2 instead of Hyper-V" (recommended)
Click "OK"
Restart your computer when prompted
After restart, launch Docker Desktop from Start Menu
Wait for Docker to start (you'll see "Docker Desktop is running" in system tray)
Accept the Docker Subscription Service Agreement
Verify Installation:

docker --version
docker run hello-world
Expected Output:
Hello from Docker!
This message shows that your installation appears to be working correctly.

Step 3: Install kubectl
Open PowerShell as Administrator (Right-click Start Menu → Windows PowerShell (Admin))
Run the following commands:
powershell

# Download kubectl
curl.exe -LO "https://dl.k8s.io/release/v1.28.0/bin/windows/amd64/kubectl.exe"

# Move to System32 folder
Move-Item .\kubectl.exe C:\Windows\System32\

# Close and reopen PowerShell
Verify Installation:

kubectl version --client
Step 4: Install Minikube
Open PowerShell as Administrator
Run the following commands:
powershell


# Create minikube directory
New-Item -Path 'c:\' -Name 'minikube' -ItemType Directory -Force

# Download minikube
Invoke-WebRequest -OutFile 'c:\minikube\minikube.exe' -Uri 'https://github.com/kubernetes/minikube/releases/latest/download/minikube-windows-amd64.exe' -UseBasicParsing

# Add to PATH
$oldPath = [Environment]::GetEnvironmentVariable('Path', [EnvironmentVariableTarget]::Machine)
if ($oldPath.Split(';') -inotcontains 'C:\minikube'){
  [Environment]::SetEnvironmentVariable('Path', $('{0};C:\minikube' -f $oldPath), [EnvironmentVariableTarget]::Machine)
}

# Close and reopen PowerShell
Verify Installation:
minikube version

Step 5: Create Accounts
GitHub Account:
Go to: https://github.com/signup
Complete the setup
Docker Hub Account:
Go to: https://hub.docker.com/signup
Enter:
Docker ID (username - remember this!)
Login to Docker Hub
Login to Docker Hub from Command Line:

docker login
Enter your Docker Hub username
Enter your Docker Hub password

Login Succeeded

Part 2: Start Minikube Cluster

Step 1: Start Minikube
Open Command Prompt (not PowerShell):

minikube start --driver=docker


Expected Output:

😄  minikube v1.32.0 on Windows 10
✨  Using the docker driver based on user configuration
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🐳  Preparing Kubernetes v1.28.3 on Docker 24.0.7 ...
🔎  Verifying Kubernetes components...
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster
Step 2: Verify Cluster
minikube status

minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured

kubectl get nodes
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   2m    v1.28.3

Part 3: Clone Repository & Build Docker Images
Step 1: Clone the Repository

# Create working directory
mkdir C:\kubernetes-lab
cd C:\kubernetes-lab

# Clone repository
git clone https://github.com/benjamin-stack/VIIT--demo.git

# Navigate to project
cd VIIT--demo

# Switch to dev branch
git checkout dev

# View files
dir
Step 2: Build Frontend Image

# Navigate to frontend directory
cd frontend

# View Dockerfile
type Dockerfile

# Build image (Replace 'yourdockerhubusername' with YOUR Docker Hub username)

docker build -t test/viit-frontend:v1 .

docker images | findstr viit-frontend
Push to Docker Hub:

docker push test/viit-frontend:v1
Go back to root:

cd ..
Step 3: Build Backend Java Image

# Navigate to backend-java directory
cd backend-java

# View Dockerfile
type Dockerfile

# Build image
docker build -t test/viit-backend-java:v1 .

# Verify image
docker images | findstr viit-backend-java

# Push to Docker Hub
docker push test/viit-backend-java:v1

# Go back to root
cd ..
Step 4: Build Backend Python Image

# Navigate to backend-python directory
cd backend-python

# View Dockerfile
type Dockerfile

# Build image
docker build -t test/viit-backend-python:v1 .

# Verify image
docker images | findstr viit-backend-python

# Push to Docker Hub
docker push test/viit-backend-python:v1

# Go back to root
cd ..
Step 5: Verify All Images

# List all images
docker images | findstr viit
Expected Output:

Copy code

test/viit-frontend         v1    abc123    2m ago   50MB
test/viit-backend-java     v1    def456    5m ago   200MB
test/viit-backend-python   v1    ghi789    8m ago   150MB

Part 4: Create Kubernetes Manifests
Step 1: Create Manifests Directory

# Create directory
mkdir C:\kubernetes-lab\manifests
cd C:\kubernetes-lab\manifests
Step 2: Create Frontend Manifest
Open VS Code:

code frontend.yml
Copy and paste this content:

# Check syntax
kubectl apply --dry-run=client -f frontend.yml
kubectl apply --dry-run=client -f backend-java.yml
kubectl apply --dry-run=client -f backend-python.yml
Part 5: Deploy to Kubernetes
Step 1: Deploy Frontend
cmd
Copy code

# Deploy
kubectl apply -f frontend.yml

# Check deployment
kubectl get deployments

# Check pods
kubectl get pods

# Check service
kubectl get services
Expected Output:

Copy code

deployment.apps/frontend created
service/frontend-service created

NAME       READY   UP-TO-DATE   AVAILABLE   AGE
frontend   2/2     2            2           30s
Step 2: Deploy Backend Java
cmd
Copy code

# Deploy
kubectl apply -f backend-java.yml

# Check deployment
kubectl get deployments

# Check pods
kubectl get pods

# Check service
kubectl get services
Step 3: Deploy Backend Python
cmd
Copy code

# Deploy
kubectl apply -f backend-python.yml

# Check deployment
kubectl get deployments

# Check pods
kubectl get pods

# Check service
kubectl get services
Step 4: Verify All Deployments

# View all resources
kubectl get all

# Check pod details
kubectl get pods -o wide

# Check pod logs (replace pod-name with actual pod name)
kubectl logs <pod-name>
Example:

kubectl logs frontend-7d8f9c5b6d-abcde

kubectl port-forward service/backend-java-service 8080:8080
Open browser: http://localhost:8080

For Backend Python:


kubectl port-forward service/backend-python-service 5000:5000
Open browser: http://localhost:5000

Part 7: Useful Commands
View Resources
cmd
Copy code

# All resources
kubectl get all

# Specific resource
kubectl get pods
kubectl get deployments
kubectl get services

# Detailed info
kubectl describe pod <pod-name>
kubectl describe deployment <deployment-name>
View Logs
cmd
Copy code

# Single pod
kubectl logs <pod-name>

# Follow logs
kubectl logs -f <pod-name>

# All pods of a deployment
kubectl logs -l app=frontend
Scale Deployment


kubectl scale deployment frontend --replicas=3
Delete Resources


# Delete specific deployment
kubectl delete -f frontend.yml

# Delete all
kubectl delete -f .
Stop Minikube


minikube stop
Start Minikube Again


minikube start
Delete Minikube Cluster


minikube delete
Troubleshooting
Docker Not Running


# Start Docker Desktop from Start Menu
# Wait for "Docker is running" in system tray
Minikube Won't Start


minikube delete
minikube start --driver=docker
Image Pull Error


# Make sure you pushed images to Docker Hub
docker images | findstr viit

# Verify username in manifests matches Docker Hub username
Pods Not Running
cmd
Copy code

# Check pod status
kubectl get pods

# View pod details
kubectl describe pod <pod-name>

# Check logs
kubectl logs <pod-name>
