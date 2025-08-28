Deployment Guide
================

This guide provides comprehensive instructions for deploying and integrating the Athena API in production environments. It covers client generation, service configuration, security considerations, data handling policies, and operational best practices.

Code Generation
---------------

Generate Client Libraries
~~~~~~~~~~~~~~~~~~~~~~~~~~

The first step in deployment is generating client libraries from the protobuf definitions for your target programming language.

Protocol Buffer Compiler Setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Install the Protocol Buffer compiler and language-specific plugins:

**For Python:**

.. code-block:: bash

   # Install protoc
   sudo apt-get install protobuf-compiler  # Ubuntu/Debian
   brew install protobuf                   # macOS

   # Install Python gRPC tools with uv
   uv add grpcio grpcio-tools

   # Generate Python code
   uv run python -m grpc_tools.protoc \
       --proto_path=athena \
       --python_out=generated \
       --grpc_python_out=generated \
       athena/athena.proto

**For Java:**

.. code-block:: bash

   # Add to build.gradle
   plugins {
       id 'com.google.protobuf' version '0.9.4'
   }

   dependencies {
       implementation 'io.grpc:grpc-netty:1.57.2'
       implementation 'io.grpc:grpc-protobuf:1.57.2'
       implementation 'io.grpc:grpc-stub:1.57.2'
   }

   protobuf {
       protoc {
           artifact = "com.google.protobuf:protoc:3.24.0"
       }
       plugins {
           grpc {
               artifact = 'io.grpc:protoc-gen-grpc-java:1.57.2'
           }
       }
       generateProtoTasks {
           all()*.plugins {
               grpc {}
           }
       }
   }

**For C#:**

.. code-block:: bash

   # Install tools
   dotnet tool install -g Grpc.Tools

   # Add to .csproj
   <PackageReference Include="Grpc.Net.Client" Version="2.57.0" />
   <PackageReference Include="Grpc.Tools" Version="2.57.0">
     <PrivateAssets>all</PrivateAssets>
     <IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>
   </PackageReference>

**For Go:**

.. code-block:: bash

   # Install protoc-gen-go and protoc-gen-go-grpc
   go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
   go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest

   # Generate Go code
   protoc --go_out=. --go_opt=paths=source_relative \
          --go-grpc_out=. --go-grpc_opt=paths=source_relative \
          athena/athena.proto

Client Configuration
--------------------

Connection Management
~~~~~~~~~~~~~~~~~~~~~

Production deployments require robust connection management with proper error handling, retry logic, and connection pooling.

**Configuration Guidance:**

Key connection settings to consider:

* **Keep-alive settings**: Configure appropriate timeouts for your network environment
* **Message size limits**: Set appropriate limits based on your image sizes
* **TLS configuration**: Enable transport security for production environments
* **Connection pooling**: Implement pooling for high-throughput scenarios
* **Dependency Management**: Use uv for fast, reliable Python dependency management

Refer to your language-specific gRPC documentation for exact configuration syntax.

Authentication and Security
---------------------------

TLS Configuration
~~~~~~~~~~~~~~~~~

Always use TLS in production environments to encrypt data in transit.

**TLS Configuration Considerations:**

* **Server Certificate Validation**: Configure proper root CA certificates
* **Mutual TLS (mTLS)**: Use client certificates for enhanced security
* **Certificate Management**: Implement proper certificate rotation
* **Hostname Verification**: Ensure proper hostname validation

Consult your gRPC library documentation for specific TLS configuration methods.

Data Privacy and Handling
--------------------------

Understanding Athena's data handling policies is crucial for compliance and security planning.

Client Library Processing
~~~~~~~~~~~~~~~~~~~~~~~~~~

**Pre-transmission Processing:**

The client library performs several operations before sending data to Crisp:

* **Image Hashing**: Automatic generation of MD5 and SHA1 hashes for integrity verification
* **Image Resizing**: Optimizes image dimensions for processing efficiency and bandwidth reduction
* **Metadata Creation**: Generates correlation IDs and hash metadata for tracking
* **Format Validation**: Ensures images are in supported formats before transmission

Server-Side Data Handling
~~~~~~~~~~~~~~~~~~~~~~~~~~

**Data Reception and Processing:**

Crisp's Athena service receives and processes data with strict privacy controls:

* **Data Scope**: Only receives image data, hash metadata, and correlation metadata
* **Ephemeral Processing**: Images are processed in memory and immediately discarded after classification
* **No Storage**: Images are never written to disk or persistent storage
* **Immediate Disposal**: Images are discarded as soon as the classification call completes

**Audit and Billing:**

* **Audit Records**: Crisp creates audit records for each processed image for billing purposes
* **No Image Data**: Audit records contain only metadata - no actual image data is retained
* **Correlation Tracking**: Audit records include correlation IDs for request tracking
* **Billing Integration**: Audit data is used solely for billing calculations

