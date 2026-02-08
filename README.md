# Distributed Microservices Architecture

A production-grade distributed microservices system built with Go, demonstrating enterprise patterns for service communication, authentication, logging, and event-driven architecture.

## Learning & Exploration

Go microservices ecosystem exploring HTTP, RPC, gRPC, and event-driven communication patterns. Built following Trevor Sawler's course to understand when distributed architecture is and isn't warranted.

## Overview

This project showcases a complete microservices ecosystem with multiple communication patterns (HTTP, RPC, gRPC, RabbitMQ), demonstrating real-world distributed system design. The architecture features a broker service acting as an API gateway, coordinating requests across authentication, logging, and mail services through various protocols.

## Architecture

```
┌─────────────────┐
│   Front-End     │
│   (Port 8082)   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Broker Service │ ◄─── API Gateway & Request Router
│   (Port 8080)   │
└────────┬────────┘
         │
         ├──────────────┬──────────────┬──────────────┐
         ▼              ▼              ▼              ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│    Auth      │ │    Logger    │ │     Mail     │ │   Listener   │
│ Service      │ │   Service    │ │   Service    │ │   Service    │
│ (Port 8081)  │ │ (Port 8082)  │ │ (Port 8083)  │ │  (RabbitMQ)  │
└──────┬───────┘ └──────┬───────┘ └──────┬───────┘ └──────┬───────┘
       │                │                │                │
       ▼                ▼                ▼                ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│  PostgreSQL  │ │   MongoDB    │ │   MailHog    │ │  RabbitMQ    │
│  (Port 5432) │ │ (Port 27017) │ │ (Port 1025)  │ │ (Port 5672)  │
└──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘
```

## Key Features

### Multi-Protocol Communication

- **HTTP/REST** - Traditional RESTful API calls
- **RPC** - Go's native remote procedure calls
- **gRPC** - High-performance protocol buffer-based RPC
- **AMQP/RabbitMQ** - Asynchronous event-driven messaging

### Core Services

- **Broker Service** - API gateway routing requests to appropriate microservices
- **Authentication Service** - User authentication with bcrypt password hashing
- **Logger Service** - Centralized logging with MongoDB persistence
- **Mail Service** - SMTP email sending with HTML templates
- **Listener Service** - RabbitMQ consumer for event-driven workflows

### Enterprise Patterns

- Service discovery and routing
- Event-driven architecture
- Centralized logging
- Database per service
- API gateway pattern
- Message queue integration

## Technology Stack

**Backend**

- Go 1.18
- Chi Router (HTTP mux)
- gRPC & Protocol Buffers
- Go RPC

**Databases**

- PostgreSQL 14.0 (user data)
- MongoDB 4.2.16 (logs)

**Message Queue**

- RabbitMQ 3.9
- AMQP Protocol

**Infrastructure**

- Docker & Docker Compose
- MailHog (email testing)

**Key Libraries**

- `go-chi/chi` - HTTP routing
- `jackc/pgx` - PostgreSQL driver
- `mongo-driver` - MongoDB client
- `rabbitmq/amqp091-go` - RabbitMQ client
- `golang.org/x/crypto` - bcrypt hashing
- `xhit/go-simple-mail` - SMTP client

## Prerequisites

- Docker Desktop 4.0+
- Docker Compose 2.0+
- Go 1.18+ (for local development)
- Make (optional, for convenience commands)

## Quick Start

### 1. Clone the Repository

```bash
git clone <repository-url>
cd distributed-microservices
```

### 2. Start All Services

```bash
cd project
make up_build
```

This will:

- Build all Docker images
- Start all microservices
- Initialize PostgreSQL with user schema
- Start MongoDB for logging
- Launch RabbitMQ message broker
- Start MailHog for email testing

### 3. Access the Application

