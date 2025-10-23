# Patient Management System

A comprehensive microservices-based patient management system built with Spring Boot, featuring REST APIs, gRPC communication, event-driven architecture with Kafka, and JWT-based authentication. **Currently working on** automating the entire cloud infrastructure (ECS, RDS, MSK) using Infrastructure as Code (IaC) with AWS CloudFormation and LocalStack.

## 🏗️ Architecture Overview

This system follows a microservices architecture pattern with the following components:

```
┌─────────────────┐
│   API Gateway   │ ← Entry point for all client requests
└────────┬────────┘
         │
    ┌────┴────┐
    │         │
┌───▼────┐ ┌──▼──────────┐
│  Auth  │ │   Patient   │
│Service │ │   Service   │
└────────┘ └──────┬──────┘
              │   │
         ┌────┘   └────────┐
         │                 │
    ┌────▼─────┐     ┌─────▼─────────┐
    │ Billing  │     │   Analytics   │
    │ Service  │     │    Service    │
    └──────────┘     └───────────────┘
       (gRPC)            (Kafka)
```

## 🚀 Services

### 1. **API Gateway** (Port: TBD)
- **Technology**: Spring Cloud Gateway
- **Purpose**: Central entry point that routes requests to appropriate microservices
- **Features**:
  - Request routing
  - Load balancing
  - API composition

### 2. **Patient Service** (Port: 8080)
- **Technology**: Spring Boot, JPA, PostgreSQL
- **Purpose**: Core service managing patient records and data
- **Features**:
  - RESTful CRUD operations for patient management
  - Patient registration and profile management
  - Kafka event publishing for patient events (CREATE, UPDATE, DELETE)
  - gRPC client integration with Billing Service
  - OpenAPI/Swagger documentation
- **Database**: PostgreSQL (with H2 in-memory option for development)
- **Communication**:
  - REST API for external clients
  - Kafka producer for analytics events
  - gRPC client for billing service

### 3. **Auth Service** (Port: 4005)
- **Technology**: Spring Boot, Spring Security, JWT
- **Purpose**: Authentication and authorization service
- **Features**:
  - User authentication with JWT tokens
  - Token validation and management
  - Password encryption with BCrypt
  - User registration and login
- **Database**: PostgreSQL

### 4. **Billing Service** (Port: 4001, gRPC: 9001)
- **Technology**: Spring Boot, gRPC
- **Purpose**: Manages billing accounts for patients
- **Features**:
  - Create billing accounts via gRPC
  - Synchronous communication using Protocol Buffers
  - High-performance RPC calls
- **Communication**: gRPC server

### 5. **Analytics Service**
- **Technology**: Spring Boot, Kafka
- **Purpose**: Processes and analyzes patient events
- **Features**:
  - Real-time event consumption from Kafka
  - Patient activity tracking and analytics
  - Event-driven data processing
- **Communication**: Kafka consumer

### 6. **Integration Tests**
- Comprehensive integration testing suite
- End-to-end testing across microservices

## 🛠️ Technology Stack

### Backend Framework
- **Java 17**
- **Spring Boot 3.5.6**
- **Spring Cloud 2025.0.0**
- **Maven** for dependency management

### Communication Protocols
- **REST API** - Patient and Auth services
- **gRPC 1.69.0** - High-performance RPC (Billing Service)
- **Apache Kafka** - Event streaming (Analytics)
- **Protocol Buffers 4.29.1** - Data serialization

### Security
- **Spring Security**
- **JWT (JSON Web Tokens)** - JJWT 0.12.6
- **BCrypt** password hashing

### Database
- **PostgreSQL** - Production database
- **H2** - In-memory database for development/testing
- **Spring Data JPA** - Data persistence

### Documentation & Testing
- **SpringDoc OpenAPI 2.8.13** - API documentation
- **JUnit** - Unit testing
- **Spring Test** - Integration testing

### DevOps
- **Docker** - Containerization
- **Multi-stage Dockerfiles** for optimized images
- **AWS CloudFormation** - Infrastructure as Code (IaC)
- **LocalStack** - Local AWS cloud environment for development

