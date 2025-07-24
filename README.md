# Eyego Deployment CI/CD Pipeline

This repository automates building, scanning, and deploying a Node.js application to AWS EKS using GitLab CI/CD.

---

## Project Structure

```
├── cluster-workloads/
│   ├── deployment.yaml      # Kubernetes Deployment manifest
│   ├── pod.yaml             # (Optional) Kubernetes Pod manifest
│   └── service.yaml         # Kubernetes Service manifest
├── nodejs/
│   ├── app.js               # Simple Node.js web server
│   ├── Dockerfile           # Container image definition
│   └── package.json         # Project dependencies and metadata
└── .gitlab-ci.yml           # GitLab CI/CD pipeline definition
```

---

## GitLab CI/CD Pipeline

### Stages

* **notify**: Slack notification before the deployment starts
* **build**: Build and save the Docker image
* **security-scan**: Scan the image with Trivy
* **deploy**: Push image to AWS ECR and deploy to AWS EKS

### Key Jobs

#### 1. `slack-notify`

Sends a Slack message with deployment details using webhook `$SLACK_WEBHOOK`.

#### 2. `docker-build-node`

Builds the Docker image from `nodejs/` and saves it as `node-image.tar`.

#### 3. `trivy-node`

Runs [Trivy](https://github.com/aquasecurity/trivy) to perform vulnerability scanning:

* Uses a cache for faster scans
* Generates a JSON report for GitLab

#### 4. `docker-push-image`

* Loads the image
* Tags it with `$PUBLIC_ECR_IMAGE`
* Pushes it to AWS ECR Public

#### 5. `deploy-workload`

* Installs `kubectl` and AWS CLI
* Authenticates to EKS using IAM credentials
* Applies Kubernetes manifests (`deployment.yaml`, `service.yaml`)
* Outputs deployment details

---

## Requirements

Set the following CI/CD variables in GitLab:

* `AWS_IAM_ACCESS_KEY`
* `AWS_IAM_SECRET_KEY`
* `AWS_REGION`
* `CLUSTER_NAME`
* `PUBLIC_ECR_IMAGE`
* `SLACK_WEBHOOK`

---

## Kubernetes Deployment Files

Located in `cluster-workloads/`. These define how the application is deployed, exposed, and managed in the cluster.

* `deployment.yaml`: Defines a Deployment with image pulled from ECR public registry.
* `service.yaml`: Exposes the app using a LoadBalancer service

---

## CI/CD Security Scanning

Trivy scans the Docker image using 3 commands:

* `exit-code 0`: Report all vulnerabilities
* `exit-code 1 --severity CRITICAL`: Fail pipeline on critical vulnerabilities
* Output stored as `$FULL_IMAGE_NAME-scanning-report.json`

---

## Final Deployment Output

After deployment, logs include:

* Running pods
* Deployment status
* Service status
* External Load Balancer URL:

```
Deployment URL:
<load-balancer-host>:<port>
```

