# Integrator Sample Test Cases

A curated subset of test images for quick integration testing.

## Source

Images are selected from `live_model/` which contains frames from the
**UCF101 Action Recognition Dataset**:

- **License**: CC0 (Public Domain)
- **URL**: https://www.kaggle.com/datasets/matthewjansen/ucf101-action-recognition

## Contents

10 carefully selected images where:

- classA weight = 0 (no high-risk classification)
- classB weight = 0 (no high-risk classification)

This makes them safe for demonstration and integration testing while still
exercising the full classification pipeline.

## Selection Criteria

Images were chosen to provide:

1. **Safety**: No high-risk (classA/classB) classifications
2. **Variety**: Mix of indicative, adult, and distraction classifications
3. **Coverage**: Different action categories (YoYo, Diving, Brushing Teeth, etc.)

## Usage

Ideal for:

- Quick smoke tests during development
- Integration testing with production model
- Demonstrating client functionality to integrators
- CI pipeline validation (fast, ~10 images)

For comprehensive testing, use `live_model/` (50 images).
