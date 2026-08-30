---
title: SpaceXAI
description: Instructions on how to use Grok conversation and AI tasks with SpaceXAI
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

## Data updates

SpaceXAI is contacted when you send a conversation or AI task request. The integration does not poll in the background.

## Known limitations

- Attachments from earlier messages are not sent again with later requests.
- Empty attachments and file types other than JPEG, PNG, and PDF are not supported.
- Image editing accepts JPEG and PNG only, with no more than five images per request.
- Image generation has no integration-specific aspect ratio or resolution setting.
- The models available during setup depend on the signed-in account.

## Troubleshooting

### The sign-in code expired

If the code expires before you finish signing in, select **Submit** to request a new code.

### Grok rejects the sign-in

If your authorization expired or was revoked, remove the SpaceXAI integration and add it again to complete a new browser sign-in.

### Grok cannot access an entity

Make sure you selected the Assist API during setup and [exposed the entity to Assist](/voice_control/voice_remote_expose_devices/).

## Removing the integration

This integration follows standard integration removal. No extra steps are required.

{% include integrations/remove_device_service.md %}
