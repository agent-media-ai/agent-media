# Tools

The hosted connector's `tools/list` on the loose surface, with each input schema rendered from the same zod definitions the server validates with (`packages/schema/src/v2/generate.ts`). If this page and `tools/list` ever disagree, `tools/list` wins and CI is broken.

Optional tools open_upload_panel and get_uploads appear only when temporary uploads are enabled. Discover them before use.

Before the first generation, call get_account to confirm this connection and read available credits. If get_account is absent from a cached tool catalog, list_models includes the same account check. Then call the matching quote tool with the intended inputs: quote_ugc for make_ugc, or quote for generate_image, generate_video and generate_audio. Compare the quoted credits to the available balance before submitting. A public model catalog or an upload alone does not prove credit readiness. If the balance check is unavailable, retry it; do not assume zero credits or ask the user to pay again. A balance snapshot does not reserve credits or guarantee provider availability. Upload-only requests do not need generation credits.
When the user wants to provide a product photo, portrait, screenshot, or image reference, check tools/list. If open_upload_panel is available, offer it first: call open_upload_panel with {}, show its inline panel or returned browser link, and wait for the user to finish. Then call get_uploads with the returned session_id and use the exact ready image_url in refs, first_frame, last_frame, or the matching skill image field. Do not start a reference-dependent generation until the required images are ready. The panel accepts still PNG, JPEG, and WebP, up to 10 images and 25 MB each. Images expire 24 hours after the panel is created; show the returned expiry, request a new upload if expired, and never make a permanent copy. Uploading uses no generation credits. Do not ask the user for base64, shell commands, or a public hosting service; do not continuously poll while waiting for them. Only call these tool names when discovered. If they are missing from the connected tool catalog, call upload_image with {} to open the same panel; use the returned panel: upload_key with upload_image to retrieve images after the user finishes. If that reports the panel is unavailable, use upload_image only for an already accessible file or URL. Do not build an upload page or request a local folder as a substitute for the existing panel. Never invent access to a chat attachment: if you cannot read it, explain that and offer the panel when available. Browser fallback works without inline UI support; do not promise an inline panel in every client.
Inspect the native image previews returned with retrieved uploads before describing them or writing image-specific prompts. Never infer what the image shows from an email domain, account metadata, or filename. Use the original URLs for generation, not preview bytes. If the session ID was lost, call get_uploads with {} to list recent sessions, or upload_image with {"upload_key":"panel:recent"} for cached catalogs; retrieve the matching session before asking for re-upload. Account-wide recent uploads may belong to other conversations, so clarify ambiguous selection. Uploading stores an image; it does not attach it to a generation automatically. If the user already requested a generation with these images, continue that request using the returned URLs; do not stop at "upload complete" or ask again whether to use them. If make_ugc fits the requested outcome, pass a person photo as image or a product photo as product_image, then call quote_ugc before spending. Otherwise compose the direct generators: for generate_image, put the relevant image_url values in refs; for generate_video, use refs for product/person/appearance references, or first_frame for animating a still (last_frame only for an explicitly requested ending frame), and never combine refs with frame fields. Preserve the user's intended image roles and model limits; ask only if the role or selection is unclear. Include the URL in the tool arguments, not only in the prompt. Do not substitute a newly generated image for the uploaded reference. If the request was upload-only, report readiness and wait for a generation request; uploading alone does not authorize spending credits. After submitting, poll get_run_status and return the result.

The JSON schema is the envelope; the per-model, per-mode limits (seconds, aspects, qualities, how many refs of each kind) are checked at submit against the catalog cell, see [models.md](models.md).

## make_ugc

