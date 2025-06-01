# Basic Setup Guide

This repository contains the basic guidelines and project structure for our backend service.

## Project Overview

This is an example of our project tree if later on we want to add new service 

## Project Structure
### current example is using go and need to adjust if later on we want use other programming language

```
└── auth-service/
    ├── cmd/                    # Application entry points
    │   ├── app/
    │   │   └── main.go        # Main application entry point
    │   └── server/
    │       ├── router.go      # HTTP router configuration
    │       └── health.go      # Health check endpoints
    │
    ├── configs/               # Configuration files
    │   ├── env.json          # Environment variables
    │   └── config.go         # Configuration loader
    │
    ├── database/             # Database connection and setup
    │   └── database.go       # Database initialization
    │
    ├── middlewares/          # HTTP middlewares
    │   └── authorization.go  # Authorization middleware
    │
    ├── docs/                 # Documentation
    │   └── swagger/         # API documentation
    │       ├── swagger.json
    │       ├── swagger.yml
    │       └── docs.go
    │
    ├── internal/            # Internal packages
    │   └── user/           # User domain
    │       ├── mocks/      # Mock implementations for testing
    │       │   ├── UserRepositoryGen.go
    │       │   └── UserUsecaseGen.go
    │       ├── entity/     # Domain entities
    │       │   └── user.go
    │       ├── delivery/   # Delivery layer (HTTP/gRPC handlers)
    │       │   ├── http/
    │       │   │   └── user_delivery.go
    │       │   └── grpc/
    │       │       └── user_delivery.go
    │       ├── usecase/    # Business logic
    │       │   └── user_usecase.go
    │       └── repository/ # Data access layer
    │           ├── pg/     # PostgreSQL implementation
    │           │   └── user_repository.go
    │           └── redis/  # Redis implementation
    │               └── user_repository.go
    │
    ├── pkg/                # Public packages
    │   └── helpers/       # Utility functions
    │       └── utils.go
    │
    ├── go.mod             # Go module definition
    ├── go.sum             # Go module checksums
    ├── Makefile          # Build and development commands
    └── README.md         # This file
```

#### the information will be updated accordingly later on
