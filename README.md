# hugo-shortcodes-tts

[Hugo](https://gohugo.io/) shortcode leveraging [gopipertts](https://github.com/nbr23/gopipertts) to add a "Listen to this post" feature to you hugo posts.

## Usage

### Configuration

In your hugo site repo, add the shortcode submodule:

`git submodule add -b theme git@github.com:nbr23/hugo-shortcodes-tts.git themes/hugo-shortcodes-tts`

Register it in your `config.{yaml,toml}`:

```yaml
theme: [..., 'hugo-shortcodes-tts']
params:
  tts:
    apiUrl: "http://tts-api:8080/api/tts"
    disabled: false  # Optional: set to true to disable TTS (useful during drafting/testing)
```

or

```toml
theme = [..., 'hugo-shortcodes-tts']
[Params.Tts]
  ApiUrl = "http://tts-api:8080/api/tts"
  Disabled = false  # Optional: set to true to disable TTS (useful during drafting/testing)
```

You will likely need to increase the timeout for page generation as well as the TTS process can take a while

`timeout: 120s`

### Styling

The shortcode renders a `.text-to-speech-container` div that is hidden by default and displayed via JavaScript when the browser supports audio playback. You'll need to add some CSS to style it. For example:

```css
.text-to-speech-container {
  display: none;
  align-items: center;
  gap: 16px;
}
```

The `display: none` is required — the shortcode's inline script sets it to `flex` when the browser can play the audio.

### Add shortcode to posts

In your hugo posts, add the following tags at beginning and end:

```
---
title: XXX
---
{{< text-to-speech >}}
[POST CONTENT]
{{< /text-to-speech >}}
```

To exclude specific regions from being spoken, you can use the `<!-- noTTS -->` tags:


```
---
title: XXX
---
{{< text-to-speech >}}
This will be spoken

<!-- noTTS -->
`printf("this code sample will not be spoken")`
<!-- /noTTS -->

And we will resume speaking here

{{< /text-to-speech >}}
```

### Generate TTS

See the example project for a sample configuration and Makefile to build your static site with TTS:

```bash
cd example
make serve
```

then

`open http://localhost:1313`