```json
{
  "type": "object",
  "properties": {
    "script": {
      "type": "string",
      "minLength": 1,
      "maxLength": 1200,
      "description": "What the person SAYS (lip-synced), any length. A one-liner makes one clip; a full monologue makes the full multi-take video automatically, never trimmed. Usually the only field you set. EXCEPTION: when you also pass product_image the video is a SINGLE take of at most 15s, so the script must be 46 words or fewer."
    },
    "scene_action": {
      "type": "string",
      "minLength": 3,
      "maxLength": 400,
      "description": "A silent action clip (dancing, b-roll, vibes), no dialogue. Use instead of script; needs a `character`."
    },
    "person": {
      "type": "string",
      "minLength": 8,
      "maxLength": 400,
      "description": "Describe the person in words. Omit if you pass image or character."
    },
    "scene": {
      "type": "string",
      "minLength": 3,
      "maxLength": 120,
      "description": "WHERE the video happens, in a few words: 'on a sunny beach, sea behind her', 'in a gym by the weights', 'in bed at night with a lamp on', 'in a parked car'. Set it whenever the user names a place. Without it the video is set in an ordinary room."
    },
    "image": {
      "type": "string",
      "description": "A photo of the person, an https URL. The face is locked to it. If you only hold bytes, call `upload_image` first and pass the URL it returns; base64 is still accepted here but it is printed into the user’s chat and re-sent on every retry."
    },
    "character": {
      "type": "string",
      "description": "Reuse a saved character: its character_id (char_…) OR its character_sheet_url from list_characters."
    },
    "product_image": {
      "type": "string",
      "description": "A photo of a PRODUCT to show/hold/wear, an https URL. If you only hold bytes, call `upload_image` first and pass the URL it returns: base64 travels inside this tool call, is printed into the user’s chat, and is re-sent on every retry. Turns the video into a product ad; needs a `character` to hold it, and limits `script` to ONE take of at most 15s (46 words)."
    },
    "name": {
      "type": "string",
      "maxLength": 80,
      "description": "Name/age/vibe hint, e.g. 'Sophia, 28'."
    },
    "broll_url": {
      "type": "string",
      "format": "uri",
      "pattern": "^https:\\/\\/",
      "description": "A b-roll / gameplay / product video overlaid on the lower half while the person narrates."
    },
    "duration": {
      "type": "number",
      "enum": [
        5,
        10,
        15,
        20,
        25,
        30
      ],
      "description": "IGNORED when `script` is set: length follows how long the words take to say (~2.9 words/sec; one take: up to 15 words → 5s, up to 31 → 10s, up to 46 → 15s; longer scripts become several takes sized the same way). A video is never stretched to reach a requested duration; silence between takes and at the end is cut, so the final video runs about as long as the speech. To get a longer video, write more words (~2.8 words per extra second). The response's take_plan shows the takes. Only used for a silent `scene_action` clip, which has no words to measure."
    },
    "captions": {
      "type": "boolean",
      "description": "Burn in TikTok/Hormozi captions. OFF unless set true, ASK the user whether they want captions (and which caption_style) before generating; never add captions unprompted."
    },
    "caption_style": {
      "type": "string",
      "enum": [
        "hormozi",
        "tiktok",
        "minimal"
      ],
      "default": "hormozi"
    },
    "look": {
      "type": "string",
      "enum": [
        "natural",
        "commercial",
        "raw_iphone"
      ],
      "default": "natural"
    },
    "aspect_ratio": {
      "type": "string",
      "enum": [
        "9:16",
        "1:1"
      ],
      "default": "9:16"
    },
    "music": {
      "anyOf": [
        {
          "type": "boolean"
        },
        {
          "type": "string",
          "maxLength": 120
        }
      ]
    }
  },
  "additionalProperties": false
}
```

## quote_ugc

