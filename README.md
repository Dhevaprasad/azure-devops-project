Azure DevOps CI/CD Pipeline with AKS, ACR & HPA

End-to-end DevOps project demonstrating automated build, test, containerization, deployment, and autoscaling of a Python Flask application using Terraform, Docker, Azure Container Registry, Azure Kubernetes Service, and Azure DevOps.

End-to-End Flow

Developer → GitHub → Azure DevOps Pipeline
                 ├── PyTest
                 ├── Docker@2 → Azure Container Registry
                 └── KubernetesManifest@1 → AKS
                                      ├── Deployment
                                      ├── LoadBalancer
                                      └── HPA


Technologies

•	Azure, Azure DevOps, AKS, Azure Container Registry (ACR)
•	Terraform, Docker, Kubernetes, Python, Flask, PyTest
•	Git, GitHub, YAML, Azure RBAC, Workload Identity Federation (OIDC)
•	Horizontal Pod Autoscaler (HPA)


Project Structure

azure-devops-project/
├── app/
│   ├── app.py
│   ├── requirements.txt
│   └── test_app.py
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── hpa.yaml
├── terraform/
│   ├── provider.tf
│   ├── variables.tf
│   ├── main.tf
│   └── outputs.tf
├── Dockerfile
├── azure-pipelines.yml
├── .gitignore
└── README.md


Application & Testing

Python Flask application with a root endpoint and /health endpoint.
python3 -m pip install flask pytest
pytest
curl http://localhost:5000
curl http://localhost:5000/health

Docker

The application is containerized and tested locally before CI/CD deployment.
docker build -t myapp:v1 .
docker run -d -p 5000:5000 --name myapp myapp:v1

Terraform Infrastructure
Terraform provisions the Azure Resource Group, Azure Container Registry, AKS cluster, managed identity, and required RBAC.
terraform init
terraform validate
terraform plan
terraform apply
terraform destroy

Azure DevOps CI/CD

•	Checkout source code from GitHub
•	Install Python dependencies
•	Run PyTest
•	Build and push the Docker image using Docker@2
•	Publish the image to Azure Container Registry
•	Deploy Kubernetes manifests using KubernetesManifest@1
•	Roll out the application to AKS

Workload Identity Federation

Azure DevOps service connections use Workload Identity Federation (OIDC), avoiding long-lived client secrets. Separate connections are used for ACR and AKS.
ACR-Service-Connection → Docker Registry → Workload Identity Federation
AKS-Service-Connection → Azure Resource Manager → Workload Identity Federation

Kubernetes Deployment & LoadBalancer

The application is deployed using Kubernetes Deployment and exposed externally with a LoadBalancer Service. Azure provides a public IP for the service.

Internet
   ↓
Azure Public IP
   ↓
LoadBalancer Service
   ↓
Application Pods

Horizontal Pod Autoscaler

HPA scales the myapp Deployment according to CPU utilization.
Target Deployment: myapp
Minimum Replicas: 3
Maximum Replicas: 6
CPU Target: 60%
During load testing, CPU reached approximately 88–90% and HPA scaled the application from 3 → 5 → 6 Pods. After the load stopped and CPU dropped to approximately 1%, the application scaled back down to 3 Pods.

Kubernetes Verification

kubectl get nodes
kubectl get pods
kubectl get svc
kubectl get endpoints
kubectl get hpa
kubectl get hpa -w
kubectl get pods -w
kubectl top nodes
kubectl top pods

Azure RBAC

CI/CD and runtime permissions are separated. The ACR service connection publishes images, the AKS service connection deploys manifests, and the AKS identity pulls images from ACR.
Kubernetes Health Probes
The Kubernetes Deployment includes application health probes using the Flask health endpoint. Readiness determines whether a Pod is ready to receive traffic, while liveness helps Kubernetes detect an unhealthy application container and restart it when necessary.
Health endpoint: /health

## Screenshots

### HPA Scaling

![HPA and Pod scaling](screenshots/hpa-scaling.png)

### Azure DevOps Pipeline

![Azure DevOps Pipeline](screenshots/pipeline.png)

### Kubernetes Verification

![Kubernetes verification](screenshots/kubernetes.png)

### Service Connections

![Workload Identity Federation](screenshots/service-connections.png)


Cleanup

terraform destroy
AKS and ACR are billable Azure resources, so the Terraform environment should be destroyed when the project is not in use.

Project Highlights
•	Infrastructure as Code with Terraform
•	Automated Azure DevOps YAML CI/CD pipeline
•	Docker image build and ACR publishing
•	AKS deployment with Kubernetes manifests
•	Secretless authentication using Workload Identity Federation
•	Azure RBAC for controlled access
•	Public application exposure using LoadBalancer
•	CPU-based HPA with validated scale-up and scale-down

