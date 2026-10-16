# Choosing a model

Generated from `packages/schema/src/v2/models.ts`. Call the `list_models` MCP tool (or `GET /v1/models`) for the live version.

## The one rule

**Default paid video to `seedance-2.5`.** It is the recommended paid model for talking-head UGC, product-in-hand, animating a still and crazy-look at 60 credits/s at 480p, 125 at 720p, 225 at 1080p. `seedance-2.0` remains selectable for the older faster path at 30 credits/s at 480p, 60 at 720p, 150 at 1080p. Default quality: 480p on `seedance-2.5`, 720p on `seedance-2.0`; 480p is the cheap draft, 1080p the dear finish.

## How to select

Pass the id as `model` to `generate_video` / `generate_image` / `generate_audio` (MCP) or `POST /v2/generate/<kind>` (REST). Omit it for the default, or pass `"auto"`. Live video models are also the `engine` of the fixed REST skills (`/v2/selfie`, `/v2/crazy-look`, CLI `--engine`). Only live models are accepted; a planned id returns a 400 naming the live ones.

## Video modes

The mode follows from the fields: `first_frame` (and optional `last_frame`) is image-to-video, `refs` / `video_refs` / `audio_refs` is reference, neither is text. Frames and refs cannot be mixed on Seedance. Reference clip seconds (video_refs) are billed at the same per-second rate as output seconds. Credits below are per output second.

| Model | Mode | Inputs | Seconds | Aspects | Credits/s | Verified |
|---|---|---|---|---|---|---|
| seedance-2.0 | text | prompt only | 4 to 15 | 9:16, 16:9, 1:1, 4:3, 3:4, 21:9, adaptive (default 9:16) | 30 at 480p, 60 at 720p, 150 at 1080p | 2026-09-06: 4s, 16:9, 480p, native audio; rendered in 231 s |
| seedance-2.0 | image | first_frame + optional last_frame | 4 to 15 | 9:16, 16:9, 1:1, 4:3, 3:4, 21:9, adaptive (default adaptive) | 30 at 480p, 60 at 720p, 150 at 1080p | 2026-09-06: first_frame only (4s, 480p, adaptive) and first_frame + last_frame (task-unified-1788681109-w09254uu); both rendered in about 3 to 5 min |
| seedance-2.0 | reference | refs up to 9, video_refs up to 3 (15 s total), audio_refs up to 3 (15 s total); audio needs an image or video beside it | 4 to 15 | 9:16, 16:9, 1:1, 4:3, 3:4, 21:9, adaptive (default 9:16) | 30 at 480p, 60 at 720p, 150 at 1080p | 2026-09-06: image ref (4s, 480p) and image ref + 8 s reference clip (task-unified-1788681110-hdbufncn); earlier 5s 720p portrait run b0011e92 via Claude Code |
| seedance-2.5 | text | prompt only | 4 to 30 | 9:16, 16:9, 1:1, 4:3, 3:4, 21:9, adaptive (default 9:16) | 60 at 480p, 125 at 720p, 225 at 1080p | 2026-09-05: generate_video, text only, 4s, 720p, 396 credits; the provider took about 25 min |
| seedance-2.5 | image | first_frame + optional last_frame | 4 to 30 | adaptive (default adaptive) | 60 at 480p, 125 at 720p, 225 at 1080p | 2026-09-06: first_frame, 4s, 480p, adaptive; rendered in 262 s |
| seedance-2.5 | reference | refs up to 30, video_refs up to 10 (30 s total), audio_refs up to 10 (30 s total) | 4 to 30 | 9:16, 16:9, 1:1, 4:3, 3:4, 21:9, adaptive (default 9:16) | 60 at 480p, 125 at 720p, 225 at 1080p | 2026-09-06: generate_video with a product image ref via Claude Code, 8s, 720p (perfume UGC run) |

Notes for seedance-2.0:

