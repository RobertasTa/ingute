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

Piper 1.9.0 will ship with the Lithuanian phonemizer built in (it is already
merged into `piper1-gpl`). Ingutė has already been submitted to the official
Piper catalogue; once 1.9.0 is released, she should appear in the Home Assistant
Piper add-on with no extra files. Until then, use the Python path above.

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
