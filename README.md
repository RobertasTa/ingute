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
| `wyoming_reginute.py` | Wyoming tarnyba Home Assistant'ui (bendra abiem seserims): tempą ir balso vardą ima iš modelio ir jo konfigo. Žr. „Home Assistant". |
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

**Jau dabar — per Wyoming, apie 5 minutes.** Ingutė prijungiama kaip atskira
Wyoming tarnyba šalia jūsų esamo Piper priedo; nieko esamo neliečia.

Bet koks Linux, root'u (kitaip — su `sudo`); išbandyta švariame Ubuntu 24.04 konteineryje, 2 branduoliai, 2 GB:

```bash
apt install -y python3-venv
python3 -m venv /opt/ingute/venv
/opt/ingute/venv/bin/pip install piper-tts==1.8.0 wyoming huggingface_hub
# visi balso failai ir serveris — iš Hugging Face (~65 MB)
/opt/ingute/venv/bin/python -c "from huggingface_hub import snapshot_download as s; s('RobertasTa/lt_LT-ingute-medium', local_dir='/opt/ingute/voice')"
# bandymui
/opt/ingute/venv/bin/python /opt/ingute/voice/wyoming_reginute.py --model /opt/ingute/voice/lt_LT-ingute-medium.onnx --uri tcp://0.0.0.0:10250
```

Visada leiskite **venv'o** Pythonu (pilnu keliu, kaip aukščiau): jei `pip` įdiegė į vieną
Python, o `python3` yra kitas, gausite `No module named 'piper'`, nors `piper-tts` įdiegtas.
Kad pasileistų pati — `/etc/systemd/system/ingute.service`:

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

Įspėjimas `onnxruntime … pthread_setaffinity_np failed` konteinerio žurnale — nekenksmingas.

Paleidus žurnale turi būti:
`ingute klauso tcp://0.0.0.0:10250 (tempas 1.15 iš konfigo, plėtiklis: taip)`.
Home Assistant: **Settings → Devices & Services → Add integration → Wyoming
Protocol**, serverio adresas ir portas 10250 — balsas atsiras kaip `ingute`;
Assist konvejeryje pasirinkite jį teksto į šneką varikliu. Tempas ir balso
vardas imami iš modelio ir jo konfigo — nurodyti nieko nereikia.

Kai išeis Piper 1.9.0 (lietuviškas fonemizatorius jau įlietas į `piper1-gpl`),
Ingutė — oficialiame Piper kataloge nuo 2026-10-20 — turėtų atsirasti Home
Assistant Piper priede be jokių papildomų failų, ir šio serverio nebereikės.

### Jei skamba ne taip, kaip pavyzdyje

| Ką girdite | Kodėl | Ką daryti |
|---|---|---|
| **Ištęstai, lėtai, „robotiškai"** | Ingutė kalba Reginutės tempu (1.30), ne savo (1.15) — ~13 % lėčiau. Taip būna su `wyoming_reginute.py`, paimtu iš Reginutės repo iki 2026-10-24: jis visiems balsams duodavo 1.30. | Imkite serverį iš šio repo arba pridėkite `--length-scale 1.15`. Žurnale turi būti „tempas 1.15". |
| **Švelniai trūkčioja tarp frazių** (Voice PE ar kita kolonėlė) | Serveris siunčia garsą frazėmis, kai tik kiekviena sukurta, o Home Assistant pradeda groti iš karto. Jei procesorius nespėja sukurti kitos frazės, kol skamba ankstesnė, atsiranda mikro pauzė. Piper sukasi **tik procesoriuje** — vaizdo plokštė jam nepadeda. | Mūsų namuose 2 branduolių konteineris (Ryzen 7 8845HS) 16 s kalbos sukuria per 1,8 s — atsarga dešimteriopa, tad kaltas dažniausiai **užimtas** procesorius. Jei tame pačiame kompe sukasi Ollama, `ollama ps` stulpelis PROCESSOR turi rodyti `100% GPU`: jei dalis — CPU, kalbos modelis netilpo į vaizdo plokštę ir atsakymo metu dalijasi branduoliais su balsu. Duokite balsui savo branduolius ar konteinerį. |
| **Trūkčioja ir kiti balsai** | Ne balsas, o Wi-Fi tarp kolonėlės ir Home Assistant. | Patikrinkite su bet kuriuo kitu Piper balsu; jei trūkčioja ir jis — kolonėlę ar prieigos tašką arčiau. |
| Serveris nepasileidžia: `phoneme_type='lithuanian'` | Paimtas konfigas iš oficialaus Piper katalogo — jis skirtas Piper 1.9.0+. | Šiam serveriui reikia `lt_LT-ingute-medium.onnx.json` iš šio repo (`phoneme_type: text`). |
| Balsas Home Assistant'e vadinasi `reginute1` | Senas serveris iš Reginutės repo. | Šio repo serveris vardą ima iš modelio failo (`ingute`); po atnaujinimo Assist konvejeryje balsą pasirinkite iš naujo. |

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
