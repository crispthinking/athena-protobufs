Classifier API Performance Best Practices
=========================================

Overview
--------

The Athena Classifier Service provides two main APIs for classification:

- ``Classify``: A streaming API for batch classification.
- ``ClassifySingle``: A unary API for single input classification.

Choosing the right API and following best practices can significantly impact performance, scalability, and reliability.

API Tradeoffs
-------------

**Classify (Streaming API)**

- **Use Case**: Designed for batch processing of multiple inputs in a single request.
- **Performance**: Highly efficient for large volumes of data. Inputs are processed in parallel, reducing overall latency.
- **Resource Utilization**: Better throughput and lower per-input overhead. Suitable for high-throughput scenarios.
- **Error Handling**: Channel-level errors can abort the entire stream. Individual input errors are returned per input.
- **Best For**: Bulk classification, real-time pipelines, and scenarios where latency per input is less critical than overall throughput.

**ClassifySingle (Unary API)**

- **Use Case**: Processes a single input per request.
- **Performance**: Simpler, but incurs higher overhead per input due to connection setup and teardown.
- **Resource Utilization**: Less efficient for bulk operations. Each request is handled independently.
- **Error Handling**: Errors are returned directly in the response for the specific input.
- **Best For**: Interactive use, low-frequency requests, or when per-input latency and error isolation are critical.

Performance Best Practices
--------------------------

1. **Prefer Streaming for Bulk Operations**

   Use the ``Classify`` API when classifying multiple inputs. Streaming allows parallel processing and reduces network overhead.

2. **Batch Inputs Appropriately**

   Group inputs into batches that balance throughput and latency. Very large batches may increase memory usage; very
   small batches may reduce efficiency. A batch size of 10-50 inputs is often a good starting point, but this may vary
   based on specific use cases and system capabilities.

3. **Handle Errors Gracefully**

   - For ``Classify``, check both global errors (e.g., deployment/channel errors) and per-input errors.
   - For ``ClassifySingle``, handle errors in the returned output object.

4. **Optimize Input Validation**

   Invalid inputs are rejected early. Validate inputs client-side before sending to reduce unnecessary network and server load.

5. **Monitor and Tune Timeouts**

   - Streaming requests may take longer; ensure client and server timeouts are configured appropriately.
   - For ``ClassifySingle``, use the ``RequestTimeout`` setting to avoid hanging requests.

6. **Resource Management**

   - Streaming requests consume server resources for the duration of the stream. Close streams promptly when done.
   - Avoid opening multiple concurrent streams from the same client unless necessary. Fewer clients will likely be able
     to better batch requests.

7. **Deployment ID Consistency**

   - For streaming, all inputs in a stream must use the same deployment ID.
   - Mismatched deployment IDs will abort the stream.
   - Multiple instances can connect to the same deployment ID for load balancing.
   - New deployment IDs can be created at any time without affecting existing streams, but there is a latency
     cost for the initial use of a deployment.

8. **Use Appropriate Encoding and Formats**

   - Use the ``IMAGE_FORMAT_RAW_UINT8_BGR`` format for optimal performance with image data.
   - Use the ``RequestEncoding`` field to specify the encoding of image data. We suggest Brotli for optimal compression.

Summary Table
-------------

+----------------+-------------------+-------------------+
| Feature        | Classify          | ClassifySingle    |
+================+===================+===================+
| Batch Support  | Yes (streaming)   | No (single input) |
+----------------+-------------------+-------------------+
| Throughput     | High              | Low               |
+----------------+-------------------+-------------------+
| Latency        | Lower per batch   | Lower per input   |
+----------------+-------------------+-------------------+
| Affinity       | Per-deployment    | Per-request       |
+----------------+-------------------+-------------------+
| Error Handling | Per input & global| Per input         |
+----------------+-------------------+-------------------+
| Use Case       | Bulk, pipelines   | Interactive, test |
+----------------+-------------------+-------------------+

References
----------

* API Reference: :doc:`api_reference`
* Proto Definitions: :doc:`proto/index`
