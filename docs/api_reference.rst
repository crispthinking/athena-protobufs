API Reference
=============

This section provides detailed documentation for all Protocol Buffer messages, services, and enums defined in the Athena API, including client library behavior and data handling policies.

Services
--------

ClassifierService
~~~~~~~~~~~~~~~~~

The main service interface for image classification and deployment management.

.. code-block:: protobuf

   service ClassifierService {
     rpc Classify(stream ClassifyRequest) returns (stream ClassifyResponse);
     rpc ListDeployments(google.protobuf.Empty) returns (ListDeploymentsResponse);
   }

Methods
^^^^^^^

Classify
""""""""

**RPC Type**: Bidirectional Streaming

**Description**: Processes images for classification within deployment-based sessions. Multiple clients can join the same deployment to share classification responses.

**Request**: Stream of ``ClassifyRequest`` messages
**Response**: Stream of ``ClassifyResponse`` messages

**Features**:
- Real-time bidirectional streaming
- Session-based deployment grouping
- Multi-client collaboration support
- Correlation-based request/response matching

**Usage Pattern**:
1. Establish stream connection with deployment ID
2. Send multiple ClassifyRequest messages
3. Receive ClassifyResponse messages for the deployment
4. Use correlation IDs to match requests with responses

ListDeployments
"""""""""""""""

**RPC Type**: Unary

**Description**: Retrieves information about all currently active deployments, including backlog statistics.

**Request**: ``google.protobuf.Empty``
**Response**: ``ListDeploymentsResponse``

**Use Cases**:
- Monitoring active classification sessions
- Debugging deployment connectivity
- Load balancing decisions
- Performance analysis

Messages
--------

Request Messages
~~~~~~~~~~~~~~~~

ClassifyRequest
^^^^^^^^^^^^^^^

Container message for batch image classification requests within a deployment.

.. code-block:: protobuf

   message ClassifyRequest {
     string deployment_id = 1;
     repeated ClassificationInput inputs = 2;
   }

**Fields**:

* ``deployment_id`` (string): Unique identifier for the deployment session. All clients with the same deployment_id will receive shared responses.
* ``inputs`` (repeated ClassificationInput): Array of images to classify in this batch. Allows efficient processing of multiple images per request.

**Constraints**:
- deployment_id must be non-empty
- At least one input must be provided
- All correlation_ids within inputs must be unique within the deployment

**Data Handling**:
- Images are processed ephemerally and discarded immediately after classification
- Only image data, hash metadata, and correlation metadata is received by the API
- Responses are available on the deployment for 1 hour, then automatically purged

ClassificationInput
^^^^^^^^^^^^^^^^^^^

Individual image data and metadata for classification.

.. code-block:: protobuf

   message ClassificationInput {
     string affiliate = 1;
     string correlation_id = 2;
     RequestEncoding encoding = 3;
     bytes data = 4;
     ImageFormat format = 5;
     repeated ImageHash hashes = 6;
   }

**Fields**:

* ``affiliate`` (string): Source system or organization identifier. Used for access control and analytics.
* ``correlation_id`` (string): Unique identifier for matching this input with its response. Must be unique within the deployment scope.
* ``encoding`` (RequestEncoding): Compression format of the image data (uncompressed, Brotli, etc.).
* ``data`` (bytes): Raw image data in the specified encoding format.
* ``format`` (ImageFormat): Image file format (JPEG, PNG, etc.) for proper parsing.
* ``hashes`` (repeated ImageHash): Optional cryptographic hashes of the image data for integrity verification.

**Client Library Processing**:
- Client library performs image hashing before transmission
- Client library handles image resizing for optimal processing
- Hash metadata is generated automatically by the client library

**Validation Rules**:
- affiliate must be permitted for the current client
- correlation_id must be non-empty and unique within deployment
- data must not be empty
- format must be supported by the service

**Server-Side Processing**:
- Images are processed in memory and immediately discarded after classification
- Crisp creates audit records for billing purposes (no image data retained)
- No image storage - images only used for the classification call

ImageHash
^^^^^^^^^

Cryptographic hash information for image integrity verification.

.. code-block:: protobuf

   message ImageHash {
     string value = 1;
     HashType type = 2;
   }

**Fields**:

* ``value`` (string): Hexadecimal representation of the hash value.
* ``type`` (HashType): Algorithm used to generate the hash (MD5, SHA1, etc.).

Response Messages
~~~~~~~~~~~~~~~~~

ListDeploymentsResponse
^^^^^^^^^^^^^^^^^^^^^^^

Response containing active deployment information.

.. code-block:: protobuf

   message ListDeploymentsResponse {
     repeated Deployment deployments = 1;
   }

**Fields**:

* ``deployments`` (repeated Deployment): List of currently active deployments with their metadata.

Deployment
^^^^^^^^^^

Information about a single active deployment.

.. code-block:: protobuf

   message Deployment {
     string deployment_id = 1;
     int32 backlog = 2;
   }

**Fields**:

* ``deployment_id`` (string): Unique identifier for the deployment.
* ``backlog`` (int32): Number of pending classification responses in the deployment queue.

**Deployment Lifecycle**:
- Deployments are automatically removed after 24 hours of inactivity
- Active deployments continue as long as clients are connected and processing

ClassifyResponse
^^^^^^^^^^^^^^^^

Response message containing classification results for a batch of images.

.. code-block:: protobuf

   message ClassifyResponse {
     ClassificationError global_error = 1;
     repeated ClassificationOutput outputs = 2;
   }

**Fields**:

* ``global_error`` (ClassificationError): Error affecting the entire request batch. If present, outputs will be empty.
* ``outputs`` (repeated ClassificationOutput): Individual classification results, one per input image.

**Behavior**:
- If global_error is set, the entire request failed and outputs will be empty
- If global_error is not set, outputs contains results for each input
- Individual images may still have errors in their respective ClassificationOutput

**Response Availability**:
- Responses are available on the deployment for 1 hour after generation
- After 1 hour, responses are automatically purged and no longer accessible
- All clients in the same deployment receive the same responses

ClassificationOutput
^^^^^^^^^^^^^^^^^^^^

Classification result for a single image.

.. code-block:: protobuf

   message ClassificationOutput {
     string correlation_id = 1;
     repeated Classification classifications = 2;
     ClassificationError error = 3;
   }

**Fields**:

* ``correlation_id`` (string): Matches the correlation_id from the corresponding ClassificationInput.
* ``classifications`` (repeated Classification): All detected classifications for this image.
* ``error`` (ClassificationError): Error information if this specific image failed to process.

**Result Interpretation**:
- If error is set, classification failed for this image
- If error is not set, classifications contains the detection results
- Empty classifications with no error indicates no detections above threshold

Classification
^^^^^^^^^^^^^^

Individual classification result with label and confidence.

.. code-block:: protobuf

   message Classification {
     string label = 1;
     float weight = 2;
   }

**Fields**:

* ``label`` (string): Human-readable classification label (e.g., "CatA", "CatB", "Indicitive").
* ``weight`` (float): Confidence score between 0.0 and 1.0, where higher values indicate greater certainty.

**Confidence Interpretation**:
- 0.0: No confidence / definitely not this classification
- 0.5: Uncertain / borderline case
- 1.0: Maximum confidence / definitely this classification

Error Messages
~~~~~~~~~~~~~~

ClassificationError
^^^^^^^^^^^^^^^^^^^

Detailed error information for failed classification attempts.

.. code-block:: protobuf

   message ClassificationError {
     ErrorCode code = 1;
     string message = 2;
     string details = 3;
   }

**Fields**:

* ``code`` (ErrorCode): Structured error code for programmatic handling.
* ``message`` (string): Human-readable error description.
* ``details`` (string): Additional context or technical details about the error.

**Error Handling**:
- Use code for programmatic error handling and retry logic
- Display message to users for error reporting
- Include details in logs for debugging purposes

Enumerations
------------

ErrorCode
~~~~~~~~~

Enumeration of possible classification error types.

.. code-block:: protobuf

   enum ErrorCode {
     ERROR_CODE_UNSPECIFIED = 0;
     ERROR_CODE_IMAGE_TOO_LARGE = 2;
     ERROR_CODE_MODEL_ERROR = 3;
     ERROR_CODE_AFFILIATE_NOT_PERMITTED = 4;
   }

**Values**:

* ``ERROR_CODE_UNSPECIFIED`` (0): Unknown or unspecified error condition.
* ``ERROR_CODE_IMAGE_TOO_LARGE`` (2): Image exceeds maximum size limits for processing.
* ``ERROR_CODE_MODEL_ERROR`` (3): Internal machine learning model encountered an error.
* ``ERROR_CODE_AFFILIATE_NOT_PERMITTED`` (4): Client lacks permission to process images for the specified affiliate.

**Retry Recommendations**:
- ``UNSPECIFIED``: May be retryable depending on underlying cause
- ``IMAGE_TOO_LARGE``: Not retryable, reduce image size
- ``MODEL_ERROR``: Possibly retryable after delay
- ``AFFILIATE_NOT_PERMITTED``: Not retryable, check client permissions

RequestEncoding
~~~~~~~~~~~~~~~

Enumeration of supported data encoding formats.

.. code-block:: protobuf

   enum RequestEncoding {
     REQUEST_ENCODING_UNSPECIFIED = 0;
     REQUEST_ENCODING_UNCOMPRESSED = 1;
     REQUEST_ENCODING_BROTLI = 2;
   }

**Values**:

* ``REQUEST_ENCODING_UNSPECIFIED`` (0): Default encoding, treated as uncompressed.
* ``REQUEST_ENCODING_UNCOMPRESSED`` (1): Raw, uncompressed image data.
* ``REQUEST_ENCODING_BROTLI`` (2): Brotli-compressed data for bandwidth optimization.

**Usage Guidelines**:
- Use ``UNCOMPRESSED`` for local networks or when CPU is limited
- Use ``BROTLI`` for bandwidth-constrained environments
- ``UNSPECIFIED`` defaults to uncompressed behavior

ImageFormat
~~~~~~~~~~~

Enumeration of supported image file formats.

.. code-block:: protobuf

   enum ImageFormat {
     IMAGE_FORMAT_UNSPECIFIED = 0;
     IMAGE_FORMAT_GIF = 1;
     IMAGE_FORMAT_JPEG = 2;
     IMAGE_FORMAT_BMP = 3;
     IMAGE_FORMAT_DIB = 4;
     IMAGE_FORMAT_PNG = 5;
     IMAGE_FORMAT_WEBP = 6;
     IMAGE_FORMAT_PBM = 7;
     IMAGE_FORMAT_PGM = 8;
     IMAGE_FORMAT_PPM = 9;
     IMAGE_FORMAT_PXM = 10;
     IMAGE_FORMAT_PNM = 11;
     IMAGE_FORMAT_PFM = 12;
     IMAGE_FORMAT_SR = 13;
     IMAGE_FORMAT_RAS = 14;
     IMAGE_FORMAT_TIFF = 15;
     IMAGE_FORMAT_HDR = 16;
     IMAGE_FORMAT_PIC = 17;
     IMAGE_FORMAT_RAW_UINT8 = 18;
   }

**Common Formats**:

* ``IMAGE_FORMAT_JPEG`` (2): JPEG format including .jpeg, .jpg, .jpe extensions
* ``IMAGE_FORMAT_PNG`` (5): PNG format with transparency support
* ``IMAGE_FORMAT_WEBP`` (6): Modern WebP format with compression
* ``IMAGE_FORMAT_GIF`` (1): GIF format including animations
* ``IMAGE_FORMAT_TIFF`` (15): TIFF format including .tiff, .tif extensions

**Professional Formats**:

* ``IMAGE_FORMAT_HDR`` (16): High Dynamic Range images
* ``IMAGE_FORMAT_RAW_UINT8`` (18): Raw RGB data in C-order array format

**Scientific/Legacy Formats**:

* ``IMAGE_FORMAT_PBM/PGM/PPM`` (7-9): Portable bitmap formats
* ``IMAGE_FORMAT_BMP/DIB`` (3-4): Windows bitmap formats
* ``IMAGE_FORMAT_SR/RAS/PIC`` (13-14, 17): Sun Raster and other legacy formats

HashType
~~~~~~~~

Enumeration of supported cryptographic hash algorithms.

.. code-block:: protobuf

   enum HashType {
     HASH_TYPE_UNKNOWN = 0;
     HASH_TYPE_MD5 = 1;
     HASH_TYPE_SHA1 = 2;
   }

**Values**:

* ``HASH_TYPE_UNKNOWN`` (0): Unspecified or unknown hash algorithm.
* ``HASH_TYPE_MD5`` (1): MD5 hash algorithm (128-bit).
* ``HASH_TYPE_SHA1`` (2): SHA-1 hash algorithm (160-bit).

**Security Considerations**:
- MD5 and SHA-1 are provided for compatibility but are cryptographically weak
- Use for data integrity verification rather than security purposes
- Consider these hashes as checksums rather than secure hashes

Language-Specific Options
-------------------------

The protobuf schema includes language-specific options for code generation:

**C#**: ``Resolver.Athena.Grpc`` namespace
**Java**: ``com.resolver.athena.grpc`` package, ``AthenaProto`` outer class
**Objective-C**: ``RAT`` class prefix
**PHP**: ``Resolver\Athena\Grpc`` namespace
**Ruby**: ``Resolver::Athena::Grpc`` package
**Swift**: ``RAT`` prefix

These options ensure consistent naming conventions across different programming languages when generating client code from the protobuf definitions.
