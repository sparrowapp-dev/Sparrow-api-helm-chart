
# Sparrow API Helm Chart

This repository contains the Helm chart for deploying the `sparrow-api` application. The chart allows for easy deployment, scaling, and configuration of the API on Kubernetes clusters.

## Prerequisites

Before installing the `sparrow-api` Helm chart, make sure the following dependencies are set up:

1. **MongoDB**: Install MongoDB using Helm.
   ```bash
   helm repo add bitnami https://charts.bitnami.com/bitnami
   helm install sparrow-mongodb bitnami/mongodb --namespace sparrow-dev
   ```

2. **Kafka**: Install Kafka using Helm.
   ```bash
   helm repo add bitnami https://charts.bitnami.com/bitnami
   helm install sparrow-kafka bitnami/kafka --namespace sparrow-dev
   ```

Ensure that MongoDB and Kafka services are running and retrieve the connection URLs for each service, as they are required to configure the `sparrow-api` chart.

## Installing the Chart

1. **Add the Sparrow Helm Repository** (if hosted externally):
   ```bash
   helm repo add sparrow-api https://sparrowapp-dev.github.io/Sparrow-api-helm-chart
   helm repo update
   ```

2. **Install the Chart**:
   Replace `<namespace>` with your desired namespace (e.g., `sparrow-dev`).

   ```bash
   helm upgrade --install sparrow-api sparrow-api/sparrow-api --namespace <namespace>
   ```

### Configuring MongoDB and Kafka URLs

To connect the chart to MongoDB and Kafka, provide their URLs using the `--set` flag or by updating the `values.yaml` file.

#### Example with `--set`
```bash
helm install sparrow-api sparrow-api/sparrow-api --namespace sparrow-dev   --set secrets.DB_URL="mongodb://<username>:<password>@mongodb-service/"   --set secrets.KAFKA_BROKER="<KAFKA_URL>:9092"
```

### Configuration

The following table lists common configurable parameters of the `sparrow-api` chart and their default values in `values.yaml`:

| Parameter                             | Description                                          | Default                     |
|---------------------------------------|------------------------------------------------------|-----------------------------|
| `replicaCount`                        | Number of replicas for the API                       | `1`                         |
| `autoscaling.enabled`                 | Enable horizontal pod autoscaling                    | `true`                      |
| `autoscaling.minReplicas`             | Minimum number of replicas                           | `1`                         |
| `autoscaling.maxReplicas`             | Maximum number of replicas                           | `10`                        |
| `autoscaling.targetCPUUtilizationPercentage` | Target CPU usage percentage for scaling       | `70`                        |
| `image.repository`                    | Docker image repository for sparrow-api              | `sparrowapi/sparrow-api`    |
| `image.tag`                           | Image tag to use                                     | `v1`                        |
| `service.port`                        | Port on which the service is exposed                 | `80`                        |
| `service.targetPort`                  | Port the container listens on                        | `9000`                      |
| `secrets.DB_URL`                      | MongoDB connection URL                               | (empty)                     |
| `secrets.KAFKA_BROKER`                | Kafka broker URL                                     | (empty)                     |
| `resources.requests.memory`           | Minimum memory guaranteed for the pod                | `1024Mi`                     |
| `resources.limits.memory`             | Maximum memory allowed for the pod                   | `1648Mi`                    |

### Example `values.yaml`

Here's an example `values.yaml` configuration file:

```yaml
replicaCount: 1

autoscaling:
  enabled: true
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70

image:
  repository: sparrowapi/sparrow-api
  pullPolicy: IfNotPresent
  tag: v1

service:
  type: ClusterIP
  port: 80
  targetPort: 9000

secrets:
  DB_URL: "mongodb://<username>:<password>@mongodb-service/"
  KAFKA_BROKER: "<KAFKA_URL>:9092"

resources:
  requests:
    memory: 560Mi
  limits:
    memory: 1024Mi
```

## Accessing the Application

1. **Using Port Forwarding** (for ClusterIP services):

   ```bash
   export POD_NAME=$(kubectl get pods --namespace sparrow-dev -l "app.kubernetes.io/name=sparrow-api" -o jsonpath="{.items[0].metadata.name}")
   kubectl port-forward $POD_NAME 8080:9000 --namespace sparrow-dev
   ```

2. **Using LoadBalancer IP** (for LoadBalancer services):

   ```bash
   kubectl get svc --namespace sparrow-dev sparrow-api -o jsonpath="{.status.loadBalancer.ingress[0].ip}"
   ```

   Copy the IP address and access your application at `http://<LoadBalancer_IP>:80`.


## Uninstalling the Chart

To remove the `sparrow-api` release, use the following command:

```bash
helm uninstall sparrow-api --namespace sparrow-dev
```

This will remove all resources associated with the Helm release.

---

Feel free to reach out for any additional guidance or information!