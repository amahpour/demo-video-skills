# Recording: the scenario API

A take is one real run of the product, recorded as video. `record.py` holds the scenario: a function that opens the windows, puts the app in its prepared state, calls `take.ready()`, then plays the human and marks every moment the edit will cut to. `demo_video.record.run(scenario)` gives it the command line (`--build`, `--dry`, `--fps`, `--force`) and cleans everything up afterwards, even on an error.

```python
from demo_video.record import Take, run

def scenario(take: Take) -> None:
    app = take.window("app", "http://localhost:3000", css=(1280, 720), scale=2.0)
    app.wait_for("#inbox")
    take.ready()
    app.click("#compose")
    take.mark("typing", box=app.box("#editor"))
    app.type("Lunch on Friday?", cps=11)
    app.press("Enter")
    app.wait_for_text("#sent", "Lunch on Friday?")
    take.mark("sent", box=app.box("#sent"))

if __name__ == "__main__":
    run(scenario)
```

## What a take writes

| File | What it is |
|---|---|
| `<window>.mkv` | One recording per window, H.264 at CRF 14 (near-lossless, so the edit can crop), with wall-clock timestamps (`-copyts`) |
| `timeline.jsonl` | One JSON object per line, each with `t` (wall-clock seconds): `mark` records (`what` plus whatever you passed), `term` records (a terminal's text whenever it changed), `clock` records (a system clock step) |
| `views.json` | Each window's CSS size, device scale and pixel size; `cells` for terminal windows (where the character cells are) |
| `*.log` | Xvfb, Chrome, ffmpeg and the terminal viewer's logs |
| `dry-*.png` | `--dry` only: a screenshot of each window at `take.ready()` |

## Take

| Call | What it does |
|---|---|
| `take.window(name, url, css=(1280, 720), scale=2.0, color_scheme="dark")` | A Chrome app window (no tabs or address bar) on its own virtual display, `css` CSS pixels at `scale` device pixels each. Returns a `Window`. `color_scheme=None` leaves the page's own setting. |
| `take.ready()` | The prepared state is on screen. In a dry run: screenshot every window and stop. Otherwise start recording every window and mark `recording`. Windows opened later are recorded from when they open. |
| `take.mark(what, **extra)` | A moment the edit can cut to. Pass boxes (`box=window.box(...)`) and anything else JSON can hold. Marks with the same name are numbered in order: `take.mark("message")` three times gives the edit `mark("message", 0)`, `1` and `2`. |
| `take.spawn(argv, env=None, log_name="")` | A background process killed with the take (a dev server, for example). |
| `take.cleanups.append(fn)` | Run `fn` when the take ends. |

## Window

Boxes are `[left, top, right, bottom]` in CSS pixels, as `getBoundingClientRect()` gives them.

| Call | What it does |
|---|---|
| `wait_for(selector, timeout=20)` | Until a matching element is visible. |
| `wait_for_text(selector, text, timeout=20)` | Until the element's text contains `text`. |
| `box(selector)` / `boxes(selector)` | One element's box (or None), or every match's text and box. |
| `click(selector)` | A real mouse click at the element's centre (move, press, release), after scrolling it into view. |
| `type(text, cps=11, selector=None)` | Types into the focused element (or `selector`) at a person's pace. About 11 characters a second reads as typing; terminal commands read fine at about 22. |
| `press(key)` | `Enter`, `Tab`, `Escape`, `Backspace`, `Delete`, `Space`, the arrows, `Home`, `End`, `PageUp`, `PageDown`. Enter submits forms as a real key press does. |
| `eval(js)` | A JavaScript expression's value (promises awaited). For state the page holds that a selector can't reach. |
| `goto(url)` / `reload()` | Navigate and wait for the load. |
| `color_scheme("dark")` | Make the page see `prefers-color-scheme: dark` (or `light`). |
| `screenshot(path)` | A PNG of the window, at device pixels. |
| `cookies()` / `set_cookies(cookies)` | Sign a second window in with the first one's session, in memory (then `reload()`). |

## Terminals

`demo_video.terminal.Terminals` records terminals as video: each is a tmux session on a private tmux server, shown read-only in xterm.js in its own window. Nothing typed into that window reaches the shell; the scenario types through tmux.

```python
from demo_video.terminal import Terminals

terms = Terminals(take, cols=100, rows=28)
terms.open("shell", title="you@laptop", subtitle="zsh", badge="$", color="#7ece8e",
           cwd="/path/to/demo/checkout", env={"HOME": "/path/to/throwaway/home"})
terms.watch("shell", r"passed|failed")   # new matching lines become "work" marks (pane, row, line)
take.ready()
terms.type("shell", "make test")
terms.wait_for("shell", r"\d+ passed", timeout=300)
take.mark("tests", row=...)
```

| Call | What it does |
|---|---|
| `open(name, title, subtitle, badge, color, argv=None, cwd=None, env=None, css=(1280, 720), scale=1.5)` | A pane running a clean bash (or `argv`), and its window. `env` is the pane's whole environment on top of a minimal default: give it a throwaway `HOME` whenever the command writes config. |
| `type(name, text, cps=22, enter=True)` / `key(name, key)` | Type a command, or press a tmux key (`Enter`, `Escape`, `C-c`, `Up`). |
| `screen(name)` / `wait_for(name, pattern, timeout)` | The pane's text now; wait for a regular expression in it. |
| `watch(name, pattern)` | Mark `work` whenever a new line matches. |

The terminal window loads xterm.js and its fonts from public CDNs, so the machine needs network access while recording terminals.

## Habits that make takes usable

- **Prepared state, off camera.** Sign in, seed data, open the right page before `take.ready()`. Use a demo account and made-up data; nothing private on screen, ever.
- **Mark generously.** A mark costs nothing; a missing one means guessing in the edit. Mark before a typed line (`typing`), after it lands, each result, and the finished state (`end`) with its box.
- **Measure boxes after the layout moves.** Opening a panel can shift everything: take the box after the click.
- **Hold the end.** Sleep a second or two after the last mark: the cold open and the GIF's first frame come from the finished state.
- **Second windows** go on the same origin as the first: `take.window("board", origin + "/")`, then `set_cookies(first.cookies())` and `reload()`.
- **One take per folder.** `run()` refuses to record over a take; use `take-2`, `take-3`. Keep takes in `~/recordings/<project>/`, never in a scratch directory and never in git.
- **Dry run first.** `--dry` saves `dry-<window>.png` at `take.ready()`: check legibility, the colour scheme and that nothing private shows before recording.
- **Clock steps.** If the recorder warns that the system clock stepped (common on WSL2), check the shots near that moment in `--stills`, or record another take.
