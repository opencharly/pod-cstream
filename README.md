# pod-cstream

The `cstream` candy of the OpenCharly candy library, as a standalone repo
(kind-prefixed naming). It is the cstream transport spine: the Wayland parent that
**is** the streamer, the browser-facing gateway, the PAM login broker, and the
gates that prove the stack is actually *streaming* rather than merely started.

## What it provides

Builds `cstream-gateway` (Go) and `cstream-leader`, `cstream-streamer` and
`cstream-input` (Rust) from a pinned `charly-streamer` commit. It also stages the
PAM service, the PipeWire audio endpoints and the session target. Two supervised
services run the pipeline:

| Property | Value |
|---|---|
| Services | `cstream-streamer` (the Wayland parent, priority 10), `cstream-gateway` (the browser player, login and signalling proxy, priority 18) |
| Port | `8080` (browser-facing gateway) |
| Requires | `pod-pipewire`, `layer-supervisord`, `layer-gst-wayland-display` |
| Packages | `go`, `rust`, `pam`, `socat`, `foot`, `gst-plugins-good`/`bad`, `gst-plugin-rswebrtc`, `gst-plugin-pipewire` |
| Env | `CSTREAM_AUDIO=on`, `CSTREAM_ENCODE=auto`, `CSTREAM_FRAME_DIR`; accepted: `CSTREAM_RENDER_NODE` (default: charly's `DRINODE`), `CSTREAM_ENCODE`, `CSTREAM_LOCK_AFTER` (default off) |
| PAM | `/etc/pam.d/cstream` (its own service, not a borrowed `login` stack) |

The streamer creates the parent compositor and holds the render node, so it *is*
the Wayland parent — a separate parent would negotiate a WebRTC session with no
media. **No desktop is composed here.** The consuming pod nests one into this
parent; pod-kde-cstream nests KDE Plasma through pod-kde-nested.

The render node's kernel driver picks the encoder:
- NVIDIA through CUDA (`nvh264enc`) on an `nvidia` node, as pod-cdi injects it through
  CDI;
- VA-API (`vah264enc`) on `amdgpu`/`i915`/`xe`.

## Why the gates are unusual

Three gates exist because the obvious checks are not evidence:

- A service check proves a process is up — measured, charly's own `service:` verb
  reports PASS even for a supervisord program parked in FATAL
  (`opencharly/charly#456`).
- A screenshot proves the compositor draws, but not that the encoder and transport
  carry those pixels.
- "The frame is not uniform" passes on a wallpaper, so a frozen pipeline serving
  one static frame satisfies it.

So the streaming gate compares frames pulled **through the pipeline** before and
after a real client window maps on the streamer's own compositor. The encoder gate
reads the encoder the streamer reports it OPENED and fails on software, the named
silent fallback to CPU. The audio gates assert the live PipeWire nodes and that the
speaker's **monitor** ports (both channels) are captured.

The streaming gate and the resize gates CHANGE the deployment: one maps a window over
the desktop, the other resizes every viewer's stream. So they are not in this candy's
baked plan, which `charly check live` runs against real deployments. They run in the
consuming pods' disposable beds, as `run:` steps. pod-kde-cstream's
`check-kde-cstream-pod` resizes through a matrix of sizes, from portrait 390x844 to
3840x2160, and observes each one in the stream's frames, KWin's output and the pointer.
The baked plan keeps what a live deployment can prove without change: the control
socket is served and reports the stream's current size. Sizes are fitted to the chosen
encoder's own limits and to even dimensions only, so 1366x768 streams as 1366x768.

## How to use it

Compose the candy into a box (the consuming pod supplies the nested desktop):

```yaml
my-cstream:
  candy:
    candy:
      - '@github.com/opencharly/pod-cstream:<tag>'
```

```bash
charly box build my-cstream
charly start my-cstream
# gateway on :8080
```

## Layout

- `charly.yml` — the `cstream:` candy entity.
- `etc/` — the PAM service, the PipeWire drop-in and the session target. The
  `cstream:` verb's probes (login, frame, and the bed-only `stream-probe`) ship with
  that verb in `opencharly/plugin-cstream`.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — this user overview.

## Related

- `/charly-selkies:kde-cstream` — pod-kde-cstream, the KDE Plasma desktop
  streamed through this candy, and its beds. This candy has no `skill:` entity of
  its own (recorded on
  [opencharly/opencharly#291](https://github.com/opencharly/opencharly/issues/291)).
- `/charly-check:check` — the `cstream:` verb's probes and the beds that compose
  this candy.
- `/charly-infrastructure:dbus-layer` / `/charly-selkies:selkies` — the desktop
  transport and nested-compositor background.
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