```json
{
  "type": "object",
  "properties": {
    "script": {
      "type": "string",
      "minLength": 1,
      "maxLength": 1200,
      "description": "What the person SAYS (lip-synced), any length. A one-liner makes one clip; a full monologue makes the full multi-take video automatically, never trimmed. Usually the only field you set. EXCEPTION: when you also pass product_image the video is a SINGLE take of at most 15s, so the script must be 46 words or fewer."
    },
    "scene_action": {
      "type": "string",
      "minLength": 3,
      "maxLength": 400,
      "description": "A silent action clip (dancing, b-roll, vibes), no dialogue. Use instead of script; needs a `character`."
    },
    "person": {
      "type": "string",
      "minLength": 8,
      "maxLength": 400,
      "description": "Describe the person in words. Omit if you pass image or character."
    },
    "scene": {
      "type": "string",
      "minLength": 3,
      "maxLength": 120,
      "description": "WHERE the video happens, in a few words: 'on a sunny beach, sea behind her', 'in a gym by the weights', 'in bed at night with a lamp on', 'in a parked car'. Set it whenever the user names a place. Without it the video is set in an ordinary room."
    },
    "image": {
      "type": "string",
      "description": "A photo of the person, an https URL. The face is locked to it. If you only hold bytes, call `upload_image` first and pass the URL it returns; base64 is still accepted here but it is printed into the user’s chat and re-sent on every retry."
    },
    "character": {
      "type": "string",
      "description": "Reuse a saved character: its character_id (char_…) OR its character_sheet_url from list_characters."
    },
    "product_image": {
      "type": "string",
      "description": "A photo of a PRODUCT to show/hold/wear, an https URL. If you only hold bytes, call `upload_image` first and pass the URL it returns: base64 travels inside this tool call, is printed into the user’s chat, and is re-sent on every retry. Turns the video into a product ad; needs a `character` to hold it, and limits `script` to ONE take of at most 15s (46 words)."
    },
    "name": {
      "type": "string",
      "maxLength": 80,
      "description": "Name/age/vibe hint, e.g. 'Sophia, 28'."
    },
    "broll_url": {
      "type": "string",
      "format": "uri",
      "pattern": "^https:\\/\\/",
      "description": "A b-roll / gameplay / product video overlaid on the lower half while the person narrates."
    },
    "duration": {
      "type": "number",
      "enum": [
        5,
        10,
        15,
        20,
        25,
        30
      ],
      "description": "IGNORED when `script` is set: length follows how long the words take to say (~2.9 words/sec; one take: up to 15 words → 5s, up to 31 → 10s, up to 46 → 15s; longer scripts become several takes sized the same way). A video is never stretched to reach a requested duration; silence between takes and at the end is cut, so the final video runs about as long as the speech. To get a longer video, write more words (~2.8 words per extra second). The response's take_plan shows the takes. Only used for a silent `scene_action` clip, which has no words to measure."
    },
    "captions": {
      "type": "boolean",
      "description": "Burn in TikTok/Hormozi captions. OFF unless set true, ASK the user whether they want captions (and which caption_style) before generating; never add captions unprompted."
    },
    "caption_style": {
      "type": "string",
      "enum": [
        "hormozi",
        "tiktok",
        "minimal"
      ],
      "default": "hormozi"
    },
    "look": {
      "type": "string",
      "enum": [
        "natural",
        "commercial",
        "raw_iphone"
      ],
      "default": "natural"
    },
    "aspect_ratio": {
      "type": "string",
      "enum": [
        "9:16",
        "1:1"
      ],
      "default": "9:16"
    },
    "music": {
      "anyOf": [
        {
          "type": "boolean"
        },
        {
          "type": "string",
          "maxLength": 120
        }
      ]
    }
  },
  "additionalProperties": false
}
```

## generate_video

