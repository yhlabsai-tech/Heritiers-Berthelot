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

*What this gives the association: [à valider avec toi] the platform's core
loop — sign up, get listed, post, poll — is provably real now instead of
planned, and the association has a public face for the first time, which
means the alumni-outreach ask sent to teachers this week finally has
somewhere to point people.*

