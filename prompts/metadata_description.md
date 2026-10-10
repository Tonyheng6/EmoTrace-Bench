# `prompts.json` — Field and Label Description

This file documents `prompts.json`, the frozen prompt set used for every experiment reported in the paper. All 448 prompts were submitted verbatim (identical English text) to all 12 evaluated text-to-video systems.

## 1. File layout

```text
{
  "benchmark": "EmoTrace-Bench",
  "description": "...",
  "counts": { "prompts": 448, "intensity_pairs": 224, ... },
  "emotions": [ ...8 labels... ],
  "templates": [ ...14 template names... ],
  "channels": { "C1": "facial expression", ... },
  "hash_definition": "...",
  "prompts": [ { ...one record per prompt... }, ... ]   // 448 records
}
```

Design: 8 emotions × 2 scenes per emotion × 14 templates × 2 intensity levels = 448 prompts = 224 strong–weak pairs. Every template contributes exactly 32 prompts; every emotion contributes exactly 56.

## 2. Per-record fields

| Field | Type | Meaning |
|---|---|---|
| `prompt_id` | string | Unique identifier. Encodes emotion, template, prompt-side channel roles, intensity and scene (see §5). Use it to join with any released per-video outputs. |
| `prompt` | string | The complete English prompt exactly as submitted to the generators. This is the only text used for generation; nothing was prepended or appended. |
| `target_emotion` | string | One of the 8 target emotions (§3). |
| `scene_id` | string | `<emotion>_FP1` or `<emotion>_FP2`. Each emotion has two fixed scenes; all 14 templates × 2 intensities are instantiated within each scene. |
| `template` | string | One of the 14 prompt templates, written as in the paper (Table B.1), e.g. `FACE BODY`. |
| `template_code` | string | Same template in identifier form (underscores), e.g. `FACE_BODY`; this is the form that appears inside `prompt_id`. |
| `template_family` | string | Design family of the template: `emotion-label baseline`, `single channel`, `paired addition`, `representation`, `temporal probe`, `stress test`, `no-character` (Table B.1, column "Family"). |
| `template_complexity` | string | `atomic`, `composed` or `stress` (Table B.1, column "Complexity"). |
| `has_character` | bool | `false` only for the two no-character templates (`EMPTY MOOD`, `EMPTY HARD`). |
| `target_intensity` | string | `weak` or `strong`. The requested intensity of the target emotion. This value is hidden from both human annotators and the automated evaluator during scoring. |
| `pair_id` | string | Identifier of the strong–weak pair: `<scene_id>_<template_code>`. Exactly two records share each `pair_id`, one `weak` and one `strong`. |
| `paired_prompt_id` | string | `prompt_id` of the other member of the same pair. |
| `base_prompt` | string | The shared base sentence of the pair (subject and setting). Identical for both members of a pair. |
| `channel_roles` | object | Prompt-side role of each of the ten cue channels for this prompt (§4). |
| `explicit_channels` | list | Channels whose role is `explicit` (convenience view of `channel_roles`). |
| `implicit_channels` | list | Channels whose role is `implicit` (convenience view of `channel_roles`). |
| `intensity_varied_segments` | list | The text segments in which the two pair members differ; everything else in the pair is identical (§6). This is the field the paper refers to as "allowed changed atoms" (Appendix B.6); `|atoms|` in Table B.1 is its length. |
| `word_count` | int | Number of whitespace-separated tokens in `prompt`. Medians per template match Table B.1 ("Words"). |
| `sentence_count` | int | Number of sentences in `prompt`. |
| `prompt_sha256` | string | SHA-256 (hex) of the UTF-8 bytes of `prompt`. All 448 hashes are distinct. |

## 3. Emotions

`amusement`, `excitement`, `awe`, `contentment` (positive) and `anger`, `fear`, `disgust`, `sadness` (negative). The emotion definitions used by the automated evaluator are given in each system prompt under `evaluator/` (field `emotion_definition`).

## 4. Cue channels and prompt-side roles

| Code | Channel | Group |
|---|---|---|
| C1 | facial expression | character |
| C2 | bodily behavior | character |
| V1 | lighting | visual form |
| V2 | color | visual form |
| V3 | composition | visual form |
| V4 | visual dynamics (camera and on-screen motion) | visual form |
| A1 | sound | audio |
| X1 | scene / space | context |
| X2 | event / reaction | context |
| T1 | temporal organization | temporal |

`channel_roles` values:

| Value | Meaning |
|---|---|
| `explicit` | The prompt contains a text segment that directly specifies this channel ("E" in Table B.2). |
| `implicit` | The channel is not written in the prompt but is within the template's intended scope and is still observed in the generated video ("I" in Table B.2). |
| `not_applicable` | Outside the intended scope of this template ("–" in Table B.2). Such channels are excluded from that prompt's cue-channel denominators; they are never scored 0. |
| `output_only` | Used only for A1 (sound). Sound is never written in any prompt and is scored only from the generated audio track; it is therefore not part of the prompt-side role matrix. |

Notes:

