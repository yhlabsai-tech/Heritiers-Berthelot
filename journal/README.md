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

# 2026-08-02 — 2026-08-08 - Mac stack rebuilt, statutes and internal rules sent

## What I did

Slower week overall — school holidays, and the team is scattered. Still, two things
moved: the association's founding documents went out, and I rebuilt the automation
stack on my own machine.

**Association (1–8 August):**

- Logo designed and validated by the team.
- Statutes and internal rules finalised over the week and **sent officially today**.
- Pace deliberately slower than in July. Everyone is on holiday; I'd rather have
  documents that hold up than documents sent fast.

**Automation stack (Mac):**

Rebuilt the whole chain on the MacBook — Docker Desktop → n8n + PostgreSQL →
n8n-mcp → Claude Code + VS Code — so I stop depending on my father's Windows
laptop.

- Inspected the existing `docker-compose.yml` before touching anything. It was
  already PostgreSQL 16 with named volumes, matching the Windows architecture.
  The migration I'd planned turned out to be unnecessary.
- Hit an encryption-key mismatch: the `n8n_data` volume held one key, the `.env`
  another. n8n refuses to start in that state.
- Lost the n8n password. Reset the owner account from the CLI
  (`n8n user-management:reset`) after dumping the database — 6.7 MB backup.
  Workflows and credentials survived intact, including the meeting-minutes
  pipeline from July.
- Installed Node.js, Claude Code (native installer), and registered `n8n-mcp`
  at project scope.
- First MCP call failed: `n8n-mcp` ships an SSRF guard that blocks localhost by
  default. Fixed with `WEBHOOK_SECURITY_MODE=moderate`.
- Cleaned up VS Code (7 extensions down to 6) and installed the Claude Code
  extension.

**Mistake worth recording:** I assumed Claude Code needed Node.js and installed
Node for that reason. It doesn't — the native installer is standalone. Node was
still needed, but for `npx`, which runs `n8n-mcp`. Right action, wrong reasoning.
I checked the official docs only after giving myself the answer.

## What I decided (and why)

**Align the `.env` to the volume's key, never the reverse.** The obvious move was
to wipe the volume and start clean. That would have been the worst outcome:
workflows and credentials live in PostgreSQL, encrypted with the key stored in
the n8n volume. Wiping one leaves the other orphaned — visible workflows with
permanently unreadable credentials. Aligning the `.env` is reversible and keeps
both volumes coherent.

**Back up before any reset, even a documented one.** The `user-management:reset`
command is supposed to leave workflows untouched, and it did. I dumped the
database first anyway. Ten seconds against the risk of losing July's pipeline.

**Register the MCP server at project scope, not globally.** This finally explains
the rule I'd noted on Windows without understanding it — that I must `cd` into the
project before launching `claude`. The configuration lives in
`~/n8n-local/.mcp.json`; launch from elsewhere and the file isn't read. It was
never a quirk of the tool, just a consequence of the scope I'd chosen. In VS Code
the same rule becomes: open `n8n-local` as the workspace root.

**A technical lock beats a written convention.** I wanted a rule against deleting
workflows through prompts. Writing it in a `CLAUDE.md` would only be a
recommendation the model follows willingly. Instead I disabled the tool itself
(`DISABLED_TOOLS=n8n_delete_workflow`). Deletion is still possible from the n8n
interface, where I can see what I'm doing. **Honest caveat:** `n8n_workflow_versions`
is still enabled and can restore or prune versions, so the destructive path isn't
fully closed.

**`moderate`, not `permissive`, for the SSRF guard.** The permissive mode would
also open private network ranges. I only need localhost.

**VS Code over the bare terminal.** No performance difference — same engine, same
model, same MCP tools. The choice is purely ergonomic, and my work is
file-and-Git-centred, which is exactly where a bare terminal loses.

**Security note:** VS Code 1.132 now ships a native agent host. It read my
`.mcp.json`, which contains the n8n API key in plaintext. Low risk here — local
instance, key expires 6 November — but the principle stands: any agent opened on
that folder sees that file. `.mcp.json` never goes into Git.

## What's next

**Association (team):**

- Open LinkedIn and Instagram accounts — the communication division's first
  concrete step.
- Keep prospecting for future mentors.

**My side:**

- Carry on building out the ecosystem on my own machine, and keep training on it.
  Concretely, the open threads are: a `CLAUDE.md` for the stack (architecture,
  conventions, validate-before-publish), n8n's instance-level MCP server so
  automations can be triggered and not only built, and WhisperX restarted on
  demand to re-test the meeting pipeline.
- Train specifically on the automations the association will need: administrative
  workflows, Drive management, automated emails.

*What this gives the association: the environment set up today — Claude Code and
VS Code, configured and verified — is the one I'll build the platform in. The
Next.js work starts from here rather than from a blank machine.*

# 2026-08-09 — First event planning meeting, Conda incident, and prompt hardening
## What I did

Team meeting to organize the association's first event — a welcome gathering for incoming ECG1 students — plus a technical fix on the meeting-minutes pipeline and the last steps of the VS Code / Claude Code setup.

**Association:**

Reviewed the week's objectives with the team: statutes sent ✅, minutes drafted ✅, mail to Thamma for the alumni network ✅, mentorat/tutorat document created ✅, Romain's Drive split up ✅, list of incoming students requested from Amyot ✅.
Wrote and circulated a full planning document on the association's WhatsApp, covering location and schedule, event goals, food/drinks, activities, equipment, communication (before and during the event), and the responsibilities still needing an owner (food, games, setup, photos/video, comms, welcoming, cleanup).
Assigned the open tasks for the coming days: Rayane — file the association with the prefecture and propose a logo; Elyes — contact the town hall and advance the tutor/mentor handbook; Enzo — fill out and improve the two mentorat/tutorat docs; Alexandre — logo.

**Automation stack:** 

WhisperX had stopped responding on port 9000. Root cause: a prior troubleshooting session (outside this stack, using ChatGPT) had misdiagnosed the problem, checked the wrong Conda paths (~/miniconda3, ~/anaconda3 instead of the real /opt/anaconda3), and installed a duplicate Miniconda via Homebrew — which silently masked the real whisperx environment instead of fixing anything. Diagnosed the real path, confirmed the original environment and its binaries were untouched, cleaned the duplicate conda initialize block out of .zshrc, and restarted the Docker stack.
Rewrote the OpenAI structuring prompt: split into a System message (fixed instructions only) and a User message (transcript as an expression variable), added two new JSON keys — pistes and points_non_tranches — and upgraded the model. Re-ran it on the actual meeting transcript from this session; the output correctly captured details the previous version missed (domain name cost, membership fee amount, bakery partnerships, newsletter format) and correctly separated an individual's funding wish from an actual group decision.
Finished the VS Code + Claude Code setup: n8n-mcp verified at project scope (24 tools), Docker images pinned to exact versions, CLAUDE.md updated.

## What I decided (and why)

Check the real path before trusting someone else's diagnosis — including a past AI session's. The Conda "break" wasn't a break at all: the environment was fine at /opt/anaconda3/envs/whisperx, but a previous session never checked that path and layered a second Conda install on top. One command against the actual path settled it, instead of reinstalling anything.
All event-planning points get locked before August 17. That's the date the administration needs the information to relay it to incoming ECG1 students, so the whole planning doc treats it as the hard deadline rather than a soft target.
New JSON keys plug into the existing formatting node, not into a new one. pistes and points_non_tranches extend the same Code node that already builds the HTML — confirms the analysis/formatting split decided two weeks ago was the right call, since adding fields didn't require touching the LLM step's architecture.

## What's next

Lock the event's final date, activity list, headcount estimate, and material/food ownership before August 17.
Update the JavaScript formatting node so pistes and points_non_tranches actually reach the email output — the prompt already returns them.
Enable diarisation (pyannote) in WhisperX for speaker attribution.
Run a full end-to-end pipeline test on a real, unedited meeting recording.
Association: préfecture filing (Rayane), town hall contact (Elyes), first social media accounts.

