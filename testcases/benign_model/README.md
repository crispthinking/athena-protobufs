# Benign Model Test Cases

Test images for validating classification with benign/development models.

## Source

Images are sourced from **Pexels** under the [Pexels License](https://www.pexels.com/license/)
(free to use for any purpose, no attribution required).

## Contents

Categories include:

- `ducks/` - Duck photographs
- `goose/` - Goose photographs  
- `lakeside/` - Lake and waterside scenery
- `mountains/` - Mountain landscapes
- `seashore/` - Beach and coastal scenes
- `trees/` - Forest and tree photographs

## Usage

These test cases are designed for:

- Local development without production model access
- CI/CD pipelines with benign classification models
- Testing image processing and client behavior with known-safe content

Expected outputs are calibrated for a benign classification model that
classifies all images as safe/distraction content.
