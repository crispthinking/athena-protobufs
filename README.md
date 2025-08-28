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
