# tts-ro

Fine-tuning state-of-the-art TTS models to produce natural, high-quality Romanian speech.

## Why
De ce nu? Tot apar modele TTS, tot mai performante, dar nici unul nu suporta limba Romanna.
This project systematically fine-tunes and evaluates multiple TTS architectures to find the best one for Romanian.

## Results

### XTTS-v2 — Epoch 15 (training in progress)

| Voice | Avg WER | Quality | Samples |
|-------|---------|---------|---------|
| Female 1 | 21.5% | Good | [Listen](models/xtts-v2/results/epoch-15/samples/speaker_female_hp/) |
| Female 2 | 24.8% | Good | [Listen](models/xtts-v2/results/epoch-15/samples/speaker_female_adr/) |
| Male 1 | 28.0% | Good | [Listen](models/xtts-v2/results/epoch-15/samples/speaker_male_literature/) |
| Male 2 | 28.1% | Good | [Listen](models/xtts-v2/results/epoch-15/samples/speaker_male_bible/) |
| Male 3 | 33.5% | Working | [Listen](models/xtts-v2/results/epoch-15/samples/speaker_male_4/) |
| **Overall** | **27.2%** | **Good** | |

WER (Word Error Rate) is measured by generating audio from fixed Romanian test sentences, transcribing with Whisper large-v3, and comparing against the input text. Lower is better. See [Methodology](docs/methodology.md) for details.

### Models Under Evaluation

| Model | Params | Status | Notes |
|-------|--------|--------|-------|
| [XTTS-v2](models/xtts-v2/) | 518M | Training (epoch 15) | Two-phase fine-tuning, 27.2% WER |
| Qwen3-TTS | 1.7B | Planned | Official fine-tuning pipeline |

## Trained Checkpoints

Coming soon.

## Repository Structure

```
tts-ro/
├── docs/                          # Methodology and model roadmap
└── models/
    └── xtts-v2/                   # One directory per model
        ├── README.md              # Model card (architecture, training config, lessons)
        └── results/
            └── epoch-15/          # One directory per evaluation checkpoint
                ├── results.json   # Full WER metrics
                └── samples/       # Generated audio (WAV)
```

New models get their own directory under `models/`. New evaluation checkpoints get their own directory under `results/`.

## Listening to Samples

Browse the [demo page](https://eduardm.github.io/tts-ro/) to listen to all samples with inline audio players, or clone the repo and open WAV files locally.
