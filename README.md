# eks-minikube
#  EKS-Style Secure Deployment Simulation 

This project simulates a production-like secure Kubernetes deployment (similar to AWS EKS) using **Minikube** in a custom VM. It includes:

- Microservice deployment using Helm
- RBAC and ServiceAccount-based IAM simulation with MinIO
- NetworkPolicies for microservice isolation
- Monitoring via Prometheus + Grafana

---

## 📁 Project Structure

.
├── charts/ # Helm charts for each microservice
│ ├── auth-service/
│ ├── data-service/
│ └── gateway/
├── manifests/ # Static Kubernetes resources
│ ├── network-policies.yaml
│ └── namespaces.yaml
├── values/ # Helm values.yaml files
│ ├── auth-service.yaml
│ ├── data-service.yaml
│ └── gateway.yaml
├── README.md

---

## 🧰 Prerequisites

- VirtualBox VM with Minikube installed (`minikube start` is working)
- Helm installed
- Git configured
- Internet access to pull Helm charts

## Ingress-egress

allow-auth-to-data:
Only pods with label app=auth-service can send traffic to data-service.

deny-all-egress-auth:
Blocks all egress traffic from auth-service, including internet or other services.


##Architecture Digram:
                             ┌──────────────────────────┐
                             │       Developer VM       │
                             │     (Minikube Host)      │
                             └────────────┬─────────────┘
                                          │
                                          ▼
                             ┌──────────────────────────┐
                             │      Minikube Cluster     │
                             │       (Single Node)       │
                             └──────────────────────────┘
                                          │
                   ┌──────────────────────┴────────────────────────┐
                   │                                               │
        ┌──────────▼─────────┐                         ┌───────────▼─────────┐
        │   Namespace: `app` │                         │  Namespace: `system`│
        └──────────┬─────────┘                         └───────────┬─────────┘
                   │                                               │
   ┌───────────────▼─────────────┐                  ┌──────────────▼──────────────┐
   │  🧠 auth-service (Pod)       │                  │    🗂️ MinIO (Pod)            │
   │  - Uses ServiceAccount       │◄──────RBAC───────│ - Simulates S3 IAM          │
   │  - Egress Restricted         │                  └──────────────┬──────────────┘
   │  - Only allowed to contact  ▼                                   │
   │    data-service via        NetworkPolicy                        │
   └────────────────────────────┘                                    │
                                                                     │
   ┌────────────────────────────┐                ┌───────────────────▼────────────┐
   │  📊 Gateway (Pod)           │                │  📈 Prometheus & Grafana (Pods) │
   └────────────────────────────┘                └─────────────────────────────────┘
                   ▲
                   │
     ┌─────────────┴────────────┐
     │    External Access via   │
     │     NodePort/Ingress     │
     └──────────────────────────┘


| Component              | Description                                                                     
 ----------------------  ------------------------------------------------------------------------------- 
 `auth-service`        :  Service restricted by a `NetworkPolicy` to talk only to `data-service`.         
 `data-service`        :  Core service with RBAC access to MinIO using a specific `ServiceAccount`.       
 `gateway`             : Acts as a frontend or API gateway with Ingress exposure.                        
 `MinIO`               : Simulates AWS S3 bucket and IAM using keys + RBAC for access control.           
 `Prometheus + Grafana`:  Provides observability into the system metrics.                                 
 `NetworkPolicy`       : Prevents unauthorized east-west communication between services.                 
 `RBAC`                : Used to simulate IAM-like permissions for pods (via `Role` + `ServiceAccount`). 

