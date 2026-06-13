# Secure Kubernetes Prometheus Monitoring

## Overview

This project demonstrates how to secure and operationalize monitoring for a containerized Django application deployed on Kubernetes. The application originally included incomplete Prometheus monitoring and unsafe telemetry that could expose sensitive information. I hardened the monitoring implementation, added a meaningful application metric, deployed Prometheus, and organized the Kubernetes security artifacts into a cleaner project structure.

The project focuses on DevSecOps practices across application security, Kubernetes deployment, secret management, and observability.

## Security Problem

The original deployment had several security and reliability issues:

* Sensitive values were at risk of being exposed through application monitoring.
* The application had Prometheus instrumentation, but Prometheus was not fully deployed to collect the metrics.
* Runtime secrets needed to be separated from normal Kubernetes configuration.
* Monitoring needed to provide useful operational signals without leaking confidential data.

## What I Implemented

### Secure Application Metrics

Unsafe monitoring logic was removed from the Django application so that secrets would not be exposed through the metrics endpoint.

A Prometheus counter was added to track intentional database-related 404 responses:

```text
database_error_return_404
```

In Prometheus, this counter appears as:

```text
database_error_return_404_total
```

This metric helps identify cases where backend database problems are being surfaced to users as 404 responses.

### Prometheus Deployment

Prometheus was deployed into the Kubernetes environment using YAML configuration files. The monitoring setup allows Prometheus to scrape application metrics and make them available for inspection through the Prometheus UI.

Prometheus-related files include:

```text
prometheus.yaml
prometheus-deployment.yaml
prometheus-service.yaml
```

### Secret Management

Plaintext secrets were removed from normal Kubernetes deployment files and replaced with a sealed secret workflow.

The sealed secret artifact is stored here:

```text
k8/secrets/sealed-django-secret.yaml
```

The raw unsealed secret file is intentionally excluded from the repository:

```text
django-secret-raw.yaml
```

Additional notes about the secret management design are available in:

```text
docs/secret-management.md
```

## Technologies Used

* Python
* Django
* Docker
* Kubernetes
* Minikube
* kubectl
* Prometheus
* Prometheus Python Client
* Kubernetes ConfigMaps
* Kubernetes Sealed Secrets
* GitHub Actions

## Repository Structure

```text
.
├── .github/workflows/          # GitHub Actions workflow files
├── GiftcardSite/               # Django application source code
├── assets/                     # Supporting project assets
├── db/                         # Database container and Kubernetes files
├── docs/                       # Project documentation
│   └── secret-management.md    # Secret management notes
├── k8/                         # Additional Kubernetes security artifacts
│   └── secrets/
│       └── sealed-django-secret.yaml
├── proxy/                      # Reverse proxy configuration
├── scripts/                    # Setup and helper scripts
├── prometheus.yaml             # Prometheus configuration
├── prometheus-deployment.yaml  # Prometheus Kubernetes deployment
├── prometheus-service.yaml     # Prometheus Kubernetes service
├── Dockerfile                  # Django application container build file
├── requirements.txt            # Python dependencies
└── setup.sh                    # Environment setup script
```

## Security Improvements

This project improves the deployment by:

* Removing sensitive telemetry from application monitoring.
* Adding focused Prometheus metrics for database-related errors.
* Deploying Prometheus so metrics are actually collected.
* Moving secret handling into a Kubernetes sealed secret workflow.
* Keeping raw secrets out of source control.
* Organizing security documentation under `docs/`.

## Example Commands

Start Minikube:

```bash
minikube start
```

Build the Docker images:

```bash
docker build -t nyuappsec/assign3:v0 .
docker build -t nyuappsec/assign3-proxy:v0 proxy/
docker build -t nyuappsec/assign3-db:v0 db/
```

Apply the sealed secret:

```bash
kubectl apply -f k8/secrets/sealed-django-secret.yaml
```

Apply the application resources:

```bash
kubectl apply -f db/k8
kubectl apply -f GiftcardSite/k8
kubectl apply -f proxy/k8
```

Apply the Prometheus resources:

```bash
kubectl apply -f prometheus.yaml
kubectl apply -f prometheus-deployment.yaml
kubectl apply -f prometheus-service.yaml
```

Check running pods and services:

```bash
kubectl get pods
kubectl get services
```

Open the application through the proxy:

```bash
minikube service proxy-service
```

Open Prometheus:

```bash
minikube service prometheus-server
```

## Monitoring Value

The implemented metric helps identify database-related failures that are being returned to users as 404 responses. This provides a useful operational signal because repeated increases in this counter could indicate backend instability, database connectivity problems, broken queries, or error-handling paths that need review.

Other useful metrics for this type of application could include:

* Failed login attempts
* Gift card redemption errors
* Request latency
* HTTP 5xx response counts
* Database connection failures

## Lessons Learned

* Monitoring should never expose secrets.
* Metrics should be designed around useful security and reliability signals.
* A metrics endpoint is not enough; Prometheus must be deployed and configured to scrape it.
* Kubernetes ConfigMaps are useful for Prometheus configuration.
* Kubernetes Sealed Secrets help keep sensitive runtime values out of public source control.
* Project structure matters when presenting security work in a portfolio.
