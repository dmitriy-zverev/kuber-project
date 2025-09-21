# SynergyChat - Enterprise Kubernetes Deployment

[![Kubernetes](https://img.shields.io/badge/Kubernetes-1.20+-blue.svg)](https://kubernetes.io/)
[![Production Ready](https://img.shields.io/badge/Production-Ready-brightgreen.svg)](https://github.com/dmitriy-zverev/kuber-project)

SynergyChat is a production-grade, cloud-native chat application built for enterprise environments. This repository contains the complete Kubernetes deployment manifests for a scalable, resilient, and observable chat platform.

## 🏗️ Architecture Overview

SynergyChat follows a microservices architecture pattern with the following components:

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Web Frontend  │    │   API Gateway   │    │  Crawler Service│
│   (React/Vue)   │    │   (REST API)    │    │  (Multi-instance)│
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌─────────────────┐
                    │   Ingress       │
                    │   Controller    │
                    └─────────────────┘
```

### Core Components

- **Web Frontend**: React-based user interface with auto-scaling capabilities
- **API Service**: RESTful backend with persistent storage and database integration
- **Crawler Service**: Multi-instance web crawling service with shared cache
- **Load Testing**: CPU and RAM stress testing components for performance validation

## 🚀 Quick Start

### Prerequisites

- Kubernetes cluster (v1.20+)
- kubectl configured and connected to your cluster
- NGINX Ingress Controller installed
- Metrics Server for HPA functionality

### Deployment

1. **Clone the repository**
   ```bash
   git clone https://github.com/dmitriy-zverev/kuber-project.git
   cd kuber-project
   ```

2. **Create the crawler namespace**
   ```bash
   kubectl create namespace crawler
   ```

3. **Deploy the application stack**
   ```bash
   # Deploy ConfigMaps first
   kubectl apply -f api-configmap.yaml
   kubectl apply -f web-configmap.yaml
   kubectl apply -f crawler-configmap.yaml
   kubectl apply -f testram-configmap.yaml

   # Deploy storage
   kubectl apply -f api-pvc.yaml

   # Deploy applications
   kubectl apply -f api-deployment.yaml
   kubectl apply -f web-deployment.yaml
   kubectl apply -f crawler-deployment.yaml

   # Deploy services
   kubectl apply -f api-service.yaml
   kubectl apply -f web-service.yaml
   kubectl apply -f crawler-service.yaml

   # Deploy ingress and autoscaling
   kubectl apply -f app-ingress.yaml
   kubectl apply -f web-hpa.yaml
   kubectl apply -f testcpu-hpa.yaml
   ```

4. **Deploy load testing components (optional)**
   ```bash
   kubectl apply -f testcpu-deployment.yaml
   kubectl apply -f testram-deployment.yaml
   ```

5. **Configure DNS or hosts file**
   ```bash
   # Add to /etc/hosts or configure DNS
   <INGRESS_IP> synchat.internal
   <INGRESS_IP> synchatapi.internal
   ```

## 📊 Monitoring & Observability

### Health Checks

Monitor application health using the live configuration:
```bash
kubectl apply -f live.yaml
```

### Scaling Metrics

The application includes Horizontal Pod Autoscalers (HPA) for:
- **Web Frontend**: Scales 1-4 replicas based on 50% CPU utilization
- **Test CPU Service**: Scales based on CPU metrics for load testing

### Key Metrics to Monitor

- Pod CPU/Memory utilization
- Request latency and throughput
- Database connection pool status
- Crawler job completion rates

## 🔧 Configuration

### Environment Variables

#### API Service
- `API_PORT`: Service port (default: 8080)
- `API_DB_FILEPATH`: Database file path for persistence
- `CRAWLER_BASE_URL`: Base URL for crawler service communication

#### Crawler Service
- `CRAWLER_PORT`: Primary crawler port
- `CRAWLER_PORT_2`: Secondary crawler instance port
- `CRAWLER_PORT_3`: Tertiary crawler instance port
- `CRAWLER_DB_PATH`: Shared database path
- `CRAWLER_KEYWORDS`: Keywords for crawling operations

### Storage Configuration

The API service uses persistent storage:
- **Volume**: `synergychat-api-pvc`
- **Mount Path**: `/persist`
- **Storage Class**: Default cluster storage class

## 🔄 Scaling & Performance

### Horizontal Pod Autoscaling

The web frontend automatically scales based on CPU utilization:
```yaml
minReplicas: 1
maxReplicas: 4
targetCPUUtilizationPercentage: 50
```

### Crawler Service Architecture

The crawler service runs multiple instances within a single pod:
- 3 crawler containers per pod
- Shared cache volume (`emptyDir`)
- Load distribution across multiple ports

### Performance Testing

Use the included load testing components:
```bash
# Deploy CPU stress test
kubectl apply -f testcpu-deployment.yaml

# Deploy RAM stress test  
kubectl apply -f testram-deployment.yaml
```

## 🛡️ Security Considerations

### Network Policies
- Implement network policies to restrict inter-pod communication
- Use service mesh for advanced traffic management

### Secrets Management
- Store sensitive configuration in Kubernetes Secrets
- Use external secret management systems (HashiCorp Vault, AWS Secrets Manager)

### RBAC
- Implement Role-Based Access Control for service accounts
- Follow principle of least privilege

## 🔍 Troubleshooting

### Common Issues

1. **Pods not starting**
   ```bash
   kubectl describe pod <pod-name>
   kubectl logs <pod-name> -c <container-name>
   ```

2. **Ingress not accessible**
   ```bash
   kubectl get ingress
   kubectl describe ingress app-ingress
   ```

3. **HPA not scaling**
   ```bash
   kubectl get hpa
   kubectl describe hpa web-hpa
   ```

### Debug Commands

```bash
# Check all resources
kubectl get all

# Check crawler namespace
kubectl get all -n crawler

# View logs
kubectl logs -f deployment/synergychat-api
kubectl logs -f deployment/synergychat-web
kubectl logs -f deployment/synergychat-crawler -n crawler

# Check resource usage
kubectl top pods
kubectl top nodes
```

## 📈 Production Deployment Checklist

- [ ] Configure resource limits and requests for all containers
- [ ] Set up monitoring and alerting (Prometheus/Grafana)
- [ ] Implement backup strategy for persistent volumes
- [ ] Configure log aggregation (ELK/EFK stack)
- [ ] Set up CI/CD pipeline for automated deployments
- [ ] Implement security scanning for container images
- [ ] Configure network policies for micro-segmentation
- [ ] Set up disaster recovery procedures
- [ ] Implement secrets management
- [ ] Configure SSL/TLS certificates for ingress

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

**Built with ❤️ for enterprise-grade Kubernetes deployments**
