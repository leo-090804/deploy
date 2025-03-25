# Spring Boot Application trên Kubernetes

Dự án này là một ví dụ về cách triển khai ứng dụng Spring Boot lên Kubernetes, được xây dựng dựa trên hướng dẫn chính thức của Spring. Dự án bao gồm hai service chính:

1. `hello-service`: Service cung cấp API REST cơ bản trả về lời chào "Hello World"
2. `hello-caller`: Service gọi API từ hello-service để minh họa cách service discovery hoạt động trong môi trường Kubernetes

## Yêu cầu hệ thống

- Docker Desktop (cung cấp cả Docker và Kubernetes)
- Kubernetes CLI (kubectl)
- Maven
- JDK 17 trở lên

## Cấu trúc dự án

```
top-spring-on-kubernetes/
├── hello-caller/           # Service gọi API
│   ├── src/               # Source code
│   ├── k8s/               # Kubernetes manifests
│   └── pom.xml           # Maven configuration
└── hello-service/         # Service cung cấp API
    ├── src/               # Source code
    ├── k8s/               # Kubernetes manifests
    └── pom.xml           # Maven configuration
```

## CI/CD với GitHub Actions

Dự án sử dụng GitHub Actions để tự động build và push Docker images lên GitHub Container Registry (ghcr.io) mỗi khi có push hoặc pull request vào nhánh main.

Workflow sẽ:
1. Build Docker images cho cả hai service
2. Push images lên GitHub Container Registry
3. Tự động cập nhật README với tags của images mới nhất

### Latest Docker Images

Latest images are available at:
- hello-service: `ghcr.io/datfan06/top-spring-on-kubernetes-hello-service:latest`
- hello-caller: `ghcr.io/datfan06/top-spring-on-kubernetes-hello-caller:latest`

## Hướng dẫn chạy

### 1. Build Docker Image (Local)

Build image cho hello-service:
```bash
cd hello-service
./mvnw spring-boot:build-image -Dspring-boot.build-image.imageName=hello-service:latest
```

Build image cho hello-caller:
```bash
cd hello-caller
./mvnw spring-boot:build-image -Dspring-boot.build-image.imageName=hello-caller:latest
```

### 2. Kiểm tra Docker Image

```bash
docker images | grep hello
```

### 3. Triển khai lên Kubernetes

Triển khai hello-service:
```bash
kubectl apply -f hello-service/k8s/
```

Triển khai hello-caller:
```bash
kubectl apply -f hello-caller/k8s/
```

### 4. Kiểm tra trạng thái

```bash
kubectl get all
```

### 5. Truy cập ứng dụng

Để truy cập hello-caller service từ bên ngoài cluster:
```bash
kubectl port-forward svc/hello-caller 8080:8080
```

Sau đó truy cập: http://localhost:8080/call-hello

## Demo

### 1. Build Docker Image
![Docker Build Result](hello-caller/src/main/resources/img/DockerBuildResult.png)

### 2. Kiểm tra các resource trên Kubernetes
![Kubectl Get All](hello-caller/src/main/resources/img/KubectlGetAll.png)

### 3. Kết quả chạy ứng dụng
![Kubeclt Hello World](hello-caller/src/main/resources/img/KubecltHelloWorld.png)

## API Endpoints

### Hello Service
- GET `/hello`: Trả về lời chào từ service
- GET `/actuator/health`: Kiểm tra trạng thái sức khỏe của service

### Hello Caller
- GET `/call-hello`: Gọi API từ hello-service và trả về kết quả

## Best Practices đã áp dụng

1. Sử dụng Cloud Native Buildpacks để tạo Docker image
2. Cấu hình readiness và liveness probes
3. Sử dụng Kubernetes ConfigMaps cho cấu hình
4. Service discovery trong Kubernetes cluster
5. Graceful shutdown
6. CI/CD tự động với GitHub Actions

## Công nghệ sử dụng

- Spring Boot
- Docker
- Kubernetes
- Maven
- Java 17
- Cloud Native Buildpacks
- GitHub Actions
- GitHub Container Registry

## License

MIT

## Cấu hình môi trường

Dự án sử dụng Kubernetes ConfigMap để quản lý các biến môi trường. Các file cấu hình được đặt trong thư mục `k8s` của mỗi service:

### Hello Service ConfigMap
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: hello-service-config
data:
  SPRING_APPLICATION_NAME: hello-service
  SERVER_PORT: "8080"
  GREETING_MESSAGE: "Hello from hello-service!"
  MANAGEMENT_ENDPOINTS_WEB_EXPOSURE_INCLUDE: "health,info"
  MANAGEMENT_ENDPOINT_HEALTH_PROBES_ENABLED: "true"
  MANAGEMENT_ENDPOINT_HEALTH_SHOW_DETAILS: "always"
```

### Hello Caller ConfigMap
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: hello-caller-config
data:
  SPRING_APPLICATION_NAME: hello-caller
  SERVER_PORT: "8080"
  HELLO_SERVICE_URL: "http://hello-service:8080"
  MANAGEMENT_ENDPOINTS_WEB_EXPOSURE_INCLUDE: "health,info"
  MANAGEMENT_ENDPOINT_HEALTH_PROBES_ENABLED: "true"
  MANAGEMENT_ENDPOINT_HEALTH_SHOW_DETAILS: "always"
```

Để áp dụng cấu hình môi trường:

```bash
# Áp dụng ConfigMap cho hello-service
kubectl apply -f hello-service/k8s/configmap.yaml

# Áp dụng ConfigMap cho hello-caller
kubectl apply -f hello-caller/k8s/configmap.yaml
```
