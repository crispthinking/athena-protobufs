# Live Model Test Cases

Test images for validating classification against the production Athena model.

> **Important**: All images in this test set are **completely safe and benign**.
> There is no CSAM, adult content, or any violative material. These are frames from
> a public action recognition dataset (people doing everyday activities). Any non-zero
> classification weights represent known edge cases and false positives that have been
> carefully curated for testing purposes.

## Source

Images are extracted frames from the **UCF101 Action Recognition Dataset**:

- **Dataset**: UCF101 - Action Recognition Data Set
- **URL**: https://www.kaggle.com/datasets/matthewjansen/ucf101-action-recognition
- **License**: CC0 (Public Domain)
- **Citation**: Soomro, K., Zamir, A. R., & Shah, M. (2012). UCF101: A Dataset of
  101 Human Actions Classes From Videos in The Wild. arXiv preprint arXiv:1212.0402.

## Contents

50 PNG frames from various action recognition video clips including:

- ApplyEyeMakeup, ApplyLipstick
- BabyCrawling, BlowDryHair, BlowingCandles, BrushingTeeth
- Diving, Drumming
- FrontCrawl
- Haircut, HeadMassage, HulaHoop
- JugglingBalls
- Knitting
- Nunchucks
- Punch
- SumoWrestling, Swing
- TaiChi, TrampolineJumping
- YoYo

## Usage

These test cases are designed for validating production model behavior. Expected
outputs reflect actual production model classifications at the time of capture.

Use `integrator_sample/` for a smaller, safer subset of these images.
