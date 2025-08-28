Athena API Overview
===================

The Athena API is a gRPC-based service designed for real-time image classification, specifically focused on CSAM (Child Sexual Abuse Material) detection. This document provides an architectural overview of the service, its core concepts, and data handling behavior.

Architecture
------------

Service Design
~~~~~~~~~~~~~~

The Athena API follows a streaming-first architecture that enables:

* **Real-time Processing**: Bidirectional streaming allows for immediate response to classification requests
* **Session Management**: Deployment-based grouping enables multiple clients to collaborate on processing
* **Scalable Design**: The service can handle multiple concurrent deployments with independent processing pipelines

Core Components
~~~~~~~~~~~~~~~

ClassifierService
^^^^^^^^^^^^^^^^^

The main service interface providing two primary operations:

* ``Classify``: Bidirectional streaming RPC for image classification
* ``ListDeployments``: Unary RPC for monitoring active deployments

Key Concepts
------------

Deployments
~~~~~~~~~~~

A deployment represents a logical grouping of classification requests. Key characteristics:

* **Deployment ID**: Unique identifier for grouping related requests
* **Multi-client Support**: Multiple clients can join the same deployment
* **Shared Responses**: All clients in a deployment receive responses for that deployment
* **Backlog Tracking**: Each deployment maintains a backlog count for monitoring

Affiliates
~~~~~~~~~~

Affiliates represent different source systems or organizations submitting images:

* **Access Control**: Clients can only process images for permitted affiliates
* **Tracking**: Each image is tagged with its affiliate for analytics and routing
* **Isolation**: Affiliate-based permissions ensure proper data segregation

Correlation
~~~~~~~~~~~

The correlation system enables request-response matching:

* **Correlation IDs**: Unique identifiers within a deployment scope
* **Batch Processing**: Multiple images can be sent in a single request
* **Response Matching**: Clients can correlate responses with original requests

Data Flow
---------

Request Processing
~~~~~~~~~~~~~~~~~~

1. **Client Connection**: Client establishes bidirectional stream with deployment ID
2. **Image Submission**: Client sends ClassifyRequest with image data and metadata
3. **Processing**: Server processes images and generates classifications
4. **Response Delivery**: Server streams ClassifyResponse back to all deployment clients

Image Pipeline
~~~~~~~~~~~~~~

1. **Client Processing**: Client library performs image hashing and resizing before transmission
2. **Data Reception**: Raw image bytes received with encoding and format metadata
3. **Decompression**: If compressed (Brotli), data is decompressed
4. **Format Detection**: Image format is validated and parsed
5. **Classification**: Crisp's machine learning model processes the image
6. **Result Generation**: Classification labels and confidence scores are generated
7. **Data Disposal**: Images are immediately discarded after classification

Data Retention and Availability
-------------------------------

Response Availability
~~~~~~~~~~~~~~~~~~~~~

* **Response Retention**: Classification responses are available on their deployment for 1 hour
* **Access Window**: After 1 hour, responses are no longer available for retrieval
* **Deployment Lifecycle**: Deployments are automatically removed after 24 hours of inactivity

Data Processing Workflow
~~~~~~~~~~~~~~~~~~~~~~~~

1. **Client Preparation**: Client library hashes and resizes images before sending
2. **API Processing**: Crisp receives image data, hash metadata, and correlation metadata
3. **Classification**: Images are processed by Crisp's classifier and immediately discarded
4. **Audit Logging**: Audit records created for billing purposes (no image data retained)
5. **Response Delivery**: Results delivered to all clients in the deployment
6. **Response Expiry**: Responses available for 1 hour, then automatically purged

Error Handling
--------------

Two-Level Error Model
~~~~~~~~~~~~~~~~~~~~~

* **Global Errors**: Affect entire requests (e.g., affiliate permissions)
* **Individual Errors**: Affect specific images within a request (e.g., format issues)

Error Categories
~~~~~~~~~~~~~~~~

* **Client Errors**: Invalid requests, permission issues, malformed data
* **Server Errors**: Processing failures, resource constraints, model errors
* **System Errors**: Network issues, timeouts, infrastructure problems

Supported Formats
------------------

Image Formats
~~~~~~~~~~~~~

The service supports a wide range of image formats:

* **Common Formats**: JPEG, PNG, WebP, GIF, BMP
* **Professional Formats**: TIFF, HDR, RAW
* **Scientific Formats**: PBM, PGM, PPM, PXM, PNM, PFM
* **Legacy Formats**: DIB, SR, RAS, PIC

Encoding Options
~~~~~~~~~~~~~~~~

* **Uncompressed**: Raw image data for maximum quality
* **Brotli Compressed**: Reduced bandwidth usage with compression
* **Auto-detection**: Format detection based on data headers

Performance Considerations
--------------------------

Streaming Benefits
~~~~~~~~~~~~~~~~~~

* **Low Latency**: Immediate processing without waiting for batch completion
* **Resource Efficiency**: Concurrent processing of multiple images
* **Scalability**: Independent deployment processing enables horizontal scaling

Optimization Strategies
~~~~~~~~~~~~~~~~~~~~~~~

* **Compression**: Use Brotli encoding for bandwidth-limited scenarios
* **Batching**: Send multiple images per request to reduce overhead
* **Connection Reuse**: Maintain persistent connections for multiple deployments

Monitoring and Observability
-----------------------------

Deployment Tracking
~~~~~~~~~~~~~~~~~~~

* **Active Deployments**: Real-time view of processing sessions
* **Backlog Monitoring**: Queue depth tracking for performance analysis
* **Client Management**: Visibility into client connections per deployment

Error Monitoring
~~~~~~~~~~~~~~~~

* **Error Rate Tracking**: Monitor classification failure rates
* **Error Code Analysis**: Categorize failures for root cause analysis
* **Performance Metrics**: Track processing times and throughput

Security Considerations
-----------------------

Access Control
~~~~~~~~~~~~~~

* **Affiliate Permissions**: Strict enforcement of affiliate access rights
* **Connection Security**: gRPC transport-level security (TLS)
* **Data Isolation**: Proper segregation between different affiliates

Data Handling
~~~~~~~~~~~~~

* **Client-Side Processing**: Client libraries handle image hashing and resizing before transmission
* **Data Received by API**: Only image data, hash metadata, and correlation metadata is received
* **Ephemeral Processing**: Images are processed in memory and immediately discarded after classification
* **No Image Storage**: Images are never stored on Crisp servers - only used for the classification call
* **Audit Creation**: Crisp creates audit records for each image processed for billing purposes
* **Privacy Protection**: No retention of source image data beyond the classification process
