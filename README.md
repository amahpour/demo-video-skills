# demo-video-skills

Short product demo videos from real screen recordings, made with your coding agent. It records your app on virtual displays, cuts the take by timeline marks, adds captions, a local voice-over and a music bed, and asks you to approve the sound before it renders.

Everything on screen is real app state: nothing is mocked, redrawn or stitched from screenshots.

## What's inside

- **Two agent skills**, for Codex, Claude Code and any tool that reads `SKILL.md` files: `demo-video` (the whole workflow, with its gates) and `voice-over` (local narration with Kokoro).
- **A recorder.** Each window is Chrome on its own Xvfb display, driven over the DevTools protocol and recorded by ffmpeg with wall-clock timestamps. Terminals are recorded the same way, through tmux and xterm.js. A timeline of marks says what happened when, and where on screen.
- **An editor.** Shots from the marks, crops framed by the page's real layout, captions, a splash and an end card, labelled speed-ups, and an MP4 and a looping GIF under GitHub's 10 MB.
- **Sound.** Kokoro narration in passages, a steady music bed under it, and the stems and an audio-only mix to approve before anything is rendered.
- **The rules**, learned on real videos, with the research behind them.

## Quick start

You need Linux (or WSL2) with Xvfb, ffmpeg, Chrome or Chromium, and [uv](https://docs.astral.sh/uv/). On Ubuntu: `sudo apt install xvfb ffmpeg tmux fonts-noto-core`, plus Chrome and uv. On a Mac, install [Docker Desktop](https://www.docker.com/products/docker-desktop/) and run the commands below through `in_docker.py` (see [On macOS](skills/demo-video/references/recording.md#on-macos)).

```bash
git clone https://github.com/amahpour/demo-video-skills && cd demo-video-skills
python3 skills/demo-video/scripts/check_env.py --deep
python3 skills/demo-video/scripts/new_project.py projects/sample
cd projects/sample
uv run record.py --build ~/recordings/sample/take-1
uv run edit.py --build ~/recordings/sample/take-1 --stills
uv run edit.py --build ~/recordings/sample/take-1 --gif
```

That records the bundled sample app and writes `~/recordings/sample/take-1/out/demo.mp4` and `demo.gif`. For your own app, give `new_project.py` your app's `--url` and write the scenario in `record.py`. Or ask your agent:

> Make a 30-second demo video of my app at http://localhost:3000 for the README.

## Using the skills

Inside this repository, Codex finds the skills in `.agents/skills/` and Claude Code in `.claude/skills/`: small pointer files that send the agent to the real skills in `skills/`. `CLAUDE.md` imports `AGENTS.md`, so every agent gets the same instructions.

To use the skills from any repository, link the real skill folders into your user folders:

```bash
mkdir -p ~/.agents/skills ~/.claude/skills
for s in demo-video voice-over; do
  ln -s "$PWD/skills/$s" ~/.agents/skills/$s   # Codex
  ln -s "$PWD/skills/$s" ~/.claude/skills/$s   # Claude Code
done
```

## How a video gets made

1. **Brief:** what it's for, the one thing only this product does, what the viewer should believe afterwards, a reference video, voice or not, which music.
2. **Rig:** a scenario that plays the human at a person's pace and marks every moment worth showing.
3. **Takes:** a dry run (a screenshot per window), then takes. Expect several.
4. **Cut:** shots from the marks, previewed as stills. Captions (six words or fewer) and narration passages, written after watching the take.
5. **Sound:** the soundtrack alone, approved by a person.
6. **Render:** the MP4, and a silent GIF whose first frame tells the story.

## The rules, short

1. Real footage or nothing.
2. The story is the product: show what only it can do.
3. Type at reading pace. Speed up the waiting, and say so.
4. Captions short and one per beat; narration in passages.
5. Music comes in alone, then stays at one level under the voice. No ducking.
6. Approve audio as audio, before rendering video.
7. Ship deliverables, not tooling.

The full set, with the reasons: [`lessons.md`](skills/demo-video/references/lessons.md). The research behind them: [demo video best practices](skills/demo-video/references/research-demo-videos.md) and [a survey of README demos](skills/demo-video/references/research-readme-survey.md).

## Layout

| Path | What it is |
|---|---|
| `skills/demo-video/SKILL.md` | The workflow, with its gates |
| `skills/demo-video/scripts/demo_video/` | The library: `record` (displays, windows, the timeline), `terminal` (tmux panes as video), `cut` (shots, captions, cards, MP4, GIF), `sound` (narration, the music bed, the mix) |
| `skills/demo-video/scripts/` | `new_project.py`, `check_env.py`, `audio_check.py`, `chiptune.py` (an original music bed), `in_docker.py` (recording on macOS, through Docker), the terminal viewer, the Kokoro runner |
| `skills/demo-video/assets/template/` | A new project's rig, with a sample app |
| `skills/demo-video/assets/docker/` | The Linux image `in_docker.py` records in on a Mac |
| `skills/demo-video/references/` | Lessons, research, the recording and editing APIs, delivery, troubleshooting |
| `skills/voice-over/` | Kokoro setup, auditions, narration from a script |
| `projects/` | Your rigs (git-ignored: they name your product) |
| `AGENTS.md`, `CLAUDE.md` | Instructions for coding agents |

## Voice and music

Narration runs locally with [Kokoro-82M](https://huggingface.co/hexgrad/Kokoro-82M) (Apache-2.0) in its own Python 3.12 venv; no audio or script leaves your machine. See [`skills/voice-over/SKILL.md`](skills/voice-over/SKILL.md).

No music is included. Bring a track you're allowed to use, and credit it: Creative Commons Attribution tracks need their credit line on the end card and wherever the video is posted. Or generate an original 8-bit bed in a style (chiptune, jazz, hip hop, rock, bossa, or one you write) with `skills/demo-video/scripts/chiptune.py`, timed to the cut's cards, with no credit line needed.

The terminal viewer loads [xterm.js](https://xtermjs.org/) (MIT) and its fonts from public CDNs at recording time.

## License

MIT. See [LICENSE](LICENSE).
