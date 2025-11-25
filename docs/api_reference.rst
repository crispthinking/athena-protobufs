API Reference
=============

This section provides detailed documentation for all Protocol Buffer messages,
services, and enums defined in the Athena API, including client library behavior
and data handling policies.

Services
--------

ClassifierService
~~~~~~~~~~~~~~~~~

The main service interface for image classification and deployment management.

.. code-block:: protobuf

   service ClassifierService {
     rpc Classify(stream ClassifyRequest) returns (stream ClassifyResponse);
     rpc ListDeployments(google.protobuf.Empty) returns (ListDeploymentsResponse);
     rpc ClassifySingle(ClassificationInput) returns (ClassificationOutput);
   }

Methods
^^^^^^^

Classify
""""""""

**RPC Type**: Bidirectional Streaming

**Description**: Processes images for classification within deployment-based
sessions. Multiple clients can join the same deployment to share classification
responses.

**Request**: Stream of ``ClassifyRequest`` messages
**Response**: Stream of ``ClassifyResponse`` messages

**Features**:

* Bidirectional streaming
* Deployment ID grouping
* Multi-client collaboration support
* Correlation ID based request/response matching

**Usage Pattern**:

#. Establish stream connection with deployment ID
#. Send ClassifyRequest messages
#. Receive ClassifyResponse messages for the deployment ID
#. Use correlation IDs to match requests with responses

ListDeployments
"""""""""""""""

**RPC Type**: Unary

**Description**: Retrieves information about all currently active deployments,
including request backlog statistics.

**Request**: ``google.protobuf.Empty``
**Response**: ``ListDeploymentsResponse``

ClassifySingle
"""""""""""""""

**RPC Type**: Unary

**Description**: Classifies a single image synchronously without deployment
context. Returns classification results immediately in a single request-response
cycle. Unlike the streaming Classify method, this operates independently of
deployments and does not require session management or deployment coordination.

**Request**: ``ClassificationInput``
**Response**: ``ClassificationOutput``

**Use Cases**:

* Low-throughput, low-latency classification scenarios
* Simple one-off image classifications
* Applications where immediate synchronous responses are preferred over streaming
* Testing and debugging individual image classifications

**Features**:

* Synchronous request-response pattern
* No deployment coordination required
* Independent operation from streaming sessions
* Direct correlation between input and output
* Immediate error reporting

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
- All correlation_ids within inputs must be unique within the deployment

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

* ``affiliate`` required (string): Source system or organization identifier. Used for access control.
* ``correlation_id`` required (non-empty string): Unique identifier for matching this input with its response. Must be unique within the deployment scope.
* ``encoding`` (RequestEncoding): Default RequestEncoding.Uncompressed -  Compression format of the image data (uncompressed, Brotli, etc.).
* ``data`` (bytes): Raw image data in the specified encoding format.
* ``format`` required if ``data`` is not empty (ImageFormat): Image file format (JPEG, PNG, etc.).
* ``hashes`` optional (repeated ImageHash): Hashes of the image data used for checking known CSAM.

Requests must have at least one ``data`` or ``hashes`` to see classifications in
the response.

ImageHash
^^^^^^^^^

Hash information for image known CSAM detection.

.. code-block:: protobuf

   message ImageHash {
     string value = 1;
     HashType type = 2;
   }

**Fields**:

* ``value`` (string): Hexadecimal representation of the hash value.
* ``type`` (HashType): Algorithm used to generate the hash.

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
* ``backlog`` (int32): Number of pending classification requests in the deployment queue.

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
* ``classifications`` (repeated Classification): All classifications for this image.
* ``error`` (ClassificationError): Error information if this specific image failed to process.

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

* ``UNSPECIFIED``: May be retryable depending on underlying cause
* ``IMAGE_TOO_LARGE``: Not retryable, make image size 448x448 pixels.
* ``MODEL_ERROR``: Possibly retryable after delay
* ``AFFILIATE_NOT_PERMITTED``: Not retryable, check client permissions

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
    IMAGE_FORMAT_RAW_UINT8_BGR = 19;
   }

**Raw Formats**:

  * ``IMAGE_FORMAT_RAW_UINT8`` (18): Raw RGB data in C-order array format (legacy)
  * ``IMAGE_FORMAT_RAW_UINT8_BGR`` (19): Raw BGR data in C-order array format (preferred canonical format)

HashType
~~~~~~~~

Enumeration of supported hash algorithms.

.. code-block:: protobuf

   enum HashType {
     HASH_TYPE_UNKNOWN = 0;
     HASH_TYPE_MD5 = 1;
     HASH_TYPE_SHA1 = 2;
   }
