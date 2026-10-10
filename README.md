# EmoTrace-Bench: Diagnosing Emotional Expression in Text-to-Video Generation through Controlled Cues and Intensity

**Yeheng Wang**<sup>1\*</sup>, **Liudan Shi**<sup>1\*</sup>, **Yi Zheng**<sup>1†</sup>, **Jingke Li**<sup>2</sup>, **Shiyuan Liu**<sup>2</sup>, **Xin Tao**<sup>2</sup>, **Yuanxing Zhang**<sup>2</sup>, **Guoyu Sun**<sup>1</sup>

<sup>1</sup>Communication University of China&nbsp;&nbsp;&nbsp;<sup>2</sup>Kling Team, Kuaishou Technology

<sup>\*</sup>Equal contribution&nbsp;&nbsp;&nbsp;<sup>†</sup>Corresponding author

[[Paper (arXiv)]](ARXIV_URL)

## Overview

EmoTrace-Bench evaluates whether text-to-video (T2V) models express a requested emotion, whether they control its intensity, and through which audiovisual cues the emotion is conveyed. Each generated video is judged at three levels:

1. **Target emotion** — is the requested emotion clearly conveyed?
2. **Perceived intensity** — if so, is it expressed weakly or strongly?
3. **Ten cue channels** — which channels support the emotion: facial expression (C1), bodily behavior (C2), lighting (V1), color (V2), composition (V3), visual dynamics (V4), sound (A1), scene / space (X1), event / reaction (X2), and temporal organization (T1).

The prompt set is controlled: 448 prompts = 8 emotions × 2 scenes × 14 templates × 2 intensity levels, forming 224 strong–weak pairs that differ only in preregistered intensity expressions. Templates vary which cue channels are written explicitly, so explicit instruction following can be separated from cues the model adds on its own.

In the paper we generate 5,376 videos with 12 T2V systems, score them with a frozen automated evaluator, and validate the evaluator against human annotations of 1,120 videos.

## Results

Main results from the paper (percent). *Emotion*: target-emotion accuracy. *E+I*: target emotion correct and perceived intensity matching the requested intensity. *Visual 9D* / *Full 10D*: mean cue-channel support among videos that convey the target emotion; Full 10D adds sound and is reported only for models with native audio.

| Model | Audio | Emotion | E+I | Visual 9D | Full 10D |
|---|:---:|---:|---:|---:|---:|
| Seedance-2.5 | ✓ | 87.0 | 54.2 | 83.5 | 83.4 |
| Veo-3.1 | ✓ | 84.8 | 55.1 | 76.7 | 75.2 |
| Wan-3.0 | ✓ | 79.9 | 52.0 | 72.4 | 73.0 |
| HappyHorse-1.1 | ✓ | 79.7 | 53.3 | 77.6 | 77.4 |
| MiniMax-H3-Max (fal) | ✓ | 74.3 | 49.1 | 72.0 | 70.0 |
| Kling-3.0 | ✓ | 73.2 | 44.2 | 74.1 | 70.8 |
| MiniMax-H3 | ✓ | 67.9 | 45.1 | 68.3 | 68.1 |
| LTX-2.5 | ✓ | 67.2 | 48.9 | 69.8 | 67.1 |
| Wan-2.2 | – | 67.4 | 41.7 | 63.8 | – |
| HunyuanVideo-1.5 | – | 55.6 | 38.6 | 62.7 | – |
| Kandinsky-5.0-T2V-Pro | – | 50.5 | 33.7 | 65.3 | – |
| LongCat-Video-13.6B | – | 49.3 | 29.9 | 65.1 | – |

See the paper for confidence intervals, significance groups, intensity-control results, and per-emotion and per-channel breakdowns.

## Contents

```text
EmoTrace-Bench/
├── prompts/
│   ├── prompts.json                 448 prompts with per-prompt metadata
│   └── metadata_description.md      field, channel, label and pairing documentation
├── evaluator/
│   ├── README.md                    run settings, how to call, output format, metrics
│   ├── stage1_emotion_intensity/    Stage 1 system prompts (target emotion, intensity), one per emotion
│   └── stage2_cue_channels/         Stage 2 system prompts (ten cue channels), one per emotion
├── LICENSE
└── README.md
```

### Prompts

`prompts/prompts.json` contains the exact 448 English prompts used for every reported experiment. Each record gives the target emotion, scene, template, requested intensity, the ID of its strong–weak partner, the prompt-side role of each cue channel (explicit / implicit / not applicable), the text segments that vary with intensity, and a SHA-256 hash of the prompt text. See `prompts/metadata_description.md` for every field.

```python
import json, collections

data = json.load(open("prompts/prompts.json", encoding="utf-8"))
prompts = data["prompts"]                      # 448 records
pairs = collections.defaultdict(list)
for p in prompts:
    pairs[p["pair_id"]].append(p)              # 224 strong–weak pairs

p = prompts[0]
print(p["prompt_id"], p["target_emotion"], p["template"], p["target_intensity"])
print(p["prompt"])
print("partner:", p["paired_prompt_id"])
```

To benchmark a new model, generate one video per prompt and name each file by its `prompt_id`.

### Automated evaluation rules

`evaluator/` contains the frozen rules of the automated evaluator used in the paper. Each video is judged with two independent Gemini 3.1 Pro calls (temperature 0, 8 fps): Stage 1 decides whether the target emotion is present and whether it is weak or strong; Stage 2 labels each of the ten cue channels. Each file is the complete system prompt for one target emotion, including the emotion definition, decision rules and output format. See `evaluator/README.md` for the exact call, the information the evaluator does and does not receive, output validation, and how labels become the reported metrics.

## License

The data and documents in this repository are released under the [Creative Commons Attribution 4.0 International License (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/).

## Citation

```bibtex
@article{wang2026emotrace,
  title   = {EmoTrace-Bench: Diagnosing Emotional Expression in Text-to-Video Generation through Controlled Cues and Intensity},
  author  = {Wang, Yeheng and Shi, Liudan and Zheng, Yi and Li, Jingke and Liu, Shiyuan and Tao, Xin and Zhang, Yuanxing and Sun, Guoyu},
  journal = {arXiv preprint arXiv:ARXIV_ID},
  year    = {2026}
}
```
