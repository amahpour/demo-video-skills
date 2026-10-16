# AGENTS.md

Instructions for coding agents (Codex, Claude Code, Cursor, Gemini CLI, GitHub Copilot and others) working in this repository.

## What this is

demo-video-skills makes short product demo videos from real screen recordings. A recorder runs the product on virtual displays (Chrome over the DevTools protocol, terminals through tmux and xterm.js) and records it with ffmpeg. An editor cuts the take by timeline marks and adds captions, cards, local narration (Kokoro) and a music bed. The rules learned making real videos come with it.

| Path | What it is |
|---|---|
| `skills/demo-video/` | The main skill. `SKILL.md` is the workflow; `scripts/` holds the `demo_video` library, `new_project.py`, `check_env.py`, `audio_check.py`, `in_docker.py` and the terminal viewer; `assets/template/` is a new project's rig (`record.py`, `edit.py`, `captions.json`, a sample app), and `assets/docker/` the image for recording on macOS; `references/` holds the lessons, the research, the API and the troubleshooting table. |
| `skills/voice-over/` | Kokoro setup, voice auditions, narration from a script. |
| `.agents/skills/`, `.claude/skills/` | Pointers, so Codex and Claude Code find the skills: each `SKILL.md` there sends the agent to the real one in `skills/`. Keep their `name` and `description` identical to the real skill's. |
| `projects/` | Video rigs, one folder per product. Git-ignored except its README: a rig names a product and its screens. |

## Making a video

Use the `demo-video` skill (and `voice-over` for narration). If your tool doesn't load skills, read `skills/demo-video/SKILL.md` and follow it step by step.

These hold every time:

- **Real footage only.** No stitched screenshots, redrawn or mocked UI, or on-screen data the app didn't produce.
- **Ask the brief before building** (what it's for, the one thing only this product does, references, voice, music), and agree the beats.
- **Captions of six words or fewer; narration in passages** that span several beats.
- **Music steady about 7 dB under the voice, no ducking, credited.**
- **Approve audio as audio.** Run `edit.py --audio-only`, send the WAVs, and wait. Pass `--audio-approved` only after a person said yes to that exact mix.
- **Under 10 MB per file.**
- **Never record a real desktop.** Every window goes on its own virtual display.
- **Nothing private on screen or on disk:** use a demo account and made-up data; keep sign-in tokens and keys out of frames, logs and files.
- **Takes go in `~/recordings/<project>/`**, never in a session scratch directory (it gets wiped) and never in git.
- **Deliver the files, not the tooling.** Don't copy this kit into the product's repository, and don't upload, post or publish anything unless asked.

## Working on the kit itself

- Python 3.10+, the standard library plus `pillow`, `numpy` and `websockets`, declared inline in each runnable script (PEP 723), so `uv run <script>` just works. Kokoro runs in its own Python 3.12 venv as a subprocess; never import it into the kit.
- Linux: Xvfb, ffmpeg with libx264, Chrome or Chromium; tmux for terminals. On macOS the same toolchain runs in Docker: put `python3 skills/demo-video/scripts/in_docker.py` in front of each command below that records or edits.
- Test a change on the bundled sample before calling it done: `python3 skills/demo-video/scripts/new_project.py /tmp/demo-video-check`, then in that folder `uv run record.py --build /tmp/demo-video-check/take-1`, `uv run edit.py --build /tmp/demo-video-check/take-1 --stills`, then a render with `--gif`. Run `check_env.py --deep` after touching the recorder.
- Keep the library small and readable: no frameworks, no new dependencies without a reason.
- Docs are plain sentences, one paragraph per line (no hard wraps).
- Keep the repository free of anyone's product, names, paths, recordings and music. No media files in git.
