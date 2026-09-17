# Logbook

This logbook records the main decisions, difficulties and lessons from building the alumni association and its platform.

Entries are organised chronologically. They focus on what changed, why I made certain choices, and what I learned from the process.

# 2026-07-28 — Creating the GitHub Repository

## The session

I created the structure of this logbook to document the construction of the alumni association and the platform around it.

The goal is not only to keep a record of what I build, but also to make the evolution of the project visible over time — including the decisions, mistakes and changes in direction that come with it.

# 2026-07-29 — Building a self-hosted meeting-minutes pipeline

## The session

I built the first version of a local meeting-minutes pipeline on my MacBook using n8n, PostgreSQL, Qdrant, Ollama and WhisperX. The whole stack runs locally, which keeps the prototype free and avoids sending meeting recordings to external services.

The first end-to-end test already worked: a recorded meeting could be transcribed with speaker labels. I deliberately started with local models, but the first tests showed an important limitation: a small local model is not reliable enough on long meetings, especially when it has to keep track of decisions and action items.

I also chose to record audio at the operating-system level rather than depend on the API of a specific meeting platform. This keeps the system usable for online meetings, WhatsApp calls and in-person meetings.

## Lessons learned

The first useful lesson was that the architecture should stay simple. I do not want to add automation just because I can. For example, if a recording is saved in the wrong folder, changing the recording application's output directory is preferable to adding another node whose only job is to move the file afterwards.

I also learned early that testing the limits of a model is more useful than assuming they are acceptable. A working demo is not necessarily a usable product.

# 2026-07-30 — Finishing the meeting-minutes pipeline

## The session

I finished the first end-to-end version of the meeting-minutes pipeline. A real recording could now be transcribed, analysed and turned into a formatted email automatically.

The main architectural change was separating analysis from formatting. Initially, I asked the LLM to analyse the transcript and generate the HTML email at the same time. On long inputs, it sometimes returned a polished but empty template. I therefore made the model return structured JSON only, and moved the HTML generation into a deterministic code step.

I also stopped allowing the model to generate factual information such as dates or imaginary tasks. It had already invented dates and even returned raw PHP in one output. Dates and other factual values are now injected by the workflow itself.

A deliberately long test confirmed another limitation: a single-pass summary is not reliable for very long meetings. A chunking approach would be necessary, but I chose not to build it yet because real meetings are currently much shorter.

At the same time, I completed the first major administrative work for the association and chose to become Secretary General of the application division rather than president. I wanted to stay close to the technical side and to the platform itself.

## Lessons learned

The most important principle from this session was simple: **the model should do the work that requires interpretation, while code should handle deterministic facts and formatting**.

I also started to understand that good engineering often means refusing to build something before it is necessary. Knowing that a system may eventually need map-reduce summarisation does not mean I should add it before the real use case requires it.

# 2026-08-01 — Rebuilding the automation stack on Windows

## The session

I rebuilt the automation environment from scratch on a Windows laptop using WSL2, Docker, n8n and PostgreSQL. I also connected Claude Code to n8n through MCP and successfully built a small test workflow through a natural-language prompt.

The important change was moving from a purely visual workflow approach towards treating automations as code. Workflow exports are versioned in Git so that I can keep history, inspect changes and rebuild the environment if necessary.

I deliberately chose Docker and PostgreSQL instead of the quickest possible installation. The goal was to create an environment that could support real projects rather than a disposable prototype.

## Lessons learned

I started to treat security and reproducibility as part of the architecture rather than as cleanup work. Secrets stay out of Git, workflows should be exportable, and the environment should be reversible.

I also made a small but useful mistake: I installed Node.js because I thought Claude Code required it. That was wrong. Node was useful for `npx` and `n8n-mcp`, but not for Claude Code itself. The lesson was less about Node than about my process: **check the documentation before building an explanation around an assumption**.

# 2026-08-02 — 2026-08-08 — Moving the stack and sending the founding documents

## The session

The association's founding documents were finalised and officially sent during the week. At the same time, I rebuilt the automation environment on my own MacBook so that the project no longer depended on another person's computer.

I also stoped the diarisation on whisper. Before that an audio of 20 minutes was transcript in 28 minutes, way too long. Now on my computer it is around 4 minutes.

The migration itself was more complicated than expected. The n8n data volume and the `.env` file contained different encryption keys, so n8n refused to start. I also had to reset the owner account and deal with an SSRF restriction in `n8n-mcp` before the tools worked correctly.