What this gives the association: [à valider avec toi — je te propose une piste, dis-moi si tu gardes] a hardened structuring prompt and a documented Conda incident mean the meeting-minutes pipeline survives external meddling without losing a session's worth of transcripts, right as the event-planning cadence picks up and the team starts generating more minutes to process.

# 2026-08-11 — Bureau formalized, and the platform's Phase 0 is done

## What I did

Two fronts moved today: the association's governance structure got its first
formal shape, and the web platform went from "not started" to a working
Supabase backend with a real schema.

**Association:**

- Enzo laid out the full bureau structure on WhatsApp: Président (Romain
  Very), two Vice-Présidents (Rayane Saalaoui, Enzo Pruvost-A), Trésorier
  (Lucas Corte), and three Secrétaires Généraux, one per pôle — tutorat/
  mentorat (Elyes Bahnis), application (me), and événementiel (Alexandre
  Pithon-Laumônier). Each pôle other than the application one is structured
  as a committee: one SG plus three "responsables de pôle" who sit under
  them but outside the bureau itself. The événementiel pôle got its three
  responsables named on the spot: Karla, Dorothée, Éléa.
- Event date narrowed down: August 29 proposed as a target, pending an
  availability check across the team before it's locked.
- Logo finished by Alexandre and delivered as a ZIP (building-only version +
  full logo). Instagram and LinkedIn account creation is now unblocked for
  tomorrow; Thamma/Tercero will redirect alumni toward them once they're up.
- Flagged what I'll need next for the app: a merged directory of contacts —
  what Arthur already collected, what we already have, and what's still
  missing — plus alumni emails and current P1/P2 students added to that same
  directory, since it will double as the seed data for account creation.

**Platform (Next.js + Supabase) — Phase 0:**

- Created the Supabase project (Paris region), initialized a private
  `heritiers-berthelot-app` repo in Next.js 16, pushed it to GitHub, and
  linked the Supabase CLI to the project.
- Wrote and applied the initial migration: 9 tables, RLS enabled on all of
  them, 7 custom types, 4 functions, 2 triggers (profile creation on
  signup, and a lock trigger), 26 RLS policies, and an `avatars` storage
  bucket.
- Hit three real incidents in the process — an empty migration pushed by
  mistake, a `supabase migration repair` to fix the resulting history
  mismatch, and a SQL string-escaping bug (double quotes where the file
  needed doubled single quotes, e.g. in `Côte d''Ivoire`). All three
  resolved; `supabase db push` now reports the remote database up to date.
- Generated the TypeScript types from the live schema
  (`lib/types/database.ts`, 534 lines) and confirmed a known column
  (`statut_parcours`) resolves correctly — proof the generated types match
  the real schema, not a stale draft.
- Committed and pushed: `supabase/` and `lib/types/` are now in Git, not
  just applied to the remote database.

## What I decided (and why)

- **Every file edit is a full regeneration, never a partial patch.** I'd
  been sending partial SQL edits while also asking for full files elsewhere
  — inconsistent, and it's exactly how a copy-paste lands in the wrong
  place in an editor. One rule from now on: any change to a file means the
  whole file gets regenerated, destination stated first.
- **Registration stays open to everyone in v1; the validation mechanic is
  built but dormant.** Gating signups behind manual approval would slow
  down exactly the alumni-outreach push happening on WhatsApp right now.
  The validation logic exists in the schema so it can be switched on later
  without a migration, but it isn't enforced yet.
- **Progression status and filière choice are separate fields, not one.**
  Conflating "where someone is in the program" with "which track they
  picked" would have made the RLS policies and the profile logic depend on
  a single column doing two jobs — splitting them now avoids a schema
  change later once real data exists.
- **No agent team on this project until I can read an RLS policy myself.**
  It would be faster to let an agent generate and apply policies
  autonomously, but a wrong RLS policy silently leaks or blocks data with
  no error message. I'm keeping this manual, migration-by-migration, until
  I trust my own review of what each policy actually does.
- **Application is the one pôle I took by choice, not the presidency.** The
  app was my project before the bureau existed — it's the reason I stepped
  back from the presidency in the first place (see 2026-07-30). Being SG of
  that pôle rather than président keeps me on the build side, which is
  where I actually want to be. Whether it grows a three-person committee
  like the other pôles is a separate, later question — there's no codebase
  yet to split work on.

## What's next

**Association:**

- Confirm the event date (currently August 29, pending availabilities) and
  finish assigning event-day responsibilities.
- Get Instagram and LinkedIn live tomorrow; send the "accounts are ready"
  mail to Thamma/Tercero so alumni get redirected.
- Build the merged contacts directory (Arthur's data + what we have +
  what's missing), and collect P1/P2 emails to seed it.

**Platform:**

- Environment variables and the Supabase client (`lib/supabase/`).
- Authentication: signup, login, profile page — this is the start of Phase 1.
- Non-code task, can happen between sessions: write the membership form's
  line collecting email + directory consent, since it gates everything
  downstream (the directory, the accounts, the outreach).
- Write up the migration incidents (empty push, repair, SQL escaping) in
  detail somewhere — they're the kind of mistake worth documenting exactly
  because they're the ones everyone hits once.

# 2026-08-13 — v1 scope cut down, auth flow under construction, first outreach to teachers

## What I did

Spent the session narrowing the platform's v1 scope down to something
actually buildable, then started implementing the first slice of it —
authentication — and hit a real ambiguity in Supabase's email-confirmation
flow along the way. Also sent the first formal outreach email to the
teaching staff.

**Scoping:**

- Walked through the full feature list I'd drafted (messaging, feed,
  profiles, multi-criteria search, parents) against what a handful of early
  users actually need. Only one item survived unchanged: a centralized poll
  for oraux blancs availability — the one piece of the whole list tied to a
  real, dated, currently-felt pain point.
- Cut messaging from v1 entirely, replacing it with a "request to connect"
  flow (three-line message → email to the alumni → accept/refuse). Cut
  free-text profile descriptions in favor of structured fields (promo,
  école, secteur, ville, pays from closed lists) plus a separate free bio.
  Cut parents from v1 — kept in the roadmap, not coded.
- Wrote out the GDPR constraints that shape the architecture before any
  code: Supabase region must be EU (Paris/Frankfurt, irreversible after
  project creation), registration must be closed (allowlist or manual
  validation, never open), and importing the association's existing
  spreadsheet directory into the app counts as a new processing activity
  that requires informing the people in it first.

**Data model and auth:**

- Drafted the v1 schema: `profiles`, `allowlist` (the existing directory,
  matched on email — never on name), `posts` (info/event/poll), `poll_options`
  / `poll_votes` (non-anonymous, tied to a profile), `demandes_relation`.
  Row Level Security rules sketched per table.
- Realized the sequencing question I'd been treating as "feed vs. directory
  in parallel" was really "identity first, then two branches" — the feed
  needs a profile, not the directory, so identity isn't optional scaffolding,
  it's the actual prerequisite.
- Started the real auth flow. Hit a wall: Supabase's shared email service
  won't let me customize confirmation templates without a personal SMTP
  server, and the default confirmation link's parameter format (`code` vs.
  `token_hash`) is ambiguous in the docs depending on the auth flow used.
- Built a route handler that accepts both parameter forms rather than
  guessing which one applies, and a `REPORTS.md` tracking deferred items
  (custom SMTP being the next one to become blocking).
- Testing the real signup flow with my own allowlisted address is the next
  concrete step — the actual URL received is the only way to settle which
  parameter format Supabase is sending, not the documentation.

**Association:**

- Sent the first formal email to the teaching staff: statutes and internal
  rules filed with the prefecture, the pre-rentrée event underway, the app
  foundations in progress, and three concrete asks — the list of ECG alumni
  from Berthelot, a convention on using the lycée's name and visual identity
  (logo, typography), and a WhatsApp group for bureau/teachers/administration
  coordination. Also flagged the funding reality plainly: insurance, a
  domain name, server costs, and a proposed €5–10 membership fee.