- **Front-End UI**: <http://localhost:8082>
- **Broker API**: <http://localhost:8080>
- **Authentication API**: <http://localhost:8081>
- **RabbitMQ Management**: <http://localhost:15672> (guest/guest)
- **MailHog UI**: <http://localhost:8025>

### 4. Test the System

Open the front-end UI and use the test buttons to:

- Test broker connectivity
- Authenticate user (<admin@example.com> / verysecret)
- Send logs via HTTP
- Send logs via RabbitMQ
- Send logs via gRPC
- Send test emails

## Service Documentation

### Broker Service

**Port**: 8080

**Endpoints**:

```bash
# Main request handler
POST /handle
Content-Type: application/json

{
  "action": "auth|log|mail",
  "auth": {
    "email": "admin@example.com",
    "password": "verysecret"
  },
  "log": {
    "name": "event",
    "data": "log message"
  },
  "mail": {
    "from": "sender@example.com",
    "to": "recipient@example.com",
    "subject": "Test Email",
    "message": "Email body"
  }
}

# Test endpoint
POST /
Content-Type: application/json

{
  "message": "test"
}

# gRPC logging
POST /log-grpc
Content-Type: application/json

{
  "name": "event",
  "data": "log message"
}
```

### Authentication Service

**Port**: 8081

**Endpoints**:

```bash
# Authenticate user
POST /authenticate
Content-Type: application/json

{
  "email": "admin@example.com",
  "password": "verysecret"
}

# Response
{
  "error": false,
  "message": "Logged in to authentication service",
  "data": {
    "id": 1,
    "email": "admin@example.com",
    "first_name": "Admin",
    "last_name": "User",
    "active": 1
  }
}
```

**Database**: PostgreSQL

**Default Test User**:

- Email: `admin@example.com`
- Password: `verysecret`

### Logger Service

**Port**: 8082 (HTTP), 5001 (RPC), 50001 (gRPC)

**HTTP Endpoints**:

```bash
# Write log entry
POST /log
Content-Type: application/json

{
  "name": "event",
  "data": "log message"
}
```

**RPC Methods**:

- `RPCServer.LogInfo(payload, reply)` - Write log via RPC

**gRPC Methods**:

- `LogService/WriteLog` - Write log via gRPC

**Database**: MongoDB

- Collection: `logs`
- Fields: `_id`, `name`, `data`, `created_at`, `updated_at`

### Mail Service

**Port**: 8083

**Endpoints**:

```bash
# Send email
POST /send
Content-Type: application/json

{
  "from": "sender@example.com",
  "to": "recipient@example.com",
  "subject": "Test Email",
  "message": "Email body content"
}
```

**Email Server**: MailHog (development)

- SMTP Port: 1025
- Web UI: <http://localhost:8025>

### Listener Service

**No HTTP Port** (background service)

**Function**:

- Consumes messages from RabbitMQ
- Listens on exchange: `logs_topic`
- Subscribes to topics: `log.INFO`, `log.WARNING`, `log.ERROR`
- Forwards to logger service

## Communication Patterns

### 1. Synchronous HTTP

```go
// Broker calls Authentication Service
resp, err := http.Post(
    "http://authentication-service:8081/authenticate",
    "application/json",
    bytes.NewBuffer(jsonData),
)
```

### 2. RPC (Remote Procedure Call)

```go
// Broker calls Logger via RPC
conn, err := rpc.Dial("tcp", "logger-service:5001")
var result string
err = conn.Call("RPCServer.LogInfo", payload, &result)
```

### 3. gRPC (Protocol Buffers)

```go
// Broker calls Logger via gRPC
conn, err := grpc.Dial("logger-service:50001", grpc.WithTransportCredentials(insecure.NewCredentials()))
client := logs.NewLogServiceClient(conn)
_, err = client.WriteLog(ctx, &logs.LogRequest{
    LogEntry: &logs.Log{
        Name: req.Log.Name,
        Data: req.Log.Data,
    },
})
```

### 4. Message Queue (RabbitMQ)