- text: No seed. Every render is new; keep a series consistent with references, not seeds.
- image: first_frame becomes frame one of the clip; last_frame (optional) becomes the final frame and the model animates between them.
- image: Frames only: refs, video_refs and audio_refs are not accepted in this mode. To keep an identity AND set the frame, put the frame image in refs and describe it as @image1.
- image: aspect "adaptive" (default) follows the first frame. Frame images: jpeg/png/webp, ratio between 0.4 and 2.5, 300 to 6000 px, up to 30 MB.
- reference: Address references in the prompt: "@image1 holds @image2 and says ...". Numbering is per list: refs are @image1.., video_refs are @video1.., audio_refs are @audio1...
- reference: Reference clips: 2 to 15 s each, 15 s in total, mp4/mov 480p to 1080p, 24 to 60 fps, up to 50 MB. Their seconds are billed like output seconds.
- reference: Audio references: wav/mp3, 2 to 15 s each, 15 s in total, up to 15 MB. Audio alone is not accepted: add an image or video reference.
- reference: aspect "adaptive" follows the first video reference, else the first image, else the prompt.

Notes for seedance-2.5:

- text: No seed.
- image: first_frame becomes frame one; last_frame (optional) becomes the final frame.
- image: aspect must be "adaptive" on this model in image mode: the clip takes the frame's ratio. Any other aspect is refused.
- image: Frames only: refs, video_refs and audio_refs are not accepted in this mode.
- reference: Address references as @image1.., @video1.., @audio1.. (numbered per list).
- reference: Reference clips: 2 to 30 s each, 30 s in total, up to the provider high-resolution cap, up to 200 MB; their seconds are billed like output seconds. Audio references may stand alone here.
- reference: Never write edit or extend intent ("edit the video", "add", "remove", "replace", "extend", "continue") into a reference prompt: the provider reclassifies the task and fails it minutes later. Describe the NEW clip you want.

## What the numbers mean

`list_models` / `GET /v1/models` add a `recent` block per model: the last 30 days of loose-surface runs, `runs`, `fail_rate`, `auto_score` (0 to 1, an auto-judge grades every job: 3 frames or the image against the realism rubric, prompt adherence, identity match when refs were given), `scored`, `user_score` (1 to 5 from `rate_run`), `rated`, `p50_seconds`. `null` until a model has run.

`model: "auto"` is one printed rule over those numbers: the kind's default, unless it failed more than 25% of at least 10 recent runs and another live model is healthy, or a live model within 1.5x the default's price beats its auto score by 0.10 or more over at least 10 judged runs. Only models that have the request's mode are candidates. The quote and the submit response say which model was chosen and why.

## Live models

