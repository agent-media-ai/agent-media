# wan-3.0-prime

> Generated facts come from `packages/schema/src/v2/models.ts`. Edit numbers there, not here.

The ad engine for the subscription plans: a creator and a product from two reference images, 10 s, at 480p or 720p, in about one to two minutes. Priced at provider cost x2.

| | |
|---|---|
| Kind | video |
| Tier | standard |
| Status | **live** |
| Provider | evolink |
| Modes | text-to-video, image-to-video, reference-to-video |
| Features | native-audio, first-frame, first-and-last-frame, image-references, video-references, audio-references |
| Limits | 2 to 30 s (per mode below); worker waits up to 15 min |
| User price | 12 credits/s at 480p, 24 credits/s at 720p, 48 credits/s at 1080p. Reference video seconds are billed at the same rate. |
| Quality / speed | good / fast |
| Verified | modes: text, image, reference (see below) |

## Modes

The mode is derived from the request: `first_frame` means image mode, `refs` / `video_refs` / `audio_refs` mean reference mode, neither means text mode.

| Mode | Provider model | Inputs | Seconds | Aspect | Quality | Seed | Verified |
|---|---|---|---|---|---|---|---|
| text | `wan3.0-prime-text-to-video` | prompt only | 2 to 30 | 9:16, 16:9, 1:1, 4:3, 3:4, adaptive (default 9:16) | 480p, 720p, 1080p | no | 2026-10-23. text only, 5s, 480p, 9:16, audio; rendered in about 60 s (run task-unified-1792741071-lgdeumm3) |
| image | `wan3.0-prime-image-to-video` | first_frame (+ last_frame) | 2 to 30 | adaptive (default adaptive) | 480p, 720p, 1080p | no | 2026-10-23. first_frame, 5s, 480p, audio; rendered in about 50 s (run task-unified-1792741072-41hrtgtv) |
| reference | `wan3.0-prime-reference-video` | 10 images, 5 clips (15 s total), 5 audio (15 s total) | 2 to 30 | 9:16, 16:9, 1:1, 4:3, 3:4, adaptive (default 9:16) | 480p, 720p, 1080p | no | 2026-10-23. creator portrait + product image refs, 10s, 720p, 9:16, audio; about 80 to 120 s a render (X ad batch, Oct 22-23) |

- text: Text only. No negative_prompt.
- image: first_frame becomes frame one; last_frame (optional) becomes the final frame.
- image: Frames only: refs, video_refs and audio_refs are not accepted in this mode.
- reference: address references as "Image 1", "Video 1", "Audio 1" (capitalised, space before the number).
- reference: Address references as "Image 1", "Image 2" (numbered per list, in array order). Each image should hold one subject: a person, or a product.
- reference: Reference video seconds are billed like output seconds.

## How to use it

**Pick this when** the default for ads: a creator and the product, 10 s, at 480p or 720p; fast and the cheapest per second.

**Best for:** 10 s UGC and talking ads; a creator holding the product from two reference images; many drafts fast.

**Avoid for:** lip-sync to a given audio track; shots that must keep small printed text on a label.

**Prompting:**

- Put the person in refs first and the product second, then write "the woman in Image 1 holds the bottle from Image 2".
- Say what the hands do and that there are exactly two hands; say "no captions, no on-screen text".
- Write the spoken line in quotes; keep it under about 25 words for 10 s.

**Latency:** about 1 to 2 minutes for 10 s.
<!-- /generated -->

## How to use it

- Reference mode is the usual path: the person first in `refs`, the product second, and the prompt names them as "Image 1" and "Image 2".
- Say what the hands do and that there are exactly two hands; ask for no captions and no on-screen text.
- Keep the spoken line short (about 25 words for 10 s).
