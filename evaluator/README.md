# Automated Evaluation Rules

This folder contains the frozen rules of the automated evaluator used for every result in the paper. Each video is evaluated with **two independent calls** to Gemini 3.1 Pro: Stage 1 judges the target emotion and its perceived intensity, and Stage 2 judges the ten cue channels. The two calls do not see each other's output.

```text
evaluator/
├── stage1_emotion_intensity/   one system prompt per target emotion (8 files)
└── stage2_cue_channels/        one system prompt per target emotion (8 files)
```

Each file is the complete system prompt for one target emotion. It contains the emotion definition, the decision rules, and the required JSON output format.

## Run settings

| Setting | Value |
|---|---|
| Model | Gemini 3.1 Pro |
| Temperature | 0 |
| Video input | the complete generated MP4 with its embedded audio, sampled at 8 fps |
| Calls per video | 2 (Stage 1 and Stage 2), independent |
| Evaluations per video | 1 |

## Information boundary

The evaluator receives only the generated video and the name of the target emotion (its definition is part of each system prompt). It does not receive the generation prompt, the requested intensity, the generating model, the prompt template, the prompt-side channel roles, the paired strong/weak video, or any other judgment.

## How to call

For a video whose target emotion is `<emotion>`:

1. **Stage 1.** System prompt: `stage1_emotion_intensity/<emotion>.txt`. User message: the fixed text below, followed by the video.
2. **Stage 2.** System prompt: `stage2_cue_channels/<emotion>.txt`, with the `auxiliary_measurements` block filled for this video (see below). User message: the same fixed text, followed by the video.

Fixed user text:

```text
Evaluate the supplied audiovisual MP4 under the system contract and return only the required JSON. CRITICAL FORMAT REQUIREMENT: every evidence string required by the schema must contain an explicit video timestamp range such as [00:01-00:03].
```

### Stage 2 auxiliary measurements

Two bounded modules supply target-neutral measurements for visual dynamics (V4) and sound (A1). They are inserted into the `auxiliary_measurements` block of the Stage 2 prompt; in the released files this block is an empty placeholder.

- **V4:** camera-motion measurements from 16 uniformly sampled frames, using bidirectional Lucas–Kanade tracks and RANSAC similarity transforms.
- **A1:** audio-level summary statistics of the embedded track, including near-silence indicators.

These measurements never assign a label; the evaluator verifies them against the complete video and owns the final V4 and A1 decisions. As stated in the prompt, A1 must be 0 when the track is flagged as near-silent or its silence ratio is at least 0.98.

## Output and validation

Stage 1 returns `emotion_match` (0/1), `emotion_evidence`, `primary_emotion_carrier`, `carrier_strength` (0/1/2), `perceived_intensity` (`not_match` / `weak` / `strong`) and `intensity_evidence`. Stage 2 returns a 0/1 label and evidence for each of C1, C2, V1, V2, V3, V4, A1, X1, X2, T1 in that order.

A response is accepted only if it is valid JSON with the exact fields, integer labels, a timestamp range in every evidence string, and consistent dependencies (`emotion_match = 0` requires `perceived_intensity = not_match` and null intensity evidence). If a response fails validation, the same call is repeated with the following text appended to the user message; this retry concerns format only:

```text
FORMAT RETRY: keep a fresh semantic judgment but obey the exact JSON schema, dependency, and timestamp requirements. The previous response failed validation with: <validation error>.
```

## From labels to metrics

- **Channel applicability.** A channel counts for a prompt only if its role in `prompts/prompts.json` (`channel_roles`) is `explicit` or `implicit`. Channels marked `not_applicable` are excluded, never scored 0. A1 is excluded for videos without a usable native audio track.
- **Metrics.** Emotion, joint emotion-and-intensity (E+I), per-channel depth, Visual 9D, Full 10D and paired intensity control are computed from these labels as defined in Appendix C of the paper. Requested intensities and strong–weak pairs come from `prompts/prompts.json` (`target_intensity`, `pair_id`).

## Relation to the prompts used in the paper

The files were generated from the frozen rule set with the same compilation code used in the paper's runs. Stage 1 prompts are identical to those used in the paper except that the benchmark name in the first line was updated to EmoTrace-Bench. Stage 2 prompts are identical except for the same name update, the empty measurement placeholder described above, and one sentence in the event/reaction (X2) rules in which internal version labels were replaced with plain wording ("the calibrated boundary", "the stable boundary").
