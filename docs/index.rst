Athena Protobufs Documentation
==============================

Welcome to the Athena Protobufs documentation. This repository contains the
Protocol Buffer definitions for the Athena API, which provides CSAM (Child
Sexual Abuse Material) detection capabilities through gRPC services.

.. toctree::
   :maxdepth: 2
   :caption: Contents:

   api_reference

Service Overview
----------------

Athena is an enterprise-grade CSAM detection platform, for both known and unknown
CSAM images, that delivers scalable, secure image classification capabilities
through a modern gRPC streaming API.

**Core Architecture**

Athena operates as a bi-directional streaming gRPC service, enabling clients to
submit image classification requests and receive results. This streaming
architecture ensures optimal performance and scalability, allowing for
continuous processing of large image volumes while maintaining low latency
response times.

**Security & Access Control**

Athena implements comprehensive authentication and authorization controls to
ensure secure operations:

- **Client Authentication**: All clients must be properly authenticated before accessing the service
- **Affiliate Authorization**: The system validates that each client is authorized to submit requests for specific affiliates

**Deployment Isolation**

Each client operates within isolated deployment ids, ensuring independence and security:

- **Dedicated Processing**: Each client submit requests to their own dedicated deployment ids
- **Operational Independence**: Each client's processing pipeline operates independently, preventing cross-contamination
- **Resource Isolation**: Deployment-level separation ensures consistent performance and security boundaries

Quick Start
-----------

The Athena API consists of two main RPC methods:

1. **Classify**: Stream images for classification with deployment-based session management
2. **ListDeployments**: Get information about active deployments and their backlogs

Deployments
~~~~~~~~~~~

The Classify endpoint uses deployment IDs to group classification requests.
Multiple clients can join the same deployment to share responses. Clients can
use different deployment IDs to process images seperately.

Image Pre-processing
~~~~~~~~~~~~~~~~~~~~
Images must be resized to 448x448 pixels before sending to the the API.

Image hashes should be generated for known CSAM detection, multiple hashes for
the same image can be added to the request (SHA1, MD5) if e.g. the image was
saved with a different extension.

For example, if an image was saved as a JPEG and a PNG, both hashes could be
included in the request.

For example Image1 hashes might be:

* Image1.jpg -> SHA1: ``1234567890abcdef1234567890abcdef12345678``
* Image1.png -> SHA1: ``456789abcdef123456789abcdef1234567812312``

These can be used in a single request, the response will tell you if any of the
hashes match a known CSAM image.

Requests must have at least one of the following, with both preferred:

* Image hashes (SHA1, MD5) - for known CSAM detection
* Image data - for unknown CSAM detection

Data Handling and Behavior
---------------------------

Client Library Responsibilities
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The client library handles image preparation before transmission:

* **Image Hashing**: Generates hashes for CSAM detection
* **Image Resizing**: Sets the image dimensions for processing
* **Metadata Creation**: Prepares correlation ID tracking

Server-Side Processing
~~~~~~~~~~~~~~~~~~~~~~~~~~

The Athena service processes images with strict data handling policies:

* **Data Reception**: Only receives image data, hash metadata, and correlation metadata
* **Ephemeral Processing**: Images are processed in memory and immediately discarded after classification
* **No Storage**: Images are never stored on our servers - only used for the classification call
* **Audit Creation**: Creates audit records for each processed image for billing purposes (no image data retained)

Data Retention Policies
~~~~~~~~~~~~~~~~~~~~~~~~~~~

* **Response Availability**: Classification responses remain available on their deployment ID for 1 hour
* **Response Expiry**: After 1 hour, responses are automatically purged and no longer accessible
* **Deployment Lifecycle**: Deployments are automatically removed after 24 hours of inactivity
* **Privacy Protection**: No source image data is retained beyond the classification process

Key Features
------------

Multi-Format Support
~~~~~~~~~~~~~~~~~~~~

Athena supports various image formats including JPEG, PNG, WebP, TIFF, and many
others. Images can be sent compressed (Brotli) or uncompressed for bandwidth
optimization.

Correlation Tracking
~~~~~~~~~~~~~~~~~~~~

Each image in a request includes a correlation ID that allows clients to match
responses with their original requests.

Getting Started
---------------

Check the `protocol buffer documentation <https://protobuf.dev/getting-started/>`_ for information on how to generate code
for your language of choice.
Review the :doc:`api_reference` for detailed message and service definitions

Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