Response and Deployment Lifecycle
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Response Availability:**

* **Retention Period**: Classification responses are available on their deployment for 1 hour
* **Automatic Purge**: After 1 hour, responses are automatically deleted and no longer accessible
* **Shared Access**: All clients in the same deployment can access responses during the availability window

**Deployment Management:**

* **Inactivity Timeout**: Deployments are automatically removed after 24 hours of inactivity
* **Data Cleanup**: All deployment data, including responses, is purged when deployments expire
* **Resource Management**: Automatic cleanup ensures optimal resource utilization

Privacy Compliance
~~~~~~~~~~~~~~~~~~~

**Data Protection Principles:**

* **Minimal Data Retention**: Only essential metadata is retained for billing purposes
* **Purpose Limitation**: Image data is used exclusively for classification - no secondary use
* **Automatic Deletion**: All data follows automatic retention policies with no manual intervention required
* **Transparency**: Clear data flow and retention policies for compliance reporting

API Key Authentication
~~~~~~~~~~~~~~~~~~~~~~

**API Key Authentication:**

Add authentication credentials to gRPC metadata:

* Use the 'authorization' header with Bearer tokens
* Include API keys in request metadata
* Implement credential refresh logic for long-running connections
* Store credentials securely and avoid hardcoding

Example metadata structure: ``('authorization', 'Bearer <your-api-key>')``

Error Handling and Resilience
------------------------------

Retry Logic
~~~~~~~~~~~

**Retry Logic Implementation:**

Implement exponential backoff for transient failures:

* **Retryable Errors**: UNAVAILABLE, DEADLINE_EXCEEDED, RESOURCE_EXHAUSTED
* **Non-retryable Errors**: PERMISSION_DENIED, INVALID_ARGUMENT, NOT_FOUND
* **Backoff Strategy**: Use exponential backoff with jitter
* **Max Retries**: Limit retry attempts to prevent infinite loops

Formula: ``delay = base_delay * (2 ^ attempt) + random_jitter``

Circuit Breaker Pattern
~~~~~~~~~~~~~~~~~~~~~~~

**Circuit Breaker Pattern:**

Implement circuit breaker to prevent cascading failures:

* **States**: CLOSED (normal), OPEN (failing), HALF_OPEN (testing)
* **Failure Threshold**: Number of failures before opening circuit
* **Recovery Timeout**: Time before attempting to close circuit
* **Success Criteria**: Requirements for closing circuit from half-open state

This pattern helps prevent overwhelming a failing service and provides graceful degradation.

Monitoring and Observability
-----------------------------

Metrics Collection
~~~~~~~~~~~~~~~~~~

**Metrics Collection:**

Track key performance indicators:

* **Request Metrics**: Count, rate, and latency per deployment
* **Error Metrics**: Error rates by type and deployment
* **Resource Metrics**: Connection pool utilization, memory usage
* **Business Metrics**: Images processed, classification accuracy

Export metrics to monitoring systems like Prometheus, CloudWatch, or Datadog.

Structured Logging
~~~~~~~~~~~~~~~~~~

**Structured Logging:**

Implement consistent logging for better observability:

* **Log Format**: Use JSON or structured format for machine parsing
* **Required Fields**: timestamp, level, correlation_id, deployment_id
* **Context**: Include request context and trace information
* **Error Details**: Log full error context for debugging

Log aggregation tools like ELK stack or Splunk can help analyze logs.

Health Checks
~~~~~~~~~~~~~

**Health Checks:**

Implement service health monitoring:

* **Health Endpoint**: Use ListDeployments RPC as a basic health check
* **Health Metrics**: Track success/failure rates and response times
* **Graceful Degradation**: Handle partial service availability
* **Integration**: Connect to service discovery and load balancers
* **Deployment Lifecycle**: Monitor deployment age and plan for 24-hour cleanup
* **Response Windows**: Track response availability within 1-hour windows

Configure health check intervals based on your infrastructure requirements.

Load Balancing
--------------

Client-Side Load Balancing
~~~~~~~~~~~~~~~~~~~~~~~~~~

**Load Balancing:**

Implement client-side load balancing strategies:

* **Round Robin**: Distribute requests evenly across endpoints
* **Random**: Simple random selection for basic distribution
* **Health-aware**: Route only to healthy endpoints
* **Weighted**: Distribute based on endpoint capacity

Consider using gRPC's built-in load balancing features or service mesh solutions for production deployments.

Performance Tuning
-------------------

Connection Pool Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Connection Pool Optimization:**

Design efficient connection pooling:

