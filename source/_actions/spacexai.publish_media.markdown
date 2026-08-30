---
title: Publish media
action: spacexai.publish_media
domain: spacexai
description: Create a temporary signed link to an image or video in Home Assistant's local media.
related_actions:
  - spacexai.generate_video
---

The **Publish media** action creates a one-hour signed link to an existing image or video in Home Assistant's local media. Use it to share a file temporarily or obtain a new link after an earlier one expires.

This action does not generate content, upload anything to SpaceXAI, or copy files to a public directory.

## Using this action from the user interface

To create a link from an automation or script:

1. Go to {% my automations title="**Settings** > **Automations & scenes**" %}.
2. Open an automation or script. For a new automation, add a trigger in **When**.
3. In **Then do**, select **Add action** and search for **Publish media**.
4. Under **Media**, select a locally stored image or video.
5. Set a **Response variable**, such as `published_media`, to receive the link.
6. Select **Save**.

This action does not use an account field or an entity, device, or area target.

### Options in the UI

{% options_ui %}
Media:
  description: An existing local image or video to share. Remote media is not supported.
  required: true
{% endoptions_ui %}

{% include actions/yaml_header.md %}

In YAML, refer to this action as `spacexai.publish_media`. This is an action step for an automation or script. Select your file in the visual editor to obtain its media identifier.

{% example %}
action: |
  action: spacexai.publish_media
  data:
    media:
      media_content_id: "media-source://media_source/local/welcome.png"
      media_content_type: "image/png"
  response_variable: published_media
{% endexample %}

### Options in YAML

{% options_yaml %}
media:
  description: "A media-selector mapping with `media_content_id` and `media_content_type`. The selected media must resolve to an existing local file."
  required: true
  type: map
{% endoptions_yaml %}

## Response data

Set `response_variable` on the action to receive:

- `media_source_id`: The selected media's Home Assistant identifier.
- `content_type`: The media content type.
- `path`: A signed relative path, valid for one hour.
- `url`: The signed path with a Home Assistant base URL when one is available; otherwise, a relative path.

## Good to know

- This action requires administrator access.
- Anyone with the signed link can access the file while the link is valid. Share it only with intended recipients.
- External recipients need a reachable Home Assistant URL. A relative path works only when opened against your Home Assistant instance.
- The link expires after one hour. The file is not deleted, and existing links are not renewed when you run the action again.
- The action does not contact SpaceXAI or incur a video-generation charge.

## Try it yourself

Open {% my developer_services title="**Settings** > **Tools** > **Actions**" %}, select **Publish media**, and choose a local file. Select **Perform action** and inspect the returned link. Treat that link as temporary access to the file.

{% include actions/more_examples.md %}

### Automation: send a welcome image link on arrival

When a person arrives home, send a link to a welcome image you already uploaded. Replace the example person, notification entity, and local media identifier with ones from your installation.

- **Trigger**: State: the person becomes home
- **Action**: Publish media
  - **Media**: Your local welcome image
  - **Response variable**: `published_media`
- **Action**: Send a notification message
  - **Target**: My Device (`notify.my_device`)
  - **Message**: The returned media URL

{% details "YAML example for sharing a welcome image" %}

{% example %}
automation: |
  alias: "Share a welcome image on arrival"
  triggers:
    - trigger: state
      entity_id: person.resident
      to: "home"
  actions:
    - action: spacexai.publish_media
      data:
        media:
          media_content_id: "media-source://media_source/local/welcome.png"
          media_content_type: "image/png"
      response_variable: published_media
    - action: notify.send_message
      target:
        entity_id: notify.my_device
      data:
        message: "{{ published_media.url }}"
{% endexample %}

{% enddetails %}

### Automation: show a local video link at a scheduled time

Create a fresh link to an existing local video each evening and display it in a Home Assistant notification. This shares the same saved video; it does not generate a new one. Replace the example media identifier with your file.

- **Trigger**: Time: 18:00
- **Action**: Publish media
  - **Media**: Your local video
  - **Response variable**: `published_media`
- **Action**: Create persistent notification
  - **Message**: The returned media link

{% details "YAML example for a scheduled video link" %}

{% example %}
automation: |
  alias: "Show an evening video link"
  triggers:
    - trigger: time
      at: "18:00:00"
  actions:
    - action: spacexai.publish_media
      data:
        media:
          media_content_id: "media-source://media_source/local/evening.mp4"
          media_content_type: "video/mp4"
      response_variable: published_media
    - action: persistent_notification.create
      data:
        title: "Evening video"
        message: "[Open video]({{ published_media.url }})"
{% endexample %}

{% enddetails %}

{% include actions/stuck.md %}

{% include actions/related.md %}
