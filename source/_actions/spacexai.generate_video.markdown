---
title: Generate video
action: spacexai.generate_video
domain: spacexai
description: Generate a video with Grok Imagine and save it in Home Assistant's local media.
related_actions:
  - spacexai.publish_media
---

The **Generate video** action creates a video from your prompt using Grok Imagine. You can also supply a local image to animate. Home Assistant saves the resulting MP4 in local media and returns a temporary link to it.

Video generation is billable according to your xAI account's terms. Use this action when you want to create new content, rather than share an existing file.

## Using this action from the user interface

To generate a video from an automation or script:

1. Go to {% my automations title="**Settings** > **Automations & scenes**" %}.
2. Open an automation or script. For a new automation, add a trigger in **When**.
3. In **Then do**, select **Add action** and search for **Generate video**.
4. Under **Account**, select your SpaceXAI account.
5. Enter a **Prompt** describing the video. Optionally, choose a **Source image** from local media.
6. Set a **Response variable**, such as `generated_video`, to receive the result.
7. Select **Save**.

This action uses an account field, not an entity, device, or area target.

### Options in the UI

{% options_ui %}
Account:
  description: The signed-in SpaceXAI account used to generate the video.
  required: true
Prompt:
  description: A description of the video to generate.
  required: true
Model:
  description: "The video model. Defaults to `grok-imagine-video-1.5`."
  required: false
Source image:
  description: A local JPEG, PNG, or WebP image to animate, up to 20 MiB.
  required: false
Duration:
  description: Video length, from 1 to 15 seconds. If omitted, the provider chooses it.
  required: false
Aspect ratio:
  description: "Choose 1:1, 16:9, 9:16, 4:3, 3:4, 3:2, or 2:3. If omitted, the provider chooses it; image-to-video uses the source image's ratio."
  required: false
Resolution:
  description: Choose 480p, 720p, or 1080p. If omitted, the provider chooses it.
  required: false
{% endoptions_ui %}

{% include actions/yaml_header.md %}

In YAML, refer to this action as `spacexai.generate_video`. This example is an action step for an automation or script. Replace `YOUR_SPACEXAI_ENTRY_ID` with the account selected in the visual editor.

{% example %}
action: |
  action: spacexai.generate_video
  data:
    config_entry: YOUR_SPACEXAI_ENTRY_ID
    prompt: "A calm mountain lake with clouds drifting above it."
  response_variable: generated_video
{% endexample %}

### Options in YAML

{% options_yaml %}
config_entry:
  description: The SpaceXAI configuration entry ID selected under Account in the UI.
  required: true
  type: string
prompt:
  description: A nonempty description of the video to generate.
  required: true
  type: string
model:
  description: The video model.
  required: false
  type: string
  default: grok-imagine-video-1.5
image:
  description: "A media-selector mapping with `media_content_id` and `media_content_type`. The selected file must be a local JPEG, PNG, or WebP image, up to 20 MiB."
  required: false
  type: map
duration:
  description: Video length, from 1 to 15 seconds. There is no integration default.
  required: false
  type: integer
aspect_ratio:
  description: "One of `1:1`, `16:9`, `9:16`, `4:3`, `3:4`, `3:2`, or `2:3`. There is no integration default."
  required: false
  type: string
resolution:
  description: "One of `480p`, `720p`, or `1080p`. There is no integration default."
  required: false
  type: string
{% endoptions_yaml %}

## Response data

Set `response_variable` on the action to receive:

- `media_source_id`: The saved video's Home Assistant media identifier.
- `content_type`: The media content type.
- `path`: A signed relative path, valid for one hour.
- `url`: The signed path with a Home Assistant base URL when one is available; otherwise, a relative path.
- `duration`: The duration reported by the provider, when available.
- `model`: The model reported by the provider.

## Good to know

- This action requires administrator access and a loaded SpaceXAI account.
- A configured local media directory is required. Videos are stored in its `spacexai` folder, not the public `/local` directory.
- Generation can take several minutes. Provider moderation, timeouts, and invalid downloads are reported as action errors.
- Downloads are limited to 100 MiB. Failed downloads do not leave partial videos in your media library.
- Anyone with a signed link can access the file for one hour. Share it only with intended recipients. External recipients also need a reachable Home Assistant URL.
- Link expiry does not delete the saved video. **Publish media** can create a new link later.

## Try it yourself

Open {% my developer_services title="**Settings** > **Tools** > **Actions**" %}, select **Generate video**, and fill in the account and prompt. Select **Perform action** to generate a billable video and inspect its response. Find the saved video in local media.

{% include actions/more_examples.md %}

### Automation: create a landscape clip at sunset

Generate a new short landscape clip each evening. Each run creates a billable video.

- **Trigger**: Sun: sunset
- **Action**: Generate video
  - **Account**: Your SpaceXAI account
  - **Prompt**: A calm mountain lake at sunset
  - **Response variable**: `generated_video`

{% details "YAML example for a sunset clip" %}

{% example %}
automation: |
  alias: "Generate an evening landscape clip"
  triggers:
    - trigger: sun
      event: sunset
  actions:
    - action: spacexai.generate_video
      data:
        config_entry: YOUR_SPACEXAI_ENTRY_ID
        prompt: "A calm mountain lake at sunset."
      response_variable: generated_video
{% endexample %}

{% enddetails %}

### Automation: animate a local image on request

Create an [input button](/integrations/input_button/) {% term helper %} separately and select a local image you already uploaded to media. Pressing the button generates a video from that image. Replace the example helper and media identifier with your own.

- **Trigger**: State: the input button changes
- **Action**: Generate video
  - **Account**: Your SpaceXAI account
  - **Source image**: Your local image
  - **Prompt**: Animate the clouds while keeping the landscape still
  - **Response variable**: `generated_video`

{% details "YAML example for animating a local image" %}

{% example %}
automation: |
  alias: "Animate a landscape on request"
  triggers:
    - trigger: state
      entity_id: input_button.animate_landscape
  actions:
    - action: spacexai.generate_video
      data:
        config_entry: YOUR_SPACEXAI_ENTRY_ID
        prompt: "Animate the clouds while keeping the landscape still."
        image:
          media_content_id: "media-source://media_source/local/landscape.png"
          media_content_type: "image/png"
      response_variable: generated_video
{% endexample %}

{% enddetails %}

{% include actions/stuck.md %}

{% include actions/related.md %}