```go
// Broker publishes to RabbitMQ
err = ch.Publish(
    "logs_topic",  // exchange
    "log.INFO",    // routing key
    false,
    false,
    amqp.Publishing{
        ContentType: "text/plain",
        Body:        jsonData,
    },
)

// Listener consumes from RabbitMQ
msgs, err := ch.Consume(
    q.Name,
    "",
    true,
    false,
    false,
    false,
    nil,
)
```

## Development

### Building Individual Services

```bash
cd project

# Build broker
make build_broker

# Build authentication
make build_auth

# Build logger
make build_logger

# Build mail
make build_mail

# Build listener
make build_listener
```

### Running Locally (without Docker)

```bash
# Start dependencies
dockercompose up postgres mongo rabbitmq mailhog

# Run services
cd authentication-service
go run cmd/api/main.go

cd logger-service
go run cmd/api/main.go

cd mail-service
go run cmd/api/main.go

cd listener-service
go run main.go

cd broker-service
go run cmd/api/main.go

cd front-end
go run cmd/web/main.go
```

### Make Commands

```bash
make up              # Start containers without rebuild
make up_build        # Rebuild and start containers
make down            # Stop all containers
make build_broker    # Build broker service
make build_auth      # Build authentication service
make build_logger    # Build logger service
make build_mail      # Build mail service
make build_listener  # Build listener service
make start           # Start front-end
make stop            # Stop front-end
```

## Database Schemas

### PostgreSQL - Users Table

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    first_name VARCHAR(255) NOT NULL,
    last_name VARCHAR(255) NOT NULL,
    password VARCHAR(60) NOT NULL,
    user_active INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### MongoDB - Logs Collection

```json
{
  "_id": "ObjectId",
  "name": "string",
  "data": "string",
  "created_at": "ISODate",
  "updated_at": "ISODate"
}
```

## Testing

### Manual Testing via UI

1. Navigate to <http://localhost:8082>
2. Use the provided test buttons:
   - **Test Broker** - Verify broker connectivity
   - **Test Auth** - Test authentication with default user
   - **Test Log** - Send log via HTTP
   - **Test Log via RabbitMQ** - Send log via message queue
   - **Test Log via gRPC** - Send log via gRPC
   - **Test Mail** - Send test email

### API Testing with cURL

```bash
# Test authentication
curl -X POST http://localhost:8080/handle \
  -H "Content-Type: application/json" \
  -d '{
    "action": "auth",
    "auth": {
      "email": "admin@example.com",
      "password": "verysecret"
    }
  }'

# Test logging
curl -X POST http://localhost:8080/handle \
  -H "Content-Type: application/json" \
  -d '{
    "action": "log",
    "log": {
      "name": "test-event",
      "data": "Test log message"
    }
  }'

# Test email
curl -X POST http://localhost:8080/handle \
  -H "Content-Type: application/json" \
  -d '{
    "action": "mail",
    "mail": {
      "from": "test@example.com",
      "to": "recipient@example.com",
      "subject": "Test Email",
      "message": "This is a test email"
    }
  }'
```

## Monitoring & Debugging

### RabbitMQ Management Console

- URL: <http://localhost:15672>
- Username: `guest`
- Password: `guest`
- View queues, exchanges, message rates, and connections

### MailHog Email Testing

- URL: <http://localhost:8025>
- View all emails sent by the mail service
- Test email templates without sending real emails

### MongoDB Express (optional)

Add to docker-compose.yml for MongoDB GUI:

```yaml
mongo-express:
  image: mongo-express
  ports:
    - "8888:8081"
  environment:
    ME_CONFIG_MONGODB_URL: mongodb://mongo:27017/
```

## Troubleshooting

### Services won't start

```bash
# Clean everything and rebuild
make down
docker system prune -a
make up_build
```

### PostgreSQL connection issues

```bash
# Check if PostgreSQL is running
docker ps | grep postgres

# View logs
docker logs postgres
```

