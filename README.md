# Zero-Downtime Deployment System

A production-grade deployment system using ArgoCD for GitOps, Argo Rollouts for progressive delivery, Istio for traffic routing, Prometheus for metric-based rollbacks, and OpenTelemetry for tracing.

## System Architecture

This system implements a complete zero-downtime deployment pipeline with the following components:

1. **GitOps with ArgoCD** - Continuous deployment using Git as the source of truth
2. **Progressive Delivery with Argo Rollouts** - Canary deployments with automated analysis
3. **Service Mesh with Istio** - Advanced traffic routing and management
4. **Monitoring with Prometheus** - Metrics collection and alerting
5. **Tracing with OpenTelemetry/Jaeger** - Distributed tracing for observability
6. **Security with Kyverno** - Policy enforcement for security and compliance
7. **Feature Flags with Unleash** - Runtime feature toggles
8. **Load Testing with k6** - Performance testing capabilities

## Prerequisites

- Kubernetes cluster (v1.20+)
- ArgoCD installed
- Argo Rollouts installed
- Istio installed
- Prometheus Operator installed
- Kyverno installed
- OpenTelemetry Operator installed
- Jaeger installed

## Folder Structure

```
GitOps/
├── argocd/                 # ArgoCD Application and ApplicationSet configurations
├── argorollouts/           # Argo Rollouts Analysis templates
├── backstage/              # Backstage catalog information
├── feature-flags/          # Feature flag configurations (Unleash)
├── istio/                  # Istio Gateway, VirtualService, and DestinationRule
├── kustomize/              # Kustomize overlays for different environments
│   ├── base/               # Base manifests
│   ├── dev/                # Development environment overlay
│   ├── staging/            # Staging environment overlay
│   └── prod/               # Production environment overlay
├── kyverno/                # Security policies
├── load-testing/           # Load testing jobs (k6)
├── manifests/              # Base Kubernetes manifests
├── observability/          # OpenTelemetry, Jaeger, and Grafana configurations
└── prometheus/             # Prometheus alert rules
```

## Deployment Instructions

### 1. Install Prerequisites

If not already installed, install the required components:

```bash
# Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Install Argo Rollouts
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml

# Install Istio (example using Istio operator)
kubectl apply -f https://istio.io/latest/istioctl/install/latest/istio-operator.yaml

# Install Prometheus Operator
kubectl create namespace monitoring
kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/master/bundle.yaml

# Install Kyverno
kubectl create namespace kyverno
kubectl apply -f https://raw.githubusercontent.com/kyverno/kyverno/main/config/install-latest.yaml

# Install OpenTelemetry Operator
kubectl apply -f https://github.com/open-telemetry/opentelemetry-operator/releases/latest/download/opentelemetry-operator.yaml

# Install Jaeger Operator
kubectl create namespace observability
kubectl apply -f https://github.com/jaegertracing/jaeger-operator/releases/latest/download/jaeger-operator.yaml
```

### 2. Deploy the System

1. Fork this repository to your own GitHub account
2. Update the repository URL in [argocd/application.yaml](argocd/application.yaml) and [argocd/applicationset.yaml](argocd/applicationset.yaml)
3. Apply the ArgoCD Application:

```bash
kubectl apply -f argocd/application.yaml
```

Or for multi-environment deployment:

```bash
kubectl apply -f argocd/applicationset.yaml
```

### 3. Environment Promotion

The system supports environment promotion through GitOps:

1. **Development**: Changes are automatically deployed to the dev environment
2. **Staging**: Promote by merging changes from dev to staging branch
3. **Production**: Promote by merging changes from staging to main branch

### 4. Canary Deployment Process

1. A new version is deployed as a canary
2. Traffic is gradually shifted to the canary (20% → 40% → 60% → 80% → 100%)
3. Automated analysis runs at each step using Prometheus metrics
4. If error rate exceeds threshold or latency is too high, automatic rollback occurs
5. Manual judgment step allows for human intervention if needed

### 5. Monitoring and Observability

- **Prometheus**: Metrics collection and alerting
- **Grafana**: Dashboard for golden signals (traffic, errors, latency, saturation)
- **Jaeger**: Distributed tracing
- **Kiali**: Service mesh visualization

Access Grafana dashboard using the provided [grafana-dashboard.json](observability/grafana-dashboard.json).

## Security Policies

The system enforces security through Kyverno policies:

1. **Resource Limits**: All pods must specify CPU and memory requests/limits
2. **Image Signing**: Only signed images are allowed (using Cosign)

## Feature Flags

Feature flags are managed through Unleash:

1. Deploy the Unleash server: `kubectl apply -f feature-flags/unleash.yaml`
2. Access the Unleash UI to manage feature flags
3. Integrate with your application using Unleash SDKs

## Load Testing

Run load tests using k6:

```bash
kubectl apply -f load-testing/k6-load-test.yaml
```

## Rollback Process

Automatic rollback is triggered when:

1. Error rate exceeds 5% for 2 minutes
2. Latency exceeds 1 second for 2 minutes
3. Success rate drops below 95% for 2 minutes

Manual rollback can be performed through the Argo Rollouts dashboard or CLI:

```bash
kubectl argo rollouts undo rollout/zero-downtime-app
```

## Customization

To customize this system for your application:

1. Update the image name in [manifests/rollout.yaml](manifests/rollout.yaml)
2. Modify resource requests/limits as needed
3. Update environment variables in [manifests/configmap.yaml](manifests/configmap.yaml)
4. Adjust Prometheus alert thresholds in [prometheus/alerts.yaml](prometheus/alerts.yaml)
5. Customize analysis templates in [argorollouts/analysis-templates.yaml](argorollouts/analysis-templates.yaml)

## Troubleshooting

Common issues and solutions:

1. **ArgoCD Sync Issues**: Check application status with `kubectl get applications -n argocd`
2. **Rollout Stuck**: Check rollout status with `kubectl argo rollouts get rollout zero-downtime-app`
3. **Traffic Not Routing**: Verify Istio VirtualService and DestinationRule configurations
4. **Alerts Not Firing**: Check Prometheus rules and connectivity

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.