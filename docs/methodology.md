# Evaluation Methodology

## Overview

We evaluate text-to-speech quality by measuring how accurately the generated audio reproduces the input text. The pipeline is:

1. Feed a Romanian text sentence to the TTS model
2. The model generates a WAV audio file
3. Whisper large-v3 transcribes the generated audio back to text
4. Word Error Rate (WER) is computed between the original input and the transcription

A low WER means Whisper heard exactly what we asked the model to say — the model is producing intelligible, accurate Romanian speech.

## Test Sentences

We use 8 fixed Romanian sentences covering different phonetic patterns, diacritics, and sentence structures:

| # | Sentence |
|---|----------|
| 1 | Bună ziua, mă numesc Alexandru și sunt din București. |
| 2 | România este o țară frumoasă cu munți înalți și câmpii întinse. |
| 3 | Într-o dimineață de toamnă, frunzele cădeau ușor pe aleea din parc. |
| 4 | Științele exacte și cele umaniste se completează reciproc. |
| 5 | Copiii se jucau fericiți în grădina plină de flori colorate. |
| 6 | Profesorul a explicat cu răbdare lecția dificilă de matematică. |
| 7 | Muzica populară românească are ritmuri și melodii unice în lume. |
| 8 | Această carte reprezintă o contribuție importantă la literatura contemporană. |

These sentences include all Romanian diacritics (ă, â, î, ș, ț), various vowel combinations, and a mix of common and less frequent words.

## WER Computation

Word Error Rate uses the word-level Levenshtein (edit) distance:

```
WER = (substitutions + insertions + deletions) / number_of_reference_words
```

Both reference and hypothesis are lowercased and split on whitespace before comparison.

## Quality Thresholds

| WER Range | Quality | Interpretation |
|-----------|---------|----------------|
| < 30% | Good | Text conditioning working well, speech is intelligible |
| 30–50% | Working | Text is mostly followed, some words garbled |
| 50–80% | Needs training | Text conditioning partially working |
| > 80% | Broken | Model is likely parroting reference audio or mode-collapsed |

## Why Whisper WER

Training loss alone is unreliable for TTS quality assessment. A mode-collapsed model (one that repeats the same phrase regardless of input) can show decreasing training loss while producing identical output for every sentence. WER catches this because the transcription of a repeated phrase will not match the diverse input texts.

## Whisper Model

We use [Whisper large-v3](https://github.com/openai/whisper) with `language="ro"` for transcription. This is OpenAI's largest and most accurate speech recognition model, providing a reliable reference transcription for WER computation.
