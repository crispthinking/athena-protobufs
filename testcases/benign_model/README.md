# Benign Model Test Cases

Test images for validating classification with benign/development models.

## Source

Images are believed to be sourced from **Flickr** under permissive licenses
(Creative Commons or similar). These are safe, benign images of nature scenes
and wildlife.

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

## License Note

While images are believed to be under permissive licenses, verify licensing
if using outside of internal testing purposes.