I chose to align the `.env` with the existing volume rather than wipe everything and start again. That preserved the workflows and credentials already stored in the system.

I also moved the MCP configuration to project scope and chose VS Code as the main interface for the stack because the work is heavily based on files, Git and versioned configuration.

## Lessons learned

The strongest lesson was about reversibility. When a system contains data that matters, the easiest fix is not always the safest one. Wiping a volume might have solved the immediate startup problem while creating a much worse credential-recovery problem later.

I also adopted a broader rule: **when working with AI tools, a technical restriction is often better than a written instruction**. If I do not want an agent to delete workflows, disabling the destructive tool is more reliable than asking it not to do so.

# 2026-08-09 — First event planning meeting and a useful debugging lesson

## The session

The association's first event started taking shape. We organised the responsibilities, prepared the planning document and divided the remaining work between the team.

On the technical side, WhisperX had stopped responding. A previous troubleshooting session had looked in the wrong Conda directories and installed another Miniconda environment, which only made the situation harder to understand. Once I checked the actual environment, the original WhisperX installation was still intact.

I also rewrote the meeting-minutes structuring prompt, separating the fixed instructions from the transcript input and adding fields for unresolved points and possible follow-ups. The new version handled real meeting content more accurately and was better at distinguishing a personal suggestion from an actual group decision.

## Lessons learned

The main lesson was methodological: **check the real state of the system before trusting a diagnosis, including a diagnosis produced by an AI tool**.

I also kept the architecture simple. New information was added to the existing formatting step rather than introducing another processing layer. The earlier separation between analysis and formatting made that possible.

# 2026-08-11 — Governance and the first real database

## The session

The association's bureau structure was formalised, and I continued to take responsibility for the application division.

On the platform, Phase 0 became real. I created the Supabase project, the first schema, row-level security policies, storage and the TypeScript types generated from the live database.

The migration process was not clean. I accidentally pushed an empty migration, had to repair the migration history, and hit a SQL escaping issue. All three problems were resolved and the repository now contains the database migrations and generated types rather than relying only on the remote database.

A major product decision was also made: registration would remain open in the first version while the validation mechanism stayed available in the schema. I separated study progression from academic track because they represent different concepts. I also decided not to let an AI agent autonomously write RLS policies until I could understand and review them myself.

## Lessons learned

Security-sensitive code is not the place to outsource understanding. A bad UI can be fixed visibly; a bad RLS policy can silently expose or block data.

I also learned that a database schema should describe the concepts of the product rather than trying to compress several meanings into a single field just because it is convenient in the beginning.

# 2026-08-13 — Cutting the scope of v1

## The session

I spent the session reducing the platform's first version to something that could actually be built and used.

The original feature list included messaging, profiles, search, parents and a feed. After comparing those ideas with the needs of early users, I kept the central poll for oral-exam availability and replaced full messaging with a simpler request-to-connect flow. Several profile fields were also changed from open text to structured data.

This exercise also clarified the architecture. Identity has to come first: the feed depends on profiles and roles, while the directory can exist independently. I therefore stopped thinking about identity, feed and directory as three parallel features.

The authentication flow exposed an ambiguity in Supabase's email confirmation process. Instead of assuming one URL parameter was correct, I built the route to accept the two forms involved and decided to use a real signup test to determine which one was actually being sent.

At the same time, I sent the first formal email to the teaching staff about the association and the platform.

## Lessons learned

The main lesson was product discipline: **a feature is only valuable if there is a real problem and a real reason to build it now**.

The session also reinforced an important privacy principle. The existing alumni spreadsheet was created for another purpose. Reusing it inside the platform is not a neutral technical import; it changes how the data is processed and therefore has to be treated accordingly.

# 2026-08-14 — The beta becomes functional

## The session

I completed the main feed and poll functionality of the platform. Building it exposed several problems in the original schema.

The most important was the distinction between identity and directory membership. Posts were linked to the directory, even though authorship should belong to a person's account. The same problem affected poll votes. I changed both to use the profile layer.

I also found that poll closing dates existed in the database but were not actually enforced. A date that exists in a column but has no effect is misleading, so the closing rule was integrated into the policies.

The platform became functionally complete for the initial beta: identity, directory, administration, feed and polls were working together.

Outside the platform, the association launched its LinkedIn, Instagram and Facebook presence. I wrote the first public description and worked with the team on the visual identity and messaging. We also made the ECG-only scope explicit instead of leaving people to guess who the association was for.

## Lessons learned

Building one feature often reveals problems that were already present somewhere else. The feed did not merely add functionality; it exposed weaknesses in the identity model and in the way permissions were enforced.

