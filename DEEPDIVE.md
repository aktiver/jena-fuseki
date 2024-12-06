# Apache Jena Fuseki Helm Chart

This chart deploys Apache Jena Fuseki with TDB2 and REST API support on Kubernetes.

## Features 

1. Autoscaling: Scales pods dynamically based on CPU utilization.
2. Distributed Graph: Uses TDB2, partitioned across pods with high CPU and memory limits.
3. Configurable Resources: Scales from 32 cores to 512 cores with defined heap sizes.
4. Affinity Rules: Ensures pods are distributed across nodes.
5. Configurable via `values.yaml` with persistent storage.

## File Structure:

```
jena-fuseki/
├── charts/
├── templates/
│   ├── deployment.yaml          # Configures the Fuseki service
│   ├── service.yaml             # Exposes the Fuseki service
│   ├── configmap.yaml           # Stores configuration and scripts
│   ├── pvc.yaml                 # Defines persistent volume for data
│   ├── job-load-data.yaml       # Executes dataset initialization
│   ├── hpa.yaml                 # Enables autoscaling
├── files/                       # Contains additional required files
│   ├── docker-entrypoint.sh     # Custom entry point script
│   ├── load.sh                  # Data loading script
│   ├── shiro.ini                # Security configuration
│   ├── tdbloader2               # Loader executable (binary)
├── values.yaml                  # Configures deployment options
├── Chart.yaml                   # Helm chart metadata

```

## TODO: Changes Required

- Add docker-entrypoint.sh: Place this file in files/ and reference it in the ConfigMap.

- Include load.sh: Place in files/ and use it in the job-load-data.yaml.

- Mount shiro.ini: Add this file to the ConfigMap to configure Fuseki.

- Include tdbloader2: Add this as a file in files/, and mount it in the deployment.

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

------------------------------------------

## Deployment notes
- Install the Helm Chart:
`helm install jena-fuseki ./jena-fuseki`

- Monitor the Deployment:
`kubectl get pods`

- Verify Resource Allocation:
`kubectl top pods`

- Load Datasets: If loader.enabled is true in `values.yaml`, the job automatically preloads the datasets.


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

# Apache Jena Fuseki Helm Deployment

## Key Features

### Distributed Graph Processing
- Designed for environments with high computational demands, supporting configurations to scale across **32, 64, 128, 256, and 512 cores**.
- JVM is tuned dynamically based on the `values.yaml` configuration for heap size and garbage collection threads.

### TDB2 Backed Storage
- Uses **TDB2** for efficient RDF triple storage and SPARQL query execution.
- Data is partitioned across nodes for better performance in distributed setups.

### Autoscaling
- **Horizontal Pod Autoscaler (HPA)** scales pods based on CPU utilization.
- Minimum and maximum replica counts can be adjusted in `values.yaml`.

### Persistent Storage
- **Persistent Volume Claims (PVCs)** ensure data is retained across pod restarts and redeployments.
- Storage size is configurable via `values.yaml`.

### Preloaded Datasets
- Data is initialized during deployment using a **Kubernetes Job** that runs the `load.sh` script.
- Datasets are pre-configured in `values.yaml`.

### Configurable Security
- Ships with a default `shiro.ini` configuration for user authentication and role-based access.

### Custom Entry Point
- A custom `docker-entrypoint.sh` initializes the environment, ensuring datasets are set up and configurations applied correctly.

### Kubernetes Native
- Fully managed deployment, leveraging Kubernetes ConfigMaps, PVCs, and Jobs.
- Ready to integrate with monitoring tools like **Prometheus**.

---

## Deployment Components

### 1. ConfigMaps
- **File**: `configmap.yaml`
- **Purpose**:
  - Stores configuration files (`shiro.ini`, `load.sh`, `docker-entrypoint.sh`) and the `tdbloader2` loader binary.
  - Mounted into the container for runtime execution.

### 2. Deployment
- **File**: `deployment.yaml`
- **Purpose**:
  - Configures the Fuseki container with resource limits for CPU and memory.
  - Mounts the ConfigMap and persistent storage volumes.
  - Dynamically adjusts JVM parameters (`-Xms`, `-Xmx`, `ParallelGCThreads`) based on `values.yaml`.

### 3. Persistent Volume Claims
- **File**: `pvc.yaml`
- **Purpose**:
  - Ensures RDF datasets are stored persistently.

### 4. Horizontal Pod Autoscaler
- **File**: `hpa.yaml`
- **Purpose**:
  - Scales pods based on CPU usage, ensuring the deployment handles workload spikes.

### 5. Data Loader Job
- **File**: `job-load-data.yaml`
- **Purpose**:
  - Runs a one-time job to preload RDF datasets into TDB2 databases during initialization.

### 6. Service
- **File**: `service.yaml`
- **Purpose**:
  - Exposes Fuseki’s REST API on a configurable port (default: 3030).
  - Uses a **ClusterIP** service type by default, suitable for internal usage.

---

## Configuration (`values.yaml`)

| Parameter                                   | Description                                    | Default         |
|---------------------------------------------|------------------------------------------------|-----------------|
| `replicaCount`                              | Initial number of pods                        | `1`             |
| `autoscaling.enabled`                       | Enable Horizontal Pod Autoscaling            | `true`          |
| `autoscaling.minReplicas`                   | Minimum number of replicas                    | `1`             |
| `autoscaling.maxReplicas`                   | Maximum number of replicas                    | `16`            |
| `autoscaling.targetCPUUtilizationPercentage`| Target CPU utilization for scaling            | `75`            |
| `resources.requests.memory`                 | Minimum memory allocation per pod            | `16Gi`          |
| `resources.limits.memory`                   | Maximum memory allocation per pod            | `256Gi`         |
| `resources.requests.cpu`                    | Minimum CPU allocation per pod               | `4`             |
| `resources.limits.cpu`                      | Maximum CPU allocation per pod               | `512`           |
| `javaOpts.minHeap`                          | JVM minimum heap size                        | `32G`           |
| `javaOpts.maxHeap`                          | JVM maximum heap size                        | `256G`          |
| `javaOpts.gcThreads`                        | JVM Parallel GC threads                      | `512`           |
| `datasets`                                  | List of datasets to initialize               | `graph1, graph2`|
| `persistence.size`                          | Storage size for PVC                         | `500Gi`         |
| `admin.password`                            | Administrator password                       | `admin`         |

---

## Workflow

### Initialization
- `docker-entrypoint.sh` sets up the `shiro.ini` file, initializes datasets, and starts the Fuseki service.

### Dataset Loading
- `load.sh` is executed via a Kubernetes Job, loading datasets into TDB2 databases.

### Autoscaling
- The Horizontal Pod Autoscaler monitors CPU usage and scales pods as needed.

### Graph Querying
- Users can interact with Fuseki via its REST API (port 3030 by default).

---

## How to Deploy

### Install the Helm Chart
```
helm install jena-fuseki ./jena-fuseki

Access the Fuseki Service

kubectl port-forward svc/jena-fuseki 3030:3030

Verify Datasets
Check logs for dataset initialization:

kubectl logs job/jena-fuseki-load-data
```
Query the RDF Graph
Access Fusekis web interface or REST API to run SPARQL queries.

## Contributing
Feel free to open issues or submit PRs for improvements.