- V1–V4 are always specified jointly by the visual-form segments; they are never manipulated independently (Table B.1 note).
- T1 is never written directly; where present it is always `implicit`.
- The role assignment is fixed before generation and hidden during scoring.

Role matrix reproduced from the file (identical to Table B.2 of the paper):

| Template | C1 | C2 | V1 | V2 | V3 | V4 | X1 | X2 | T1 |
|---|---|---|---|---|---|---|---|---|---|
| EMO | I | I | I | I | I | I | I | I | I |
| FACE | E | I | I | I | I | I | – | – | I |
| BODY | I | E | I | I | I | I | – | – | I |
| SCENE | I | I | I | I | I | I | E | – | I |
| EVENT | I | I | I | I | I | I | – | E | I |
| FACE BODY | E | E | I | I | I | I | – | – | I |
| SCENE REACT | E | E | I | I | I | I | E | – | I |
| EVENT REACT | E | E | I | I | I | I | – | E | I |
| CONTEXT MOOD | I | I | E | E | E | E | – | – | I |
| CONTEXT HARD | I | I | E | E | E | E | – | – | I |
| FACE TRANSITION | E | I | I | I | I | I | – | – | I |
| ALL DYNAMIC | E | E | E | E | E | E | E | E | I |
| EMPTY MOOD | – | – | E | E | E | E | E | – | – |
| EMPTY HARD | – | – | E | E | E | E | E | – | – |

## 5. Reading a `prompt_id`

Example: `fear_ALL_DYNAMIC_A-C1-C2-V1-V2-V3-V4-X1-X2_E-T1_I-WEAK_R-FP1`

| Part | Meaning |
|---|---|
| `fear` | target emotion |
| `ALL_DYNAMIC` | template code |
| `A-C1-C2-V1-V2-V3-V4-X1-X2` | channels written explicitly in the prompt (`A-NA` = none) |
| `E-T1` | channels within scope but not written, i.e. implicit (`E-NA` = none) |
| `I-WEAK` / `I-STRONG` | target intensity |
| `R-FP1` / `R-FP2` | scene index within the emotion |

The identifier is informational; the authoritative role information is `channel_roles`.

## 6. Strong–weak pairs and `intensity_varied_segments`

Both members of a pair share the same emotion, scene, template and `base_prompt`, and differ only in the preregistered intensity expressions inside the segments listed in `intensity_varied_segments`. Segment names:

| Segment | Content that changes between weak and strong |
|---|---|
| `emotion_label` | the explicit emotion word or phrase (EMO template) |
| `face` | facial-expression description (C1) |
| `body` | bodily-behavior description (C2) |
| `face_transition` | facial-expression change over time (FACE TRANSITION template) |
| `scene_appraisal` | how the scene is described and appraised (X1) |
| `event_appraisal` | the event and its appraisal (X2) |
| `mood` | experiential atmosphere wording for lighting, color, composition and motion (V1–V4, "MOOD" templates) |
| `hard_visual_form` | explicit cinematic-form wording for V1–V4 ("HARD" templates) |

To find the partner of any prompt: look up `paired_prompt_id`, or group records by `pair_id`. To compare intensity control, evaluate both members of a pair and compare the perceived intensity of the two generated videos (see the paper's paired intensity-control metric, Appendix C).

## 7. Cross-template contrasts

The seven controlled template contrasts analyzed in the paper (Table B.3) are formed by selecting, within the same `scene_id` and `target_intensity`, the records of the two templates named in each contrast (e.g. `SCENE` vs `SCENE REACT`). No additional field is needed.

## 8. Integrity checks

The file satisfies the following, which can be re-verified from the file alone:

- 448 records; 224 distinct `pair_id`, each with exactly one `weak` and one `strong` member;
- 448 distinct `prompt` strings and 448 distinct `prompt_sha256` values;
- `prompt_sha256` equals `sha256(prompt.encode("utf-8")).hexdigest()` for every record;
- within every pair, `base_prompt`, `target_emotion`, `scene_id` and `template` are identical;
- every template has 32 records, every emotion 56, every (emotion, scene) 28.

These correspond to the first five rows of Table B.4 in the paper (448 records; 224 / 224 strong / weak; 448 / 448 unique texts; 448 / 448 unique hashes; 224 / 224 pairs passing the pairing contract). The two segment-reuse rows of Table B.4 were computed on the segment inventory from which prompts were assembled; this file contains the assembled prompts, so those two rows are not re-derivable from it alone.

## 9. Correspondence with paper terminology

| Paper | This file |
|---|---|
| template / prompt condition | `template`, `template_code` |
| scene (two per emotion) | `scene_id` |
| strong–weak pair, pair pointer | `pair_id`, `paired_prompt_id` |
| prompt-side cue roles, E / I / – (Table B.2) | `channel_roles` = `explicit` / `implicit` / `not_applicable` |
| allowed changed atoms, \|atoms\| (Table B.1, B.6) | `intensity_varied_segments` |
| Words (Table B.1, median) | `word_count` |
| prompt hash | `prompt_sha256` |