| Model | Kind | Tier | Price | Modes and limits | Best for | Avoid for |
|---|---|---|---|---|---|---|
| [seedance-2.0](models/seedance-2.0.md) | video | standard | 30 credits/s at 480p, 60 at 720p, 150 at 1080p | text (prompt only): 4 to 15 s, default aspect 9:16, 480p/720p/1080p; image-to-video (first_frame + optional last_frame): 4 to 15 s, default aspect adaptive, 480p/720p/1080p; reference (refs / video_refs / audio_refs, up to 9 images, 3 clips, 15 s total, 3 audio, 15 s total, not alone): 4 to 15 s, default aspect 9:16, 480p/720p/1080p | faster paid drafts; bulk daily posts; legacy 720p workflows; talking-head UGC; animating a still (first frame) into a clip | clips over 15s; hero shots where 2.5 detail is the default expectation; anything that needs a seed |
| [seedance-2.5](models/seedance-2.5.md) (default) | video | premium | 60 credits/s at 480p, 125 at 720p, 225 at 1080p | text (prompt only): 4 to 30 s, default aspect 9:16, 480p/720p/1080p; image-to-video (first_frame + optional last_frame): 4 to 30 s, aspect adaptive only, 480p/720p/1080p; reference (refs / video_refs / audio_refs, up to 30 images, 10 clips, 30 s total, 10 audio, 30 s total): 4 to 30 s, default aspect 9:16, 480p/720p/1080p | paid video default; hero product ads; close-up faces; one clip that has to be the best | jobs where the user explicitly wants the older faster 2.0 engine; anyone who cannot wait 15 to 30 minutes |
| [gpt-image-2.5](models/gpt-image-2.5.md) (default) | image | premium | 20 credits per image | 1024x1024, 1024x1536, 1536x1024; refs up to 4 | portraits and character sheets that a video has to keep; the first frame of a clip; product in hand; edits that must not lose the face | bulk throwaway drafts where gpt-image-2.5-flare is faster |
| [gpt-image-2.5-flare](models/gpt-image-2.5-flare.md) | image | standard | 20 credits per image | 1024x1024, 1024x1536, 1536x1024; refs up to 4 | variants and drafts at the same quality tier; batches of frames; anything where a few seconds matter | the one sheet a whole series depends on, where gpt-image-2.5 edits hold identity a little better |
| [gpt-image-2](models/gpt-image-2.md) | image | standard | 20 credits per image | 1024x1024, 1024x1536, 1536x1024; refs up to 4 | the previous generation, kept selectable for runs that were built on it | new work: gpt-image-2.5 is the default and holds identity better |
| [nano-banana-pro](models/nano-banana-pro.md) | image | premium | 26 credits per image | 1K, 2K; refs up to 4 | image edits with complex instructions; text-heavy product graphics; composites that need reasoning over references | cheap drafts; jobs where GPT Image identity continuity is already enough |
| [elevenlabs-tts](models/elevenlabs-tts.md) (default) | audio | standard | 16 credits per 1,000 characters | up to 4000 characters per call | voiceover on b-roll; narration; a standalone voice file; an audio reference for generate_video | lip-synced talking head: generate_video renders speech natively |

## How to use each model

From the catalog `usage` card. `list_models` returns the same text plus the last 30 days of real results.

### seedance-2.0

- **Pick it when** you explicitly want the older faster standard engine, or you are making volume drafts where the shorter wait matters more than 2.5 detail.
- **Best for:** faster paid drafts; bulk daily posts; legacy 720p workflows; talking-head UGC; animating a still (first frame) into a clip.
- **Avoid for:** clips over 15s; hero shots where 2.5 detail is the default expectation; anything that needs a seed.
- **Latency:** about 3 minutes for a 5 s clip at 720p.
- **Price:** 30 credits/s at 480p, 60 at 720p, 150 at 1080p; reference clip seconds (video_refs) are billed at the same per-second rate as output seconds.
- **Modes:**
  - text (prompt only): 4 to 15 s, default aspect 9:16, 480p/720p/1080p
  - image-to-video (first_frame + optional last_frame): 4 to 15 s, default aspect adaptive, 480p/720p/1080p
  - reference (refs / video_refs / audio_refs, up to 9 images, 3 clips, 15 s total, 3 audio, 15 s total, not alone): 4 to 15 s, default aspect 9:16, 480p/720p/1080p
- **Prompting:**
  - Write the shot as a director: who (age, look), where (setting, light), what happens, phone framing, and the exact spoken words in quotes. About 2.3 words per second.
  - Keep a person consistent across clips with refs (a portrait or a character sheet), addressed as @image1; the model has no seed.
  - To animate a specific still, pass it as first_frame; add last_frame to control where the motion ends.

### seedance-2.5 (default video)

