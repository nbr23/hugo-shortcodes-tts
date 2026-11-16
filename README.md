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
    model: "gpt-4o-mini-tts"  # Optional, openai provider only: model name sent to the server
    speed: 1.0  # Optional, openai provider only: synthesis speed sent to the server
    instructions: "Speak calmly"  # Optional, openai provider only: voice instructions sent to the server
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
  Model = "gpt-4o-mini-tts"  # Optional, openai provider only: model name sent to the server
  Speed = 1.0  # Optional, openai provider only: synthesis speed sent to the server
  Instructions = "Speak calmly"  # Optional, openai provider only: voice instructions sent to the server
  Timeout = "10m"  # Optional: per-request timeout (default 10m)
```

With `provider: "openai"`, `apiUrl` points at any OpenAI-compatible speech endpoint, e.g. Phonon's [ptts-openai-server](https://github.com/gradium-ai/xn-ptts) at `http://phonon:8880/v1/audio/speech`. Text is sent in chunks of at most 4000 characters and the audio is concatenated, so posts longer than that require `mp3`.

`voice`, `model`, `speed` and `instructions` are only sent when set, so a server that rejects one of them can be used by leaving it out.

`speed` is baked into the generated audio, and changing it regenerates every post. Readers pick their own playback speed in the player, which costs nothing (see [Styling](#styling)).

#### OpenAI API

The OpenAI API requires `model` and `voice`, and an API key:

```yaml
params:
  tts:
    provider: "openai"
    apiUrl: "https://api.openai.com/v1/audio/speech"
    model: "gpt-4o-mini-tts"
    voice: "alloy"
security:
  http:
    mediaTypes:
      - "audio/mpeg"
    methods:
      - (?i)GET|POST
    urls:
      - ^https://api\.openai\.com/
```

The key is read from `params.tts.apiKey` and sent as `Authorization: Bearer <key>` (openai provider only). Do not put it in your config file: pass it through the environment, which Hugo maps onto that param:

```bash
HUGO_PARAMS_TTS_APIKEY=sk-... hugo
```

Hugo caches remote responses keyed on the URL and the request options (body and headers), so an unchanged post is not sent again, and changing the text, the key or any of the params above regenerates its audio.

A failed request (error status, unreachable server, empty response) fails the build with the post name and the server's response.

Hugo 0.141 or later is required.

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

The container also holds a `.text-to-speech-rate` select for the playback speed, from `0.75×` to `2×`. It sets `playbackRate` on every player on the page, and the choice is saved in `localStorage` under `tts-playback-rate`.

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

`voice`, `model`, `speed` and `instructions` can be overridden the same way.

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

### Alternate texts and languages

`params.tts.substitutions` lists terms to say differently, or in another language. A post can add its own under `tts.substitutions` in its front matter; those are applied first, so they win over the site's:

```yaml
params:
  tts:
    substitutions:
      - match: kubectl
        say: cube control
      - match: déjà vu
        lang: fr
    languages:
      fr:
        apiUrl: "http://tts-fr:8880/v1/audio/speech"
        voice: "default"
```

`match` is literal and case-sensitive, and only matches whole words. It is matched against the text as the reader sees it, before the rewrites above. `say` replaces it, `lang` reads it (or its `say`) in that language, and both can be set.

For a one-off, mark up the text inline. The comments do not show on the page:

```
{{% text-to-speech %}}
Released in <!-- tts say="twenty twenty six" -->2026<!-- /tts -->, to a chorus of <!-- tts lang="fr" -->bon appétit<!-- /tts -->.
{{% /text-to-speech %}}
```

Each language span is sent as its own request, using `params.tts.languages.<lang>`: `apiUrl`, `voice`, `model`, `speed` and `instructions`, each falling back to the post's settings. A server that only speaks one language, like Phonon, needs a second server for the other. The audio is joined, so this requires `mp3`, and expect a short pause around each span. A `lang` with no settings fails the build. With `provider: "gopipertts"`, only `apiUrl` and `voice` apply.

### Generate TTS

See the example project for a sample configuration and Makefile to build your static site with TTS:

```bash
cd example
make serve
```

then

`open http://localhost:1313`
