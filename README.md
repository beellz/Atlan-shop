# E-commerce Application Kubernetes Deployment Guide

This repository contains Kubernetes manifests for deploying a complete e-commerce application with monitoring and logging capabilities.


## Architecture Overview

The application consists of the following components:
- Frontend: React application served by Nginx
- Backend: Node.js microservices
- Database: MongoDB cluster
- Message Queue: RabbitMQ
- Monitoring: Prometheus and Grafana
- Logging: ELK Stack (Elasticsearch, Logstash, Kibana)

## Prerequisites

- Kubernetes cluster (v1.20+)
- kubectl CLI tool
- Helm (optional, for additional deployments)
- Storage class supporting ReadWriteOnce access mode
- Ingress controller (e.g., nginx-ingress)

## Directory Structure

```
k8s/
├── frontend/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   ├── ingress.yaml
│   └── hpa.yaml
├── backend/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   └── hpa.yaml
├── mongodb/
│   ├── statefulset.yaml
│   ├── service.yaml
│   └── secret.yaml
├── rabbitmq/
│   ├── statefulset.yaml
│   ├── service.yaml
│   └── configmap.yaml
├── monitoring/
│   ├── namespace.yaml
│   ├── prometheus-config.yaml
│   ├── prometheus-deployment.yaml
│   └── grafana-deployment.yaml
├── logging/
│   ├── elasticsearch-statefulset.yaml
│   ├── logstash-deployment.yaml
│   ├── kibana-deployment.yaml
│   └── filebeat-config.yaml
└── namespace.yaml
```

## Installation Steps

1. Create namespaces:
```bash
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/monitoring/namespace.yaml
kubectl create namespace logging
```

2. Deploy core infrastructure:
```bash
# Deploy MongoDB
kubectl apply -f k8s/mongodb/
# Deploy RabbitMQ
kubectl apply -f k8s/rabbitmq/
```

3. Deploy application components:
```bash
# Deploy backend services
kubectl apply -f k8s/backend/
# Deploy frontend application
kubectl apply -f k8s/frontend/
```

4. Deploy monitoring stack:
```bash
# Deploy Prometheus
kubectl apply -f k8s/monitoring/prometheus-config.yaml
kubectl apply -f k8s/monitoring/prometheus-deployment.yaml
# Deploy Grafana
kubectl apply -f k8s/monitoring/grafana-deployment.yaml
```

5. Deploy logging stack:
```bash
# Deploy Elasticsearch
kubectl apply -f k8s/logging/elasticsearch-statefulset.yaml
# Deploy Logstash
kubectl apply -f k8s/logging/logstash-deployment.yaml
# Deploy Kibana
kubectl apply -f k8s/logging/kibana-deployment.yaml
# Deploy Filebeat configuration
kubectl apply -f k8s/logging/filebeat-config.yaml
```

## Monitoring Setup

### Grafana Setup
1. Access Grafana UI:
```bash
kubectl port-forward svc/grafana 3000:3000 -n monitoring
```

2. Login credentials:
- Username: admin
- Password: (retrieve from secret)
```bash
kubectl get secret grafana-secret -n monitoring -o jsonpath="{.data.admin-password}" | base64 -d
```

## Logging Setup


### Kibana Access
1. Access Kibana UI:
```bash
kubectl port-forward svc/kibana 5601:5601 -n logging
```

2. Create index patterns:
- Pattern: filebeat-*
- Time field: @timestamp

## Scaling Configuration

### Horizontal Pod Autoscaling
- Frontend scaling:
  - Min replicas: 3
  - Max replicas: 10
  - CPU target: 70%
  - Memory target: 80%

- Backend scaling:
  - Min replicas: 3
  - Max replicas: 15
  - CPU target: 75%
  - Memory target: 80%

Monitor HPA status:
```bash
kubectl get hpa -n ecommerce
```

---
