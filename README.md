# End-to-End CI/CD Pipeline with Kubernetes Monitoring

Complete DevOps pipeline with Terraform, Ansible, Jenkins, Docker, Kubernetes, Prometheus, and Grafana.

## 🏗️ Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Developer     │    │   Jenkins CI    │    │   Kubernetes    │
│                 │───▶│                 │───▶│                 │
│   Git Push      │    │   Build & Test  │    │   Deploy App    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │                        │
                                ▼                        ▼
                       ┌─────────────────┐    ┌─────────────────┐
                       │   Docker Hub    │    │   Monitoring    │
                       │                 │    │                 │
                       │   Image Store   │    │ Prometheus+Loki │
                       └─────────────────┘    └─────────────────┘
```

## 🚀 Quick Start

### Prerequisites
- AWS Account with CLI configured
- SSH key pair: `~/.ssh/bookstore_poc`
- DockerHub account

### 1. Infrastructure Setup
```bash
# Deploy AWS infrastructure
cd terraform
terraform init
terraform apply

# Configure servers
cd ../ansible
# Update inventory.ini with terraform output IPs
ansible-playbook -i inventory.ini playbook.yml
```

### 2. Application Deployment
```bash
# Deploy to Kubernetes
cd ../k8s
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

### 3. Monitoring Setup
```bash
# Deploy Prometheus + Grafana
chmod +x deploy-monitoring-with-app.sh
./deploy-monitoring-with-app.sh

# Deploy Loki for logs
chmod +x deploy-loki.sh
./deploy-loki.sh
```

## 📁 Project Structure

```
├── bookstore-app/
│   ├── app/                    # Node.js application
│   │   ├── src/               # Source code with metrics
│   │   ├── package.json       # Dependencies + test scripts
│   │   └── Dockerfile         # Container definition
│   ├── ci/                    # Continuous Integration
│   │   └── Jenkinsfile        # CI pipeline (build, test, security)
│   ├── cd/                    # Continuous Deployment
│   │   └── Jenkinsfile        # CD pipeline (deploy to K8s)
│   ├── k8s/                   # Kubernetes manifests
│   │   ├── deployment.yaml    # App deployment
│   │   ├── service.yaml       # Load balancer service
│   │   ├── prometheus-*.yaml  # Monitoring stack
│   │   ├── loki-*.yaml        # Log aggregation
│   │   └── *.json            # Grafana dashboards
│   ├── terraform/             # Infrastructure as Code
│   │   ├── main.tf           # AWS VPC, EC2, Security Groups
│   │   └── variables.tf      # Configuration variables
│   └── ansible/              # Configuration Management
│       ├── playbook.yml      # Server setup automation
│       └── inventory.ini     # Server inventory
└── load-test.sh              # Performance testing
```

## 🔧 Components

### Application Stack
- **Frontend**: Node.js Express server
- **API**: RESTful endpoints for DevOps product store
- **Metrics**: Prometheus metrics integration
- **Health Checks**: Kubernetes readiness/liveness probes

### CI/CD Pipeline
- **CI**: Build → Test → Security Scan → Docker Push → Trigger CD
- **CD**: Deploy → Verify → Health Check
- **Security**: npm audit, secret detection, container scanning
- **Automation**: Auto-trigger CD on successful CI

### Infrastructure
- **AWS**: VPC, EC2 instances, Security Groups
- **Kubernetes**: K3s single-node cluster
- **Load Balancer**: NodePort services for external access

### Monitoring & Observability
- **Metrics**: Prometheus + Grafana dashboards
- **Logs**: Loki + Promtail log aggregation
- **Alerts**: Request rate, error rate, response time
- **Dashboards**: Application, infrastructure, and pod monitoring

## 🌐 Access Points

