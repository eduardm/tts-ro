# XTTS-v2 — Romanian Fine-Tuning

[XTTS-v2](https://github.com/coqui-ai/TTS) is a GPT-based text-to-speech model by Coqui AI with zero-shot voice cloning. It supports 16 languages out of the box — we added Romanian as the 17th.

## Vocabulary Extension

XTTS-v2's tokenizer does not include Romanian characters. We extended it with 6 new tokens:

| Token | Description |
|-------|-------------|
| ă | a-breve |
| â | a-circumflex |
| î | i-circumflex |
| ș | s-comma-below |
| ț | t-comma-below |
| [ro] | Romanian language token |

These tokens start from **random initialization** because the checkpoint loader skips embedding keys with shape mismatches. This is why a dedicated warmup phase is needed.

## Two-Phase Training Strategy

### Why Two Phases?

The 6 new tokens have random embeddings while the rest of the 518M-parameter model has pre-trained weights. Training everything together from the start injects noisy gradients from the random embeddings into the pre-trained layers. Phase 1 lets the new tokens "catch up" before the full model is unfrozen.

### Configuration

| Parameter | Phase 1 (Embedding Warmup) | Phase 2 (Full Fine-Tune) |
|-----------|---------------------------|--------------------------|
| **Freeze strategy** | `embeddings_only` (1.4% of params) | `full_gpt` (all params) |
| **Learning rate** | 1e-4 | 5e-6 |
| **Batch size** | 4 | 4 |
| **Gradient accumulation** | 8 | 63 |
| **Effective batch** | 32 | 252 |
| **Epochs** | 2 | 50+ |
| **text_ce_weight** | 0.01 | 0.01 |
| **Optimizer** | AdamW (betas=0.9, 0.96) | AdamW (betas=0.9, 0.96) |
| **gpt_use_masking_gt_prompt_approach** | True | True |
| **gpt_use_perceiver_resampler** | True | True |

### Mistakes done in previous tries

**Effective batch size >= 252.** The [official Coqui recipe](https://github.com/coqui-ai/TTS/blob/dev/recipes/ljspeech/xtts_v2/train_gpt_xtts.py) uses batch=3, grad_accum=84 (effective=252). Our first run used effective batch=32 and the model mode-collapsed — it memorized a single Romanian phrase and repeated it for every input. ([Issue #3992](https://github.com/coqui-ai/TTS/issues/3992))

**text_ce_weight = 0.01.** This is the default and intentional — text prediction is an auxiliary regularizer, not the primary objective. The mel prediction loss (weight=1.0) is the main training signal. Setting text_ce_weight=1.0 caused the model to overfit to text prediction instead of learning proper audio generation.

**gpt_use_masking_gt_prompt_approach = True.** Without this, the model learns to copy the reference audio into the output instead of conditioning on the input text. ([Issue #3122](https://github.com/coqui-ai/TTS/issues/3122))

## Results

### Epoch 15

Overall WER: **27.2%** (Good)

| Voice | Avg WER | Quality |
|-------|---------|---------|
| Female 1 | 21.5% | Good |
| Female 2 | 24.8% | Good |
| Male 1 | 28.0% | Good |
| Male 2 | 28.1% | Good |
| Male 3 | 33.5% | Working |

Full per-sentence results: [results/epoch-15/results.json](results/epoch-15/results.json)
Audio samples: [results/epoch-15/samples/](results/epoch-15/samples/)

## Reproducing

### Dependencies

```
Python 3.11
TTS==0.22.0
trainer==0.0.36
PyTorch 2.x
whisper (openai-whisper)
```

### Required Patches to TTS Library

The installed `TTS==0.22.0` requires 4 patches for PyTorch 2.x and Romanian support:

1. **torch.load weights_only** — Add `weights_only=False` to all `torch.load()` calls in TTS and trainer packages (PyTorch 2.x changed the default)
2. **Romanian tokenizer** — Add `"ro"` to the supported language set in `tokenizer.py`, add Romanian ordinal regex, make abbreviation/symbol dicts safe for unknown languages via `.get(lang, [])`
3. **Checkpoint size mismatch** — Patch `GPTTrainer.load_checkpoint` to skip keys with shape mismatches (needed for extended vocab)
4. **torch.device syntax** — Fix `torch.device("cpu", weights_only=False)` → `torch.device("cpu"), weights_only=False` in `gpt_trainer.py` and `hifigan_decoder.py`


