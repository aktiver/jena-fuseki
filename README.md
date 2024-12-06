# Apache Jena Fuseki Helm Chart

This chart deploys Apache Jena Fuseki with TDB2 and REST API support on Kubernetes.

## Features
- Supports autoscaling.
- Distributed graph processing across multiple cores and memory limits.
- Persistent storage for datasets.
- Configurable via `values.yaml`.

## Prerequisites
- Kubernetes 1.20+
- Helm 3.0+
- Persistent Volume provisioner

## Installation
1. Clone this repository:
   ```bash
   git clone <repo_url>
   cd jena-fuseki
   ```

2. Install the chart:
   ```bash
   helm install jena-fuseki ./jena-fuseki
   ```

3. Verify the deployment:
   ```bash
   kubectl get pods
   ```

4. Access the service:
   ```bash
   kubectl port-forward svc/jena-fuseki 3030:3030
   ```

## Configuration
Modify `values.yaml` to customize the deployment:
- `replicaCount`: Number of initial replicas.
- `resources`: Define CPU and memory requests/limits.
- `datasets`: Predefined datasets to load.

## Cleaning Up
To uninstall the chart:
```bash
helm uninstall jena-fuseki
```

## Contributing
Feel free to open issues or submit PRs for improvements.