The broader lesson was that **a feature is only finished when the rules around it are enforced**, not when the database contains the right columns or the interface looks correct.

# 2026-08-16 — Going live, and real-world failures

## The session

The platform went live at `https://lesheritiersdeberthelot.fr`. I registered the domain with OVH, created functional email addresses for the association, configured OVH SMTP, added SPF/DKIM/DMARC and deployed the application on Vercel.

The most interesting production bug came from password reset. The link was reported as expired almost immediately. The logs showed two verification requests in the same second. The cause was not my application code: corporate mail systems can inspect links before the user opens them, and the single-use token was being consumed by that inspection.

The fix was to make email confirmation and password-related validation require an explicit user action rather than silently validating on page load. I also added a way to resend the link.

Another hour was lost because a fix had been written and tested locally but had never actually reached production. The Git commit was rejected because my Git author email still contained a placeholder, so Vercel kept serving the previous build.

I also received teacher feedback that exposed inconsistencies in the association's statutes, especially around membership fees, and made it clear that the school administration — especially the headmaster — should have been contacted earlier.

## Lessons learned

Two lessons stand out.

First, **when a fix appears not to work, verify deployment before debugging the code again**. I spent an hour investigating a change that was simply not live.

Second, I realised that I had become too dependent on AI tools. Reaching my usage limit forced me to stop and made me recognise that I had started to confuse speed with progress. I could build faster, but I was not always taking the time to understand what I was building.

That also exposed a problem with this logbook: many entries had been written by AI instead of by me. I decided to restructure the journal so that it records the decisions and lessons that actually mattered.

# 2026-08-20 — Statutes, membership fees and user feedback

## The session

A long bureau meeting led to several important decisions for the association: the annual membership fee was set at €10, tutoring would remain collective rather than becoming individual paid lessons, and tutorat and mentorat would become separate areas.

I rewrote the statutes accordingly. Updating the membership fee revealed contradictions in other articles, including membership confirmation and the consequences of non-payment. I also added rules around bank-account access and corrected the statutes so that they matched the actual validation process of the application.

The platform was tested by other members of the team, which exposed many issues I had not noticed myself: an unclear charter interface, insufficient profile fields, weak sector coverage, incomplete profile states, missing notifications and non-clickable links.

I also introduced a genuine bug in file deletion by trying to make a database trigger delete files directly from Supabase Storage. I had made the same assumption earlier with CV files without ever having triggered the failing path.

Finally, I accidentally deleted my own test account from the Supabase dashboard.

## Lessons learned

The strongest lesson was that **real users find different bugs from the person who built the product**. Several of the most useful changes came from watching someone else use the application rather than from my own tests.

I also changed the way I think about product documentation: a policy or charter should describe the screen that actually exists. A written rule cannot compensate for a product that implements something different.

Another principle became clearer: controls should exist because they do something. Decorative permissions, fake checkboxes and unenforced dates create the impression of security without providing it.

# 2026-08-22 — Reaching out before everything is ready

## The session

This session was much less technical. I wrote to an HGG teacher to explain the project and to make the association known without asking her to take on a role she had not agreed to.

The association also fixed the pre-rentrée event for 29 August, continued work on the bank account, discussed the tutor and mentor network, and iterated on the presentation deck.

We decided to announce the tutor charter even though the question of possible remuneration was not settled yet. Waiting several more weeks for certainty would have meant losing the people we were trying to involve.

I also decided not to rely on the lycée to provide a list of students. Each person should register themselves on the platform rather than having the association receive a list of personal contact details from the school.

Finally, after the problems with membership fees, I decided the platform should not be opened beyond the bureau before the payment mechanism was ready.

## Lessons learned

This session showed that project progress is not only measured in lines of code. Communication, timing and institutional relationships can become the real bottlenecks.

It also exposed a coordination problem inside the team: two people independently started redesigning the same presentation. The result was useful, but the duplication was avoidable. Saying what you are working on before starting matters almost as much as doing the work itself.

# 2026-08-23 — When something looks finished but is not

## The session

I rewrote the meeting-minutes prompt so that it first filters a transcript for association-relevant information, then categorises what remains as decisions, tasks, leads, unresolved points or other useful items. Personal matters and off-topic discussion are ignored rather than mentioned.

I also worked on several platform issues. The mobile banner was hidden because I had assumed it would be unreadable, but testing showed that removing it entirely was the wrong solution. I replaced the fixed-height approach with a ratio-based layout.

