# pod-cstream

The cstream transport spine: the Wayland parent that **is** the streamer, the browser-facing
gateway, the PAM login stack, and the gates that prove the stack is actually *streaming* rather
than merely started.

Composes `layer-gst-wayland-display` (the parent compositor element), `pod-hyprland` (the nested
desktop) and `layer-supervisord`, and builds `cstream-gateway` and `cstream-leader` from a pinned
`charly-streamer` commit.

**Three of its gates exist because the obvious checks are not evidence.** A service check proves a
process is up — measured, charly's own `service:` verb even reports PASS for a supervisord program
parked in FATAL (`opencharly/charly#456`). A screenshot proves the compositor draws, but says
nothing about whether the encoder and transport carry those pixels. And "the frame is not uniform"
passes on a wallpaper, so a frozen pipeline serving one static frame forever satisfies it. So the
streaming gate compares frames pulled **through the pipeline** before and after a real client
window maps: a dead, frozen or placeholder pipeline yields identical frames and fails. The encoder
gate fails when a VA-capable node is present but hardware H.264 is not actually producing a
stream — the spec's named silent fallback to CPU.

The login stack is `/etc/pam.d/cstream`, deliberately **not** `login`: `login` carries
`pam_loginuid` (needs `CAP_AUDIT_CONTROL`) and `pam_securetty` (gates on a tty a streamed session
has none of), so borrowing it makes a *correct* password fail with "cannot retrieve authentication
info" — an error that names the wrong cause.