## What I decided (and why)

- **Identity is a prerequisite, not a feature to schedule alongside others.**
  I'd been planning to build the feed and the directory "in parallel." They
  can, but only after profiles, roles, and validation exist — the feed
  depends on identity, the directory doesn't need the feed. Attacking all
  three at once would produce three half-built features and ship none of
  them.
- **No feature ships without an external deadline.** A prépa schedule will
  always lose to actual math homework if the only pressure on this project
  is my own motivation. Oraux blancs (October–December) gives the poll
  feature a real date; the pre-rentrée event gives the directory an email
  list to seed itself with. Both are now anchored to dates I don't control.
- **Roles are never self-declared.** They come from the allowlist or an
  admin, never from a signup form field — otherwise the first person to
  register can call themselves "professeur" and post as one.
- **Diagnose before guessing, even against ambiguous documentation.** Rather
  than picking one interpretation of Supabase's auth-link format and hoping
  it's right, the route handler accepts both known forms, and the real
  answer comes from testing an actual signup and reading the resulting URL.
- **Existing directory data isn't free to reuse.** It was collected for a
  spreadsheet, not an app — importing it into the platform is a new GDPR
  processing activity that needs its own notice to the people in it before
  the import happens.

## What's next

- Run the real signup test (my own allowlisted address) and read the
  confirmation URL to settle the `code` vs. `token_hash` question for good.
- Configure a personal SMTP provider (Resend) before opening registration,
  to unblock French-language templates and remove the "before code" excuse
  it currently has.
- Finish the identity layer (profiles + roles + validation), then move to
  the feed and directory branches.
- Write the GDPR notice that has to go out before the existing spreadsheet
  directory is imported as `allowlist`.
- Association: follow up with teachers on the alumni list, the naming/logo
  convention, and the WhatsApp group request.

# 2026-08-14 — Feed and polls close the beta loop; social presence goes live

## What I did

Two fronts closed today: the platform's beta became functional end to end,
and the événementiel/communication pôle took the association's first public
presence live on LinkedIn, Instagram, and Facebook.

**Platform — feed and polls:**

- Built the feed and polls feature (posts, poll options, votes) on top of the
  existing schema, then pushed the migration and regenerated the TypeScript
  types.
