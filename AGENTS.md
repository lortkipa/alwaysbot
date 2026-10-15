# Always Bot — Project Context for AI Agents

This file describes what Always Bot is and where it is headed. Read it before
making changes so your work stays aligned with the project's goals.
It is tool-agnostic: Claude Code reads it via `CLAUDE.md`, and other agents
(Codex, Cursor, Copilot, etc.) read `AGENTS.md` directly.

## What Always Bot is

Always Bot is an **open-source AI agent that is always on, 24/7**. It keeps
working even when the user's own computer is off.

- **One cloud VM per user.** Each user gets their own Linux virtual machine in
  the cloud, and their bots run there.
- **Full computer control.** Bots have complete control of that machine and can
  do anything a human can do on a computer: place orders, write code, send
  email, and more.
- **Long-running work.** Bots can work for hours at a time without stopping.
- **Watch or take over.** Users can watch their bots work live, and can take
  control of the cloud computer with their own mouse and keyboard at any time.

## Bot behavior

- Bots have full control and do whatever the user tells them to do. They are
  not gated behind approval prompts for each action.
- When a bot is **uncertain** about something (an ambiguous instruction, a
  choice the user didn't specify), it may pause and ask the user to confirm
  before continuing.

## AI models

- Bots are powered by **OpenAI** models.
- All model calls go **through the backend**. Clients never call OpenAI
  directly and never hold an OpenAI API key.
- On the official backend, the maintainer's OpenAI key is used. A self-hosted
  backend is configured with its own key.

## Clients

Always Bot will ship on every major platform:

| Surface | Platforms |
| ------- | --------- |
| Web     | Browser |
| Desktop | Linux, Windows, macOS |
| Mobile  | Android, iOS |

All clients talk to the same backend. Features should be designed so they work
across all of them, including live viewing and taking control of the VM.

## Backend and hosting

Users have two options:

1. **Official hosted backend (default, recommended).** Run by the project
   maintainer. It is currently planned to run on the maintainer's home server
   and may move to a professional hosting provider later. Don't hard-code
   anything that assumes a specific host.
2. **Self-hosted.** Anyone can clone this repository and run the server
   themselves. Self-hosting is a first-class use case, not an afterthought.

What this means when you contribute:

- Keep the backend deployable by people outside the project: configuration
  goes through environment variables or config files, setup is documented, and
  nothing depends on the maintainer's infrastructure.
- Clients must be able to point at either the official backend or a
  self-hosted one.

## Business model

The official backend is **freemium**: a free tier plus paid tiers. Code should
support per-plan limits (for example VM time, resources, or model usage).
Billing and plan limits must be optional so self-hosters can turn them off.

## Tech stack

| Area | Decision |
| ---- | -------- |
| Clients | **One React web app** wrapped for desktop and mobile, so all six platforms share a single codebase |
| Desktop wrapper | **Electron** (Linux, Windows, macOS) |
| Mobile wrapper | **Capacitor** (Android, iOS) |
| Backend | **TypeScript (Node.js)** |
| Language | TypeScript across the whole monorepo, so types are shared between backend and clients |
| User computers | **Incus-managed KVM virtual machines**, Linux only, one per user |
| Screen streaming and takeover | **VNC in each VM, viewed through noVNC** in the clients |

Why these choices:

- **One web app:** the client's main job is chat plus a remote-screen view, and
  noVNC is a web library, so a single web codebase works everywhere.
- **React:** the largest ecosystem and contributor pool, and the framework AI
  coding agents handle best.
- **Electron + Capacitor:** native code stays in TypeScript (Swift or Kotlin
  only for rare mobile edge cases). Electron bundles Chromium, so the app and
  the noVNC screen render the same on every desktop OS. Capacitor has mature
  push-notification support, which bots need to reach users when they ask for
  confirmation. Accepted trade-off: larger desktop apps. Electron windows must
  use secure settings (`contextIsolation` on, `nodeIntegration` off, a narrow
  preload API).
- **TypeScript:** one language for the whole repo makes contributing easier,
  and OpenAI ships official TypeScript SDKs.
- **Incus VMs:** bots get root and run untrusted code from the internet, so
  each user needs the strong isolation of a real VM rather than a container.
  Incus has a simple REST API, snapshots, and images, and installs on any Linux
  host, which keeps self-hosting easy. Hosts need hardware virtualization
  (KVM).
- **VNC + noVNC:** simple and proven, works in every client, and supports
  mouse and keyboard takeover out of the box. WebRTC streaming may replace it
  later if latency becomes a problem.

## Repository

Everything lives in **one repository (monorepo)**: backend, VM tooling, and all
clients.

## Guidelines for contributors and their agents

- **Isolation and security come first.** Bots act with real power on users'
  behalf. Keep users' VMs isolated from each other and from the host's local
  network, authenticate every request, and keep secrets (API keys, user
  credentials) on the backend.
- **The user stays in control.** Live viewing and manual takeover are core
  features. Don't design anything that makes them harder.
- **Open source.** The project is MIT-licensed (see `LICENSE`). Never commit
  secrets, credentials, or personal data.

## Project status

The project is at an early stage: the repository contains only this document
and the license. Still undecided:

- Backend web framework, database, and monorepo tooling
- Whether the bot pauses automatically while the user has taken over the VM

Update this file as those decisions are made, and add the repository layout and
build commands once they exist.
