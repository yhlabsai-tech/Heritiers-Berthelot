# Héritiers Berthelot — Building an Alumni Association from Scratch

> A public logbook of how I founded **Les Héritiers de Berthelot**, the alumni
> association of the *classe préparatoire* (CPGE) at Lycée Marcelin Berthelot,
> from the ground up — legal structure, governance, and the platform that runs it.

I started this the year I left the *prépa*. This repository is the running
record of that work: the decisions, the drafts, the mistakes, and the code.

---

## What is this, in plain terms?

In France, the **CPGE** (*classes préparatoires aux grandes écoles*) are
intensive, selective, post-secondary programmes that prepare students over two
years for competitive entrance exams to top business and engineering schools.
They are hosted inside a *lycée* but sit **above** the secondary curriculum —
closer to the first years of a demanding university track than to high school.

**Les Héritiers de Berthelot** is the alumni association for the business-track
CPGE (ECG) at Lycée Marcelin Berthelot. It is a non-profit under the French
*loi 1901*. Its purpose:

- **Support current students** — tutoring, mentoring, mock interviews.
- **Open doors** — internships, professional introductions, conferences.
- **Build a network** — connect alumni across graduating years and schools.

## My role

Founder, and Secretary General of the application division. I coordinate a
six-person founding team across three parallel workstreams:

1. **Student support** — designing the tutoring and mentoring programme.
2. **Association creation** — statutes, internal regulations, governance,
   compliance (*loi 1901*).
3. **Platform** — the web application the association runs on. This one is mine
   to build.

## What's in this repository

| Folder | What it holds |
| --- | --- |
| [`journal/`](./journal) | Dated logbook entries. One per working session. |
| [`docs/`](./docs) | Non-sensitive artefacts: diagrams, public notes, decisions. |
| `app/` *(coming soon)* | The association's web platform. |

> **Note on privacy.** This repository is public. It documents the *journey*,
> not the internals. Member data, personal information, and internal strategy
> stay off GitHub — that's a GDPR requirement and plain common sense.

## What counts as a working session

Anything that moves the association forward gets an entry — including the time
I spend learning the tools rather than shipping features.

I run the platform workstream, and I did not start this project knowing how to
build it. So a session spent standing up a self-hosted service, breaking a
pipeline and understanding why, or working out how to keep secrets out of Git is
project work, not a detour: it is the apprenticeship the platform is built on.
Entries of that kind close with a line stating what the session gives back to
the association. If I can't write that line honestly, the session doesn't belong
in this logbook.

## The platform (in progress)

The association will run on a custom web app so alumni and students can register,
confirm membership, and connect.

- **Framework:** Next.js
- **Backend / database:** Supabase
- **Hosting:** Vercel

Deliberately built **independent** from the school's own website.

Status: not started. The stack is chosen, the groundwork is being laid, and the
first prototype is the next milestone.

## Timeline

| Date | Milestone |
| --- | --- |
| 2026-06 | Idea born during my oral exams. |
| 2026-07 | First conversations with teachers and classmates; drafting begins. |
| 2026-07 | Statutes and internal regulations drafted; outreach for the tutoring programme. |
| 2026-07-28 | Public logbook opened. |
| 2026-07-30 | Statutes and administrative groundwork finished; I take the role of Secretary General of the application division. |
| 2026-08-01 | Self-hosted automation stack rebuilt and versioned, ready to serve the association's back office. |
| *next* | Statutes filed with the *préfecture*; first prototype of the web app. |

---

*This logbook is written in English so it can be read widely. The association's
official documents are in French.*
