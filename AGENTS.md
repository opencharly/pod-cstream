# AGENTS.md — pod-cstream

Standalone candy repo for the `cstream` candy — the cstream transport spine: the
Wayland parent that is the streamer, the browser-facing gateway, the PAM login
broker, and the streaming/encoder gates. The candy lives in `charly.yml` at the
repo root plus its `etc/` staged files.

Canonical files:

- `charly.yml` — the `cstream:` candy entity (description, `require`, `port`,
  `package`, `plan`, `service`).
- `etc/` — the Wayland parent wrapper (`cstream-parent`), the PAM service
  (`pam-cstream`), the PipeWire drop-in (`pipewire-cstream.conf`), the session
  target (`cstream-session.target`), and the probes (`stream-probe`,
  `encoder-probe`, `login-probe`, `cstream-frame-probe`).
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-distros:omarchy-cstream` — the closest family skill: it documents the
  cstream transport spine (the `wl_compositor` v6 constraint, the nested-Hyprland
  composition). **This candy has no `skill:` entity of its own** — the gap is
  recorded on [opencharly/opencharly#291](https://github.com/opencharly/opencharly/issues/291).
- `/charly-check:wl` — the `wl:` check verb, including the Hyprland `wl: hypr-*`
  methods the resize gates use (served by `plugin-wl`).
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
  steps assert the three built binaries, the registered GStreamer elements
  (`webrtcsink`, `webrtcbin`, `pipewiresrc`), the live PipeWire audio nodes, the
  captured speaker monitor (both channels), the PAM stack, the four probes, and
  the `wl:` resize apply/observe pairs.
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no**
  per-repo candy gate. Its only workflow file is
  `.github/workflows/tag-on-merge.yml`.

## Modify this repo

- Edit the `cstream:` candy entity in `charly.yml`; there is no `skill:` entity in
  this repo (the gap is tracked on opencharly/opencharly#291).
- The `charly-streamer` commit pin is the `codeload` SHA in the build step; bump
  it deliberately and keep the WP-lock check in step.
- `pod-hyprland` is deliberately **not** required here — the nested compositor is
  supplied by the consumer layer `layer-cstream-desktop`. Do not add it back: two
  pins on one candy produced `resolved to multiple versions`.
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
