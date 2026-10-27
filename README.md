
# Campus Equipment Booking — DevOps Lab

## 1. Project Overview

This project is a Spring Boot REST API for managing campus equipment and booking requests. The lab covers Git-based development, Maven build and testing, Docker containerization, and Kubernetes deployment, scaling, and self-healing.

**Technology stack**
- Java 17
- Spring Boot 3.3.5
- Maven
- Docker
- Kubernetes with Minikube
- Git and GitHub

## 2. Setup and Execution

### Prerequisites

Install Java 17, Maven, Git, and Docker. For Kubernetes tasks, install `kubectl` and Minikube.

Clone the repository and switch to the feature branch:

```bash
git clone https://github.com/Varsha-S2005/Campus-Equipment-Booking.git
cd Campus-Equipment-Booking-Student-Starter
git checkout feature/booking-validation
```

If the cloned repository uses a different root directory, enter the directory containing `pom.xml`.

### Build, test, and package

```bash
mvn clean
mvn test
mvn clean package
```

- `mvn clean` removes previous build output.
- `mvn test` executes the automated tests.
- `mvn clean package` compiles the application, runs the configured tests, and packages it into an executable JAR under `target/`.

Run the packaged application locally:

```bash
java -jar target/campus-equipment-booking-1.0.0.jar
```

The application listens on port `8080` by default.

## 3. Git Evidence

**Feature branch:** `feature/booking-validation`

The work was developed on a feature branch and pushed to GitHub.

Display the branch and commit history:

```bash
git branch
git status
git log --oneline --decorate -10
git diff
```

Use the actual output of these commands as Git evidence. The commit history should show the changes made during the lab.

## 4. Maven Evidence

The project uses `pom.xml` to configure dependencies, Java compatibility, and the Maven build.

Commands used:

```bash
mvn test
mvn clean package
ls -lh target/campus-equipment-booking-1.0.0.jar
```

The baseline test run completed with **6 tests passing**. Record the final test count, failures, and errors from the final `mvn test` or `mvn clean package` output after the code changes.

**Package result:** The executable JAR was generated at:

`target/campus-equipment-booking-1.0.0.jar`

## 5. Code Changes

### Inventory validation defect

In `BookingService.java`, the original inventory check allowed a request to exceed available inventory by one because it added `1` to the available quantity.

The validation was corrected to compare the requested quantity directly against the available quantity:

```java
if (quantity > e.getAvailableQuantity()) {
    throw new IllegalStateException(
        "Requested quantity exceeds available inventory"
    );
}
```

This prevents bookings from exceeding the available equipment inventory.

### Maximum booking quantity

The assigned change request limits an individual booking to a maximum quantity of two. The validation rejects requests exceeding that limit with an `IllegalArgumentException`.

## 6. Docker Evidence

### Dockerfile

```dockerfile
FROM eclipse-temurin:17-jre
WORKDIR /app
COPY target/campus-equipment-booking-1.0.0.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

The Dockerfile uses a Java 17 runtime image, copies the packaged JAR into the image, and starts the Spring Boot application.

### Build and run

Build the Docker image:

```bash
mvn clean package
docker build -t campus-equipment-booking:set-a .
```

Run the container:

```bash
docker run -d \
  --name campus-booking-set-a \
  -p 8081:8080 \
  campus-equipment-booking:set-a
```

Host port `8081` is mapped to container port `8080` because port `8080` on the host was already occupied by Jenkins.

Check the container:

```bash
docker ps
docker logs campus-booking-set-a
```

### API and health verification

Check the equipment endpoint:

```bash
curl -i http://localhost:8081/api/equipment
```

Create a booking:

```bash
curl -i -X POST http://localhost:8081/api/bookings \
  -H "Content-Type: application/json" \
  -d '{"borrowerId":"STU1001","equipmentId":"LAPTOP01","quantity":2}'
```

The successful request returned HTTP `201 Created` with a confirmed booking.

Test invalid quantity validation:

```bash
curl -i -X POST http://localhost:8081/api/bookings \
  -H "Content-Type: application/json" \
  -d '{"borrowerId":"STU1001","equipmentId":"LAPTOP01","quantity":0}'
```

The invalid request returned HTTP `400 Bad Request`.

Check application health:

```bash
curl -i http://localhost:8081/actuator/health
```

The health endpoint returned HTTP `200 OK` with status `UP`.

## 7. Kubernetes Evidence

Minikube was used to deploy the Docker image to a local Kubernetes cluster.

Start the cluster:

```bash
minikube start
kubectl get nodes
```

### Deployment

The `deployment.yaml` manifest defines the application Deployment, its labels and selector, the container image, and the container port.

The initial deployment used two replicas.

Apply and verify:

```bash
kubectl apply -f deployment.yaml
kubectl get deployments
kubectl get pods
```

### Service

The application was exposed through a NodePort Service:

```bash
kubectl expose deployment campus-booking \
  --type=NodePort \
  --port=8080 \
  --target-port=8080
```

The Service was saved to `service.yaml`:

```bash
kubectl get service campus-booking -o yaml > service.yaml
```

Inspect the Service:

```bash
kubectl get services
kubectl get service campus-booking
```

The Service exposed port `8080` through NodePort `30740` in the recorded cluster output.

### Verify the application through Kubernetes

```bash
minikube service campus-booking --url
```

Use the URL returned by that command:

```bash
curl -i "$(minikube service campus-booking --url)/actuator/health"
curl -i "$(minikube service campus-booking --url)/api/equipment"
```

Both endpoints returned HTTP `200 OK`; the health endpoint reported status `UP`.

### Scaling

Scale the Deployment from two replicas to three:

```bash
kubectl scale deployment campus-booking --replicas=3
kubectl get deployments
kubectl get pods
```

The Deployment reached `3/3` ready replicas.

### Self-healing

Delete one of the application Pods:

```bash
kubectl delete pod <pod-name>
kubectl get pods
```

Replace `<pod-name>` with an actual Pod name returned by `kubectl get pods`.

Kubernetes created a replacement Pod, restoring the desired replica count.

### How the Deployment controller works

A Kubernetes Deployment declares the desired state of an application, including the number of replicas. Its controller continuously compares the actual state with the desired state. If a Pod is deleted, the controller arranges for a replacement to be created. Changing the replica count updates the desired state, allowing Kubernetes to scale the application.

## 8. Modified and Submitted Files

The submission includes:

- `src/main/java/edu/university/equipment/BookingService.java` — booking and inventory validation.
- `src/test/java/edu/university/equipment/BookingServiceTest.java` — automated tests.
- `pom.xml` — Maven dependencies and build configuration.
- `Dockerfile` — container image definition.
- `deployment.yaml` — Kubernetes Deployment.
- `service.yaml` — Kubernetes NodePort Service.
- `README.md` — setup, execution, evidence, and technical explanation.

The remaining application source files and configuration are included as part of the project.

## 9. Known Limitations

- The Kubernetes manifests reference the local image `campus-equipment-booking:set-a`; another environment must have access to that image or an appropriately available image registry.
- The NodePort address and port depend on the local Minikube configuration.
- The final automated test count and Git commit hashes should be taken from the actual final command output and repository history.
- This submission demonstrates local development and deployment; it does not establish production readiness, external database persistence, or production-grade security.
