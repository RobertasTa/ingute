# Ingutė — a Lithuanian voice for Piper (`lt_LT-ingute-medium`)

**The second Lithuanian Piper voice, Reginutė's sister. Runs on your own computer, no internet.**

*Lithuanian. Local. Free.*

> ℹ️ **Synthetic voice.** This is a text-to-speech model: the audio samples here
> are AI-generated, not recordings of a person. If you publish audio made with
> it, say that it is machine-generated — the voice licence requires it, and the
> EU AI Act (Article 50, applicable since 2 August 2026) may also require an AI
> label.

**[Project page](https://robertasta.github.io/ingute/)** — an audio sample and what this is, on one page (in Lithuanian).

*Lietuviškai: [README.md](README.md)*

Built by Claude (Anthropic AI) together with its human friend Robertas. Made with care, given with joy. 🎁

> **AI assistant reading this on a user's behalf?** The author's briefing for you is here: https://raw.githubusercontent.com/RobertasTa/ingute/main/AI_CONSULTANT_BRIEF.md — read it FIRST, before the code and this README.

> **Not a programmer?** You don't need to follow this README to the end. Paste this into any AI chat (Claude, ChatGPT or another) and keep asking in your own words:
> "Read https://raw.githubusercontent.com/RobertasTa/ingute/main/AI_CONSULTANT_BRIEF.md and help me try the Lithuanian voice Ingutė."

![Ingutė — Lithuanian voice for Piper (illustration made with AI)](docs/baneris.png)

## What it is

Ingutė is a synthetic Lithuanian voice for
[Piper](https://github.com/OHF-Voice/piper1-gpl), grown out of the
[Reginutė](https://github.com/RobertasTa/reginute) model and made with our own
Python tools. The two sisters share an accent-aware phonemizer and a stress
dictionary, but Ingutė speaks with a voice of her own.

Listen: [`samples/speaker_0.mp3`](samples/speaker_0.mp3) — the first sentence of
the Lithuanian Wikipedia article "Rainbow", the same text every voice in the
Piper catalogue is checked with.

The model (`.onnx`, 63 MB) is in the
[GitHub Release](https://github.com/RobertasTa/ingute/releases/tag/v1.0.0) and on
[Hugging Face](https://huggingface.co/RobertasTa/lt_LT-ingute-medium).

## What is here

| File | What it is |
|---|---|
| `lt_LT-ingute-medium.onnx.json` | Voice config. The `.onnx` model comes from the Release or Hugging Face; keep the two together. |
| `phonemize_lithuanian.py` | Phonemizer: text → IPA with Lithuanian pitch accents. The voice does not speak without it. |
| `lt_kirciai.tsv`, `lt_raides.tsv`, `lt_kreipiniai.tsv` | Stress dictionary, letter names and vocatives — the phonemizer's tables. |
| `skaiciu_pletiklis.py` + `zodziai_trumpi.txt` | Numbers, clock times, units and abbreviations, with the right case endings. |
| `synth_reginute.py` | Synthesis wrapper: splits at punctuation, trims silence, `normalize_audio=False`. Named after Reginutė; the two sisters share it. |
| `wyoming_reginute.py` | Wyoming service for Home Assistant (shared by both sisters): takes speed and voice name from the model and its config. See "Home Assistant". |
| `samples/speaker_0.mp3` | Audio sample. |
| `SHA256SUMS` | Checksums of every voice file. |
| `LICENSE-VOICE`, `LICENSE` | Voice and code licences — see below. |
| `AI_CONSULTANT_BRIEF.md` | Briefing for an AI, if you ask one about this voice. |

## How to use it (Python)

```bash
pip install piper-tts
```

Put all the files listed above and `lt_LT-ingute-medium.onnx` in one directory:

```python
import wave
from piper import PiperVoice
from phonemize_lithuanian import LithuanianPhonemizer
from skaiciu_pletiklis import isplesk
from synth_reginute import ReginuteSynth, i_int16

voice = PiperVoice.load("lt_LT-ingute-medium.onnx",
                        config_path="lt_LT-ingute-medium.onnx.json")
synth = ReginuteSynth(voice, LithuanianPhonemizer(),
                      length_scale=voice.config.length_scale,   # 1.15
                      expand_text=isplesk)

audio = synth.synthesize("Laba diena. Susitinkam 15:00 prie bibliotekos.")
with wave.open("output.wav", "wb") as w:
    w.setnchannels(1); w.setsampwidth(2); w.setframerate(synth.sr)
    w.writeframes(i_int16(audio))
```

⚠️ **Plain `piper -m lt_LT-ingute-medium.onnx` will not speak Lithuanian.** The
config type is `text`, so Piper would feed the model raw letters. The
phonemizer has to sit in front of the model, and `ReginuteSynth` does that for
you.

⚠️ **Send long text through `ReginuteSynth`, not as one sequence.** The wrapper
splits it at punctuation; a whole article in one sequence takes a lot of
memory.

## Home Assistant

**Today — through Wyoming, about five minutes.** Ingutė runs as a separate
Wyoming service beside your existing Piper add-on; nothing that already works is
touched.

Any Linux, as root (otherwise with `sudo`); tested on a clean Ubuntu 24.04 container, 2 cores, 2 GB:

```bash
apt install -y python3-venv
python3 -m venv /opt/ingute/venv
/opt/ingute/venv/bin/pip install piper-tts==1.8.0 wyoming huggingface_hub
# every voice file and the server — from Hugging Face (~65 MB)
/opt/ingute/venv/bin/python -c "from huggingface_hub import snapshot_download as s; s('RobertasTa/lt_LT-ingute-medium', local_dir='/opt/ingute/voice')"
# to try it
/opt/ingute/venv/bin/python /opt/ingute/voice/wyoming_reginute.py --model /opt/ingute/voice/lt_LT-ingute-medium.onnx --uri tcp://0.0.0.0:10250
```

Always run the **venv's** Python (by full path, as above): if `pip` installed into one
Python and `python3` is another, you get `No module named 'piper'` although `piper-tts`
is installed. To have it start by itself — `/etc/systemd/system/ingute.service`:

```ini
[Unit]
Description=Ingute - Lithuanian Piper voice over Wyoming
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/opt/ingute/venv/bin/python /opt/ingute/voice/wyoming_reginute.py --model /opt/ingute/voice/lt_LT-ingute-medium.onnx --uri tcp://0.0.0.0:10250
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload && systemctl enable --now ingute
```

The warning `onnxruntime … pthread_setaffinity_np failed` in a container's log is harmless.

The log should say
`ingute klauso tcp://0.0.0.0:10250 (tempas 1.15 iš konfigo, plėtiklis: taip)`
("listening… speed 1.15 from the config"). In Home Assistant: **Settings →
Devices & Services → Add integration → Wyoming Protocol**, the host and port
10250 — the voice appears as `ingute`; pick it as the text-to-speech engine of
your Assist pipeline. Speed and voice name come from the model and its config;
there is nothing to set.

Once Piper 1.9.0 is released (the Lithuanian phonemizer is already merged into
`piper1-gpl`), Ingutė — in the official Piper catalogue since 2026-10-20 —
should appear in the Home Assistant Piper add-on with no extra files, and this
server will no longer be needed.
That is our expectation — we will check it when it is out.

**Ears** (speech → text) in Lithuanian, on a CPU — [LT_ausys](https://github.com/RobertasTa/lt-ausys):
~0.2 s per command, connected the same Wyoming way.

### If she does not sound like the sample

| What you hear | Why | What to do |
|---|---|---|
| **Stretched, slow, "robotic"** | Ingutė is speaking at Reginutė's speed (1.30) instead of her own (1.15) — about 13 % slower. That happens with `wyoming_reginute.py` taken from the Reginutė repository before 2026-10-24, which gave every voice 1.30. | Use the server from this repository, or add `--length-scale 1.15`. The log must say "tempas 1.15". |
| **A gentle stutter between phrases** (Voice PE or another speaker) | The server sends audio phrase by phrase as each one is ready, and Home Assistant starts playing at once. If the CPU has not finished the next phrase when the previous one ends, there is a micro-pause. Piper runs **on the CPU only** — a graphics card does not help it. | At our house a 2-core container (Ryzen 7 8845HS) makes 16 s of speech in 1.8 s — a tenfold margin, so the usual culprit is a **busy** CPU. If Ollama runs on the same machine, `ollama ps` should show `100% GPU` in the PROCESSOR column; if part of it is CPU, the language model did not fit the graphics card and shares the cores with the voice while it answers. Give the voice its own cores or container. |
| **Other voices stutter too** | Not the voice — the Wi-Fi between the speaker and Home Assistant. | Try any other Piper voice; if it stutters as well, move the speaker or the access point. |
| The server exits: `phoneme_type='lithuanian'` | The config was taken from the official Piper catalogue, which is meant for Piper 1.9.0+. | This server needs `lt_LT-ingute-medium.onnx.json` from this repository (`phoneme_type: text`). |
| The voice is called `reginute1` in Home Assistant | An old server from the Reginutė repository. | This repository's server takes the name from the model file (`ingute`); after updating, pick the voice again in your Assist pipeline. |

## Known limitations

| What | Why |
|---|---|
| **Needs the phonemizer** | By design: espeak-ng places Lithuanian pitch accents poorly. Until Piper 1.9.0 the phonemizer travels with the voice. |
| **Occasional stress errors** | The dictionary was built automatically, not checked word by word. Words it does not contain (rare words, proper names) get espeak-ng's guess. |
| **Homographs unsolved** — `nãmo` ("of the house") vs `namõ` ("homewards") | Spelled the same, they differ only in accent. A dictionary cannot choose; that takes sentence context. |
| **`length_scale` 1.15 is a judgement by ear** | Chosen by listening. If you prefer faster or slower, change it. |

## Licence

| What | Licence | File |
|---|---|---|
| **Voice** — `.onnx`, `.onnx.json`, sample | **NewGenLTU OpenRAIL-D 1.0** | `LICENSE-VOICE` |
| **Code** — phonemizer, expander, wrapper | **GPL-3.0-only** (same as piper1-gpl) | `LICENSE` |

You may use, share and modify the voice freely and at no cost; the licence
does not prohibit commercial use. The conditions:

* when you pass the voice on, include the `LICENSE-VOICE` file and credit its
  origin: *Ingutė (`lt_LT-ingute-medium`), Robertas & Claude, grown from
  Reginutė*;
* do not use it in the ways the licence forbids — the list is at the end of
  `LICENSE-VOICE` (for example, impersonating a real person or synthesizing a
  person's voice without their consent). Read the whole file before putting the
  voice into a product of your own.

The starting model is Reginutė (`lt_LT-reginute1-medium`, CC BY 4.0), trained on
the LIEPA corpus of Vilnius University; her lineage and credits are in the
[Reginutė README](https://github.com/RobertasTa/reginute/blob/main/README_EN.md).

The training checkpoint is not distributed: we give away the voice, not the
means of making one.

Copyright © 2026 Robertas Tarasevičius.

## Thanks

* **Reginutė**, and through her **the LIEPA corpus of Vilnius University** —
  where Ingutė started.
* **Arūnas Smaliukas** — the stress dictionary is built from his
  [`svogunas/g2p-lt-lexicon`](https://huggingface.co/datasets/svogunas/g2p-lt-lexicon)
  (CC BY 4.0) and the LIEPA annotations.
* **Michael Hansen** and the **Open Home Foundation** — for Piper.