- **Pick it when** you are making paid video and the user did not explicitly ask for the older faster 2.0 engine; default 480p keeps the draft tier focused.
- **Best for:** paid video default; hero product ads; close-up faces; one clip that has to be the best.
- **Avoid for:** jobs where the user explicitly wants the older faster 2.0 engine; anyone who cannot wait 15 to 30 minutes.
- **Latency:** 12 to 25 minutes per clip; plan the wait.
- **Price:** 60 credits/s at 480p, 125 at 720p, 225 at 1080p; reference clip seconds (video_refs) are billed at the same per-second rate as output seconds.
- **Modes:**
  - text (prompt only): 4 to 30 s, default aspect 9:16, 480p/720p/1080p
  - image-to-video (first_frame + optional last_frame): 4 to 30 s, aspect adaptive only, 480p/720p/1080p
  - reference (refs / video_refs / audio_refs, up to 30 images, 10 clips, 30 s total, 10 audio, 30 s total): 4 to 30 s, default aspect 9:16, 480p/720p/1080p
- **Prompting:**
  - Same director-style prompt and @image1 references as seedance-2.0.
  - In image mode leave aspect out (it is adaptive); the frame decides the ratio.
  - Do not put edit or extend wording in a reference prompt.

### gpt-image-2.5 (default image)

- **Pick it when** you are making the image a video will be built on: a portrait, a character sheet, a first frame, or an edit that has to keep the same person.
- **Best for:** portraits and character sheets that a video has to keep; the first frame of a clip; product in hand; edits that must not lose the face.
- **Avoid for:** bulk throwaway drafts where gpt-image-2.5-flare is faster.
- **Latency:** about 30 seconds, a little longer for an edit with references.
- **Price:** 20 credits per image.
- **Prompting:**
  - Concrete subject, age, framing, light, what the hands do; it follows layout instructions like "four poses on a plain background" or "headroom for a caption".
  - With refs it edits or composes from them and holds the identity across poses, which is what makes a character sheet usable as a video reference.
  - Pass the result straight to generate_video as first_frame (to animate it) or in refs (to keep that person across clips).

### gpt-image-2.5-flare

- **Pick it when** you want the same look as gpt-image-2.5 but faster, or you are making several images at once.
- **Best for:** variants and drafts at the same quality tier; batches of frames; anything where a few seconds matter.
- **Avoid for:** the one sheet a whole series depends on, where gpt-image-2.5 edits hold identity a little better.
- **Latency:** about 15 to 30 seconds.
- **Price:** 20 credits per image.
- **Prompting:**
  - Same prompts as gpt-image-2.5; it is the speed tier of the same family.

### gpt-image-2

- **Pick it when** you are reproducing something that was made on gpt-image-2; otherwise take the default.
- **Best for:** the previous generation, kept selectable for runs that were built on it.
- **Avoid for:** new work: gpt-image-2.5 is the default and holds identity better.
- **Latency:** under a minute.
- **Price:** 20 credits per image.
- **Prompting:**
  - Concrete subject, age, framing, light, what the hands do; it follows layout instructions like "headroom for a caption".
  - With refs it EDITS or composes from them (a product into a hand, a portrait re-lit); without refs it paints from the prompt alone.

### nano-banana-pro

- **Pick it when** you need Gemini/Nano Banana Pro image reasoning, text rendering or reference editing from generate_image.
- **Best for:** image edits with complex instructions; text-heavy product graphics; composites that need reasoning over references.
- **Avoid for:** cheap drafts; jobs where GPT Image identity continuity is already enough.
- **Latency:** about 30 to 90 seconds.
- **Price:** 26 credits per image.
- **Prompting:**
  - Be explicit about text, layout and relationships between referenced objects.
  - With refs it edits or composes from them; pass only the images that must influence the final composition.

### elevenlabs-tts (default audio)

- **Pick it when** you need a clean voice track and no face.
- **Best for:** voiceover on b-roll; narration; a standalone voice file; an audio reference for generate_video.
- **Avoid for:** lip-synced talking head: generate_video renders speech natively.
- **Latency:** seconds.
- **Price:** 16 credits per 1,000 characters.
- **Prompting:**
  - Emotion tags like [excited] or [whispers] are honoured; keep sentences short for pacing.
  - Seven named voices (sarah default) or a raw ElevenLabs voice id.

100 credits = 1 USD.
