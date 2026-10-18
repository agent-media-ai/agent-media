# seedance-2.5

> Generated facts come from `packages/schema/src/v2/models.ts`. Edit numbers there, not here.

The premium video engine. Same inputs and pipeline as 2.0, the provider's newer generation, at about twice the credits per second of 2.0 at 480p and 720p, and 1.5 times at 1080p. No side-by-side quality comparison has been recorded yet; treat the quality tier as the provider's claim.

| | |
|---|---|
| Kind | video (default for `generate_video`) |
| Tier | premium |
| Status | **live** |
| Provider | evolink |
| Modes | text-to-video, image-to-video, reference-to-video |
| Features | native-audio, lip-sync, first-frame, first-and-last-frame, image-references, video-references, audio-references |
| Limits | 4 to 30 s (per mode below); worker waits up to 90 min |
| User price | 60 credits/s at 480p, 125 credits/s at 720p, 225 credits/s at 1080p. Reference video seconds are billed at the same rate. |
| Quality / speed | premium / slow |
| Verified | modes: text, image, reference (see below) |

## Modes

The mode is derived from the request: `first_frame` means image mode, `refs` / `video_refs` / `audio_refs` mean reference mode, neither means text mode.

| Mode | Provider model | Inputs | Seconds | Aspect | Quality | Seed | Verified |
|---|---|---|---|---|---|---|---|
| text | `seedance-2.5-text-to-video` | prompt only | 4 to 30 | 9:16, 16:9, 1:1, 4:3, 3:4, 21:9, adaptive (default 9:16) | 480p, 720p, 1080p | no | 2026-09-05. generate_video, text only, 4s, 720p, 396 credits; the provider took about 25 min (run 431f82ba-9e9e-4644-beb3-b1f67c0de91e) |
| image | `seedance-2.5-image-to-video` | first_frame (+ last_frame) | 4 to 30 | adaptive (default adaptive) | 480p, 720p, 1080p | no | 2026-09-06. first_frame, 4s, 480p, adaptive; rendered in 262 s (run task-unified-1788681109-pzf704jd) |
| reference | `seedance-2.5-reference-to-video` | 30 images, 10 clips (30 s total), 10 audio (30 s total) | 4 to 30 | 9:16, 16:9, 1:1, 4:3, 3:4, 21:9, adaptive (default 9:16) | 480p, 720p, 1080p | no | 2026-09-06. generate_video with a product image ref via Claude Code, 8s, 720p (perfume UGC run) |

- text: No seed.
- image: first_frame becomes frame one; last_frame (optional) becomes the final frame.
- image: aspect must be "adaptive" on this model in image mode: the clip takes the frame's ratio. Any other aspect is refused.
- image: Frames only: refs, video_refs and audio_refs are not accepted in this mode.
- reference: address references as @image1 @video1 @audio1 (numbered per list, from 1).
- reference: Address references as @image1.., @video1.., @audio1.. (numbered per list).
- reference: Reference clips: 2 to 30 s each, 30 s in total, up to the provider high-resolution cap, up to 200 MB; their seconds are billed like output seconds. Audio references may stand alone here.
- reference: Never write edit or extend intent ("edit the video", "add", "remove", "replace", "extend", "continue") into a reference prompt: the provider reclassifies the task and fails it minutes later. Describe the NEW clip you want.

## How to use it

**Pick this when** the user asks for Seedance 2.5 or the best quality, or the clip needs what only 2.5 makes (longer than 15 s, more references, audio references alone).

**Best for:** hero product ads; close-up faces; one clip that has to be the best; clips longer than 15 s.

**Avoid for:** quick drafts where the shorter seedance-2.0 wait matters more than detail; anyone who cannot wait 10 to 25 minutes.

**Prompting:**

- Same director-style prompt and @image1 references as seedance-2.0.
- In image mode leave aspect out (it is adaptive); the frame decides the ratio.
- Do not put edit or extend wording in a reference prompt.

**Latency:** about 10 minutes per clip (30-day median), up to about 24; plan the wait.
<!-- /generated -->

## Usage notes

- This is the paid default for loose video generation and the paid fixed `selfie` / `crazy-look` engine.
- Recorded runs: text mode job 431f82ba (2026-09-05, 4s, 396 credits, about 25 minutes at the provider) and a reference-mode 8s product clip (2026-09-06). Budget for the wait (2.0 renders 5s in about 3 minutes).
- `generate_video` renders 4 to 30 s on it; the fixed `selfie` / `crazy-look` skills take 5, 10 or 15 s per take.

## How to select it

- `generate_video` over MCP by omitting `model` (the paid default) or with `"model": "seedance-2.5"`; `POST /v2/generate/video` over REST, same body. The provider model follows the mode: `first_frame` uses `seedance-2.5-image-to-video`, references use `seedance-2.5-reference-to-video`, a bare prompt uses `seedance-2.5-text-to-video`
- `agent-media selfie ...` or `agent-media crazy-look ...` with no `--engine`, or pass `--engine seedance-2.5` explicitly
- `POST /v2/selfie` / `POST /v2/crazy-look` over REST with `"engine": "seedance-2.5"` or with `engine` omitted
- `make_ugc` has no engine field: a paid run renders on the Seedance 2.0 family; only the free first ad renders on seedance-2.5
