Examples and Usage Patterns
============================

This section provides guidance on common usage patterns for the Athena API, including client library behavior and data handling policies. For specific implementation details, please refer to the generated client libraries for your programming language.

Basic Classification Workflow
------------------------------

The typical workflow for using the Athena API involves:

1. **Image Preparation**: Client library performs hashing and resizing of images
2. **Establish Connection**: Create a gRPC channel to the Athena service
3. **Create Request Stream**: Build ClassifyRequest messages with processed image data
4. **Process Responses**: Handle ClassifyResponse messages with results
5. **Manage Errors**: Implement appropriate error handling

Client Library Responsibilities
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Before transmission, the client library automatically handles:

* **Image Hashing**: Generates cryptographic hashes (MD5, SHA1) for integrity verification
* **Image Resizing**: Optimizes image dimensions for processing efficiency
* **Metadata Preparation**: Creates hash metadata and correlation tracking information

This preprocessing ensures optimal performance and data integrity while reducing bandwidth usage.

Key Concepts
------------

Deployment-Based Sessions
~~~~~~~~~~~~~~~~~~~~~~~~~

All classification requests are grouped by deployment ID. Multiple clients can join the same deployment to share responses, enabling collaborative processing scenarios.

**Deployment Lifecycle**:
* Deployments are automatically removed after 24 hours of inactivity
* Active deployments continue as long as clients are connected and processing
* Response data is purged when deployments expire

Correlation Tracking
~~~~~~~~~~~~~~~~~~~~

Each image includes a correlation ID that allows you to match responses with original requests. This is essential for batch processing where multiple images are sent in a single request.

**Data Flow**:
* Client library generates correlation metadata automatically
* Server processes images and maps results to correlation IDs
* Responses maintain correlation for proper request/response matching

Batch Processing
~~~~~~~~~~~~~~~~

For optimal performance, group multiple images into single ClassifyRequest messages rather than sending individual requests for each image.

**Processing Behavior**:
* Images are processed ephemerally in memory
* No image storage occurs on Crisp servers
* Images are immediately discarded after classification
* Only audit records are created for billing purposes (no image data retained)

Error Handling Strategies
~~~~~~~~~~~~~~~~~~~~~~~~~

The API provides two levels of error reporting:

* **Global Errors**: Affect entire requests (e.g., permission issues)
* **Individual Errors**: Affect specific images within a request (e.g., format issues)

Implement retry logic based on error codes:

* ``ERROR_CODE_AFFILIATE_NOT_PERMITTED``: Not retryable, check permissions
* ``ERROR_CODE_IMAGE_TOO_LARGE``: Not retryable, reduce image size or use client library resizing
* ``ERROR_CODE_MODEL_ERROR``: Possibly retryable after delay

**Data Handling During Errors**:
* Failed images are still immediately discarded from memory
* Error information is included in audit records
* No image data is retained regardless of processing outcome

Performance Optimization
-------------------------

Client-Side Optimization
~~~~~~~~~~~~~~~~~~~~~~~~

* **Image Preprocessing**: Client library handles resizing to reduce transmission size
* **Hash Generation**: Leverage client-side hashing for integrity verification
* **Metadata Efficiency**: Use correlation IDs for efficient request/response matching

Connection Management
~~~~~~~~~~~~~~~~~~~~~

* Use connection pooling for high-throughput scenarios
* Reuse gRPC channels when possible
* Implement proper connection lifecycle management

Batch Size Optimization
~~~~~~~~~~~~~~~~~~~~~~~

* Group 10-50 images per request for optimal throughput
* Balance latency requirements with batch efficiency
* Monitor deployment backlogs for load balancing
* Consider that client library processing time scales with batch size

Compression
~~~~~~~~~~~

* Use Brotli compression (REQUEST_ENCODING_BROTLI) for bandwidth-limited environments
* Test compression ratios vs. CPU overhead for your use case
* Client library resizing reduces the need for compression in many cases

Response Management
~~~~~~~~~~~~~~~~~~~