The city system was also too limited: the database contained only a small hand-picked list of towns. I separated the reference list of French communes from the actual list of cities used by member profiles and added server-side search.

The association's communication revealed several preventable mistakes. A teacher email contained a broken sentence, an unreplaced placeholder and an incorrect membership-fee figure. The event invitation contained links that were visible but not clickable, and even the day of the week was wrong.

I also discovered that a supposed database backup had not run, and that a migration had been pushed to the database while the code using it remained uncommitted.

## Lessons learned

The common thread was **verification**. Something can exist and still be unusable: a backup file can exist without containing data, a link can be displayed without being clickable, a migration can exist remotely without the corresponding code being deployed, and a document can be sent without being internally consistent.

I also learned that explicit tests beat intuition. The mobile layout, city search and PDF links all improved after I tested the actual user experience rather than checking only the source code.

# 2026-08-25 — Guards that guarded nothing

## The session

I reviewed the first version of the WhatsApp-to-association workflow and found that several of its safeguards were protecting the wrong thing.

The database review exposed a missing default recipient, weak task validation and an immutability mechanism that relied on RLS even though the workflow writes through `service_role` and therefore bypasses it. I replaced that with explicit database triggers where immutability really matters.

The biggest discovery was the backup system. `supabase db dump --linked` had been creating schema-only files, not full backups. Three days of supposed backups therefore contained no member data. I changed the backup process to include a separate data dump and verified the first real backup by inspecting its contents.

The WhatsApp extraction prompt was also tested twice against a real conversation. Tasks were extracted well, but decisions were not reliable enough: the model sometimes recorded a suggestion as a decision or kept a decision that had later been reversed. I therefore decided not to treat model-generated decisions as authoritative data.

The workflow itself contained several integration problems that only appeared when executed: outdated n8n node versions, an incorrect response mode, malformed Postgres array parameters, missing permissions for `service_role`, and configuration assumptions that were not true in the current n8n version.

Finally, I found that sensitive credentials had appeared in screenshots or copied text, including the n8n encryption key, the database password and the SMTP password.

## Lessons learned

The central lesson is that **a safeguard is only useful if it protects the actual path through which the system operates**. RLS on a service-role write path is not a safeguard. A schema-only dump is not a backup. A prompt saying that decisions are important does not make the model a reliable source of immutable decisions.

This session also reinforced a broader principle from the rest of the project: when the cost of being wrong is high, automation should stop and require a human to confirm the information.

# 2026-08-26 — Two days on the wrong problem

## The session

Two more days on the WhatsApp automation, and it is still not finished. Four parts out of five now work — a conversation pasted into a form becomes tasks in the database, and the first real run pulled nine of them out of a day of messages without inventing anything. But the documents it is meant to write are still not connected, and the notification part has not been started.

Most of those two days went into failures that reported success: every step green while the database stayed empty. The worst of them was a security mechanism I had built, and then spent two evenings debugging, because a note I had written myself claimed the tool had no built-in authentication. It has three.

But the bugs are not what I got wrong.

I chose to work on this automation instead of finishing what the platform actually needs: its security, its design, and making sure the whole thing is smooth to use.

I still think the automation is a good idea, and I am not walking that back. People often stop contributing simply because they do not know what is left to do. Having every task visible at all times — who is on what, and which ones are free for anyone to pick up — will make the whole association run better. The problem is not the idea. It is when I chose to build it.

And I do not have unlimited time in which to be wrong. I am on the Pro plan, and I reach the weekly limit one to four days before it resets, nearly every week. The tier above costs €100 a month, which the association does not have and neither do I. So this is not a temporary annoyance I can spend my way out of: I get four or five days of building a week, not seven, and I spent them on the wrong thing.

Two obstacles moved, though, and neither the way we planned.

We will not get the list of incoming students. It only exists the day before term starts, and the obstacle is the administration, not the teachers. But the same conversation opened a better route: summer assignments are published on the national admissions platform, which every incoming student already visits, and we can add our link there. It also fits a decision we made earlier — never accept a file of personal contact details from the school. We do not go and collect people; we make ourselves findable.

And our welcome event has to move to the first week of September. It was built for the window when everyone is still home; a week later most alumni are back at their schools, and realistically only those near Paris will come. We kept the event and lost part of its audience.

## Lessons learned

**A good idea built at the wrong time is still a mistake.** I stand by the automation itself. But sequencing is a decision too, and I made it badly: I picked the problem I found most interesting over the one that was blocking everyone else.

