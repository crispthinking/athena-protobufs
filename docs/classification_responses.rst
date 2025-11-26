Understanding Classification Responses
=====================================

Overview
--------

The Athena Classifier Service returns classification results as arrays of labels with associated weights. Understanding the structure and meaning of these responses is crucial for interpreting classifier output and integrating with your applications.

Classification Response Structure
---------------------------------

Each classification output contains a list of classifications with the following fields:

- **Label**: A string identifier describing the classification category
- **Weight**: A numerical confidence score (typically between 0.0 and higher values)

Example Classification Response
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A typical classification response might look like::

    Found 9 classifications:
      1. Label: UnknownCSAM-classA, Weight: 0.000
      2. Label: UnknownCSAM-classB, Weight: 0.068
      3. Label: UnknownCSAM-classC, Weight: 0.175
      4. Label: UnknownCSAM-adult, Weight: 0.122
      5. Label: UnknownCSAM-indicative, Weight: 0.068
      6. Label: UnknownCSAM-distraction, Weight: 0.566
      7. Label: UnknownCSAM-Entropy, Weight: 1.655
      8. Label: UnknownCSAM-PCSAM, Weight: 0.582
      9. Label: KnownCSAM-MD5, Weight: 0.000

Label Structure and Prefixes
----------------------------

Classification labels follow a structured naming convention:

**Prefix-Category Format**

Labels are prefixed to group them into logical sections:

- **UnknownCSAM-**: Prefix for unknown CSAM classification categories (ML-based detection)
- **KnownCSAM-**: Prefix for known CSAM detection results (hash-based exact matching)
- Individual category names follow the prefix (e.g., "classA", "adult", "indicative")

**Special Labels**

The classifier produces two special labels alongside individual category labels:

- **Entropy**: Represents the uncertainty or randomness in the classification
- **PCSAM**: Represents the overall probability that the input represents any form of CSAM content without regard to specific categories. This is the sum of the raw UnknownCSAM category weights.

**Known CSAM Detection**

The classifier also performs hash-based matching against known CSAM databases:

- **KnownCSAM-MD5**: Indicates MD5 hash match against known CSAM database
- **KnownCSAM-SHA1**: Indicates SHA1 hash match against known CSAM database

When a hash match is found, the corresponding KnownCSAM label will have a non-zero weight. These results provide definitive identification of previously catalogued CSAM content. Exact hash weights will only ever be either 0.0 (no match) or 1.0 (match).

Classifier Types
----------------

The Athena system supports different classifier configurations with distinct label sets:

Benign Classifier
~~~~~~~~~~~~~~~~~

**Purpose**: Integration testing and development

**Important**: The benign classifier is intended **for integration testing only**. The classification values and labels are largely meaningless and should not be used for production content analysis.

**Labels**:

- Lakeside
- Mountains  
- Trees
- Seashore
- Ducks
- Goose

**Characteristics**:

- Individual labels typically sum to approximately 1.0
- Includes Entropy and PCSAM special labels
- All labels are prefixed with "UnknownCSAM-", just as the live classifier labels would be.

Live Classifier
~~~~~~~~~~~~~~~

**Purpose**: Production CSAM detection

**Labels**:

- classA
- classB  
- classC
- adult
- indicative
- distraction

**Characteristics**:

- Individual labels typically sum to approximately 1.0
- Includes Entropy and PCSAM special labels
- All labels are prefixed with "UnknownCSAM-"

Weight Interpretation
--------------------

Understanding Weight Values
~~~~~~~~~~~~~~~~~~~~~~~~~~

- **Individual Category Weights**: Typically range from 0.0 to 1.0 and collectively sum to approximately 1.0
- **Entropy Weight**: Can exceed 1.0 and represents classification uncertainty
- **PCSAM Weight**: Typically in the range of 0.0 to 1.0. Indicates overall likelihood of CSAM content regardless of specific categories. This may not be the exact sum of the UnknownCSAM category weights due to normalization and scaling factors applied during classification, and may exceed 1.0 in some cases.

**Weight Significance**:

- Higher weights indicate stronger classification confidence for that category
- Weights should be interpreted relative to other weights in the same response
- The distribution of weights across categories provides insight into classification certainty. You may want to factor the Entropy into your classification decision rather than relying solely on individual category weights and a fixed threshold.

Understanding Missing or Elided Labels
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The labels available in a response may vary based on the exact classification performed:

- **KnownCSAM labels** will only be present if hash data was provided in the request.
- **UnknownCSAM labels** may be elided if no image data was provided.
- **UnknownCSAM labels** may also be elided if a KnownCSAM match is found, since the image is already identified as known CSAM.
In the case of a model or classification error that prevents classification, all labels may be elided and only an error message returned. Alternatively, only the labels from the stage that was successfully completed may be returned.

Response Processing Guidelines
-----------------------------

When processing classification responses:

1. **Check for Errors**: Always verify that no ``ClassificationError`` is present before processing results

2. **Match Correlation IDs**: Use the ``correlation_id`` field to match responses with your original requests

3. **Interpret Weight Distributions**: Consider the relative weights across all categories, not just individual values

4. **Handle Special Labels**: Process Entropy and PCSAM labels separately from category-specific labels

5. **Process Known CSAM Results**: Check KnownCSAM labels for hash-based matches, which provide definitive CSAM identification

6. **Handle Elided Labels**: Any or all of the labels may be elided from the response. The meaning and interpretation of the missing labels should be considered in your application logic.

7. **Prefer PCSAM Weight for Simple Binary Decisions**: When assessing the likelihood of CSAM content, prioritize the PCSAM weight over the sum of individual UnknownCSAM category weights, as it provides a more accurate overall probability, however less granularity.

Important Notes
--------------

**Benign Classifier Limitations**

.. warning::
   The benign classifier is **not suitable for production use**. It is designed exclusively for:
   
   - Integration testing
   - API development and debugging  
   - System validation
   
   The classification results from the benign classifier have no meaningful relationship to actual content analysis and should never be used for content moderation decisions. The accuracy of the benign classifier is not representative of real-world performance of the live classifier.

See Also
--------

- :doc:`api_reference` - Complete API documentation
- :doc:`best_practices` - Performance optimization guidelines