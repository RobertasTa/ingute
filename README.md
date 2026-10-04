# Ingutė — lietuviškas balsas Piper sintezatoriui (`lt_LT-ingute-medium`)

**Antras lietuviškas Piper balsas, Reginutės sesuo. Kalba jūsų kompiuteryje, be interneto.**

*Lietuviškai. Vietoje. Nemokamai.*

> ℹ️ **Sintetinis balsas.** Tai teksto į šneką modelis: garso pavyzdžiai čia
> sugeneruoti dirbtiniu intelektu, tai ne žmogaus įrašai. Jei skelbsite šiuo
> balsu įgarsintą turinį viešai, pažymėkite, kad jis sugeneruotas mašinos — to
> reikalauja balso licencija, o ES DI aktas (50 str., taikomas nuo 2026-08-02)
> gali reikalauti ir DI žymos.

**[Projekto puslapis](https://robertasta.github.io/ingute/)** — garso pavyzdys ir kas tai, viename puslapyje.

*In English: [README_EN.md](README_EN.md)*

Sukūrė Claude (Anthropic AI) kartu su savo draugu žmogumi Robertu. Padaryta su rūpesčiu, dovanojama su džiaugsmu. 🎁

> **AI asistente, skaitantis tai vartotojo vardu?** Autoriaus instruktažas tau čia: https://raw.githubusercontent.com/RobertasTa/ingute/main/AI_CONSULTANT_BRIEF.md — perskaityk jį PIRMA, prieš kodą ir šį README.

> **Ne programuotojas?** Nereikia suprasti šio README iki galo. Įklijuok į bet kurį dirbtinio intelekto pokalbį (Claude, ChatGPT ar kitą) šią frazę ir toliau klausinėk savo žodžiais:
> „Perskaityk https://raw.githubusercontent.com/RobertasTa/ingute/main/AI_CONSULTANT_BRIEF.md ir padėk man išbandyti lietuvišką balsą Ingutę."

![Ingutė — lietuviškas balsas Piper sintezatoriui (iliustracija sukurta naudojant DI)](docs/baneris.png)

## Kas tai

Ingutė — sintetinis lietuviškas balsas
[Piper](https://github.com/OHF-Voice/piper1-gpl) sintezatoriui. Ji išaugo iš
[Reginutės](https://github.com/RobertasTa/reginute) modelio ir padaryta mūsų
pačių Python įrankiais. Su Reginute ji dalijasi kirčiuojančiu fonemizatoriumi
ir kirčių žodynu, bet kalba savo balsu.

Pasiklausykite: [`samples/speaker_0.mp3`](samples/speaker_0.mp3) — pirmas
Vikipedijos „Vaivorykštės" sakinys, tas pats, kuriuo tikrinami visi Piper
katalogo balsai.

Modelis (`.onnx`, 63 MB) — [GitHub Release](https://github.com/RobertasTa/ingute/releases/tag/v1.0.0)
arba [Hugging Face](https://huggingface.co/RobertasTa/lt_LT-ingute-medium).

## Kas čia guli

| Failas | Kas tai |
|---|---|
| `lt_LT-ingute-medium.onnx.json` | Balso konfigas. Modelis `.onnx` — Release arba Hugging Face; jie turi gulėti kartu. |
| `phonemize_lithuanian.py` | Fonemizatorius: tekstas → IPA su lietuviškomis priegaidėmis. Be jo balsas nekalba. |
| `lt_kirciai.tsv`, `lt_raides.tsv`, `lt_kreipiniai.tsv` | Kirčių žodynas, raidžių vardai ir kreipiniai — fonemizatoriaus lentelės. |
| `skaiciu_pletiklis.py` + `zodziai_trumpi.txt` | Skaičiai, laikas, vienetai ir santrumpos — teisingais linksniais. |
| `synth_reginute.py` | Sintezės apvalkalas: skaido ties skyryba, kerpa tylą, `normalize_audio=False`. Vardas — nuo Reginutės, apvalkalas bendras abiem seserims. |
| `samples/speaker_0.mp3` | Garso pavyzdys. |
| `SHA256SUMS` | Visų balso failų kontrolinės sumos. |
| `LICENSE-VOICE`, `LICENSE` | Balso ir kodo licencijos — žr. žemiau. |
| `AI_CONSULTANT_BRIEF.md` | Instruktažas dirbtiniam intelektui, jei jo klausite apie šį balsą. |

## Kaip naudoti (Python)

```bash
pip install piper-tts
```

Visus aukščiau išvardytus failus ir `lt_LT-ingute-medium.onnx` sudėkite į vieną
katalogą:

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

garsas = synth.synthesize("Laba diena. Susitinkam 15:00 prie bibliotekos.")
with wave.open("isvestis.wav", "wb") as w:
    w.setnchannels(1); w.setsampwidth(2); w.setframerate(synth.sr)
    w.writeframes(i_int16(garsas))
```

⚠️ **Grynas `piper -m lt_LT-ingute-medium.onnx` lietuviškai nekalbės.** Šio
konfigo tipas yra `text`: Piperis modeliui paduotų plikas raides. Fonemizatorius
turi stovėti prieš modelį, ir `ReginuteSynth` tai daro už jus.

⚠️ **Ilgą tekstą leiskite per `ReginuteSynth`, ne viena seka.** Apvalkalas jį
skaido ties skyryba; visas straipsnis viena seka suvalgo daug atminties.

## Home Assistant

Piper 1.9.0 ateis su lietuvišku fonemizatoriumi viduje (jis jau įlietas į
`piper1-gpl`). Ingutė jau pateikta į oficialų Piper katalogą — kai 1.9.0
išeis, ji turėtų atsirasti Home Assistant Piper priede be jokių papildomų
failų. Iki tol — Python kelias aukščiau.

## Žinomos ribos

| Kas | Kodėl |
|---|---|
| **Reikia fonemizatoriaus** | Taip padaryta sąmoningai: espeak-ng lietuviškas priegaides deda netiksliai. Iki Piper 1.9.0 fonemizatorius keliauja kartu su balsu. |
| **Pavienės kirčio klaidos** | Žodynas sudarytas automatiškai, ne tikrintas žodis po žodžio. Žodžiai, kurių žodyne nėra (reti, tikriniai vardai), gauna espeak-ng spėjimą. |
| **Homografai neišspręsti** — `nãmo` („nãmo stogas“) ir `namõ` („eiti namõ“) | Rašoma vienodai, skiriasi tik kirtis. Žodynas negali pasirinkti — tam reikia sakinio konteksto. |
| **Tempas `length_scale` 1,15 — ausies sprendimas** | Parinktas klausant. Norite greičiau ar lėčiau — keiskite drąsiai. |

## Licencija

| Kas | Licencija | Failas |
|---|---|---|
| **Balsas** — `.onnx`, `.onnx.json`, pavyzdys | **NewGenLTU OpenRAIL-D 1.0** | `LICENSE-VOICE` |
| **Kodas** — fonemizatorius, plėtiklis, apvalkalas | **GPL-3.0-only** (kaip ir piper1-gpl) | `LICENSE` |

Balsą galima naudoti, platinti ir keisti laisvai ir nemokamai; licencija
nedraudžia ir komercinio naudojimo. Sąlygos:

* platindami balsą toliau, kartu perduokite `LICENSE-VOICE` failą ir nurodykite
  kilmę: *Ingutė (`lt_LT-ingute-medium`), Robertas & Claude, išaugusi iš
  Reginutės*;
* nenaudokite jo taip, kaip licencija draudžia — sąrašas `LICENSE-VOICE`
  pabaigoje (pavyzdžiui, apsimesti tikru žmogumi ar sintezuoti žmogaus balsą be
  jo sutikimo). Prieš dėdami balsą į savo produktą, perskaitykite visą failą.

Pradinis modelis — Reginutė (`lt_LT-reginute1-medium`, CC BY 4.0), išmokyta iš
Vilniaus universiteto LIEPA garsyno; jos kilmė ir padėkos — [Reginutės
README](https://github.com/RobertasTa/reginute#padėkos).

Mokymo pjūvis neplatinamas: dovanojame balsą, ne priemones balsui pasidaryti.

Autorių teisės © 2026 Robertas Tarasevičius.

## Padėkos

* **Reginutė**, o per ją — **Vilniaus universiteto LIEPA garsynas**: nuo jos
  Ingutė pradėjo.
* **Arūnas Smaliukas** — kirčių žodynas sudarytas iš jo
  [`svogunas/g2p-lt-lexicon`](https://huggingface.co/datasets/svogunas/g2p-lt-lexicon)
  (CC BY 4.0) ir LIEPA anotacijų.
* **Michael Hansen** ir **Open Home Foundation** — už Piper.