* **Pool Size**: Balance connection overhead with concurrency needs
* **Connection Reuse**: Implement proper connection lifecycle management
* **Overflow Handling**: Allow temporary connection creation under load
* **Cleanup**: Properly close and cleanup connections

Monitor pool utilization and adjust sizing based on actual usage patterns.

Batch Optimization
~~~~~~~~~~~~~~~~~~

**Batch Optimization:**

Optimize batching for performance:

* **Batch Size**: Test different sizes (typically 10-50 images)
* **Timeout Strategy**: Balance latency vs. efficiency
* **Dynamic Sizing**: Adjust batch size based on load
* **Memory Management**: Consider memory usage for large batches

Monitor throughput and latency to find optimal batch parameters for your workload.

Deployment Patterns
-------------------

Container Deployment
~~~~~~~~~~~~~~~~~~~~

**Container Deployment Considerations:**

* **Base Images**: Use appropriate language runtime images
* **Code Generation**: Include protobuf compilation in build process
* **Dependency Management**: Use uv for faster, more reliable Python builds
* **Environment Configuration**: Use environment variables for service configuration
* **Resource Limits**: Set appropriate CPU and memory limits
* **Health Checks**: Implement liveness and readiness probes
* **Secrets Management**: Use secure methods for API keys and certificates

Refer to your container orchestration platform documentation for specific deployment configurations.

Configuration Management
~~~~~~~~~~~~~~~~~~~~~~~~

**Configuration Management:**

Implement flexible configuration:

* **Environment Variables**: Use for deployment-specific settings
* **Configuration Files**: For complex configurations
* **Default Values**: Provide sensible defaults
* **Validation**: Validate configuration at startup
* **Secrets**: Use secure storage for sensitive values
* **Dependency Management**: Use uv sync for consistent environments

Example environment variables:
- ``ATHENA_HOST``: Service hostname
- ``ATHENA_PORT``: Service port
- ``USE_TLS``: Enable TLS (true/false)
- ``ATHENA_API_KEY``: Authentication key

Compliance and Privacy
----------------------

Data Protection Implementation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Privacy by Design:**

* **Ephemeral Processing**: Leverage Athena's ephemeral processing model for GDPR compliance
* **Data Minimization**: Only send necessary image data - client library optimization reduces data transmission
* **Automatic Deletion**: Built-in data retention policies eliminate manual data management requirements
* **Audit Trail**: Correlation IDs provide audit capabilities without retaining sensitive data

**Compliance Considerations:**

* **Data Residency**: Understand that no image data is stored on Crisp servers
* **Retention Policies**: Document the 1-hour response retention and 24-hour deployment lifecycle
* **Third-Party Processing**: Clearly communicate that Crisp processes but does not retain image data
* **Billing Records**: Account for metadata-only audit records in compliance documentation

Client Implementation Requirements
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Required Safeguards:**

* **Input Validation**: Validate images before client library processing
* **Error Handling**: Properly handle client library processing failures
* **Logging Policies**: Avoid logging sensitive image data in client applications
* **Access Controls**: Implement proper affiliate-based access restrictions

Security Best Practices
-----------------------

1. **Network Security**
   - Always use TLS in production
   - Implement mutual TLS for high-security environments
   - Use VPNs or private networks when possible

2. **Authentication**
   - Rotate API keys regularly
   - Use short-lived tokens when possible
   - Implement proper key management

3. **Data Protection**
   - Never log image data
   - Use encryption for data at rest
   - Implement proper access controls for affiliates

4. **Monitoring**
   - Monitor for unusual access patterns
   - Track failed authentication attempts
   - Implement alerting for security events

Troubleshooting
---------------

Common Issues
~~~~~~~~~~~~~

**Connection Timeouts:**
- Increase connection timeout values
- Check network connectivity
- Verify firewall configurations

**High Error Rates:**
- Check affiliate permissions
- Validate image formats and sizes
- Monitor server capacity

**Performance Issues:**
- Optimize batch sizes
- Implement connection pooling
- Use compression for large images

**Memory Issues:**
- Limit concurrent requests
- Stream large datasets
- Implement proper cleanup

Debug Logging
~~~~~~~~~~~~~

**Debug Logging:**

Enable comprehensive logging for troubleshooting:

* **gRPC Logging**: Enable framework-level logging
* **Request/Response Logging**: Log key message details (avoid sensitive data)
* **Performance Logging**: Track timing and resource usage
* **Error Context**: Include full error context and stack traces

Use appropriate log levels (DEBUG for development, INFO/WARN/ERROR for production).

**Privacy Considerations:**

* **No Image Logging**: Never log actual image data in any environment
* **Metadata Only**: Log only correlation IDs, deployment IDs, and processing metrics
* **Audit Alignment**: Align logging with Crisp's audit record structure for consistency
* **Retention Alignment**: Consider log retention policies in context of 1-hour response availability
