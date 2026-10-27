# Campus Equipment Booking Service
Java 17 / Spring Boot REST API for a CI/CD lab assessment.

## Build and test
`mvn clean test`

## Run
`mvn spring-boot:run` — port 8080.

## Endpoints
- `GET /api/equipment`
- `GET /api/equipment/{id}`
- `POST /api/bookings`
- `GET /api/bookings/{id}`
- `POST /api/bookings/{id}/cancel`
- `GET /actuator/health`

Inventory is in memory and resets on restart. No database is required.

CI pipeline is triggered automatically through GitHub Webhooks.

## Kubernetes Deployment

The Campus Equipment Booking Service can be deployed to a local Kubernetes cluster using Minikube.

### Prerequisites
- Java 17
- Maven
- Docker
- Minikube
- kubectl

### Build and Test

Run the unit tests and package the application:

```bash
mvn clean test
mvn clean package
```

### Build the Docker Image

```bash
docker build -t campus-equipment-booking:set-e .
```

### Deploy to Minikube

Start the cluster:

```bash
minikube start --driver=docker
```

Load the Docker image into Minikube:

```bash
minikube image load campus-equipment-booking:set-e
```

Apply the Kubernetes resources:

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

### Verify the Deployment

Check the Deployment, Pods, and Service:

```bash
kubectl get deployments
kubectl get pods
kubectl get services
```

Test the application health endpoint:

```bash
curl -i http://$(minikube ip):30081/actuator/health
```

The expected response contains:

```json
{
  "status": "UP",
  "groups": [
    "liveness",
    "readiness"
  ]
}
```

### Scale the Application

Scale the application to two replicas:

```bash
kubectl scale deployment campus-equipment-booking --replicas=2
```

Verify the replicas:

```bash
kubectl get deployments
kubectl get pods
```

### Continuous Integration

Jenkins is configured to run the CI pipeline when GitHub sends a push webhook notification. The pipeline checks Java and Maven, runs tests, packages the application, and builds the Docker image.

The Jenkins webhook uses a temporary ngrok HTTPS endpoint for local development. The ngrok tunnel must remain active for webhook notifications to reach Jenkins.
