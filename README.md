# Athena Protobufs

This repository contains protobuf files relating to the gRPC interface for the
Athena API (Crisp CSAM Detection).

## Overview

Athena is a gRPC-based image classification service designed for CSAM (Child Sexual Abuse Material) detection by Crisp. The service provides real-time image classification through bidirectional streaming with session-based deployment management and multi-affiliate support.

## Structure

The protobuf definitions are organized in a modular structure:

### `athena/athena.proto`
Contains the service definitions for the Athena API:
- `ClassifierService`: Main gRPC service with streaming classification and deployment management

### `athena/models.proto`
Contains all message and enum definitions used by the service:
- Request/Response messages (`ClassifyRequest`, `ClassifyResponse`, etc.)
- Data models (`Classification`, `ClassificationInput`, etc.)
- Enums (`ErrorCode`, `ImageFormat`, `RequestEncoding`, `HashType`)

This separation allows:
- **Shared Model Libraries**: Clients can generate code only for models without service stubs
- **Cleaner Dependencies**: Services can import models without circular dependencies
- **Better Maintainability**: Models and services can evolve independently
- **Language-specific Benefits**: Some languages can benefit from separate model packages

## Features

- **Real-time Classification**: Bidirectional streaming for immediate image processing
- **Session Management**: Deployment-based grouping enables collaborative processing
- **Multi-format Support**: Supports JPEG, PNG, WebP, TIFF, and many other image formats
- **Compression**: Optional Brotli compression for bandwidth optimization
- **Error Handling**: Comprehensive error codes and detailed error messages
- **Monitoring**: Active deployment tracking and backlog monitoring

## Usage

### Generating Code

When generating code from these protobuf files, you'll need to ensure both files are included in your compilation:

```bash
# Example: Generate Python code
protoc --python_out=. --grpc_python_out=. \
  athena/models.proto athena/athena.proto

# Example: Generate Go code
protoc --go_out=. --go-grpc_out=. \
  athena/models.proto athena/athena.proto
```

### Importing in Other Proto Files

If you're extending these definitions, you can import selectively:

```protobuf
// Import only models
import "athena/models.proto";

// Import service (which automatically includes models)
import "athena/athena.proto";
```

## Documentation

Comprehensive documentation is available in the `docs/` directory and includes:

- **API Reference**: Detailed protobuf message and service definitions
- **Examples**: Code examples in multiple programming languages
- **Deployment Guide**: Production deployment and integration guidance
- **Overview**: Architecture and concepts explanation

### Building Documentation

To build the documentation:

```bash
# Install uv if not already installed
curl -LsSf https://astral.sh/uv/install.sh | sh

# Sync dependencies and build
uv sync
cd docs
make html
```

The built documentation will be available in `docs/_build/html/index.html`.