### Cloud Infrastructure
- **AWS ECS (Elastic Container Service)** - Container orchestration
- **AWS RDS (Relational Database Service)** - Managed PostgreSQL databases
- **AWS MSK (Managed Streaming for Kafka)** - Managed Kafka clusters
- **Infrastructure as Code** - Automated provisioning with CloudFormation templates
- **AWS CloudFormation** - Infrastructure as Code (IaC)
- **LocalStack** - Local AWS cloud environment for development

## 📋 Prerequisites

- **Java 17+** installed
- **Maven 3.9+** installed
- **PostgreSQL** database (or Docker)
- **Apache Kafka** (for Analytics Service)
- **Docker** (optional, for containerized deployment)
- **AWS CLI** (for CloudFormation deployment)
- **LocalStack** (for local AWS development and testing)

## 🚦 Getting Started

### 1. Clone the Repository
```bash
git clone <repository-url>
cd patient-management
```

### 2. Database Setup

#### PostgreSQL
Create databases for each service:
```sql
CREATE DATABASE patient_db;
CREATE DATABASE auth_db;
```

#### H2 (Development Only)
The Patient Service can use H2 in-memory database for development. Uncomment the H2 configuration in `patient-service/src/main/resources/application.properties`.

### 3. Kafka Setup
Start Kafka and Zookeeper (if using Analytics Service):
```bash
# Using Docker
docker run -d --name zookeeper -p 2181:2181 zookeeper
docker run -d --name kafka -p 9092:9092 \
  --link zookeeper \
  -e KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 \
  -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://localhost:9092 \
  confluentinc/cp-kafka
```

### 4. Build All Services
```bash
# Build all services from root directory
mvn clean install
```

Or build individual services:
```bash
cd patient-service
mvn clean package

cd ../auth-service
mvn clean package

# ... repeat for other services
```

### 5. Run Services

#### Option A: Run with Maven
Start each service in a separate terminal:

```bash
# Terminal 1 - Auth Service
cd auth-service
mvn spring-boot:run

# Terminal 2 - Patient Service
cd patient-service
mvn spring-boot:run

# Terminal 3 - Billing Service
cd billing-service
mvn spring-boot:run

# Terminal 4 - Analytics Service
cd analytics-service
mvn spring-boot:run

# Terminal 5 - API Gateway
cd api-gateway
mvn spring-boot:run
```

#### Option B: Run with Docker
```bash
# Build Docker images
docker build -t patient-service ./patient-service
docker build -t auth-service ./auth-service
docker build -t billing-service ./billing-service
docker build -t analytics-service ./analytics-service
docker build -t api-gateway ./api-gateway

# Run containers
docker run -d -p 8080:8080 patient-service
docker run -d -p 4005:4005 auth-service
docker run -d -p 4001:4001 -p 9001:9001 billing-service
docker run -d analytics-service
docker run -d -p 8000:8000 api-gateway
```

## 📚 API Documentation

### Patient Service API

**Base URL**: `http://localhost:8080`

#### Create Patient
```http
POST /patients
Content-Type: application/json

{
  "name": "John Doe",
  "email": "john.doe@example.com",
  "address": "123 Main St",
  "dateOfBirth": "1990-01-01",
  "registeredDate": "2024-01-01"
}
```

#### Get All Patients
```http
GET /patients
```

#### Get Patient by ID
```http
GET /patients/{id}
```

#### Update Patient
```http
PUT /patients/{id}
Content-Type: application/json

{
  "name": "John Doe Updated",
  "email": "john.updated@example.com",
  "address": "456 New St",
  "dateOfBirth": "1990-01-01",
  "registeredDate": "2024-01-01"
}
```

#### Delete Patient
```http
DELETE /patients/{id}
```

### Auth Service API

**Base URL**: `http://localhost:4005`

#### Login
```http
POST /auth/login
Content-Type: application/json

{
  "username": "user@example.com",
  "password": "password123"
}
```

#### Validate Token
```http
POST /auth/validate
Content-Type: application/json

{
  "token": "your-jwt-token"
}
```

### Billing Service (gRPC)

**Endpoint**: `localhost:9001`

#### Create Billing Account
```http
GRPC localhost:9001/BillingService.CreateBillingAccount

{
  "patientId": "12345",
  "name": "John Doe",
  "email": "john.doe@example.com"
}
```

### Swagger UI
Access interactive API documentation:
- Patient Service: `http://localhost:8080/swagger-ui.html`