### RabbitMQ connection issues

```bash
# Restart RabbitMQ
docker restart rabbitmq

# Check management console
open http://localhost:15672
```

### Port conflicts

If ports are already in use, modify `docker-compose.yml`:

```yaml
services:
  broker-service:
    ports:
      - "8080:80" # Change 8080 to available port
```

## Project Structure

```
distributed-microservices/
├── authentication-service/
│   ├── cmd/api/          # Application entry point
│   ├── data/             # Database models and queries
│   └── Dockerfile
├── broker-service/
│   ├── cmd/api/          # Application entry point
│   ├── event/            # RabbitMQ event handling
│   ├── logs/             # gRPC proto definitions
│   └── Dockerfile
├── logger-service/
│   ├── cmd/api/          # Application entry point
│   ├── data/             # MongoDB models
│   ├── logs/             # gRPC implementation
│   └── Dockerfile
├── mail-service/
│   ├── cmd/api/          # Application entry point
│   ├── templates/        # Email templates
│   └── Dockerfile
├── listener-service/
│   ├── event/            # RabbitMQ consumer
│   ├── main.go
│   └── Dockerfile
├── front-end/
│   ├── cmd/web/          # Web server
│   ├── templates/        # HTML templates
│   └── Dockerfile
└── project/
    ├── docker-compose.yml # Service orchestration
    ├── Makefile          # Build automation
    └── users.sql         # Database initialization
```

## Learning Objectives

This project demonstrates:

1. **Microservices Architecture** - Independent, loosely coupled services
2. **API Gateway Pattern** - Single entry point for client requests
3. **Service Communication** - Multiple protocols (HTTP, RPC, gRPC, AMQP)
4. **Event-Driven Design** - Asynchronous messaging with RabbitMQ
5. **Database per Service** - Each service owns its data
6. **Containerization** - Docker for consistent environments
7. **Service Orchestration** - Docker Compose for multi-container apps
8. **Security Practices** - Password hashing, secure communication
9. **Protocol Buffers** - Efficient service definitions
10. **Message Patterns** - Publish/subscribe, request/reply

## Performance Considerations

### When to Use Each Protocol

- **HTTP** - Simple CRUD, external APIs, debugging
- **RPC** - Internal service calls, Go-to-Go communication
- **gRPC** - High-performance requirements, polyglot services
- **RabbitMQ** - Async operations, decoupling, event broadcasting

### Scaling Services

```yaml
# In docker-compose.yml
services:
  broker-service:
    deploy:
      replicas: 3 # Run 3 instances
```

Add load balancer (nginx, traefik) for production deployments.

## Security Best Practices

- Passwords hashed with bcrypt (cost factor 12)
- Environment variables for sensitive configuration
- No hardcoded credentials in code
- Database credentials managed via Docker secrets (production)
- CORS properly configured
- Input validation on all endpoints

## Production Deployment

For production deployment, consider:

1. **Service Discovery** - Consul, etcd
2. **Load Balancing** - nginx, HAProxy, Traefik
3. **Orchestration** - Kubernetes, Docker Swarm
4. **Monitoring** - Prometheus, Grafana
5. **Logging** - ELK Stack, Loki
6. **Tracing** - Jaeger, Zipkin
7. **API Gateway** - Kong, Tyk
8. **Secrets Management** - Vault, AWS Secrets Manager
9. **CI/CD** - GitHub Actions, GitLab CI, Jenkins
10. **HTTPS** - Let's Encrypt certificates

## Contributing

Contributions are welcome! Please follow these guidelines:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgments

- Built with Go and the amazing Go community libraries
- Inspired by microservices best practices
- RabbitMQ for reliable messaging
- Docker for containerization

## Support

For questions, issues, or contributions:

- Open an issue on GitHub
- Check existing documentation
- Review Docker and service logs

---

**Happy Coding!** 🚀