**A capped budget of hours is a design constraint.** Running out of tooling several days early is not a scheduling annoyance — it is the thing that should decide what I open on Monday morning.

**Check a constraint before building around it.** One wrong sentence in my own notes cost two evenings, spent defending against a problem that did not exist.

# 2026-08-27 — Asking a teacher to proofread the machine

## The session

I split the work across two Claude accounts: one keeps the WhatsApp automation, the other takes the platform's security and design. They were sharing a repository and stepping on each other. Then I paused the automation entirely — term starts in days, and it is not what the association needs this week. Same sequencing mistake as two entries ago, except this time I made the call before it cost me anything.

Security took most of the two days. A policy written the week before would have broken the whole site if it had shipped, and it sat uncommitted because I could not prove it worked. The test I had planned — stay logged in, come back an hour later, click a link — proved nothing: a browser renews its own session for as long as a tab stays open, so I would have stayed logged in whether the code was right or broken. Closing the tab and waiting two hours was the only version that meant anything. It passed, and it went live.

Then I asked one of my former maths teachers to try the site. She signed up, navigated it, and sent six remarks. Not one was a bug. An address overflowed its box on a phone. A sentence meant to encourage students to contact alumni opened with something close to a reproach, aimed at people who already feel they are intruding. A filter called a subject an "option" — the very subject taught by the teachers whose help we need most.

Most of this site was written with AI, and that is exactly where such details slip through. The AI gave us a platform we could never have paid for. What it cannot do is sound like us. So we now ask teachers to be deliberately picky about wording and interface. That is not a correction applied to the project — it is the project: the machine builds it, people make it ours.

Our welcome event on 29 August is cancelled. We still have no list of incoming students, and rather than run two small events we will run one bigger one in the first or second week of term. Next year we should reach them through the national admissions platform instead of waiting on a list.

## Lessons learned

**A test that cannot fail proves nothing.** Mine would have passed against broken code, because the browser was doing the work I was trying to attribute to the server. I now ask what result would tell me I am wrong.

**Verify as the visitor you are testing for.** A file meant for search engines was being redirected to the login page. In my browser it looked perfect — because I was logged in.

**AI can build the thing; it cannot make it sound like us.** All six remarks were about language and judgment. No amount of generated polish would have produced them.

# 2026-09-10 — Back to school, and the project stalls

## The session

With back to school, the project has lost velocity. The members of the association are less committed, and the high school and the head teacher are busy and do not respond to our mails.

What we need now is a bank account, but the Qonto verification is taking longer than it should and is still pending.

In clear terms, we are stuck. It is part of the process; the road is not linear.

I am currently trying to remobilise the members of the association through messages, giving each of them clear tasks to do:

- set up a meeting with Bolloré
- run a session at the high school with the students and the teachers, and a meeting with the administration (Elyes)
- follow up with the tutors and mentors (volunteers: Rayane for the ones he approached, Elyes for the 2006 cohort)
- keep the WhatsApp community groups active
- reach out to students for the tutoring and mentoring programme (volunteer: Enzo)
- onboard Edgar, Samuel and possibly Paul-Antoine (volunteer: Rayane)
- reactivate the event group (volunteers: everyone)
- update the document for new members with the latest news (volunteer: Rayane)
- finalise Qonto
- build a payment interface on the platform (Yanis)

## Lessons learned

The project will be less exciting for me from now on: the platform is almost done. But if we want our work to last in the long run, we have to pursue our efforts to get a sustainable alumni network.

2026-09-17 — Small hiccups, but we haven't let go
The session

The meeting-minutes automation is still giving me trouble. Nothing serious, just a couple of small bugs I'm chasing down — I'll finish it tomorrow.

On the association side, there's real momentum back. We had a bureau call, and tomorrow Ryan, Lucas and Romain are going to Lycée Berthelot to talk to the first- and second-year students in classes A and B, and to try to catch the headmaster in person, since he still doesn't answer our emails — maybe it'll pass better face to face. We also got an email from Madame Tama today, asking how things are going to be organized on our end. A real sign of interest. We haven't let go these past weeks, and it shows. The onboarding period for the rest of the bureau is also running right now.

One thing that's starting to weigh on me: the school still hasn't funded us. I'm on a Claude Pro plan at €20 a month, and between the association's work and what I now need for my own use in business school, I hit the weekly limit in three or four days. It's not a wall I can just push through by spending more time on it.

Lessons learned

An automation is like the network itself: it doesn't run on its own, it has to be kept alive and actually used. It's fine that things go quiet for a while — the point isn't to be active every single day, it's to not give up on it.
