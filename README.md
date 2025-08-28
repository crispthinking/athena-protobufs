# Athena Protobufs

This repository contains protobuf files relating to the gRPC interface for the
Athena API (Crisp CSAM Detection).

## Overview

Athena is a gRPC-based image classification service designed for CSAM (Child Sexual Abuse Material) detection by Crisp. The service provides real-time image classification through bidirectional streaming with session-based deployment management and multi-affiliate support.

## Features

- **Real-time Classification**: Bidirectional streaming for immediate image processing
- **Session Management**: Deployment-based grouping enables collaborative processing
- **Multi-format Support**: Supports JPEG, PNG, WebP, TIFF, and many other image formats
- **Compression**: Optional Brotli compression for bandwidth optimization
- **Error Handling**: Comprehensive error codes and detailed error messages
- **Monitoring**: Active deployment tracking and backlog monitoring

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

For development with auto-rebuilding:

```bash
cd docs
make livehtml
```

## Protocol Buffer Schema

The main protobuf definition is located at `athena/athena.proto` and defines:

- `ClassifierService`: Main gRPC service with `Classify` and `ListDeployments` methods
- Request/Response messages for image classification
- Error handling with specific error codes
- Support for multiple image formats and compression options

## Quick Start

### Generating Client Code

**Python:**
```bash
# Ensure dependencies are available
uv sync

# Generate protobuf code
uv run python -m grpc_tools.protoc \
    --proto_path=athena \
    --python_out=generated \
    --grpc_python_out=generated \
    athena/athena.proto
```

**Java:**
```bash
# Add protobuf plugin to build.gradle and run:
./gradlew generateProto
```

**Go:**
```bash
protoc --go_out=. --go_opt=paths=source_relative \
       --go-grpc_out=. --go-grpc_opt=paths=source_relative \
       athena/athena.proto
```

### Quick Start with uv

For the fastest setup experience:

```bash
# Clone and setup
git clone https://github.com/crispthinking/athena-protobufs.git
cd athena-protobufs

# Install dependencies (uv will create virtual environment automatically)
uv sync

# Build documentation
cd docs && make html

# Start development server
python dev_server.py
```

### Basic Usage

See the [examples documentation](docs/examples.rst) for usage patterns and guidance.

### Data Handling

- **Client Processing**: Client library performs image hashing and resizing
- **Ephemeral Processing**: Images processed in memory and immediately discarded
- **No Storage**: Images never stored on Crisp servers
- **Response Availability**: Responses available for 1 hour on deployment
- **Deployment Lifecycle**: Deployments removed after 24h of inactivity
- **Audit Records**: Only metadata retained for billing purposes

## Contributing

When making changes to the protobuf schema:

1. Update the `.proto` files
2. Regenerate client code for all supported languages
3. Update documentation in the `docs/` directory
4. Test with `cd docs && make clean && make html`
5. Run `make linkcheck` to verify documentation links

## Development

This project uses `uv` for fast Python dependency management:

```bash
# Install dependencies
uv sync

# Install development dependencies
uv sync --extra dev

# Update dependencies
uv sync --upgrade

# Build documentation
cd docs && make html
```

## Repository Information

This repository will be open sourced and documentation will be deployed to GitHub Pages.

## License

[Add your license information here]
