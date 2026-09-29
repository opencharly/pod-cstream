# pod-cstream

The `cstream` candy of the OpenCharly candy library, as a standalone repo
(kind-prefixed naming). It is the cstream transport spine: the Wayland parent that
**is** the streamer, the browser-facing gateway, the PAM login broker, and the
gates that prove the stack is actually *streaming* rather than merely started.

## What it provides

Builds `cstream-gateway` (Go) and `cstream-leader` + `cstream-streamer` (Rust)
from a pinned `charly-streamer` commit, stages the Wayland parent wrapper, the
PAM service, the PipeWire audio endpoints, the session target, and four probes.
Two supervised services run the pipeline:

| Property | Value |
|---|---|
| Services | `cstream-parent` (the streamer itself — the Wayland parent, priority 10), `cstream-gateway` (priority 18) |
| Port | `8080` (browser-facing gateway) |
| Requires | `pod-pipewire`, `layer-supervisord`, `layer-gst-wayland-display` |
| Packages | `go`, `rust`, `pam`, `socat`, `foot`, `gst-plugins-good`/`bad`, `gst-plugin-rswebrtc`, `gst-plugin-pipewire` |
| Env | `CSTREAM_AUDIO=on`, `CSTREAM_ENCODE=auto`, `CSTREAM_RENDER_NODE`, `CSTREAM_FRAME_DIR`, `CSTREAM_LOCK_AFTER=10` |
| PAM | `/etc/pam.d/cstream` (its own service, not a borrowed `login` stack) |

The streamer creates the parent compositor and holds the render node, so it *is*
the Wayland parent — a separate parent would negotiate a WebRTC session with no
media. The nested desktop (Hyprland) is supplied by the consumer layer
`layer-cstream-desktop`; this candy does not pin it.

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
after a real client window maps. The encoder gate fails when a VA-capable node is
present but hardware H.264 is not actually producing a stream — the named silent
fallback to CPU. The audio gates assert the live PipeWire nodes and that the
speaker's **monitor** ports (both channels) are captured.

## How to use it

Compose the candy into a box (the consumer layer supplies the nested desktop and
the session):

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
- `etc/` — the Wayland parent wrapper, the PAM service, the PipeWire drop-in, the
  session target, and the `stream-probe` / `encoder-probe` / `login-probe` /
  `frame-probe` probes.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — this user overview.

## Related

- Closest family skill: `/charly-distros:omarchy-cstream` — documents the cstream
  transport spine (the `wl_compositor` v6 constraint, the nested-Hyprland
  composition). This candy has no `skill:` entity of its own (recorded on
  [opencharly/opencharly#291](https://github.com/opencharly/opencharly/issues/291)).
- `/charly-check:wl` — the `wl:` check verb (including the Hyprland `wl: hypr-*`
  methods the resize gates use), served by `plugin-wl`.
- `/charly-infrastructure:dbus-layer` / `/charly-selkies:selkies` — the desktop
  transport and nested-compositor background.
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