* **Response Availability**: Responses are available for 1 hour on the deployment
* **Immediate Processing**: Process responses promptly to avoid expiry
* **Deployment Cleanup**: Plan for automatic deployment removal after 24h inactivity

Monitoring and Debugging
-------------------------

Deployment Monitoring
~~~~~~~~~~~~~~~~~~~~~

Use the ListDeployments RPC to:

* Monitor active classification sessions
* Track deployment backlogs
* Debug connectivity issues
* Make load balancing decisions
* Plan for 24-hour deployment lifecycle

Correlation Tracking
~~~~~~~~~~~~~~~~~~~~

* Use meaningful correlation IDs for request tracing
* Include session/batch identifiers in correlation IDs
* Log correlation IDs for debugging and audit trails
* Track client library processing time for optimization

Data Flow Monitoring
~~~~~~~~~~~~~~~~~~~~

* **Image Processing**: Monitor client library hashing and resizing performance
* **Transmission Efficiency**: Track data size before/after client processing
* **Response Timing**: Monitor the 1-hour response availability window
* **Audit Integration**: Coordinate with Crisp's billing audit records

Error Rate Monitoring
~~~~~~~~~~~~~~~~~~~~~

Track error rates by:

* Error code type
* Deployment ID
* Affiliate
* Time period
* Client library processing failures

Best Practices
--------------

Data Handling
~~~~~~~~~~~~~

* **Client Processing**: Trust client library for image hashing and resizing
* **Response Timing**: Process responses within the 1-hour availability window
* **Deployment Planning**: Account for 24-hour automatic deployment cleanup
* **Privacy Assurance**: Leverage the ephemeral processing model for privacy compliance

Security
~~~~~~~~

* Always use TLS in production environments
* Implement proper affiliate access controls
* Never log sensitive image data (images are not stored server-side)
* Use secure credential management
* Understand that only metadata and audit records are retained

Reliability
~~~~~~~~~~~

* Implement circuit breaker patterns for fault tolerance
* Use exponential backoff for transient failures
* Monitor and alert on error rates
* Implement proper timeout handling
* Plan for deployment lifecycle management

Performance
~~~~~~~~~~~

* Monitor response times and throughput
* Use appropriate batch sizes for your workload
* Implement connection pooling
* Consider deployment-based load distribution
* Optimize client library processing for your image types
* Leverage automatic image resizing to reduce bandwidth

Development and Testing
-----------------------

Local Development
~~~~~~~~~~~~~~~~~

* Use insecure channels for local testing
* Implement mock services for integration testing
* Use smaller batch sizes during development
* Test client library image processing with sample data

Integration Testing
~~~~~~~~~~~~~~~~~~~

* Test error scenarios (invalid images, permission errors)
* Verify correlation ID matching
* Test deployment isolation
* Validate compression/decompression
* Test client library hashing and resizing functionality
* Verify response availability within the 1-hour window
* Test deployment cleanup after 24 hours of inactivity

Production Readiness
~~~~~~~~~~~~~~~~~~~~

* Test with production-like data volumes
* Validate TLS configuration
* Test failover scenarios
* Monitor resource utilization
* Validate client library performance with your image types
* Test deployment lifecycle management
* Verify audit record creation for billing integration
* Use `uv sync` for consistent dependency management across environments

Data Privacy Testing
~~~~~~~~~~~~~~~~~~~~

* Confirm images are not retained after processing
* Verify only metadata is logged in audit records
* Test response expiry after 1 hour
* Validate deployment cleanup processes

Getting Started
---------------

1. Set up your development environment with `uv sync`
2. Generate client code from the protobuf definitions using `uv run python -m grpc_tools.protoc`
3. Review the API Reference for message details and data handling policies
4. Understand client library responsibilities (hashing, resizing)
5. Implement basic connection and request/response handling
6. Add error handling and retry logic
7. Plan for response availability windows and deployment lifecycle
8. Optimize for your specific use case
9. Add monitoring and logging with privacy considerations

For detailed implementation examples, please refer to your language-specific gRPC documentation and the generated client libraries. Use `uv` for fast, reliable dependency management throughout your development process.