## 🔄 Event Flow

### Patient Event Processing
1. Client creates/updates/deletes a patient via REST API
2. Patient Service processes the request and saves to database
3. Patient Service publishes event to Kafka topic
4. Analytics Service consumes event and processes analytics
5. Patient Service calls Billing Service via gRPC (if needed)
6. Billing Service creates billing account

### Event Types
- `PATIENT_CREATED`
- `PATIENT_UPDATED`
- `PATIENT_DELETED`

## 🧪 Testing

### Unit Tests
```bash
# Run tests for specific service
cd patient-service
mvn test
```

### Integration Tests
```bash
cd integration-tests
mvn verify
```

### API Testing
HTTP request files are provided in the `api-requests/` directory:
- `api-requests/patient-service/` - Patient Service requests
- `api-requests/auth-service/` - Auth Service requests
- `grpc-requests/billing-service/` - gRPC requests

Use IntelliJ IDEA's HTTP Client or similar tools to execute these requests.

## 📁 Project Structure

```
patient-management/
├── api-gateway/              # API Gateway service
├── patient-service/          # Patient management service
├── auth-service/            # Authentication service
├── billing-service/         # Billing management service
├── analytics-service/       # Analytics and reporting service
├── integration-tests/       # Integration test suite
├── api-requests/           # HTTP request files for testing
│   ├── patient-service/
│   └── auth-service/
└── grpc-requests/          # gRPC request files
    └── billing-service/
```

## 🔐 Security

- **JWT Authentication**: All protected endpoints require valid JWT tokens
- **Password Encryption**: User passwords are encrypted using BCrypt
## ☁️ Cloud Infrastructure (AWS) - Work in Progress

This project is **currently being developed** to include **Infrastructure as Code (IaC)** automation using AWS CloudFormation templates for complete cloud infrastructure provisioning.

### Automated Infrastructure Components

The CloudFormation templates automatically provision:

1. **AWS ECS (Elastic Container Service)**
   - ECS Cluster for container orchestration
   - Task definitions for each microservice
   - Service definitions with auto-scaling
   - Load balancers for traffic distribution

2. **AWS RDS (Relational Database Service)**
   - PostgreSQL instances for Patient Service
   - PostgreSQL instances for Auth Service
   - Automated backups and high availability
   - Security groups and network isolation

3. **AWS MSK (Managed Streaming for Kafka)**
   - Kafka cluster for event streaming
   - Topics for patient events
   - Brokers with multi-AZ deployment
   - Integration with Analytics Service


### Benefits of IaC Approach

- **Reproducibility**: Deploy identical environments consistently
- **Version Control**: Infrastructure changes tracked in Git
- **Cost Efficiency**: Use LocalStack for development without AWS charges
- **Scalability**: Easy to scale resources by modifying templates
- **Disaster Recovery**: Quick infrastructure recreation from templates

- **Token Validation**: Tokens are validated on each request to protected resources
- **HTTPS**: Should be configured for production deployment

## 🐳 Docker Support

Each service includes a multi-stage Dockerfile optimized for production:
- **Builder stage**: Compiles the application with Maven
- **Runner stage**: Creates lightweight runtime image with only JRE
- Base images use Eclipse Temurin OpenJDK 21

## 📊 Monitoring & Observability

Consider adding:
- Spring Boot Actuator for health checks
- Distributed tracing (Zipkin/Jaeger)
- Centralized logging (ELK stack)
- Metrics collection (Prometheus + Grafana)

## 🚧 Future Enhancements

- [⏳] Infrastructure as Code with AWS CloudFormation (In Progress)
- [⏳] AWS ECS, RDS, and MSK provisioning (In Progress)
- [⏳] LocalStack for local development (In Progress)
- [ ] Service discovery (Eureka)
- [ ] Centralized configuration (Spring Cloud Config)
- [ ] Rate limiting
- [ ] API versioning
- [ ] Comprehensive monitoring and alerting
- [ ] Kubernetes deployment manifests
- [ ] CI/CD pipeline configuration

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License.

## 👥 Contact

For questions or support, please contact the development team.

---

**Note**: This is a learning/demonstration project showcasing microservices architecture patterns, communication protocols (REST, gRPC, Kafka), and modern Spring Boot practices.

