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
    outputFormat: "mp3"  # Optional: "mp3" (default) or "wav"
    provider: "gopipertts"  # Optional: "gopipertts" (default) or "openai"
    voice: "default"  # Optional, openai provider only: voice name sent to the server
    timeout: "10m"  # Optional: per-request timeout (default 10m)
```

or

```toml
theme = [..., 'hugo-shortcodes-tts']
[Params.Tts]
  ApiUrl = "http://tts-api:8080/api/tts"
  Disabled = false  # Optional: set to true to disable TTS (useful during drafting/testing)
  OutputFormat = "mp3"  # Optional: "mp3" (default) or "wav"
  Provider = "gopipertts"  # Optional: "gopipertts" (default) or "openai"
  Voice = "default"  # Optional, openai provider only: voice name sent to the server
  Timeout = "10m"  # Optional: per-request timeout (default 10m)
```

With `provider: "openai"`, `apiUrl` points at any OpenAI-compatible speech endpoint, e.g. Phonon's [ptts-openai-server](https://github.com/gradium-ai/xn-ptts) at `http://phonon:8880/v1/audio/speech`. Text is sent in chunks of at most 4000 characters and the audio is concatenated, so posts longer than that require `mp3`.

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

Once playback has started, the container gets a `.text-to-speech-started` class that stays on even when paused. Use it to keep the player in view while scrolling:

```css
.text-to-speech-container.text-to-speech-started {
  position: sticky;
  top: 0;
  background: #fff;
}
```

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

The `outputFormat` parameter can be set per-shortcode to override the site-wide default:

```
{{< text-to-speech outputFormat="mp3" >}}
```

`voice` can be overridden the same way.

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

### Speech text

Before the text is sent, it is rewritten so it reads better aloud:

- Fenced code blocks are never spoken
- Inline code is wrapped in double quotes: `` `CLAUDE.md` `` becomes `"CLAUDE dot md"`
- Headers, list items and the title get a trailing `.` (unless they already end in punctuation) to force a pause
- `e.g.`, `i.e.`, `etc.` and `vs.` are expanded
- A dot followed by a lowercase letter is spoken: `settings.json` becomes `settings dot json`, `.git` becomes `dot git`
- Underscores between words become spaces, ` & ` becomes ` and `
- Parentheses become commas

### Generate TTS

See the example project for a sample configuration and Makefile to build your static site with TTS:

```bash
cd example
make serve
```

then

`open http://localhost:1313`