- Building the feed surfaced three real gaps the schema had been carrying
  since the start:
  - `posts.auteur_id` pointed at the directory (`annuaire`), which is a
    closed, curated table — a professor removed from the directory would
    have had their posts silently unsigned. Fixed by pointing authorship at
    `profiles` instead: publishing is about identity, not about being
    listed.
  - Same bug on the votes side — poll votes need to show who voted, not
    just a count, because the whole point of the oraux-blancs poll is
    knowing who to actually write to in November.
  - `cloture_le` (a poll's closing date) existed as a column but no policy
    ever read it — votes were accepted after closing, and even against a
    guessed option ID on an unpublished draft. A date with no enforcement
    is worse than no date at all.
- Confirmed multiple-choice is intentional in the schema: the primary key is
  `(option, profil)`, not `(post, profil)`, so a professor can mark
  themselves available for both October and November oraux blancs.
- Settled on a draft-then-publish pattern for poll creation: the two writes
  (post + options) aren't wrapped in a client-side transaction, so a failed
  options write must leave an invisible draft, not a poll with no answers
  visible in everyone's feed.
- Beta is now functional end to end: identity, directory, admin, feed,
  polls. What's left before opening to a second member is service work, not
  product work — logged as a five-item blocker list.

**Association — public launch:**

- LinkedIn company page is live (`Les Héritiers de Berthelot — Rassembler
  pour transmettre`, based Saint-Maur-des-Fossés), Instagram and Facebook
  pages created the same day.
- Wrote the founding "About" text: the concrete problem that started the
  project (struggling to reach alumni during oral exam prep, LinkedIn
  outreach going unanswered, Arthur's existing directory limited to top-5
  math students), the four founding pôles, and the mission.
- Iterated with the team on the banner and logo across several rounds —
  wrong LinkedIn banner dimensions caught and fixed, the logo appearing
  twice in inconsistent styles flagged and resolved, a lighter/more
  legible version chosen over the first pass.
- Split messaging by platform on purpose: LinkedIn keeps the longer
  storytelling version, Instagram gets a shorter three-pillar summary
  (Mentorat, Plateforme, Événements) to avoid the two posts reading as
  duplicates.
- Confirmed the association explicitly targets ECG-only for now — Enzo
  flagged the risk of scientifique/littéraire prépa students expecting
  tutoring access from seeing the page, so the bio and posts state the
  scope directly rather than leaving it implicit.
- Professional email address discussed and deliberately deferred — nobody
  has picked it up yet and enough is already shipping today without it.

## What I decided (and why)

- **Identity, not directory membership, is what "signing" a post means.**
  The bug the feed surfaced wasn't cosmetic: conflating "listed in the
  annuaire" with "has an account" would have silently unsigned real
  content the moment someone left the directory. Authorship now points at
  the identity layer, matching the sequencing decision from two days ago —
  everything really does depend on profiles, not on the directory.
- **A poll with no visible voters isn't useful.** Anonymizing poll votes
  would have looked more privacy-conscious but defeats the actual use case:
  the association needs to know who's available for oraux blancs, not just
  how many people are.
- **An unenforced closing date is a liability, not a feature half-built.**
  Rather than ship `cloture_le` as decorative and fix it later, it got wired
  into the policies today — a date that silently does nothing is worse than
  not having the column.
- **State the target audience explicitly rather than let people assume.**
  ECG-only wasn't a given from the outside — a prépa scientifique student
  seeing the page could reasonably expect tutoring access. Saying it plainly
  in the bio avoids disappointing people the association isn't built for
  yet, and keeps the roadmap (expanding to other filières later) honest
  about being a later step, not a current one.
- **SMTP is now a core-feature blocker, not an onboarding nice-to-have.**
  It stopped being about welcome-email wording the moment polls went live —
  without it, reopening a poll for oraux blancs has no way to notify
  professors, which recreates the exact email-chain problem the platform
  was supposed to remove. Reclassified accordingly in the project's tracked
  blockers.

## What's next

- Clear the five remaining blockers before opening the platform to a second
  member.
- Configure SMTP (Resend) — now blocking the poll notification loop, not
  just template translation.
- Finish the Instagram launch post (last slide pending, Flora posting
  tomorrow morning) and coordinate the Facebook page's admin access.
- Consider a shared visual identity (DA) system — logo, banner, and post
  formats currently don't match, and the team flagged it as worth solving
  once, not per-asset.
- Cross-post the LinkedIn/Instagram links into the existing class WhatsApp
  groups per year group.

# 2026-08-14 — Feed and polls close the beta loop; social presence goes live

## What I did

Two fronts closed today: the platform's beta became functional end to end,
and the événementiel/communication pôle took the association's first public
presence live on LinkedIn, Instagram, and Facebook.

**Platform — feed and polls:**

- Built the feed and polls feature (posts, poll options, votes) on top of the
  existing schema, then pushed the migration and regenerated the TypeScript
  types.
- Building the feed surfaced three real gaps the schema had been carrying
  since the start:
  - `posts.auteur_id` pointed at the directory (`annuaire`), which is a
    closed, curated table — a professor removed from the directory would
    have had their posts silently unsigned. Fixed by pointing authorship at
    `profiles` instead: publishing is about identity, not about being
    listed.
  - Same bug on the votes side — poll votes need to show who voted, not
    just a count, because the whole point of the oraux-blancs poll is
    knowing who to actually write to in November.
  - `cloture_le` (a poll's closing date) existed as a column but no policy
    ever read it — votes were accepted after closing, and even against a
    guessed option ID on an unpublished draft. A date with no enforcement
    is worse than no date at all.
- Confirmed multiple-choice is intentional in the schema: the primary key is
  `(option, profil)`, not `(post, profil)`, so a professor can mark
  themselves available for both October and November oraux blancs.
- Settled on a draft-then-publish pattern for poll creation: the two writes
  (post + options) aren't wrapped in a client-side transaction, so a failed
  options write must leave an invisible draft, not a poll with no answers
  visible in everyone's feed.
- Beta is now functional end to end: identity, directory, admin, feed,
  polls. What's left before opening to a second member is service work, not
  product work — logged as a five-item blocker list.

**Association — public launch:**

- LinkedIn company page is live (`Les Héritiers de Berthelot — Rassembler
  pour transmettre`, based Saint-Maur-des-Fossés), Instagram and Facebook
  pages created the same day.
- Wrote the founding "About" text: the concrete problem that started the
  project (struggling to reach alumni during oral exam prep, LinkedIn
  outreach going unanswered, Arthur's existing directory limited to top-5
  math students), the four founding pôles, and the mission.
- Iterated with the team on the banner and logo across several rounds —
  wrong LinkedIn banner dimensions caught and fixed, the logo appearing
  twice in inconsistent styles flagged and resolved, a lighter/more
  legible version chosen over the first pass.
- Split messaging by platform on purpose: LinkedIn keeps the longer
  storytelling version, Instagram gets a shorter three-pillar summary
  (Mentorat, Plateforme, Événements) to avoid the two posts reading as
  duplicates.
- Confirmed the association explicitly targets ECG-only for now — Enzo
  flagged the risk of scientifique/littéraire prépa students expecting
  tutoring access from seeing the page, so the bio and posts state the
  scope directly rather than leaving it implicit.
- Professional email address discussed and deliberately deferred — nobody
  has picked it up yet and enough is already shipping today without it.

## What I decided (and why)

- **Identity, not directory membership, is what "signing" a post means.**
  The bug the feed surfaced wasn't cosmetic: conflating "listed in the
  annuaire" with "has an account" would have silently unsigned real
  content the moment someone left the directory. Authorship now points at
  the identity layer, matching the sequencing decision from two days ago —
  everything really does depend on profiles, not on the directory.
- **A poll with no visible voters isn't useful.** Anonymizing poll votes
  would have looked more privacy-conscious but defeats the actual use case:
  the association needs to know who's available for oraux blancs, not just
  how many people are.
- **An unenforced closing date is a liability, not a feature half-built.**
  Rather than ship `cloture_le` as decorative and fix it later, it got wired
  into the policies today — a date that silently does nothing is worse than
  not having the column.
- **State the target audience explicitly rather than let people assume.**
  ECG-only wasn't a given from the outside — a prépa scientifique student
  seeing the page could reasonably expect tutoring access. Saying it plainly
  in the bio avoids disappointing people the association isn't built for
  yet, and keeps the roadmap (expanding to other filières later) honest
  about being a later step, not a current one.
- **SMTP is now a core-feature blocker, not an onboarding nice-to-have.**
  It stopped being about welcome-email wording the moment polls went live —
  without it, reopening a poll for oraux blancs has no way to notify
  professors, which recreates the exact email-chain problem the platform
  was supposed to remove. Reclassified accordingly in the project's tracked
  blockers.

## What's next

- Clear the five remaining blockers before opening the platform to a second
  member.
- Configure SMTP (Resend) — now blocking the poll notification loop, not
  just template translation.
- Finish the Instagram launch post (last slide pending, Flora posting
  tomorrow morning) and coordinate the Facebook page's admin access.
- Consider a shared visual identity (DA) system — logo, banner, and post
  formats currently don't match, and the team flagged it as worth solving
  once, not per-asset.
- Cross-post the LinkedIn/Instagram links into the existing class WhatsApp
  groups per year group.

# 2026-08-16 — Going live, and two lessons in method

`https://lesheritiersdeberthelot.fr` is live. Signup works end to end,
outgoing mail is sent from the association's own address, and anyone can now
create an account.

## What I did

**Domain and mailboxes:** `lesheritiersdeberthelot.fr` registered with OVH
under Romain's name as président, on the association's Google account.
Mailboxes are named by function — `contact@`, `donnees@`, `application@`,
`evenements@`, `tutorat@`, `bonjour@` — not by person, for the same reason
as always: a bureau turns over, and it's easier to change a password than
every address printed on every document.

**Email sending,** through OVH's SMTP, from a mailbox dedicated to the app.
I'd originally recommended a specialized transactional-email service;
writing the processing register changed my mind. A dedicated service sees
every member's address pass through it — that's one more data processor to
name in the charter, with a clause on transfers outside the EU to get
right, which means a new charter version, which means everyone re-accepting
it. OVH already hosts our mailboxes, already appears in the legal notice,
and is French — no new declaration needed. The accepted trade-off: no
deliverability statistics. At our scale, that doesn't bite.

**SPF, DKIM, and DMARC** are in place. DMARC runs in monitoring mode: it
blocks nothing, it just reports who's sending mail as us. Enforcement comes
once the reports confirm everything we send is properly signed — tightening
before looking would risk our own mail landing in spam.

**Deployment,** on Vercel's free tier. Only one person can currently
deploy — the last remaining single-person dependency, and it's now logged
as one.

**A real bug, and why I looked in the wrong place first:**

Password reset was failing — "link expired or already used," less than a
minute after receiving the email. My first assumption was a code bug. The
logs settled it:

```
17:08:16   200   POST /auth/v1/verify
17:08:16   403   POST /auth/v1/verify
```

Two verifications in the same second. The first succeeds, the second
fails — meaning the link had been opened twice, though I'd only clicked it
once. The actual cause: corporate mail systems (Microsoft/Exchange and most
enterprise gateways) pre-fetch links inside emails to scan them before
display. Our tokens are single-use, so the scanner consumed it before I
ever clicked — and my morning test on Gmail had worked precisely because
Gmail doesn't do this.

**The fix: require a human action.** The confirmation link no longer
silently validates on page load — it's a page with a button. A link
scanner performs a plain GET and leaves nothing consumed; only an actual
form submission triggers validation. It costs one click, and it protects
exactly the members most likely to have a professional email address — the
working alumni the whole network exists for.

Added a "resend the link" button alongside it. An expired link used to be
a dead end — restart signup with an address that's already taken, which
just produces a second error. Cheap to fix, costly to leave broken.

**An hour lost, and the lesson that earns it:**

Fix written, tested, pushed — and the problem persisted. I re-read the
code, chased increasingly unlikely causes, requested more logs.

The fix had never actually deployed. Vercel was silently rejecting the
build for a reason with nothing to do with the code: my
`git config user.email` literally contained the placeholder string
`TON_EMAIL_GITHUB`, copied verbatim instead of filled in. Vercel blocks
commits whose author doesn't match a GitHub account, and kept serving the
previous version. The site looked fine, nothing appeared broken, and the
change simply didn't exist.

**Renamed the pôle mentorat to tutorat** — no, that was yesterday; today's
lesson is narrower and more useful: **when a change "doesn't take," the
first move is to verify it's actually live, not to re-read the code.** A
ten-second probe — open the confirmation URL with a fake token and check
whether the new page renders — would have replaced an hour of guessing.
And more generally: **a silent failure is the worst kind.** Nothing
crashed, no alert fired, the site stayed up. That's exactly what makes this
class of bug slow to find — there's nothing to see until you look in the
right place.

## What I decided (and why)

- **OVH for email sending, not a dedicated transactional service.** A
  specialized provider adds a new data processor that has to be named in
  the charter (with an EU-transfer clause to get right), which forces a new
  charter version and a full re-acceptance from every member. OVH already
  appears in the legal notice and already hosts the association's mail —
  reusing it avoids a new declaration entirely. Giving up deliverability
  stats is an acceptable trade at this scale.
- **DMARC starts in monitoring mode, not enforcement.** Blocking mail
  before seeing who's actually sending as us risks silently dropping our
  own legitimate mail. Reports first, tightening later.
- **Email confirmation requires an explicit click, not a page load.**
  Corporate link-scanners consuming single-use tokens on page load isn't a
  hypothetical edge case — it's the exact population (working alumni on
  professional email) the platform is built to reach. A button costs one
  click and fixes it for good.
- **When a fix doesn't take effect, verify deployment before re-reading
  code.** The hour lost wasn't a debugging failure so much as a diagnostic
  ordering failure — I read the code before I checked whether the code was
  even live. That order is now the default: probe first, then reason about
  the source.

## What's next

- Get the charter formally adopted by the bureau in a meeting, and record
  it — the platform enforces version-matching now, but the bureau itself
  hasn't ratified the current text as a group.
- Fix the mobile tab bar, which currently gets cut off at "Tutorat" — a tab
  nobody can see doesn't exist in practice.
- Two open questions that aren't technical but matter more: the code
  license granted to the association, and the filing in the register of
  agreements — both logged in the decisions journal for follow-up.
- The next step isn't development anymore: invite the first person who
  didn't build this platform.
# 2026-08-16 —
## What I did
I reused the logo and visual identity of our LinkedIn account (designed by Enzo) to improve the website's design.

We received a response to the email we sent to the group of teachers. Thanks to their feedback, we identified several inconsistencies between our objectives and what was written in the association's statutes, particularly regarding the membership fee. Initially, we wanted membership to be free, but we quickly realized that this would not be financially sustainable given the costs of the server, insurance, domain name, and events.

This email also highlighted one of our biggest mistakes: we had not yet contacted the headmaster of the high school. This should have been one of our first steps before moving forward with the project.

Unfortunately, I also reached my weekly usage limit on Claude.ai, which means I have to wait until Thursday before I can continue working on the website. This unexpected break made me realize how dependent I had become on AI tools. Perhaps I confused speed with haste. In my attempt to move as quickly as possible, I did not always take the time to fully understand every tool I was using. Moreover, most of my session reports were written by Claude simply because I lacked the time to write them myself.

As a result, I plan to revisit these reports and make them more personal. I have not yet decided whether I will rewrite them completely or simply edit them while leaving some trace of the original AI-assisted version.

## What remains to be done
Reply to Thamma's email.
Amend the association's statutes.
Inform the school administration, especially the headmaster, about the project.

# 2026-08-20 — Statutes, dues, and a platform that stops guessing

Long day. A bureau meeting that ran until midnight, and a full working session
on the platform before it. Almost everything I fixed today came from someone
else using the app, not from my own testing — which is the point I want to
remember from this session.

## What I did

**Association — the meeting.** Two hours on the statutes, the dues, the
relationship with the lycée, and the pôles. I recorded it, the recording cut
halfway through, so the minutes exist in two parts and I've published both.

Decisions worth keeping: **dues at €10 a year for everyone**, tutoring stays
collective (webinars and small groups) rather than paid individual lessons,
and tutorat and mentorat become two separate pôles instead of one blurred
thing. Also a hard admission: we've been using the lycée's name, its façade
and its visual identity for weeks **without a written agreement**. That has to
be fixed before the pre-rentrée communication goes out, not after.

**Statutes, version 7.** Article 6 rewritten — €10 for every category, no
exemption, no differentiated scale. That change alone contradicted three other
articles, which is what took the time: article 5 still said the annual
confirmation was free, article 8 still made non-payment a ground for losing
membership, article 9 still described dues as hypothetical.

I also added an **article 11 bis on bank accounts**: three signatories
(president, treasurer, one designated vice-president), read-only access for
the whole bureau, credentials never shared, and access withdrawn within
fifteen days of leaving office. The last two are the ones that actually matter
in a student association where the bureau turns over every year.

And I corrected article 7, which claimed memberships are accepted
automatically on signup. They aren't — the app holds every account until the
bureau validates it. The statutes described a platform that doesn't exist.

**Platform.** A long list, almost all of it reported by Romain and Enzo
testing the site rather than found by me:

- The charter's § 10 announced "tick the box below" followed by a `☐`
  character typed into the Markdown. There was no box. Romain tried to click
  it and asked whether he was allowed to accept. The section now describes the
  real mechanism, and the page states where the reader stands — version
  accepted and date, or what's missing.
- Profile photos can be cropped and zoomed, and re-cropped later without
  re-uploading the file. That meant storing the original alongside the square,
  ~150 kB more per member.
- Job, company, and role-in-the-association added to profiles. The sector list
  was a business-school list — finance, consulting, audit — so a parent who is
  an engineer or a nurse had nothing but "Other". Fourteen sectors added.
- The profile form now hides what doesn't concern you: no employer field for a
  student still in prépa, no filière fields for a parent.
- "I can help with" became "you can ask me about", opened beyond prépa
  subjects to fifteen professional and orientation topics — and opened to
  parents, who were excluded from it entirely.
- A notification bell, an unread counter on the feed tab, and a banner for
  incomplete profiles.
- Links in free text are now clickable. Someone posted a real internship offer
  ending in "to apply: https://…" and the address was plain black text.

**A genuine bug I introduced.** Removing a profile photo failed with *Direct
deletion from storage tables is not allowed*. I had given the cleanup job to a
database trigger doing `delete from storage.objects`; Supabase forbids that
now, and the failed trigger rolled back the whole transaction. I'd made the
same bet on CV files five days earlier — that one had never run, so nobody had
seen it fail.

**And I deleted my own account by mistake**, from the Supabase dashboard,
confusing it with a test account. Locked out, no password, no reset email —
because there was no account left to send one to. The screen inside the app
makes you retype your email address before deleting. The dashboard doesn't ask
anything.

## What I decided (and why)

- **Non-payment suspends the vote, not membership.** Radiating a
  préparationnaire over €10 costs the association more than it collects, and
  article 2 promises access "without distinction". Voting rights and quorum
  counting are suspended until payment; the annuaire, the feed and the events
  stay open. Article 8's clause disappeared accordingly.
- **A text describing a screen must describe the screen that exists.** Both
  the charter's phantom checkbox and article 7's automatic admission came from
  the same habit: writing the document from a model rather than from the
  product. Both are corrected, and the rule I'm keeping is to re-read any such
  text in front of the screen it describes.
- **No notifications table.** The obvious build is one row per member per
  event, with read/unread state and a purge to write — two hundred rows for a
  single post. Everything the bell announces is already derivable: one column
  holding the last visit to the feed, compared to the date of the last post.
  What I give up is history: a notification disappears when the thing is done,
  not when it's been seen.
- **Keep the original photo, or "re-crop" is a lie.** Re-cropping the stored
  512 px square would only ever tighten the frame and degrade the image each
  time. Storing the full photo at 1024 px is the price of a crop you can
  actually revisit.
- **The emblem is not the favicon.** Enzo's crest is line art — at the 16
  pixels a browser tab actually renders, six floors of windows become a gold
  smudge. The `.ico` now holds eight images: the real crest from 32 px up
  (bookmarks, pinned tabs, search results), a simplified façade below. Nobody
  ever sees two sizes side by side.
- **Storage cleanup moves to the client, tracking stays in the database.** The
  file is deleted through the storage API from the screen that owns it; the
  trigger no longer deletes, it records what lost its row in a
  `fichiers_orphelins` table. The screen can be closed mid-action — the
  database can't forget.
- **Stop showing controls that command nothing.** Founders and
  super-admins publish and validate regardless of the two permission
  checkboxes shown next to their names. The checkboxes were decorative, and
  disabled on top of that. They're gone for those roles — they stay where they
  mean something, which is the teacher or the future maintainer given
  validation rights without publishing rights.

## What's next

- **Before opening to people outside the bureau:** a captcha on signup, the
  orphan-file cleanup job, the GDPR processing register (mandatory, and it
  doesn't exist), and the data-processing agreements to accept with Supabase
  and Vercel on behalf of the association.
- **This week, regardless:** rotate the SMTP password (it appeared in a
  screenshot days ago and I still haven't done it), start dumping the database
  before every migration — the free tier has no automatic backups — and add a
  daily ping so the project isn't paused after seven idle days.
- **Association:** get statutes v7 adopted in an extraordinary general
  meeting, file them with the prefecture, and write to the headmaster before
  any further use of the lycée's name.
- Two questions still open for the bureau: whether the pôle Association needs
  a secretary general in the list of functions, and whether to allow an
  individual dues waiver for a member without means.

---

*Note on the date: this session ran into the night of 20 August and the
meeting ended around midnight. Published the following day, dated to the day
of the work.*

# 2026-08-22 — Reaching out before the money works

Almost no code today. The session was about who we tell, when, and what we
can't promise yet — nine days before the rentrée, with a bank account that
doesn't exist and a platform that can't take a euro.

## What I did

**Wrote to an HGG teacher.** A personal email, not a press release: how the
idea came out of the oral exams, what the association is now (declared with
the prefecture, a bureau of seven, five pôles), and one concrete reason for a
teacher to care — a student in prépa can be followed by an alumnus who sat the
same orals two years earlier. Alexandre validated it: *"we're only letting her
know, we're not asking her to do anything."* That last point is the whole
register of the message.

Two things I fixed in my own draft, and they're the same mistake twice. I had
announced "three things" and then listed three abstractions — *keep the link,
give access, open contacts* — which forces the reader to count and gives them
nothing to picture. And I had written "this is what concerns you most
directly", which tells a teacher what concerns her. Concrete complements do
the work on their own: *for the orals, a school, an internship.*

**A network problem that isn't ours.** Alexandre couldn't reach the site from
his work laptop; his personal machine works, and everyone else got through.
Corporate firewall, almost certainly. Worth recording because the first
instinct was to look at the deployment — and the deployment was fine.

**The bureau's evening, on WhatsApp.** Most of the substance of this entry
comes from there rather than from the keyboard.

- **Pre-rentrée event fixed for 29 August**, with 19 alumni expected.
- **Bank account.** Lucas went through the Crédit Mutuel terms properly and
  wrote to his adviser: the association's statutory object in full, then two
  precise questions — which tariff bracket we actually fall into (movement
  commissions? is remote banking included? what does a year cost, all in?),
  and whether the account must be attached to the branch nearest the
  registered office or can sit in Saint-Maur or Ormesson. The "clarity
  agreement" banks publish is abstract enough that asking for a number is the
  only way to get one.
- **Tutors and mentors have heard nothing since late July**, nine days out.
  Rayou raised it, and he was right to.
- **The presentation deck**, iterated all afternoon: text too small, the
  opening framing too academic to speak to parents, a placeholder building
  instead of our emblem, the bureau reduced to the seven founders, and a
  "free" mention at the top of a deck that announces €10 at the bottom.

**Effort spent twice.** Rayane was already rewriting the deck when I launched
my own version through Claude — *"don't burn tokens, I'm on it"*. Two people
redesigning the same document in parallel for twenty minutes, because neither
said what he was doing before starting. The merged result is better than
either draft, which is not an argument for the method.

I've deliberately left out the identification banter from the start of the
thread. Same rule as the meeting minutes: what concerns people rather than the
association doesn't go into a document that's kept.

## What I decided (and why)

- **Announce the tutors' charter now, and say plainly that remuneration is
  being explored.** The instinct was to wait for an answer on funding before
  writing to the tutors, so as not to promise what we can't pay. Rayou's
  objection settled it: an answer isn't expected before mid-September, by
  which point the tutors will have their own term to think about and will be
  gone. Either we play sincerity — there may be remuneration, it isn't settled
  — or we announce nothing at all. Silence for six more weeks isn't the
  cautious option, it's the one that loses the people.
- **Ask the teachers to relay the message, in reply to all, rather than
  texting one of them privately.** We have a phone number and it would have
  been faster. But the other teachers were in copy on the original mail:
  answering there means everyone has the same information, and no one is put
  in the position of a private go-between. We ask nothing of anyone — that's
  what makes the request acceptable.
- **Explain the project to the first-years face to face; send the deck
  afterwards.** A first-year opening a full presentation before anyone has
  spoken to them reads a wall of information about a structure they've never
  heard of. The deck is good — it's the second step, not the first.
- **Never ask the lycée for the list of students.** Standing rule, restated
  because the question came up again in a practical form: *can she get us the
  list?* She can't give it, and we can't use it. Each person signs up
  themselves; that's already the architecture of the platform.
- **Don't open the platform beyond the bureau until payment works.** Members
  registered before dues exist are members you then have to go back and charge
  — the most thankless message an association can send. The chain is
  bank account → HelloAsso → dues → opening, and it can't be reordered.
  The margin is under a month and a half, and it's shrinking.
- **The branch is chosen for the adviser, not the calendar.** Three days
  between the 28th and the 1st against an adviser in Saint-Maur who knows the
  local firms, and possibly Berthelot itself. We're not in a three-day hurry.
- **The €10 goes on a slide of the deck, and "free" comes off the header.**
  Announcing the amount up front costs nothing and buys the right to be
  believed on the rest. A deck that says "free" at the top and €10 at the
  bottom doesn't look cheap, it looks careless.
- **Our own emblem, and no mention of the private LinkedIn group.** The deck
  carries LinkedIn, Facebook, Instagram and the platform link — four ways in.
  The private group is announced inside the platform's feed, where the people
  it concerns already are. A door for members doesn't belong on the poster.

## What's next

- **Rotate the SMTP password.** Third entry in a row it appears, still not
  done. It's ten minutes, and it must be updated on the Supabase side at the
  same time or signup mails stop leaving.
- **Apply the three fixes to the minutes automation** — the FastAPI `/pdf`
  route, the Code node that treats a string as an object, the prompt that
  sorts before it writes. Written a day ago, still not applied.
- **Send the reply to the teachers**, with the template and a short generic
  message for the incoming first-years, and write the mail on tutor
  remuneration.
- **Everyone re-reads statutes v7 for typos**, then convene the extraordinary
  general meeting. Until it's held, v6 is the text that governs.
- **Event on 29 August**, 19 alumni.
- **Bank:** wait for the adviser's answer; appointments can be booked from
  September. HelloAsso follows the account, not the other way round.
- **The platform has still never been opened on a phone.** Fifteen screens,
  and the first outside report we got this week was a network refusing to
  reach it.


# 2026-08-23 — Shipped, pushed, and still not live

A long day, and a theme I didn't choose: almost everything that went wrong was
something that looked finished and wasn't. A banner that existed but was
hidden. A link that was displayed but not clickable. A backup command that ran
and saved nothing. A migration pushed to the database while the code that used
it sat uncommitted on my disk. And a sentence sent to eight people with two
paragraphs collided in the middle of it.

## What I did

### The meeting minutes — rewritten, then made to run

**The old prompt was three lines: "summarize this meeting in JSON".** The
output was soft because the instruction was. A transcript isn't a text to
summarize, it's a recording of a conversation: people talk about the
association, and also about a shoulder, the price of Macs, and who should have
been president. A model told to *summarize* treats all of it equally and
returns minutes where "Yanis has recovered his shoulder mobility" sits next to
"the lycée is paying €1,000".

The new prompt sorts first and writes second: keep only what commits the
association, qualify each item into exactly one category (decision, task,
lead, unresolved, point raised), then write one self-contained sentence for a
reader who wasn't there. What must never appear is listed by name — health,
private life, family, housing, personal finances, personal plans, judgments
about people, off-topic conversation — and is ignored *in silence*: not
summarized, not flagged, not recorded as having taken place. Figures are
quoted with their unit and period. People are named to carry a task, never an
opinion. No section has a minimum: an empty list is a correct answer.

I also wrote out, in advance, what the prompt *should* produce for the
21 August meeting. That's the only test worth having.

**Then the flow itself**, and four settings the documentation treated as
given, none of which were: `temperature` is refused outright by the model
(`400 — Unsupported parameter`); `HTTP Request1` was sending no body at all
because `specifyBody` fell back to its `keypair` default; the disk-writing
node didn't exist in either flow; and n8n now refuses to write outside
`~/.n8n-files` unless `N8N_RESTRICT_FILE_ACCESS_TO` says otherwise. The Docker
volume mount is the one thing still open.

### The platform

**The banner was invisible on phones — on purpose.** It was hidden below 640
pixels, with the reasoning written in the file: stretched across a phone, a
5.9:1 image drops to 63 pixels tall and the signature becomes unreadable. The
reasoning was right and the conclusion was wrong. Treating a display defect
with an absence removes the identity exactly where most members open the site.

My first fix imposed a height of 96 pixels. I simulated it before deploying,
and that saved me: at 320 pixels wide the wordmark touched the edge. A height
in pixels crops *more* the narrower the screen — the opposite of what's
needed. The shipped version imposes a **ratio**, 4:1, so 68 % of the width
survives at every size and the framing is identical on an iPhone SE and a Pro
Max.

**Caen didn't exist.** The city list held fourteen French towns, chosen by hand
one evening of initial schema. A field that refuses a true answer is a field
that lies, and the member then has to lie too.

Two steps, because I got the first one half right. First, opening the list for
France with an explicit "Add" button, and a normalised key in the database —
lowercase, unaccented, everything else reduced to hyphens — so "caen", "CAEN"
and "Caen" return the same id, with the unique index on the key rather than
the spelling. Then, when Yanis said he wanted to *type* Démouville and find
it, not add it: the 34 969 communes of France, from the État's own dataset,
loaded into a separate reference table and searched server-side.

The suggestions are ranked by population, which is half the work: "paris"
returns Paris before Parisot, "limoge" returns Limoges before Limoges-Fourches.
The department is displayed because 2 234 communes share a name with another —
fourteen Sainte-Marie.

### The association

**Sent the reply to the teachers**, with all of them and Romain in copy this
time. It answers the four points raised: the contradiction between the dues
announced in the mail and article 6 of the statutes; the fact that the
prefecture filing happened before contacting the lycée's management, which
article 1 didn't foresee in that order; the platform being online but not open;
and the lack of women on the bureau, answered with the five names in the events
pôle and with the admission that only the first elections will fix it.

**Three defects went out with it, and they're worth recording.** A sentence
stops mid-word — *"Pour les futurs premières années, nousPour le groupe
WhatsApp"* — two paragraphs collided and nobody re-read the whole thing before
sending. A placeholder I had left in a draft, `[adresse de l'association]`,
was sent as-is: eight people are now invited to write to a pair of brackets.
And the dues announced are **€10 for members in prépa or school, €35 for
everyone else, per household** — a scale that is in no statute, that the
bureau voted differently on 20 August, and that contradicts the correction the
same mail was written to make.

**Built the pre-rentrée invitation as a PDF**, from Yanis's text unchanged:
banner letterhead, the practical details in a gold-bordered box, marine footer.
The links weren't clickable in the first version — displayed but dead. Fixed,
and verified in the file itself rather than by eye: two real link annotations.
The Instagram address also carried an `?igsi=…` share token, which is what
broke it.

**And the invitation says "vendredi 29 août".** 29 August 2026 is a Saturday.

### Three things I got wrong

**The backup didn't run, and I caused it.** I handed over a command with a
`#` comment appended; the shell passed it as arguments, `db dump` failed, and
`db push` ran a minute later against a database with no backup. The migration
applied cleanly, so nothing was lost — but the guard didn't guard. A command
meant to be pasted carries no commentary.

**A migration pushed while its code stayed uncommitted.** The database knew
about `chercher_communes` for an hour while the site served a build that had
never heard of it. My instructions listed the database commands and stopped
there. Deploying is not one command, it's the shortest complete list.

**Effort spent twice on the deck.** Rayou was already rewriting it when I
launched my own version. Twenty minutes of two people redesigning the same
document because neither said what he was doing first.

## What I decided (and why)

- **Minutes sort before they write, and what must never appear is ignored in
  silence.** A line saying "a personal matter was also discussed" is itself the
  disclosure, in a document that goes to seven people and is kept. Health data
  is sensitive under article 9 of the GDPR: the absence has to be total,
  including the absence of a mention of the absence.
- **A ratio, not a fixed height.** A height in pixels crops more the narrower
  the screen, which is backwards. A ratio keeps the same framing everywhere and
  only changes the size. The general form: when a constraint must hold across
  unknown screens, express it as a proportion, not a measurement.
- **Two tables for cities, not one.** `communes` is a reference — the État's
  list, nothing points at it, it exists to propose. `villes` stays the list of
  places where someone actually lives, foreign cities included, and it's what
  profiles reference. Merging them would have meant changing the type of
  `villes.id` (a `smallint`, which stops at 32 767), then of
  `profiles.ville_id`, then dismantling and rebuilding the views that depend on
  it — on a database with no automatic backup, for no gain.
- **The directory proposes only inhabited cities.** I argued for this, Yanis
  pushed back twice, and testing settled it: a filter offering 34 969 cities of
  which 34 900 return nobody makes you click for nothing. Typing a city works
  where it matters — in the profile.
- **The key decides identity, not the spelling.** `caen`, `CAEN` and `Caen` are
  one city because the unique index is on the normalised key. Without it,
  opening the field would have produced three Lyons and an unfilterable
  directory.
- **Announce the tutors' charter now, saying plainly that remuneration is being
  explored.** No answer on funding is expected before mid-September, by which
  point the tutors will have their own term to think about. Six more weeks of
  silence isn't the cautious option, it's the one that loses the people.
- **Ask the teachers to relay, in reply to all.** A private message to one of
  them would have been faster, but the others were in copy on the original
  mail: answering there means everyone has the same information and no one
  becomes a private go-between.
- **Never ask the lycée for the list of students.** Restated because it came
  back in practical form. She can't give it and we can't use it. Each person
  signs up themselves — that's already the architecture of the platform, and
  it's the answer to her worry about passing on parents' addresses.
- **Don't open the platform beyond the bureau until payment works.** Members
  registered before dues exist are members you then have to go back and charge.
  The chain is bank account → HelloAsso → dues → opening, and it can't be
  reordered.
- **The link text is the address.** The invitation displays exactly what it
  points to. Showing one address and targeting another is the mechanism of
  phishing, even when it's accidental — the same rule already applied to free
  text inside the platform.

## What's next

- **Rotate the SMTP password.** Fourth entry in a row. It also has to be updated
  on the Supabase side, or signup mails stop leaving.
- **Correct the invitation**: the 29th is a Saturday, and "time: to be
  confirmed" is the only information that decides whether anyone comes.
- **Write to the teachers again** with the association's actual address, the
  unfinished sentence completed, and — before anything else — a dues figure
  that matches the statutes. Right now three different scales are circulating:
  what the bureau voted, what the statutes say, and what the lycée has been
  told.
- **Finish the minutes flow**: mount the output folder in `docker-compose.yml`,
  then run it end to end on an old recording.
- **Statutes v7**: everyone re-reads for typos, then convene the extraordinary
  general meeting. Until it's held, v6 governs.
- **Event on 29 August**, 19 alumni. Contact M. Bolloré, submit the draft
  agreement, open the bank account.
- **The platform on a phone**: the home page is verified, the fourteen other
  screens are not. The directory and its filters, the profile form, the feed
  with its polls — in that order.

# 2026-08-25 — Guards that guarded nothing

Two days on the WhatsApp secretary, and a theme that arrived uninvited: almost
every safety mechanism I checked this week turned out to protect nothing. A
backup command that saved no data. Row-level security on a path where the
writer bypasses it. An anti-duplicate index that catches a case that won't
happen. A shared secret guarding a URL nobody outside this laptop can reach.
None of them were wrong. All of them were somewhere else than where I thought.

## What I did

### The database

**Reviewed brique 1 before pushing it, and corrected five things.** The
migration Claude Code delivered was good — French comments explaining *why*,
RLS on all four tables, a lock trigger on tasks. But
`destinataire_par_defaut('evenementiel')` returned `null`, documented as
expected behaviour because the pôle's SG post was vacant. That's not a
behaviour, it's a hole: a task nobody takes, in a pôle with no holder, alerting
nobody — exactly the task this system exists to catch. The président is now the
fallback and effaces itself as soon as the post is filled.

Also: `titre` could be empty, so a model returning `{"titre": ""}` created an
invisible task. And the immutability of `collectes` and `decisions` rested on
the absence of an `update` policy — which protects the application, and the
application writes nothing here. n8n writes, with `service_role`, which sees no
policy at all. Two `refuser_reecriture` triggers make the word "archive" true.

**Then the thing that mattered most: the backups weren't backups.**
`supabase db dump --linked` writes the **schema**, not the data. I opened the
file: 69 KB, sixteen `CREATE TABLE`, zero `COPY`, zero `INSERT`. Three days of
backups were empty skeletons. Worse, the rule that produced them is one I wrote
into the project's own memory files, so it had been giving false comfort since
the 23rd. A backup is now two files, one with `--data-only`. The first real one
weighs 2 MB.

**Aliases.** The model returns a WhatsApp display name; the database wants a
UUID. Nothing connected the two, and the real names in the conversation are
"Rayou" for Rayane, "~ Lucas" with the tilde WhatsApp adds for numbers absent
from your contacts, "Romain Very (Essec)", "Elyes Bahnis(hec)". None of them
match `prenom` or `nom`. Without the mapping, `responsable_id` stays null for
everyone, every task falls to the group's default recipient, and the nominative
email — the only output that counts — never happens.

The table keys on the display name, not the person: one account, several
aliases, and the unique index is on a normalised key so that a stray space or
capital doesn't create a second alias. The tilde falls out with the
normalisation, which makes the mapping immune to the one change we're certain
to see.

**And the routing was broken in production without anyone noticing.** Romain
had no `fonction` in the database. `destinataire_par_defaut` looks the président
up by function, so `bureau` and `general` both resolved to nobody — and
`est_bureau()` requires a non-null function, so the président couldn't read the
secretariat either. One `update` fixed it. It had been true since the tables
were created.

### The prompt

**Brique 2, tested twice on the real conversation of 24 August**, with the
expected output written before either run.

What holds: the five expected tasks with the right owners, the statutes coming
out as a single `creer` already `terminee`, an ambiguous "Jle fais" correctly
leaving the SIREN task unassigned, "the week of 14 September" giving
`echeance: null` rather than an invented Monday, and the negative check
perfect — no trace of Elyes's move, of a phone number, of an off-topic link.

What failed, both times, in the same place: **decisions.** Run A recorded an
event cancellation that was reopened ten minutes later. I rewrote the rule with
a procedure — re-read what follows the moment a decision appears settled — and
run B stopped producing that one, but still recorded a teacher's suggestion
repeated by one member, and invented a new one from a statement of fact.

### The workflow

**Brique 3 arrived written but never executed.** Reading it found three
defects: a node deleted from the canvas while the code consuming it still
called it by name, four edges converging on one input so the summary would run
once per branch, and a Postgres array parameter built with `JSON.stringify`,
which produces `["a","b"]` where Postgres wants `{a,b}` — verified by running
it and getting `malformed array literal`.

**Running it found four more**, none of which an import can catch: the Form
Trigger pinned at `typeVersion: 1`, a version so old it ignores `fieldName`
(so every downstream `$json.groupe` read `undefined`) and predates the Form
Ending mechanism entirely; `responseMode: responseNode`, which makes n8n demand
a literal Respond to Webhook node and refuse to listen without one; the four
completion screens silently emptied because their title and message had been
written inside `options` instead of at the top level; and `$env` access blocked
by default in n8n 2.32, which I had asserted was open.

**And a hole I opened myself, twice.** `service_role` carries `BYPASSRLS`, but
bypassing row-level security grants no SQL privilege: `collectes` and `taches`
had no `GRANT` for it, nor `EXECUTE` on `profil_par_alias()`. Nobody had
noticed because nothing had ever written with that key. Claude Code found it by
running the queries, not by reading the schema.

### Three things that were said out loud and shouldn't have been

The n8n encryption key and the database password went through a screenshot and
a paste — the key that decrypts every credential stored in that instance, and
the password that opens the association's database as owner. The SMTP password
of `bonjour@`, fifth journal entry in a row, is still not rotated.

## What I decided (and why)

- **The model does not record decisions.** Three successive rules failed, each
  differently, and the third produced an error the first two hadn't. Telling
  "the bureau settled it" from "somebody said yes" requires knowing who binds
  the association, which a WhatsApp transcript never says. And the asymmetry
  runs the wrong way: tasks are correctable, `decisions` is an immutable
  archive. What gets extracted automatically is what can be taken back.
- **`termine_le` comes from the collecte's period, never `now()`, never a date
  read from the text.** Accurate to the day, which is enough for minutes, and
  never wrong — the model has no closure date in its schema and so cannot
  invent one. Later, the screen where a human ticks "done" will write `now()`,
  exact to the minute. Each writer knows its own truth; no trigger arbitrates
  between them.
- **No weekly reminder email.** A recurring "come look at the Drive" ends up in
  a filter rule. The document link is pinned in each WhatsApp group's
  description instead — no code, no state to keep, and it sits where they
  already are every day. The mail that remains is rare and event-driven: one
  grouped notification per person per day, then J+3 and J+7 if a task drags.
- **The aliases are not versioned.** Twelve people's display names against
  their account ids is personal data; in a git repository it is irreversible.
  They go into the SQL editor by hand. A local `db reset` won't have them, and
  for twelve rows that's the right price.
- **The pôle members get no `fonction`.** In this schema `fonction` means
  "bureau member", and `est_bureau()` keys on it — giving it to the five
  événementiel members would open the bureau's own collectes to them. Their
  reading surface is the Drive document for their pôle. If group membership
  ever needs recording, it's a `membres_pole` table, not a hijacked column.
- **Sonnet 5, because it's the only model with evidence behind it.** Cost
  doesn't decide anything here — about ninety calls a month puts every tier
  between one and eight dollars. What decides is obedience to negative
  instructions, and the only measurements we have were taken against Claude.
  Running a production system on a model no test has ever seen is the actual
  risk.
- **The Postgres credential connects as the database owner, and the
  `service_role` grants are insurance for a path we don't use yet.** The
  Postgres node speaks to Postgres directly, not through PostgREST, so it
  authenticates with a username and password, not with the service key. The
  grants migration is correct and will matter for brique 5 or any REST access —
  but it is not what makes this workflow work, and the credential n8n now holds
  can read and destroy everything.

## What's next

- **Commit what's on disk.** Six files are untracked or modified, including
  `20260824130000_droits_du_service_role.sql`, which has been applied locally
  by `db reset` and **never pushed to production**. The database and the
  repository have drifted apart in both directions.
- **Re-export the workflow.** The `workflow-collecte.json` in the repository is
  the first version — the one with the three defects found by reading. Every
  correction since lives only inside n8n.
- **Two workflows now carry the same name**, one of them broken, because the
  import created a new one instead of replacing. Archive the old.
- **Rotate the SMTP password, and now the n8n encryption key too.**
- **The form only exists on `localhost`.** Nobody but me can deposit anything,
  which contradicts the one line of the specification that makes this survive
  my absence: one installation, several depositors. Tunnel or VPS, to be
  settled before this is announced to anyone.
- **Lucas isn't registered on the platform.** The treasurer, on the association's
  own platform, at the moment we're asking him to open a bank account and track
  dues. The five événementiel members aren't either.
- **Brique 4 and 5**: the Drive documents, one per group, then the mail flow.


