# esphome-audio

## Purpose
Audio for ESPHome: the AAC decoder and up to eight channels; one speaker that plays on several outputs and one
microphone fed by several sources, switched at runtime; echo cancellation for calls; an amplifier enable pin; codec
options (ES8311 volume range and echo reference, ES8388).

## How it works
The `resampler` speaker converts the audio for the output chosen in a select and passes it to that speaker (I2S
codec, USB sound card, Bluetooth). The `resampler` microphone takes the chosen source and, on ESP32-S3 / ESP32-P4,
runs Espressif's esp-sr AEC on it in its own task. Decoder, mixer and speaker pass up to eight channels; the audio
tasks run on the main loop's core (`LOOP_TASK_CORE` of `esp32`, from esphome-esp32-platform).

## Quick start
Chips: ESP32, ESP32-S2, ESP32-S3, ESP32-P4

Needs: [esphome-esp32-platform](https://github.com/p1ngb4ck/esphome-esp32-platform) (`LOOP_TASK_CORE` of `esp32`)

```yaml
external_components:
  - source: github://p1ngb4ck/esphome-audio@main
    components: [audio, audio_dac, es8311, es8388, i2s_audio, mixer, resampler, router, speaker, speaker_source]
  - source: github://p1ngb4ck/esphome-esp32-platform@main
    components: [esp32]
i2s_audio:
  - {id: i2s_bus, i2s_lrclk_pin: GPIO10, i2s_bclk_pin: GPIO12}
speaker:
  - {platform: i2s_audio, id: board_speaker, dac_type: external, i2s_dout_pin: GPIO9, enable_pin: GPIO11}
  - platform: resampler
    id: main_speaker
    output_speaker: [{speaker: board_speaker, name: Board, bits_per_sample: 16}]
select:
  - {platform: resampler, resampler_id: main_speaker, name: Audio output}
```

Use your board's pins. More outputs (e.g. `usb_speaker` of esphome-usb-drivers) go into the `output_speaker` list.

## Main options
| Option | What it does | Default |
|---|---|---|
| `resampler` speaker [`output_speaker`][o-spk] | outputs (`speaker`, `name`, format), chosen by a select | a speaker |
| `resampler` [`microphone`][o-mic] | list of sources, one active, chosen by a select | one source |
| [`echo_cancellation`][o-mic] `mode` | esp-sr AEC on a 2-channel 16-bit source (S3, P4) | `voip_low_cost` |
| `resampler` [select][o-sel] | `resampler_id` (outputs) or `microphone_id` (sources) | - |
| [`num_channels`][o-ch] | up to 8: audio, mixer, router, speaker pipelines, resampler outputs | the source's |
| `i2s_audio` speaker [`enable_pin`][o-i2s] | amplifier powered only while the I2S clocks run | - |
| `i2s_audio` bus [`nvs`][o-i2s] | bus enable, type and pins from the internal NVS | - |
| `es8311` [`volume_min_db` / `volume_max_db`][o-es] | dB at volume 0 % / 100 % | `-95.5` / `32.0` |
| `es8311` [`dac_reference`][o-es] | DAC playback on the ADC's right slot (AEC reference) | `false` |

[o-spk]: CONFIGURATION.md#resampler-speaker-several-outputs
[o-mic]: CONFIGURATION.md#resampler-microphone-several-sources-echo-cancellation
[o-sel]: CONFIGURATION.md#resampler-select
[o-ch]: CONFIGURATION.md#eight-channels
[o-i2s]: CONFIGURATION.md#i2s_audio
[o-es]: CONFIGURATION.md#es8311-audio_dac
[o-other]: CONFIGURATION.md#other-changes

## Catalog
Components (folder on `main` - options):
- [`audio`][c-audio] - [eight channels][o-ch], [other changes][o-other]
- [`audio_dac`][c-audio_dac] - [es8311][o-es]
- [`es8311`][c-es8311] - [es8311][o-es]
- [`es8388`][c-es8388] - [other changes][o-other]
- [`i2s_audio`][c-i2s_audio] - [i2s_audio][o-i2s]
- [`mixer`][c-mixer] - [eight channels][o-ch]
- [`resampler`][c-resampler] - [speaker][o-spk], [microphone][o-mic], [select][o-sel]
- [`router`][c-router] - [eight channels][o-ch]
- [`speaker`][c-speaker] - [eight channels][o-ch], [other changes][o-other]
- [`speaker_source`][c-speaker_source] - [eight channels][o-ch]

[c-audio]: https://github.com/p1ngb4ck/esphome-audio/tree/main/esphome/components/audio
[c-audio_dac]: https://github.com/p1ngb4ck/esphome-audio/tree/main/esphome/components/audio_dac
[c-es8311]: https://github.com/p1ngb4ck/esphome-audio/tree/main/esphome/components/es8311
[c-es8388]: https://github.com/p1ngb4ck/esphome-audio/tree/main/esphome/components/es8388
[c-i2s_audio]: https://github.com/p1ngb4ck/esphome-audio/tree/main/esphome/components/i2s_audio
[c-mixer]: https://github.com/p1ngb4ck/esphome-audio/tree/main/esphome/components/mixer
[c-resampler]: https://github.com/p1ngb4ck/esphome-audio/tree/main/esphome/components/resampler
[c-router]: https://github.com/p1ngb4ck/esphome-audio/tree/main/esphome/components/router
[c-speaker]: https://github.com/p1ngb4ck/esphome-audio/tree/main/esphome/components/speaker
[c-speaker_source]: https://github.com/p1ngb4ck/esphome-audio/tree/main/esphome/components/speaker_source

Branches:
- [main][b-main] (the whole topic)
- [pr/audio-aac-eight-channels][b-1]
- [pr/audio-dac-stopping][b-2]
- [pr/audio-decoder-discard-output][b-3]
- [pr/audio-decoder-heap][b-4]
- [pr/audio-reader-http-tx-buffer][b-5]
- [pr/audio-task-priority][b-6]
- [pr/audio-tasks-loop-core][b-7]
- [pr/es8311-dac-reference][b-8]
- [pr/es8311-volume-range][b-9]
- [pr/i2s-audio-enable-pin][b-10]
- [pr/i2s-audio-nvs][b-11]
- [pr/mixer-source-lock][b-12]
- [pr/resampler-aec][b-13]
- [pr/resampler-mic-callbacks-capture-time][b-14]
- [pr/resampler-routing][b-15]
- [pr/speaker-media-player-end][b-16]

[b-main]: https://github.com/p1ngb4ck/esphome-audio/tree/main
[b-1]: https://github.com/p1ngb4ck/esphome-audio/tree/pr/audio-aac-eight-channels
[b-2]: https://github.com/p1ngb4ck/esphome-audio/tree/pr/audio-dac-stopping
[b-3]: https://github.com/p1ngb4ck/esphome-audio/tree/pr/audio-decoder-discard-output
[b-4]: https://github.com/p1ngb4ck/esphome-audio/tree/pr/audio-decoder-heap
[b-5]: https://github.com/p1ngb4ck/esphome-audio/tree/pr/audio-reader-http-tx-buffer
[b-6]: https://github.com/p1ngb4ck/esphome-audio/tree/pr/audio-task-priority
[b-7]: https://github.com/p1ngb4ck/esphome-audio/tree/pr/audio-tasks-loop-core
[b-8]: https://github.com/p1ngb4ck/esphome-audio/tree/pr/es8311-dac-reference
[b-9]: https://github.com/p1ngb4ck/esphome-audio/tree/pr/es8311-volume-range
[b-10]: https://github.com/p1ngb4ck/esphome-audio/tree/pr/i2s-audio-enable-pin
[b-11]: https://github.com/p1ngb4ck/esphome-audio/tree/pr/i2s-audio-nvs
[b-12]: https://github.com/p1ngb4ck/esphome-audio/tree/pr/mixer-source-lock
[b-13]: https://github.com/p1ngb4ck/esphome-audio/tree/pr/resampler-aec
[b-14]: https://github.com/p1ngb4ck/esphome-audio/tree/pr/resampler-mic-callbacks-capture-time
[b-15]: https://github.com/p1ngb4ck/esphome-audio/tree/pr/resampler-routing
[b-16]: https://github.com/p1ngb4ck/esphome-audio/tree/pr/speaker-media-player-end

Needed topic repos:
- [esphome-esp32-platform](https://github.com/p1ngb4ck/esphome-esp32-platform): `esp32` (`LOOP_TASK_CORE`).
- Optional: [esphome-usb-drivers](https://github.com/p1ngb4ck/esphome-usb-drivers) (`usb_audio` outputs and sources)
  with [esphome-usb-host](https://github.com/p1ngb4ck/esphome-usb-host) (`usb_host`).
- Optional: [esphome-storage-interface](https://github.com/p1ngb4ck/esphome-storage-interface): `storage` paths for
  the audio reader, used only when `storage` is in the build.

All topics together: `github://p1ngb4ck/esphome@dev`.

---

# Detailed version

Audio changes for ESPHome: the AAC decoder, up to eight channels through decoder, mixer and speaker, one speaker
that plays on several outputs and one microphone fed by several sources - both switched at runtime from Home
Assistant - echo cancellation for calls, an amplifier enable pin, codec options (ES8311 volume range and echo
reference, ES8388), and the media player's pipeline ending cleanly at the end of a file.

Base: ESPHome `dev` at `a5a75738ad`.

## Examples

A speaker that plays on the board's codec or a USB sound card, switched with a select:

```yaml
external_components:
  - source: github://p1ngb4ck/esphome-audio@main
    components: [audio, i2s_audio, mixer, resampler, speaker]
  - source: github://p1ngb4ck/esphome-esp32-platform@main
    components: [esp32]

speaker:
  - platform: i2s_audio
    id: board_speaker
    i2s_dout_pin: GPIO09
    enable_pin: GPIO53
    dac_type: external
  - platform: resampler
    id: main_speaker
    output_speaker:
      - speaker: board_speaker
        name: Board
      - speaker: usb_speaker
        name: USB

select:
  - platform: resampler
    resampler_id: main_speaker
    name: Audio output
```

A microphone with echo cancellation (ESP32-S3 / ESP32-P4), from a codec that puts the playback reference on its
second channel:

```yaml
microphone:
  - platform: resampler
    id: call_mic
    sample_rate: 16000
    microphone:
      - microphone: codec_mic
        name: Board
        channels: [0, 1]
        echo_cancellation:
          mode: voip_low_cost
          reference_channel: 1
```

Use your board's pins; `usb_speaker` comes from `usb_audio`
([esphome-usb-drivers](https://github.com/p1ngb4ck/esphome-usb-drivers), with `usb_host` of
[esphome-usb-host](https://github.com/p1ngb4ck/esphome-usb-host)).

Needs: [esphome-esp32-platform](https://github.com/p1ngb4ck/esphome-esp32-platform) for `esp32` (the audio tasks use
its `LOOP_TASK_CORE`). Optional: esphome-usb-drivers for USB outputs and sources;
[esphome-storage-interface](https://github.com/p1ngb4ck/esphome-storage-interface) for `storage` paths in the audio
reader (used only when `storage` is in the build).

## More

**[CONFIGURATION.md](CONFIGURATION.md)**: every new option - resampler outputs and sources, echo cancellation
modes, eight channels, `i2s_audio` enable pin and pins from the NVS, ES8311 volume range and DAC reference.

## Branches

One branch per change; `main` contains every branch. Base: ESPHome `dev` at `a5a75738ad`.
Per branch: change; chips; needs. "ESP32" as chips: every ESP32 variant (ESP32, ESP32-S2, ESP32-S3, ESP32-P4).

- `pr/audio-aac-eight-channels`: the AAC decoder, `.ogg` as Opus, the re-vendored `speaker_media_player_base`, up to
  eight channels through decoder, mixer and speaker; ESP32; needs -.
- `pr/audio-decoder-heap`: AudioDecoder's objects and transfer buffer from a heap its owner gives, in PSRAM when
  asked; ESP32; needs `pr/audio-aac-eight-channels`.
- `pr/audio-decoder-discard-output`: `AudioDecoder::discard_output()`: decoded audio not yet written dropped, e.g.
  before a seek; ESP32; needs -.
- `pr/audio-reader-http-tx-buffer`: the HTTP reader's request buffer 2048 bytes (long signed media URLs); ESP32;
  needs -.
- `pr/mixer-source-lock`: mixer: the source speaker's ring buffer and audio source read under the lock; ESP32;
  needs -.
- `pr/speaker-media-player-end`: speaker media_player: the pipeline stopped at the end of a file; resampler
  `finish()` ends with its input; ESP32; needs -.
- `pr/audio-task-priority`: decode and resample tasks at priority 2, above the main loop; ESP32; needs
  `pr/speaker-media-player-end`.
- `pr/resampler-routing`: resampler speaker with a list of outputs, microphone with several sources, both switched
  at runtime (selects), USB audio outputs and sources added and removed at runtime, microphone `num_channels`;
  ESP32; needs `pr/audio-task-priority`.
- `pr/resampler-aec`: `echo_cancellation` on a microphone source (esp-sr AEC, 16 kHz 16 bit) in its own task; S3,
  P4; needs `pr/resampler-routing`.
- `pr/resampler-mic-callbacks-capture-time`: microphone data callbacks as the other microphones take them, the
  capture time carried through; ESP32; needs `pr/resampler-aec`.
- `pr/i2s-audio-enable-pin`: i2s_audio speaker `enable_pin`: the amplifier powered only while the I2S clocks run;
  ESP32; needs -.
- `pr/i2s-audio-nvs`: i2s_audio `nvs`: enable, type and pins of a bus from the internal NVS; ESP32; needs
  `pr/i2s-audio-enable-pin`.
- `pr/audio-dac-stopping`: `on_audio_stopping` hook: the ES8388 DAC stays muted while its stream stops; ESP32;
  needs `pr/i2s-audio-enable-pin`.
- `pr/es8311-volume-range`: es8311 `volume_min_db` / `volume_max_db`; all; needs -.
- `pr/es8311-dac-reference`: es8311 `dac_reference`: the DAC's playback on the ADC's right slot beside the
  microphone (echo cancellation reference); all; needs -.
- `pr/audio-tasks-loop-core`: i2s_audio, mixer, resampler, speaker tasks pinned to `LOOP_TASK_CORE`; ESP32; needs
  `pr/esp32-loop-task-core` ([esphome-esp32-platform](https://github.com/p1ngb4ck/esphome-esp32-platform)).

All topics together: `github://p1ngb4ck/esphome@dev`. `external_components` replaces components only; the ESPHome
core of the build host stays as installed.

## License

- ESPHome's own files, also where this repository changes them: the [ESPHome License](LICENSE) (MIT for the Python
  code and everything else, GPLv3 for the C++ runtime files).
- New files written for this repository (new components, their code, tests and documentation): MIT,
  [LICENSE-MIT](LICENSE-MIT).

## Thanks

Echo cancellation is Espressif's esp-sr AEC. Everyone this project builds on:
[ATTRIBUTIONS.md](https://github.com/p1ngb4ck/esphome/blob/dev/ATTRIBUTIONS.md).

## Branch structure

Each `pr/` branch holds one part of this topic on top of upstream and says in its `BRANCH.md` what it is based on.
`main` holds all of them.

| Branch | Based on |
|---|---|
| `pr/audio-aac-eight-channels` | upstream only |
| `pr/audio-dac-stopping` | `pr/i2s-audio-enable-pin` |
| `pr/audio-decoder-discard-output` | upstream only |
| `pr/audio-decoder-heap` | `pr/audio-aac-eight-channels` |
| `pr/audio-reader-http-tx-buffer` | upstream only |
| `pr/audio-task-priority` | `pr/speaker-media-player-end` |
| `pr/audio-tasks-loop-core` | `pr/esp32-loop-task-core` of p1ngb4ck/esphome-esp32-platform |
| `pr/es8311-dac-reference` | upstream only |
| `pr/es8311-volume-range` | upstream only |
| `pr/i2s-audio-enable-pin` | upstream only |
| `pr/i2s-audio-nvs` | `pr/i2s-audio-enable-pin` |
| `pr/mixer-source-lock` | upstream only |
| `pr/resampler-aec` | `pr/resampler-routing` |
| `pr/resampler-mic-callbacks-capture-time` | `pr/resampler-aec` |
| `pr/resampler-routing` | `pr/audio-task-priority` |
| `pr/speaker-media-player-end` | upstream only |
