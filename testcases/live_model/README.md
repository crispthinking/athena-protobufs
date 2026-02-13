# Live Model Test Cases

Test images for validating classification against the production Athena model.

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
