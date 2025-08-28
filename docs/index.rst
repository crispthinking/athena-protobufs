Athena Protobufs Documentation
==============================

Welcome to the Athena Protobufs documentation. This repository contains the Protocol Buffer definitions for the Athena API, which provides CSAM (Child Sexual Abuse Material) detection capabilities through gRPC services.

.. toctree::
   :maxdepth: 2
   :caption: Contents:

   overview
   api_reference
   examples
   deployment_guide

Overview
--------

Athena is a gRPC-based image classification service designed for CSAM detection. The service provides:

* Real-time image classification through bidirectional streaming
* Session-based deployment management
* Multi-affiliate support with correlation tracking
* Comprehensive error handling and monitoring

The protobuf schema defines a complete API for:

* **ClassifierService**: Main service for image classification and deployment management
* **Streaming Classification**: Bidirectional streaming for real-time processing
* **Deployment Management**: Active deployment listing and monitoring
* **Error Handling**: Comprehensive error codes and messages

Quick Start
-----------

The Athena API consists of two main RPC methods:

1. **Classify**: Stream images for classification with deployment-based session management
2. **ListDeployments**: Get information about active deployments and their backlogs

Data Handling and Behavior
---------------------------

Understanding how Athena processes data is crucial for proper implementation:

**Client Library Responsibilities**
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The client library handles image preparation before transmission:

* **Image Hashing**: Generates cryptographic hashes for integrity verification
* **Image Resizing**: Optimizes image dimensions for processing efficiency
* **Metadata Creation**: Prepares correlation and hash metadata for tracking

**Server-Side Processing**
~~~~~~~~~~~~~~~~~~~~~~~~~~

Crisp's Athena service processes images with strict data handling policies:

* **Data Reception**: Only receives image data, hash metadata, and correlation metadata
* **Ephemeral Processing**: Images are processed in memory and immediately discarded after classification
* **No Storage**: Images are never stored on Crisp servers - only used for the classification call
* **Audit Creation**: Creates audit records for each processed image for billing purposes (no image data retained)

**Data Retention Policies**
~~~~~~~~~~~~~~~~~~~~~~~~~~~

* **Response Availability**: Classification responses remain available on their deployment for 1 hour
* **Response Expiry**: After 1 hour, responses are automatically purged and no longer accessible
* **Deployment Lifecycle**: Deployments are automatically removed after 24 hours of inactivity
* **Privacy Protection**: No source image data is retained beyond the classification process

Key Features
------------

Session-Based Processing
~~~~~~~~~~~~~~~~~~~~~~~~

The Classify RPC uses deployment IDs to group related classification requests. Multiple clients can join the same deployment to share responses, enabling collaborative processing scenarios.

Multi-Format Support
~~~~~~~~~~~~~~~~~~~~

Athena supports various image formats including JPEG, PNG, WebP, TIFF, and many others. Images can be sent compressed (Brotli) or uncompressed for bandwidth optimization.

Correlation Tracking
~~~~~~~~~~~~~~~~~~~~

Each image in a request includes a correlation ID that allows clients to match responses with their original requests, essential for batch processing scenarios.

Affiliate Management
~~~~~~~~~~~~~~~~~~~~

The system supports multiple affiliates with proper access control, ensuring that clients can only process images for permitted affiliates.

Error Handling
~~~~~~~~~~~~~~

Comprehensive error handling at both global (request-level) and individual (image-level) granularity, with specific error codes for different failure scenarios.

Getting Started
---------------

To use the Athena API:

1. Review the :doc:`api_reference` for detailed message and service definitions
2. Check the :doc:`examples` for usage patterns and guidance
3. Follow the :doc:`deployment_guide` for implementation guidance

Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
