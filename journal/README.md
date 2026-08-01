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

# 2026-08-01 — Self-hosted n8n stack on Windows, driven by Claude Code (MCP)

## What I did

Rebuilt a self-hosted n8n automation stack from scratch — this time not on the MacBook but on a Windows 11 laptop (i7-13700H, 32 GB RAM), so I had to redo the whole environment. Got it running end to end, versioned, and pushed to GitHub.

Enabled WSL2 + Ubuntu, installed Docker Desktop with WSL integration. All the Linux work lives inside WSL; nothing touches Windows directly.
Deployed n8n + PostgreSQL via docker-compose, with persistent volumes, secrets isolated in a .env, and a saved N8N_ENCRYPTION_KEY.
Turned on n8n's native instance-level MCP server and generated an access token.
Installed Claude Code, fixed its PATH, and connected it to n8n over MCP. Once connected it exposes 34 n8n tools (search nodes, build, validate, run).
Built and ran a first throwaway workflow (Manual Trigger → Set) entirely through a Claude Code prompt, to prove the chain works: I describe → Claude builds in n8n → I check.
Set up Git, wrote an export script that dumps each workflow to JSON, and pushed the whole stack to a new private repo automatisation.

## What I decided (and why)

Quality over speed, even on a borrowed machine. I could have run n8n with a one-line npm install and SQLite. I chose Docker + PostgreSQL instead: SQLite buckles on concurrent executions and large automations, and I want a foundation that scales to real YH Labs work, not a throwaway. The extra setup cost buys robustness I'll need later.
Isolate everything in WSL/Docker, nothing in Windows itself. It keeps my father's machine clean and the whole thing fully reversible (unregister the distro, uninstall Docker, done). It also means no Python or other runtime to install on the host — n8n's code runs inside the container. Docker's whole point is that the execution environment is self-contained.
Native MCP + Claude Code over the visual canvas. I'd rather describe automations in natural language and version them as code than drag nodes around. It fits my GitHub/Python habits and makes the work reproducible.
Treat automations as code. The workflow JSON lives in Git, not just in n8n's database — so I get history, diffs, rollback, and can rebuild the whole instance from the repo. That's the difference between a fragile black box and a maintainable asset.
Security as a habit, not an afterthought. Secrets (.env, encryption key, tokens) never touch Git; I verified it with git check-ignore before pushing, since a push is irreversible. Honest caveat: I did leave an n8n token visible in a screenshot and chose not to regenerate it — the risk is near-zero on a local-only instance, but it's exactly the kind of corner I should stop cutting before I handle client credentials.

## What's next

Choose and build the first real automation. This is the actual point of the stack. Two candidates: YH Labs prospection (collect → qualify → sequenced follow-ups) or association management (memberships, communications). The test workflow gets deleted once a real one exists.
Make ./scripts/export.sh a reflex after every build session, so nothing lives only in n8n's database.
Open question I'm holding: whether to keep this Windows stack separate from the Mac meeting-minutes pipeline, or eventually consolidate them.
Longer term: this runs on my father's machine — move it to my own machine or a small dedicated server when the project justifies it.
