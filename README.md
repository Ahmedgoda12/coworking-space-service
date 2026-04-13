# Coworking Space Analytics Service

## Overview
This project deploys a Flask-based analytics API that reports on coworking space usage. It is containerised with Docker, built and pushed to AWS ECR via AWS CodeBuild, and deployed on AWS EKS backed by a PostgreSQL database.

## Technologies
- **AWS EKS** — runs the Kubernetes cluster that hosts the analytics app and PostgreSQL database
- **AWS ECR** — stores versioned Docker images built by the CI pipeline
- **AWS CodeBuild** — automatically builds and pushes a new Docker image to ECR on every code push
- **AWS CloudWatch** — collects and displays container logs via the Container Insights addon
- **kubectl** — used to apply manifests and inspect cluster state

## Deployment Architecture
The PostgreSQL database runs as a Kubernetes Deployment with a PersistentVolumeClaim for durable storage and is exposed inside the cluster via a ClusterIP Service. The analytics app reads database credentials from a Kubernetes Secret and non-sensitive config from a ConfigMap, then connects to PostgreSQL using the internal service DNS name. The app is exposed externally via a LoadBalancer Service backed by an AWS ELB.

## Releasing a New Build
1. Push code changes to GitHub — CodeBuild triggers automatically and builds a new Docker image tagged with the build number.
2. Update the image tag in `deployments/coworking-deployment.yaml` to the new build number.
3. Apply the updated manifest with `kubectl apply -f deployments/coworking-deployment.yaml`.
4. Verify with `kubectl rollout status deployment/coworking` and test the endpoints.

## Environment Variables
Non-sensitive variables (DB_HOST, DB_PORT, DB_NAME, DB_USERNAME) are stored in a ConfigMap. The database password is stored in a Kubernetes Secret and injected at runtime — never baked into the Docker image.

## Verifying the Application
```bash
kubectl get svc coworking
curl <EXTERNAL-IP>:5153/api/reports/daily_usage
curl <EXTERNAL-IP>:5153/api/reports/user_visits
```

## Recommended AWS Instance Type
t3.medium (2 vCPU, 4 GB RAM) is the best fit — the app is I/O-bound rather than CPU-intensive, and the burstable credit model keeps costs low during quiet periods.

## Cost Optimisation
Use Spot Instances for the EKS node group (typically 60-70% cheaper than On-Demand) and configure the Cluster Autoscaler to scale nodes to zero outside business hours. Right-sizing the PostgreSQL pod resource requests also reduces the minimum node size required.
