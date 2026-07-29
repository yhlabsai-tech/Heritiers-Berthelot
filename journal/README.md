# Logbook

One entry per working session. Each file is named by date so the folder reads
as a timeline from top to bottom.

**Naming:** `YYYY-MM-DD-short-title.md` — e.g. `2026-07-19-drafting-begins.md`

**Each entry answers three questions:**

1. **What I did** — the concrete work of the session.
2. **What I decided** — and *why* (the reasoning matters more than the choice).
3. **What's next** — the open threads I'm leaving for next time.

Keep entries short and honest.

# 2026-07-28- `Creating the GitHub Repository`
## What I did
I built the structure of my logbook to track the construction and development of the alumni association.
## What I decided
I decided to create this logbook to see the progress of my work and that of my team. The idea is also to make my alumni association visible at the national level and, why not, at the European level.

# 2026-07-29 — Self-hosted meeting-minutes pipeline (n8n + Whisper + Ollama)
## What I did

Set up a full local, self-hosted stack on my MacBook Air M2 to automate
meeting minutes, keeping everything free and running on my own machine.

- Deployed n8n, PostgreSQL and Qdrant as Docker containers (docker-compose),
  with persistent volumes.
- Installed Ollama natively to run a local LLM (llama3.1:8b) with Metal GPU
  acceleration, and connected it to n8n.
- Installed WhisperX (speech-to-text with speaker diarization) in a dedicated
  conda environment, with pyannote models for speaker separation.
- Ran a first end-to-end test: recorded audio → WhisperX transcription with
  per-speaker labels.

## What I decided (and why)

- Local models for prototyping (free, private), but I'll switch the summary
  step to an external API for the final version: an 8B model isn't reliable
  enough on long meetings (it drops decisions and action items).
- Capture audio at the OS level rather than through each platform's API, so
  the pipeline stays platform-agnostic (Meet, WhatsApp, in-person).

## What's next

- Expose WhisperX as a local service callable by n8n.
- Wire the full flow: audio file → transcription → summary → email.
- Automate the trigger on a watched folder.
