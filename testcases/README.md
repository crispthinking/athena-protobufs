# Athena E2E Test Cases

This directory contains shared end-to-end test cases for validating Athena client
implementations across different programming languages. Each client library should
consume these test cases to ensure consistent behavior.

## Directory Structure

```
testcases/
├── benign_model/       # Safe images for testing with benign/development models
├── integrator_sample/  # Small subset for quick integration testing (10 images)
├── live_model/         # Full test set for production model validation
└── README.md           # This file
```

## Test Case Format

Each test directory contains:

- `expected_outputs.json` - Expected classification results
- `images/` - Test images (tracked with Git LFS)

### expected_outputs.json Schema

```json
{
    "classification_labels": [
        "Label1",
        "Label2",
        ...
    ],
    "images": [
        ["image_filename.png", [0.1, 0.2, ...]],
        ...
    ]
}
```

- `classification_labels`: Array of label names in order
- `images`: Array of `[filename, weights]` pairs where weights correspond to labels

## Usage Example

```python
import json
from pathlib import Path

# Load test cases
with open("testcases/integrator_sample/expected_outputs.json") as f:
    data = json.load(f)

labels = data["classification_labels"]

for filename, expected_weights in data["images"]:
    image_path = Path("testcases/integrator_sample/images") / filename
    
    # Classify the image using your client
    result = classify(image_path)
    
    # Compare results
    for i, label in enumerate(labels):
        expected = expected_weights[i]
        actual = result[label]
        assert abs(expected - actual) < TOLERANCE
```

## Comparison Tolerance

Due to floating-point precision and potential minor model variations, use an
appropriate tolerance when comparing expected vs actual weights:

| Use Case | Recommended Tolerance |
|----------|----------------------|
| Exact model match | `1e-4` (0.0001) |
| Model version drift | `1e-2` (0.01) |
| Behavioral validation | `0.05` (5%) |

For CI/CD pipelines, `1e-4` is recommended as it catches regressions while
allowing for floating-point rounding differences.

## Test Directories

### `benign_model/`

Contains safe, benign images (nature scenes, animals) for testing with
development or benign classification models. Use this for:

- Local development testing
- CI pipelines without production model access
- Model behavior verification with known-safe content

### `integrator_sample/`

A curated subset of 10 images from `live_model/` specifically selected for
integrator testing. These images have no high-risk classifications (classA=0,
classB=0) making them safe for demonstration and integration testing.

### `live_model/`

Full test set with 50 images for comprehensive production model validation.
Contains a variety of classification scenarios including edge cases.

## Git LFS

Images are stored using Git LFS. Ensure LFS is installed and initialized:

```bash
git lfs install
git lfs pull
```