```json
{
  "type": "object",
  "properties": {
    "prompt": {
      "type": "string",
      "minLength": 3,
      "maxLength": 4000,
      "description": "The shot, as a director would say it: who (age, look), where (setting, light), what happens, camera (phone framing), and, if anyone speaks, the exact words in quotes. About 2.3 words per second. With references, address them as @image1, @video1, @audio1."
    },
    "model": {
      "type": "string",
      "description": "A live video model id from list_models, or \"auto\" to let agent-media pick from recent results. Omit for the default. Call list_models for what each model is good for, its modes, limits and price."
    },
    "first_frame": {
      "type": "string",
      "format": "uri",
      "description": "IMAGE-TO-VIDEO: an https image that becomes frame one of the clip (a still you want animated, a product shot, a portrait). Cannot be combined with refs, video_refs or audio_refs on Seedance."
    },
    "last_frame": {
      "type": "string",
      "format": "uri",
      "description": "Optional with first_frame: the image the clip ends on; the model animates from first to last."
    },
    "refs": {
      "type": "array",
      "items": {
        "type": "string",
        "format": "uri"
      },
      "maxItems": 30,
      "description": "REFERENCE-TO-VIDEO: image references (https URLs): a portrait, a character sheet from list_characters, a product photo. The model keeps that identity/look. Address them in the prompt as @image1, @image2..."
    },
    "video_refs": {
      "type": "array",
      "items": {
        "type": "string",
        "format": "uri"
      },
      "maxItems": 10,
      "description": "Reference clips (https mp4/mov) whose motion, framing or look the model should follow; @video1... in the prompt. Their seconds are billed like output seconds."
    },
    "audio_refs": {
      "type": "array",
      "items": {
        "type": "string",
        "format": "uri"
      },
      "maxItems": 10,
      "description": "Reference audio (https wav/mp3): a voice or a sound the clip should carry; @audio1... in the prompt."
    },
    "seconds": {
      "type": "integer",
      "minimum": 1,
      "maximum": 60,
      "default": 5,
      "description": "Clip length in seconds (the model sets the range; seedance 2.0: 4 to 15, seedance 2.5: 4 to 30). Credits = seconds x the per-second rate at the chosen quality."
    },
    "aspect": {
      "type": "string",
      "enum": [
        "9:16",
        "16:9",
        "1:1",
        "4:3",
        "3:4",
        "3:2",
        "2:3",
        "21:9",
        "adaptive"
      ],
      "description": "9:16 (default for text and reference), 16:9, 1:1, 4:3, 3:4, 3:2, 2:3, 21:9, or adaptive (follows the first frame or reference; the default and the only option in image mode on seedance-2.5)."
    },
    "quality": {
      "type": "string",
      "enum": [
        "480p",
        "720p",
        "1080p",
        "4k"
      ],
      "description": "One of 480p, 720p, 1080p, 4k where the selected model supports it. Omit for the model default. Price per second differs; see list_models."
    },
    "audio": {
      "type": "boolean",
      "default": true,
      "description": "Render native audio when the model supports an audio toggle (speech from the quoted words, ambience). false = silent clip on models that support silence."
    },
    "seed": {
      "type": "integer",
      "minimum": 0,
      "maximum": 2147483647,
      "description": "Only for models whose mode lists seed support (none of the live Seedance modes). Refused elsewhere."
    },
    "voiceover": {
      "type": "object",
      "properties": {
        "text": {
          "type": "string",
          "minLength": 1,
          "maxLength": 1000,
          "description": "Every word the person says, nothing else."
        },
        "language": {
          "type": "string",
          "pattern": "^[a-z]{2}$",
          "description": "ISO 639-1 code of the words (he for Hebrew). Omit to take it from the letters."
        },
        "voice": {
          "type": "string",
          "minLength": 1,
          "maxLength": 64,
          "description": "sulafat (female) or achird (male) for the Gemini voice, or an ElevenLabs stock voice name (sarah, roger, ...). Omit to match the speaker: the gender field, the prompt, or the person in the image. A cloned voice cannot speak Hebrew: the Gemini voice of the same gender speaks instead."
        },
        "gender": {
          "type": "string",
          "enum": [
            "female",
            "male"
          ],
          "description": "Who speaks, when no voice is named."
        }
      },
      "required": [
        "text"
      ],
      "additionalProperties": false,
      "description": "Speech in a language the video models cannot speak (Hebrew): the line is spoken by Gemini TTS and a front-facing still (first_frame, or a keyframe made from the prompt and refs) is lip-synced to it by OmniHuman 1.5. Hebrew lines in quotes get this automatically. 9:16 or 1:1, up to 35 s; not with video_refs or audio_refs. Priced as the voice + the keyframe (when there is no first_frame) + the lip-sync seconds, not the model's per-second rate."
    },
    "captions": {
      "type": "string",
      "enum": [
        "hormozi",
        "minimal",
        "bold",
        "karaoke",
        "clean",
        "tiktok",
        "neon",
        "fire",
        "glow",
        "pop",
        "aesthetic",
        "impact",
        "pastel",
        "electric",
        "boxed",
        "gradient",
        "spotlight"
      ],
      "description": "Optional: burn word-timed captions in this style into the finished clip (a second step after the render, the subtitle price added). Leave out for no captions."
    },
    "request_id": {
      "type": "string",
      "pattern": "^[A-Za-z0-9._:-]{1,128}$",
      "description": "Unique request identity (prefer a UUID). Reuse with identical inputs to recover the same job after a lost response; choose a new identity only for an intentionally new generation."
    }
  },
  "required": [
    "prompt"
  ],
  "additionalProperties": false
}
```

## generate_image

```json
{
  "type": "object",
  "properties": {
    "prompt": {
      "type": "string",
      "minLength": 3,
      "maxLength": 4000,
      "description": "What to paint. Be concrete: subject, age, framing, light, lens, mood, what the hands do."
    },
    "model": {
      "type": "string",
      "description": "A live image model id from list_models, or \"auto\" to let agent-media pick from recent results. Omit for the default (gpt-image-2.5)."
    },
    "refs": {
      "type": "array",
      "items": {
        "type": "string",
        "format": "uri"
      },
      "maxItems": 4,
      "description": "Reference images (https URLs, up to 4). With refs the model EDITS/composes from them (a product into a hand, a portrait re-lit); without, it paints from the prompt alone."
    },
    "size": {
      "type": "string",
      "enum": [
        "1024x1024",
        "1024x1536",
        "1536x1024"
      ],
      "default": "1024x1536",
      "description": "1024x1536 portrait (default, for 9:16 video), 1024x1024 square, 1536x1024 landscape."
    },
    "request_id": {
      "type": "string",
      "pattern": "^[A-Za-z0-9._:-]{1,128}$",
      "description": "Unique request identity (prefer a UUID). Reuse with identical inputs to recover the same job after a lost response; choose a new identity only for an intentionally new generation."
    }
  },
  "required": [
    "prompt"
  ],
  "additionalProperties": false
}
```

## generate_audio

