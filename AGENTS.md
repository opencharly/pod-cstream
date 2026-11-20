# AGENTS.md — pod-cstream

Standalone candy repo for the `cstream` candy — the cstream transport spine: the
Wayland parent that is the streamer, the browser-facing gateway, the PAM login
broker, and the streaming/encoder gates. The candy lives in `charly.yml` at the
repo root plus its `etc/` staged files.

Canonical files:

- `charly.yml` — the `cstream:` candy entity (description, `require`, `port`,
  `package`, `plan`, `service`).
- `etc/` holds:
  - the PAM service (`pam-cstream`);
  - the PipeWire drop-in (`pipewire-cstream.conf`);
  - the session target (`cstream-session.target`).

  The `cstream:` verb's venue probes (login, frame, and the bed-only
  `stream-probe`) live with the verb in `opencharly/plugin-cstream`.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-selkies:kde-cstream` — pod-kde-cstream, the desktop that composes this
  candy, and its beds (the R10 gate for a change here). **This candy has no
  `skill:` entity of its own** — recorded on
  [opencharly/opencharly#291](https://github.com/opencharly/opencharly/issues/291).
- `/charly-check:check` — the check/R10 framework: the `check:` step verbs, the
  `eventually:` / `retry_interval:` modifiers, and `charly check run <bed>`.
- `/charly-pod:pod` — the `kind: pod` / deploy schema reference (this candy is
  composed into a box; tree-position nesting, volumes, ports).
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs, `context: [runtime]`, service declarations).
- `/charly-internals:git-workflow` — before any git/PR action.

## Build / validate / test

- `charly box validate` at the repo root — the structural check: the manifest
  must parse and validate at the installed charly.
- The live R10 witness is a composing box's `check` bed; the candy's own `check:`
  steps assert:
  - the built binaries;
  - the registered GStreamer elements (`webrtcsink`, `webrtcbin`, `pipewiresrc`);
  - the live PipeWire audio nodes and the captured speaker monitor (both channels);
  - the PAM stack;
  - the hardware encoder the streamer reports it opened;
  - the control socket answering `geometry` (read-only).
- **The baked plan holds idempotent probes only.** `charly check live` runs it against
  real deployments, so a step that changes the deployment -- a resize, a window mapped
  over the desktop, input -- never goes here. The streaming gate and the resize matrix
  run in pod-kde-cstream's disposable beds as `run:` steps.
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no**
  per-repo candy gate. Its only workflow file is
  `.github/workflows/tag-on-merge.yml`.

## Modify this repo

- Edit the `cstream:` candy entity in `charly.yml`; there is no `skill:` entity in
  this repo (the gap is tracked on opencharly/opencharly#291).
- The `charly-streamer` commit pin is the `codeload` SHA in the build step; bump
  it deliberately and keep the WP-lock check in step.
- **No desktop is required here, and no gate may name one.** The consuming pod
  nests its desktop into this candy's parent. Every gate observes the stream or
  the streamer's own compositor, so it holds for any desktop.
- **No render-node literal.** The streamer and gateway take `CSTREAM_RENDER_NODE`,
  else charly's `DRINODE`.
- The `etc/` probes and the `service:` env are the contract the gates read; keep
  the frame-dir, the runtime dirs, and the socket paths in step.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge on
  PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the PR
  body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
