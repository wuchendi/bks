<div align="center">

# Plot

**Project management, simplified.**

Plot is a streamlined project and issue tracker. Projects, tasks, your inbox, scheduled
jobs and AI agents live in one fast workspace — and the whole thing runs as a single
binary, a Docker image, or on Cloudflare Workers.

**▶ Live demo: [bks.cdlab.workers.dev](https://bks.cdlab.workers.dev/)**

**English | [简体中文](README.zh-CN.md)**

</div>

<p align="center">
  <img src="https://cdn.jsdelivr.net/gh/cdLab996/picture-lib/wudi/bks/board.png" alt="A Plot project with its tasks on a board, grouped by status" width="100%">
</p>

---

## What is Plot?

Most trackers make you pick: a lightweight list that falls over once a team grows, or a
heavyweight suite that needs a consultant to configure. Plot aims at the middle — the
handful of things a small team actually does every day, done well and fast.

A project holds tasks. A task has a status, a priority, one owner — a person or an agent —
a due date, and child tasks when it needs breaking down. Everything that happens to it is
recorded on the task itself, everyone involved hears about it in their inbox, and the
board updates in realtime for whoever is looking.

---

## Plan the work.

*Start with a project, end with a board everyone can read at a glance.*

- **Projects →** Card and table layouts, tags with a tag filter and sidebar pins, duplicate,
  rename, archive and bulk actions. Each project pairs its task list with an overview pane:
  counters, status and priority split, and velocity.
- **List and board →** Two layouts over one shared manual order. Drag a task across status
  groups, filter with chips, bulk-select, right-click for quick actions, scroll forever.
- **Subtasks →** Break a task down up to three levels deep; progress rolls up as a ring.
- **Personal tasks →** Not everything belongs to a project. A task can live on its own
  (`T-<n>`), visible only to its creator, assignee and followers.

## Work it through.

*One task, one place for the whole conversation.*

- **Task detail →** Open it as a side pane, a drawer, or full-screen zen mode — the same view
  everywhere, with an activity feed that records every edit.
- **Markdown everywhere →** Descriptions and comments are markdown, with `@person` and task
  mentions and file attachments (paste to upload).
- **Comments →** Reply in threads and like what's useful.
- **Command palette and shortcuts →** Jump anywhere and act on anything without leaving the
  keyboard.

## Stay in the loop.

*Hear about what concerns you — and only that.*

- **Inbox →** Notifications for assignments, status changes, comments, mentions, invitations
  and due-date reminders. Read, snooze, archive, filter, or clear in bulk.
- **Realtime →** Server-sent events push changes to every open tab — only to the people who
  are allowed to see them.
- **Browser notifications →** New inbox items raise a desktop notification while Plot is in
  the background.
- **Direct messages →** Talk to a teammate without filing anything.

## Bring in agents.

*An agent is a teammate with a scope, not a superuser.*

- **Agents with an identity →** An agent is owned by a person, public or private, with
  separate grants to use it and to manage it.
- **Confined by design →** Assign a task to an agent, or mention it. It acts on the token of
  the person who put it to work, and only inside the projects that person assigned or
  mentioned it in.

## Run it your way.

*Your accounts, your data, your infrastructure.*

- **Accounts →** Local passwords, OIDC single sign-on, and virtual users that are claimed on
  first OIDC sign-in. Sessions, password-reset links and expiring read / write access tokens
  are managed from Preferences.
- **Access →** Invite-only projects: owner, direct members and shared teams, with project roles
  owner > admin > member. Global levels are user, admin (reads everything) and super admin.
- **Teams and members →** Team rosters with invitations, member pages with activity, charts
  and messages, and a scoped user directory.
- **Settings and audit →** System settings, member and invitation management, scheduled jobs
  and an audit log, each shown by level.
- **Scheduled jobs →** Log cleanup, audit retention, file garbage collection and due-date
  reminders run on a built-in scheduler.
- **Storage →** Uploads go to the local disk, any S3-compatible bucket, or Cloudflare R2 — or
  turn uploads off entirely and the UI hides them.
- **Everywhere →** English and Chinese, light / dark / system theme, desktop and mobile layouts.

---

## Deployment targets

One API codebase, four ways to run it. Nothing in the application code changes between them.

| Target | Database | Uploads |
| --- | --- | --- |
| Bun from source | SQLite file (libSQL) or a remote libSQL / Turso server | local disk or S3 |
| Single binary | same — migrations and the web app are embedded in the executable | local disk or S3 |
| Docker | same, as a non-root container with a data volume | local disk or S3 |
| Cloudflare Workers | D1, or a remote libSQL / Turso server | R2 or S3 |

The live demo runs on Cloudflare Workers.

---

## Architecture

```
                  Browser  (React SPA)
                        │
            JSON over HTTPS  +  server-sent events
                        │
          ┌─────────────┴─────────────┐
          │   Hono API  (OpenAPI 3.1)  │   Bun  ·  single binary  ·  Docker  ·  Workers
          └─────┬───────────────┬─────┘
                │               │
       ┌────────┴──────┐  ┌─────┴──────────────────┐
       │  Drizzle ORM  │  │  Blob storage          │
       │  libSQL · D1  │  │  disk · S3 · R2        │
       └───────────────┘  └────────────────────────┘
```

The web app reads the API only through types generated from the API's own OpenAPI document,
so the contract between them is checked at build time. Application code uses Web APIs only;
each host plugs in its runtime pieces — on Cloudflare, a Durable Object hosts the event bus,
the scheduler and the rate limiter.

| Layer | Stack |
| --- | --- |
| Web | React 19, Vite 8, Tailwind CSS v4, shadcn/ui, TanStack Router + Query, Tiptap, Recharts, i18next |
| API | Hono with a `@hono/zod-openapi` contract and Scalar docs, on Bun or Cloudflare Workers |
| Data | Drizzle ORM over libSQL (local file or Turso) or Cloudflare D1, with full-text search |
| Realtime | Server-sent events; a Durable Object on Workers |
| Language | TypeScript end to end |

---

## Source code

Plot is closed-source for now. This repository holds its public documentation; try the
product at **[bks.cdlab.workers.dev](https://bks.cdlab.workers.dev/)**.