```json
{
  "type": "object",
  "properties": {
    "text": {
      "type": "string",
      "minLength": 1,
      "maxLength": 4000,
      "description": "The words to speak. Emotion tags like [excited] or [whispers] are honoured. Priced per character; see list_models."
    },
    "model": {
      "type": "string",
      "description": "A live audio model id from list_models, or \"auto\". Omit for the default (elevenlabs-tts)."
    },
    "voice": {
      "type": "string",
      "minLength": 1,
      "default": "sarah",
      "description": "A voice name: jessica (young female), sarah (female), liam (young male), chris (male), roger (male), lily (elder female), bill (elder male), matilda (warm), or a raw ElevenLabs voice id. A cloned voice cannot speak Hebrew: a stock voice of the same gender speaks a Hebrew line."
    },
    "tone": {
      "type": "string",
      "enum": [
        "energetic",
        "calm",
        "confident",
        "dramatic"
      ],
      "description": "energetic | calm | confident | dramatic."
    },
    "request_id": {
      "type": "string",
      "pattern": "^[A-Za-z0-9._:-]{1,128}$",
      "description": "Unique request identity (prefer a UUID). Reuse with identical inputs to recover the same job after a lost response; choose a new identity only for an intentionally new generation."
    }
  },
  "required": [
    "text"
  ],
  "additionalProperties": false
}
```

## quote

```json
{
  "type": "object",
  "properties": {
    "kind": {
      "type": "string",
      "enum": [
        "image",
        "video",
        "audio"
      ]
    },
    "input": {
      "type": "object",
      "description": "The exact arguments you would pass to generate_<kind>."
    }
  },
  "required": [
    "kind",
    "input"
  ],
  "additionalProperties": false
}
```

## list_models

```json
{
  "type": "object",
  "properties": {
    "include_candidates": {
      "type": "boolean",
      "description": "Also return planned models (no price, not selectable). Default false."
    }
  },
  "additionalProperties": false
}
```

## get_account

```json
{
  "type": "object",
  "properties": {},
  "additionalProperties": false
}
```

## list_characters

```json
{
  "type": "object",
  "properties": {
    "limit": {
      "type": "integer",
      "minimum": 1,
      "maximum": 100,
      "description": "Max characters to return (default 50)."
    }
  },
  "additionalProperties": false
}
```

## get_run_status

```json
{
  "type": "object",
  "properties": {
    "run_id": {
      "type": "string",
      "description": "The run_id / skill_run_id / job_id returned when you submitted."
    },
    "wait": {
      "type": "boolean",
      "description": "Block until the run finishes or ~45 seconds elapse (default false). A video needs several such calls; just call again."
    }
  },
  "required": [
    "run_id"
  ],
  "additionalProperties": false
}
```

## upload_image

```json
{
  "type": "object",
  "properties": {
    "file_bytes": {
      "type": "integer",
      "description": "Step 1 of the file path: the exact size of the file in bytes (`wc -c < photo.png`). Returns a put_url and an upload_key. Full resolution, up to 25 MB."
    },
    "file_name": {
      "type": "string",
      "description": "Optional with file_bytes: the file name, so the content type is right (.png or .jpg)."
    },
    "upload_key": {
      "type": "string",
      "description": "Reuse the exact upload_key returned earlier: a panel: key retrieves all ready temporary images; a file upload key confirms the completed PUT and returns its image_url."
    },
    "image_base64": {
      "type": "string",
      "description": "The image bytes, base64-encoded. A `data:image/png;base64,...` prefix is accepted and stripped. Last resort: use file_bytes when you can run a shell."
    },
    "image_url": {
      "type": "string",
      "description": "An https URL to fetch and re-host instead."
    }
  },
  "additionalProperties": false
}
```

## rate_run

```json
{
  "type": "object",
  "properties": {
    "run_id": {
      "type": "string",
      "description": "The job id the generate tool returned."
    },
    "score": {
      "type": "integer",
      "minimum": 1,
      "maximum": 5,
      "description": "1 = unusable, 3 = usable with edits, 5 = shipped as-is."
    },
    "note": {
      "type": "string",
      "maxLength": 1000,
      "description": "One line on why (optional)."
    }
  },
  "required": [
    "run_id",
    "score"
  ],
  "additionalProperties": false
}
```

## open_upload_panel

```json
{
  "type": "object",
  "properties": {},
  "additionalProperties": false
}
```

## get_uploads

```json
{
  "type": "object",
  "properties": {
    "session_id": {
      "type": "string",
      "format": "uuid"
    }
  },
  "additionalProperties": false
}
```
