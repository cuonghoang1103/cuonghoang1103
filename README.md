## Hi, I'm Cường 👋

Software Engineering student at FPT University, Hanoi. I build and operate
**[cuongthai.com](https://cuongthai.com)** — a bilingual (VI/EN) learning
platform: courses, an AI tutor, a code lab, an exam room, and companion
iOS + desktop apps.

It isn't a tutorial project. It runs in production on a VPS I administer
myself, and I'm the one who has to fix it when it breaks at 2am.

**How I build:** I design the system and break the work into tasks, an AI
coding agent (Claude Code) writes much of the code, and I review it, test the
real behaviour, deploy it and keep it running.

---

### What that actually involves

| | |
|---|---|
| **Backend** | Node.js · Express · TypeScript · PostgreSQL + Prisma · **319 tables**, 150 migrations · Redis · Socket.IO |
| **Web** | Next.js (App Router) · React · Tailwind |
| **Mobile** | iOS — SwiftUI, shipped to TestFlight |
| **Desktop** | Electron — [130+ releases](https://github.com/cuonghoang1103/cuongthai-desktop/releases) for macOS, Windows and Linux, delivered by auto-update |
| **Infra** | Docker · GHCR · nginx · self-administered VPS · images built on a home server, post-deploy smoke tests, nginx config verified from inside the container and reverted automatically on failure |
| **AI** | One gateway over several LLM providers with per-feature model routing, per-user token quotas and daily spend limits · a self-hosted Qwen model on a home GPU behind a priority queue |

**Scale of the running system:** 4,300+ commits since June 2026 · 592
published courses · 12,100 lessons · 1,058 videos with bilingual subtitles
(330,936 aligned sentences, 4.8M words).

**Honest note:** this is a personal platform, not a startup — about 70 users.
What I'm showing is the engineering, not traction.

---

### Engineering problems I've actually had to solve

These are the ones that taught me the most — all of them share a shape:
**something reported success while doing nothing.**

<details>
<summary><b>A Docker bind-mount that read the old file forever</b></summary>

Changed `nginx.conf`, reloaded, got a green result — and nothing changed. Twice.

Docker binds a *single file* by **inode**, not by path. The deploy script
replaced the file with `mv`, which points the path at a **new** inode, so the
container kept reading the old one: `nginx -t` validated the old config,
`reload` reloaded the old config, the script reported OK.

Fix: write in place (`cat new > conf`), never `mv`/`rsync`/`sed -i`. Then
verify by comparing `sha256` on the host against
`docker exec … sha256sum /etc/nginx/nginx.conf` — because *"written"* is not
*"what the container reads"*.
</details>

<details>
<summary><b>An sshd setting that applied only half the time</b></summary>

Wrote `PasswordAuthentication no` into `70-no-password.conf`. `sshd -t` green,
`reload` green, exit 0 — and `sshd -T` still said `yes`.

`sshd_config` takes the **first** value it reads, and the `*.conf` glob is
alphabetical: a pre-existing `50-cloud-init.conf` had already won. The
`PermitRootLogin` line in the same file *did* apply — because nothing competed
for it. Half working is the signature of an ordering collision, not a syntax
error.

Fix: prefix `01-`. Verify with `sshd -T` (effective value), never `cat` of the
file you just wrote.
</details>

<details>
<summary><b>A build that was green all the way to a 7-minute outage</b></summary>

An image built outside compose picked up the default `Dockerfile` instead of
`Dockerfile.backend`. Result: an Alpine (musl) base carrying a Prisma engine
compiled for `debian-openssl-3.0.x` (glibc). Build green, push green, swap
green — then the backend restart-looped and the API was down.

Fix: a libc ↔ engine compatibility gate that runs **before** the push.
Lesson: a green build does not mean a runnable image.
</details>

<details>
<summary><b>A page that hung on its loading screen with no error at all</b></summary>

The 3D playground span two debugging sessions stuck on "loading", console
clean, network clean.

Next.js snapshots the contents of `public/` **at server start**. Rebuilding the
playground changed the bundle's content hash, so the running server returned
404 for a file that was physically on disk — no JS ran, and the loading screen
is plain HTML, so there was nothing to see fail.

Fix: restart Next after touching `public/**`. And kill the old server **by
port** — Node renames the process to `next-server`, so `pkill -f "next start"`
matches nothing and you end up debugging a zombie.
</details>

<details>
<summary><b>A link that rendered perfectly and did nothing</b></summary>

Added clickable `[2:19]` timestamps to AI answers so tapping one seeks the
video. Underline, hover colour, cursor — all correct. Clicking did nothing.

`react-markdown` runs `defaultUrlTransform` on every `href` and allows only
http/https/mailto/xmpp plus relative paths. A custom `tua://139` scheme is
silently replaced with an **empty string**. Measured, rather than guessed:

```js
defaultUrlTransform('tua://139')  // → ''
defaultUrlTransform('#tua-139')   // → '#tua-139'
```

Fix: anchor form `#tua-139`, intercepted with a delegated click handler.
</details>

The habit underneath all of these: **HTTP 200, exit 0 and a green log are not
evidence.** I verify with the thing that would differ if the change hadn't
worked — `sshd -T`, `sha256sum` from inside the container, `curl -I`, the route
table `next build` prints.

---

### Selected repositories

| | |
|---|---|
| **[api-backend](https://github.com/cuonghoang1103/api-backend)** | The platform monorepo — backend, web, desktop, infra, deploy tooling |
| **[cuongthai-desktop](https://github.com/cuonghoang1103/cuongthai-desktop)** | Release channel for the Electron app — installers and the auto-update feed |
| **[CuongHoangDev-V2](https://github.com/cuonghoang1103/CuongHoangDev-V2)** | The project before cuongthai.com: Spring Boot 3 + Flyway API, TypeScript/React front end, 287 commits in 12 days · [live demo](https://cuong-hoang-dev-v2.vercel.app) |
| **[Library-Management-System](https://github.com/cuonghoang1103/Library-Management-System)** | Spring Boot 3.4 (Java 21) + React — "one open loan per copy" enforced by a PostgreSQL partial unique index |
| **[Homestay-Booking-API](https://github.com/cuonghoang1103/Homestay-Booking-API)** | TypeScript REST API practice project with a Vitest test suite |
| **[Restaurant-Reservation-App](https://github.com/cuonghoang1103/Restaurant-Reservation-App)** | Flutter client over a Node/Express + PostgreSQL API — table booking with `SKIP LOCKED` |

---

### Currently learning

**Java + Spring Boot** — the dominant enterprise backend stack in Vietnam — and
deepening my JavaScript/React fundamentals by writing more of it by hand.

### Reach me

- 🌐 [cuongthai.com](https://cuongthai.com)
- 📍 Hanoi, Vietnam (GMT+7) · open to internships, part-time roles and small
  freelance projects — deploying and fixing Next.js / Node.js apps
