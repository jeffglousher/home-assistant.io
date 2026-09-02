---
title: SpaceXAI
description: Instructions on how to use Grok conversation, AI tasks, and speech with SpaceXAI
ha_category:
  - AI
  - Voice
ha_release: 2026.10
ha_iot_class: Cloud Polling
ha_config_flow: true
ha_codeowners:
  - '@jeffglousher'
ha_domain: spacexai
ha_integration_type: service
ha_platforms:
  - ai_task
  - conversation
  - stt
  - tts
ha_quality_scale: bronze
related:
  - docs: /voice_control/voice_remote_expose_devices/
    title: Exposing entities to Assist
  - docs: /voice_control/assist_create_open_ai_personality/
    title: Create an AI personality
  - url: https://x.ai/
    title: xAI
---

The **SpaceXAI** {% term integration %} adds a Grok-powered conversation agent from [xAI](https://x.ai/) to Home Assistant. You sign in to your xAI account in a browser by using a one-time code. The integration does not accept an API key.

To let Grok provide information about your Home Assistant entities or control them, select the Assist API during setup. Grok can access only the entities that you [expose to Assist](/voice_control/voice_remote_expose_devices/).

This integration does not integrate with [sentence triggers](/docs/automation/trigger/#sentence-trigger).

## Prerequisites

- An xAI account with access to Grok.
- A device with a web browser to complete sign-in.

{% include integrations/config_flow.md %}

During setup, you can configure the following settings:

{% configuration_basic %}
Model:
  description: "The Grok model used by the conversation agent. The available models come from your signed-in account."
Instructions:
  description: "Instructions for how Grok should respond. You can use a [Home Assistant template](/docs/configuration/templating/)."
Control Home Assistant:
  description: "The Home Assistant language model APIs that Grok can use. Select Assist to let Grok work with entities that are exposed to it."
{% endconfiguration_basic %}

## Configuration options

To add another conversation agent, open SpaceXAI under {% my integrations title="**Settings** > **Devices & services**" %} and select **Add conversation agent**. Each agent has its own model, instructions, and tool settings.

To change an existing agent, open its menu and select **Reconfigure**. The following tools are available during initial setup, when adding an agent, and when reconfiguring one:

{% configuration_basic %}
Web search:
  description: "Allow Grok to search the web. Disabled by default."
X search:
  description: "Allow Grok to search X. Disabled by default."
Code interpreter:
  description: "Allow Grok to run code using the provider's code interpreter. Disabled by default."
{% endconfiguration_basic %}

These tools run on the provider's service. They do not grant access to additional Home Assistant entities.

For an AI task, **Model** selects the model for text and structured data. Image generation uses `grok-imagine-image-2.0` and does not have a model setting.

For text-to-speech, **Speed** controls how quickly the voice speaks. It ranges from 0.7 to 1.5, with a default of 1.0. Speech-to-text has no additional configuration settings.

## Supported functionality

### Conversation

The integration creates a Grok conversation agent during setup. You can add more agents for different tasks. Use an agent in the Assist dialog or select it as the conversation agent for a [voice assistant](/voice_control/).

Grok can answer in any language supported by the selected model. If you allow it to use the Assist API, it can also get information from Home Assistant and control exposed entities.

You can attach JPEG or PNG images and PDF documents to a conversation request. Grok receives the attachments from your latest message. The combined attachment size must not exceed 20 MiB.

### AI tasks

New accounts also receive a **Grok AI Task** entity. If you already added your account, open the SpaceXAI integration and select **Add AI task**. You can change an AI task's model through **Reconfigure**.

Use the existing [AI Task actions](/integrations/ai_task/) in an automation or script:

- **Generate data** returns text or structured data from your instructions. It accepts JPEG, PNG, and PDF attachments with a combined size of up to 20 MiB. For example, you can ask Grok to summarize a document or extract a list of items from an image.
- **Generate image** creates an image from your instructions. To edit images, attach one to five JPEG or PNG images with a combined size of up to 20 MiB. Home Assistant makes the generated image available through its media system.

Select your Grok AI task entity in the action. These tasks do not use the conversation agent's Assist access or provider tools.

### Speech

New accounts also receive **Grok Speech-to-text** and **Grok Text-to-speech** entities. For an existing account, open SpaceXAI and select **Add speech-to-text service** or **Add text-to-speech service**.

To use them with Assist, go to {% my voice_assistants title="**Settings** > **Voice assistants**" %}, open your assistant, and select the Grok entities for speech-to-text and text-to-speech. Adding the integration does not change your voice assistant automatically.

Speech-to-text converts recorded audio to text. Text-to-speech turns text into MP3 audio. The default voice is Eve, and the default language is English. You can select another supported voice through the `voice` option of the [text-to-speech action](/integrations/tts/).

### Video and local media

**Generate video** creates a video from a prompt and, optionally, a local image. It saves the result in Home Assistant's local media directory. Video generation is billable according to your xAI account's terms.

**Publish media** creates a temporary link to an existing local image or video. It does not generate new content or copy files to a public directory. Both actions require administrator access.

{% include integrations/actions.md %}

## SpaceXAI automation examples

You can use the video and media actions in automations or scripts. Each action returns data, so give it a response variable to use the result in later steps.

{% include docs/paste_yaml_tip.md %}

### Automation: create a landscape clip at sunset

Use the **Generate video** action after a sun trigger to create a short landscape clip. The [video example](/actions/spacexai.generate_video/#automation-create-a-landscape-clip-at-sunset) shows how to save the action response. Each run generates a new billable video.

### Automation: share a local video at a scheduled time

Use the **Publish media** action after a time trigger to create a fresh link to a video that already exists. The [publishing example](/actions/spacexai.publish_media/#automation-show-a-local-video-link-at-a-scheduled-time) displays the link in a Home Assistant notification.

## Data updates

SpaceXAI is contacted when you send a conversation, AI task, speech, or video request. For video generation, the integration checks the submitted job until it finishes or times out. It does not poll in the background when idle. Publishing an existing local file does not contact SpaceXAI.

## Known limitations

- Attachments from earlier messages are not sent again with later requests.
- Conversation and AI data attachments must be nonempty JPEG, PNG, or PDF files.
- Image editing accepts JPEG and PNG only, with no more than five images per request.
- Image generation has no integration-specific aspect ratio or resolution setting.
- Speech-to-text accepts recorded WAV/PCM or OGG/Opus audio up to 25 MiB. It does not provide live speech-to-speech conversations.
- Choose from the supported speech languages and voices shown in Home Assistant. New provider options are not discovered automatically.
- Generated videos require a configured local media directory and must not exceed 100 MiB. Source images for video generation must be local JPEG, PNG, or WebP files up to 20 MiB.
- Media links expire after one hour. The saved files remain in local media until you remove them.
- The models available during setup depend on the signed-in account.

## Troubleshooting

### The sign-in code expired

If the code expires before you finish signing in, select **Submit** to request a new code.

### Grok rejects the sign-in

If your authorization expired or was revoked, Home Assistant asks you to sign in again. Open the SpaceXAI integration, start reauthentication, and select **Submit** to request a new one-time code. Complete the browser sign-in using the same xAI account.

Your conversation, AI task, and speech settings are preserved. Signing in with a different account is rejected so that an existing integration cannot silently switch accounts.

### Grok cannot access an entity

Make sure you selected the Assist API during setup and [exposed the entity to Assist](/voice_control/voice_remote_expose_devices/).

## Removing the integration

This integration follows standard integration removal. No extra steps are required.

{% include integrations/remove_device_service.md %}