| Service | URL | Credentials |
|---------|-----|-------------|
| Bookstore App | `http://WORKER_IP:30080` | - |
| Jenkins | `http://JENKINS_IP:8080` | admin/admin |
| Prometheus | `http://WORKER_IP:30090` | - |
| Grafana | `http://WORKER_IP:30030` | admin/admin123 |

## 📊 Monitoring Features

### Application Metrics
- HTTP request rate and response times
- Request count by endpoint and status code
- Memory and CPU usage
- Custom business metrics

### Infrastructure Metrics
- Pod status and resource usage
- Node performance metrics
- Kubernetes cluster health
- Container restart counts

### Log Aggregation
- Centralized application logs
- Error log filtering
- Real-time log streaming
- LogQL query capabilities

## 🧪 Testing

### Load Testing
```bash
# Generate traffic for monitoring
chmod +x load-test.sh
./load-test.sh  # 100K requests to test performance
```

### Manual Testing
```bash
# Test application endpoints
curl http://WORKER_IP:30080/                    # Homepage
curl http://WORKER_IP:30080/api/products        # Product API
curl http://WORKER_IP:30080/health              # Health check
curl http://WORKER_IP:30080/metrics             # Prometheus metrics
```

## 🔐 Security Features

- **Dependency Scanning**: npm audit for vulnerabilities
- **Secret Detection**: Scan for hardcoded credentials
- **Container Security**: Trivy image vulnerability scanning
- **Network Security**: AWS Security Groups with minimal access
- **Access Control**: SSH key-based authentication

## 📈 Dashboards

### Pre-built Grafana Dashboards
1. **Application Metrics**: Request rates, response times, status codes
2. **Kubernetes Pods**: Pod status, CPU/memory usage, restarts
3. **Infrastructure**: Node metrics, cluster health
4. **Logs**: Application logs with filtering and search

### Import Dashboards
```bash
# Import dashboard JSONs in Grafana UI
k8s/grafana-dashboard-bookstore.json     # App metrics
k8s/grafana-pods-dashboard.json          # Pod monitoring  
k8s/loki-dashboard.json                  # Log analysis
```

## 🚨 Troubleshooting

### Common Issues
```bash
# Check application status
kubectl get pods,svc -l app=bookstore

# View application logs
kubectl logs -l app=bookstore -f

# Test metrics endpoint
kubectl port-forward svc/bookstore 3000:3000
curl http://localhost:3000/metrics

# Check Prometheus targets
# Go to http://WORKER_IP:30090/targets
```

### Debug Scripts
```bash
# Debug Jenkins performance
./debug-jenkins.sh

# Fix kubeconfig issues  
./fix-kubeconfig-and-deploy.sh

# Check metrics collection
./debug-metrics.sh
```

## 🔄 CI/CD Workflow

1. **Developer pushes code** to repository
2. **Jenkins CI** automatically triggers:
   - Code checkout
   - Dependency installation and testing
   - Security vulnerability scanning
   - Docker image build and push
3. **Jenkins CD** automatically triggers on CI success:
   - Deploy latest image to Kubernetes
   - Verify deployment health
   - Update monitoring dashboards
4. **Monitoring** tracks application performance in real-time

## 📝 Configuration

### Environment Variables
```bash
# Update in Jenkinsfiles
DOCKERHUB = 'your-dockerhub-user/bookstore-app'
WORKER_IP = 'your-k8s-node-ip'
```

### Customize Monitoring
```bash
# Modify Prometheus scrape configs
k8s/prometheus-config.yaml

# Update Grafana dashboard queries
k8s/*.json
```

## 🤝 Contributing

1. Fork the repository
2. Create feature branch: `git checkout -b feature-name`
3. Commit changes: `git commit -am 'Add feature'`
4. Push to branch: `git push origin feature-name`
5. Submit pull request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🏷️ Tags

`devops` `kubernetes` `jenkins` `prometheus` `grafana` `terraform` `ansible` `docker` `ci-cd` `monitoring` `nodejs` `aws`
