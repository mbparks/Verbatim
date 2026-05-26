# VERBATIM

A single-file, offline voice recorder and transcriber that runs entirely in the browser. Record from your microphone, save a real 16-bit WAV, and transcribe to text locally with Whisper. No accounts, no servers, no audio leaving the device.

Instrument 002, by M.B. Parks.

## Features

- One self-contained `verbatim.html` file. No build step, no install, no dependencies to bundle.
- Microphone capture through the Web Audio API, with a live waveform scope and a tenth-of-a-second timer.
- True WAV export. Raw PCM is captured and encoded to 16-bit mono WAV in the browser, with no lossy re-encoding.
- Offline transcription with Whisper via Transformers.js, running in a Web Worker so the interface never freezes.
- Transcribe a fresh recording or load an existing audio file (any format the browser can decode) and transcribe that instead.
- Save the transcript to a `.txt` file, with the text editable before you save it.
- Selectable model size to trade speed against accuracy.
- Screen Wake Lock while recording, so phones do not sleep and cut the capture short.
- Responsive layout that adapts from desktop down to a phone screen.

## How it works

Recording uses `getUserMedia` to open the microphone, taps the raw samples through the Web Audio API, and collects them as the recording runs. On stop, the samples are written into a standard WAV container in the browser.

Transcription resamples the captured audio to the 16 kHz that Whisper expects, then hands it to a background Web Worker. The worker loads the chosen Whisper model through Transformers.js and returns the text. The first run downloads the model once from a CDN, after which the model is cached by the browser and every later run is fully offline. Audio is never uploaded anywhere.

## Models

The model selector at the bottom of the panel chooses the engine. Sizes are approximate and reflect the quantized download.

| Option   | Model               | Size    | Notes                          |
|----------|---------------------|---------|--------------------------------|
| tiny.en  | Xenova/whisper-tiny.en  | ~40 MB  | Fastest. Best choice on phones. |
| base.en  | Xenova/whisper-base.en  | ~80 MB  | Default. Good speed and accuracy balance. |
| small.en | Xenova/whisper-small.en | ~250 MB | Most accurate. Heavier download and slower inference. |

All three are English-only models.

## Requirements

- A modern browser. Chrome, Edge, Firefox, and Safari are all supported. The transcription worker uses module workers, which require Safari 16.4 or newer on iOS and iPadOS.
- A secure context for the microphone. The page must be served over `https` or run from `localhost`. Opening the file directly with a `file://` path will not grant microphone access on iOS, and is unreliable elsewhere.
- On iOS, use Safari itself. Saved-to-home-screen web apps and in-app browsers block speech and microphone features.
- A network connection for the first transcription only, to fetch the model. After that, transcription works offline.

## Getting started

Because the microphone needs a secure context, the page has to be served rather than opened from disk.

Run it locally:

```
# from the folder containing verbatim.html
python -m http.server 8000
# then open http://localhost:8000/verbatim.html
```

Host it on GitHub Pages:

1. Put `verbatim.html` in the repository. To get a clean URL, you can rename it to `index.html`.
2. In the repository settings, enable GitHub Pages for the branch.
3. Open the published `https` URL on any device, including an iPhone.

## Usage

1. Press Record and allow microphone access. Press Stop when finished. You can also press Load an audio file to bring in existing audio.
2. Press Save WAV at any point after a recording or load to keep the audio.
3. Choose a model, then press Transcribe. The first run shows a download progress bar.
4. Edit the transcript in the text box if needed, then press Save TXT.
5. Press New to clear everything and start over.

## Privacy

After the one-time model download, nothing leaves your device. Recording, encoding, and transcription all happen in the browser. There is no analytics, no telemetry, and no server component.

## Known limitations

- Transcription speed depends on the device and the model. On phones, prefer tiny.en or base.en.
- If the host does not send cross-origin isolation headers (GitHub Pages does not), Whisper falls back to single-threaded WebAssembly, which works but runs slower. Self-hosting with COOP and COEP headers enables multi-threaded inference.
- The engine is fetched from a CDN at first use. If your host enforces a strict Content Security Policy, allow the CDN origin or self-host Transformers.js.
- WAV files are uncompressed. Long recordings produce large files (roughly 1.4 MB per minute at 16 kHz, more at higher capture rates).

## Built with

- Web Audio API and MediaDevices for capture.
- Transformers.js with OpenAI Whisper models for speech to text.
- Syne and IBM Plex Mono typefaces.

## License

GPL-3.0
