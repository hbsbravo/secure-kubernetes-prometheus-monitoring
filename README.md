# Secure Kubernetes Prometheus Monitoring

## Overview

This project demonstrates how to harden monitoring for a containerized Django application running in Kubernetes. The original deployment included incomplete Prometheus monitoring and exposed sensitive data through unsafe metrics. I refactored the monitoring logic, removed sensitive telemetry, added a meaningful application metric, and deployed Prometheus so the Kubernetes cluster could collect and analyze runtime data.

The project combines application security, Kubernetes deployment, secret management, and observability practices in a realistic DevSecOps workflow.

## Problem

The original application had three major issues:

1. Sensitive values were being exposed through application monitoring.
2. The application had Prometheus client instrumentation, but the monitoring data was not being collected by a Prometheus service.
3. The Kubernetes deployment needed better configuration for secure and useful observability.

This created a common security and operations problem: metrics existed, but they were both unsafe and incomplete. Monitoring should help detect problems, not leak secrets.

## What I Implemented

### Removed Sensitive Metrics

I reviewed the Django application monitoring code in:

```text
GiftcardSite/LegacySite/views.py
```

Unsafe monitoring related to secrets was removed so sensitive values would not be exposed through the metrics endpoint.

### Added Safer Application Monitoring

I added a Prometheus counter named:

```text
database_error_return_404
```

This counter tracks cases where the application intentionally returns a 404 response because of a database-related error.

In Prometheus, this counter appears as:

```text
database_error_return_404_total
```

This is useful because it gives visibility into backend database failures that are being surfaced to users as 404 responses.

### Deployed Prometheus in Kubernetes

I added Kubernetes YAML files to deploy and configure Prometheus:

```text
prometheus.yaml
prometheus-deployment.yaml
prometheus-service.yaml
```

These files configure Prometheus to run inside the Kubernetes cluster and scrape application metrics.

### Preserved Secret Hardening from the Previous Module

The project also includes the sealed Kubernetes secret file:

```text
module2.yaml
```

The raw unsealed secret file is intentionally excluded from the repository. This keeps sensitive values out of source control while still allowing the Kubernetes deployment to use secrets at runtime.

## Technologies Used

* Python
* Django
* Prometheus Python Client
* Prometheus
* Kubernetes
* Docker
* Minikube
* kubectl
* Kubernetes ConfigMaps
* Kubernetes Sealed Secrets
* GitHub Actions

## Repository Structure

```text
.
├── .github/workflows/          # GitHub Actions workflow files
├── GiftcardSite/               # Django application
├── db/                         # Database container and Kubernetes files
├── proxy/                      # Reverse proxy container and Kubernetes files
├── scripts/                    # Supporting scripts
├── module2.yaml                # Sealed Kubernetes secret
├── prometheus.yaml             # Prometheus configuration
├── prometheus-deployment.yaml  # Prometheus Kubernetes deployment
├── prometheus-service.yaml     # Prometheus Kubernetes service
├── Dockerfile                  # Django application container build file
├── requirements.txt            # Python dependencies
└── README.md                   # Project documentation
```

## Security Improvements

This project improves the deployment by:

* Removing sensitive values from monitoring output.
* Avoiding plaintext secret exposure in the public repository.
* Using sealed Kubernetes secrets for protected runtime configuration.
* Adding focused Prometheus monitoring for database-related application errors.
* Deploying a working Prometheus service to collect and visualize metrics.

## Why This Matters

Monitoring systems are often trusted by default, but they can become a security risk if they expose secrets, tokens, passwords, or sensitive business logic. Secure observability requires collecting useful operational data without leaking confidential information.

This project shows how to apply that principle in a Kubernetes environment by replacing unsafe telemetry with meaningful, low-risk metrics.

## Example Commands

Start Minikube:

```bash
minikube start
```

Build the application images:

```bash
docker build -t nyuappsec/assign3:v0 .
docker build -t nyuappsec/assign3-proxy:v0 proxy/
docker build -t nyuappsec/assign3-db:v0 db/
```

Apply the Kubernetes resources:

```bash
kubectl apply -f module2.yaml
kubectl apply -f db/k8
kubectl apply -f GiftcardSite/k8
kubectl apply -f proxy/k8
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

## Useful Metrics to Consider

In addition to the implemented `database_error_return_404` counter, other valuable metrics for this type of application could include:

### Failed Login Attempts

A counter for failed login attempts could help detect brute-force attacks, credential stuffing, or account takeover attempts.

### Gift Card Redemption Errors

A counter for gift card redemption failures could help identify fraud attempts, validation issues, database problems, or broken transaction logic.

## Lessons Learned

* Monitoring should never expose secrets.
* Metrics should be intentionally designed around useful operational and security signals.
* A `/metrics` endpoint is not enough; a Prometheus service must collect and process the data.
* Kubernetes ConfigMaps are useful for Prometheus scrape configuration.
* Sealed Secrets help keep sensitive deployment values out of public source control.
