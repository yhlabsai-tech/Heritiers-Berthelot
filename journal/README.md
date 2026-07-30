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

# 2026-07-30 — Meeting-minutes pipeline finished, and association governance
## What I did

Finished wiring the self-hosted meeting-minutes pipeline end to end, and got a real recorded meeting to flow all the way through to a formatted email. Also advanced the governance side of the alumni association.

Pipeline (n8n + WhisperX + LLM + SMTP):

Exposed WhisperX as a local HTTP service (uvicorn, port 9000) callable from n8n, and wired the full flow: audio file → transcription → summary → email.
Added an Execute Command node that automatically picks the most recent .m4a in the recordings folder, so I no longer hard-code the filename. This required re-enabling the node in n8n 2.0 (disabled by default for security) via NODES_EXCLUDE=[] in docker-compose.
Fixed the OS-level audio capture so it records my voice and system audio together without echo.
Rebuilt the email step so the summary is delivered as clean, inline-styled HTML.
Ran a deliberate load test on a synthetic ~32k-word (≈4h) transcript to find where the model breaks.

Association:

Finished the statutes and the core administrative groundwork (object, seat, bureau, dues, board, general assembly, internal rules, moderation, data protection, dissolution, amendment procedure).
Took the role of Secretary General of the application division, in charge of the web development of the app.

## What I decided (and why)
Separate analysis from formatting. My first prompts asked the LLM to both analyze the transcript and produce the HTML. On long inputs it took the easy path — returned a nicely formatted but empty template. I split the two: the LLM now returns only structured JSON (analysis), and a Code node builds the HTML deterministically. Formatting is now perfect every time instead of depending on the model's mood.
Never let the LLM produce factual data. It kept inventing dates (even emitting raw PHP <?php echo date() ?>). The date is now injected by n8n, not the model. General rule I'm keeping: anything factual (dates, figures) comes from code, never from the model.
A single-pass summary can't handle very long meetings. The load test was conclusive: the small model accepts a 4h transcript but drops the middle; GPT-4 refuses it outright (context window exceeded). So a map-reduce approach (chunk → summarize each chunk → merge) is the only path for long meetings. I did not build it yet — real meetings are short enough that it isn't needed, and I'd rather not add complexity I can't justify.
Fix problems at the source, not with more automation. Recordings were landing in Downloads; instead of adding a node to move files around, I changed the recording app's output folder. One less moving part.
Stepped back from the presidency, on purpose. Having done the bulk of the administrative work (writing the statutes, structuring the association), I chose a technical role over a representative one. Secretary General of the application division fits my goal better: it keeps me on the build side — the web app, which is where the long-term value of the project sits — rather than on signatures and representation.
## What's next
Speed up transcription: first switch the WhisperX model from medium to small (2–3× faster, free); evaluate a cloud STT API later if speed matters more than staying fully local. Open question I'm holding: privacy vs. speed.
Build the map-reduce summarization path for long meetings.
Automate the trigger on a watched folder (Local File Trigger) so the flow runs with zero clicks.
Association: send the statutes to review, then file with the prefecture; start a prototype of the web app.
