# Campus Equipment Booking Service
Java 17 / Spring Boot REST API for a CI/CD lab assessment.

## Build and Test

### Prerequisites
- Java 17
- Maven
- Docker
- Minikube and `kubectl`

Run these commands from the project root.

**Run unit tests:**
```bash
mvn clean test
```

**Package the application:**
```bash
mvn clean package
```

The executable JAR is generated under `target/`.

## Docker Build and Run

Build the application JAR and Docker image:

```bash
mvn clean package
docker build -t campus-equipment-booking:set-e .
```

Start the container:

```bash
docker run -d --name campus-booking-container \
  -p 8081:8080 campus-equipment-booking:set-e
```

Port `8081` is used on the host because Jenkins is already using port `8080`.

Verify the running container and image:

```bash
docker ps
docker images campus-equipment-booking
```

Check application health:

```bash
curl -i http://localhost:8081/actuator/health
```

The expected response is HTTP `200 OK` with a health status of `UP`.

To stop and remove the container:

```bash
docker stop campus-booking-container
docker rm campus-booking-container
```

## Kubernetes Deployment

Start Minikube and verify that the cluster is ready:

```bash
minikube start --driver=docker
minikube status
kubectl get nodes
```

Build the application and Docker image, then make the image available inside Minikube:

```bash
mvn clean package
docker build -t campus-equipment-booking:set-e .
minikube image load campus-equipment-booking:set-e
```

Apply the Deployment and Service manifests:

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

Verify the deployment, pods, and service:

```bash
kubectl get deployments
kubectl get pods
kubectl get services
```

Wait for the application deployment to become ready:

```bash
kubectl rollout status deployment/campus-equipment-booking
kubectl get deployment campus-equipment-booking
```

Scale the application to two replicas:

```bash
kubectl scale deployment campus-equipment-booking --replicas=2
kubectl get deployment campus-equipment-booking
kubectl get pods -l app=campus-equipment-booking
```

The expected deployment status is `2/2` available replicas.

### Kubernetes Health Check

The Service exposes the application on NodePort `30081`. Retrieve the Minikube IP and check application health:

```bash
minikube ip
curl -i http://$(minikube ip):30081/actuator/health
```

The expected response is HTTP `200 OK` with a health status of `UP`.

The readiness and liveness probes use these endpoints:

- `/actuator/health/readiness`
- `/actuator/health/liveness`

## Assigned Change Request: Booking Validation

### Defect Fixed

The booking capacity validation had a boundary-condition defect. The service incorrectly rejected a booking when the requested quantity exactly matched the equipment's available quantity.

The validation was corrected to reject a request only when the requested quantity exceeds the available quantity:

```java
if (quantity > e.getAvailableQuantity()) {
    throw new IllegalArgumentException(...);
}
```

Positive quantity validation was also added to the booking request using `@NotNull` and `@Min(1)`, preventing zero or negative quantities from being accepted.

### Test Coverage

The unit tests verify that:

- A booking that exactly matches the available quantity is accepted.
- A booking exceeding the available quantity is rejected.
- Rejected over-capacity requests do not create bookings or modify inventory.
- Positive quantities are accepted when other booking conditions are satisfied.
- Zero and negative quantities are rejected.
- Invalid quantity requests do not create bookings or modify inventory.

Run the tests with:

```bash
mvn clean test
```

## CI Pipeline

The Jenkins pipeline checks Java and Maven, runs the Maven tests, packages the application, builds the Docker image, and verifies the image.

The Jenkins job is configured to respond to GitHub push webhooks for the `feature/positive-quantity-validation` branch.

A successful pipeline ends with:

```text
Finished: SUCCESS
```

## Evidence to Submit

Capture concise terminal output or screenshots showing:

1. **Git:** current branch, recent commit history, and clean working tree.
2. **Maven:** successful test results from `mvn clean test`.
3. **Docker:** the `campus-equipment-booking:set-e` image, running container, and HTTP 200 health-check response.
4. **Kubernetes:** Deployment, pods, and Service; two available replicas; HTTP 200 health-check response.
5. **Jenkins:** a successful pipeline build and, if required, a GitHub webhook delivery that triggered a build.
