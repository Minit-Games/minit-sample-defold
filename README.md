# Defold Bouncy Ball

> **Learn page:** [Defold on Minit](https://minit.studio/docs/defold) — the official guide.

A **sample**: a finished, working Minit game built on the skeleton's structure,
to read and learn from.

- Tap the ball to bounce it; each tap scores.
- A 30 second clock ends the run and reports the result.
- To start a new game, use the template instead:
  <https://github.com/Minit-Games/minit-template-defold>.

## Cloning: this repo uses Git LFS

- Art, audio and the font are in [Git LFS](https://git-lfs.com).
- GitHub Desktop asks to initialize Git LFS: click **Initialize**.
- Command line: run `git lfs install` once, before cloning.
- Cloned without it (assets are small text pointers)? Run `git lfs install`, then `git lfs checkout`.
- GitHub's **Download ZIP** already includes the real files.

## Steps (all in the Defold editor)

1. **Get the project.** Download ZIP or clone this repo, then in Defold:
   **Open From Disk** → `game.project`.
2. **Project → Fetch Libraries.** The
   [Minit SDK](https://github.com/Minit-Games/minit-defold), declared in `game.project`,
   downloads as a `minit` folder under **Dependencies** in Defold's Assets pane, not into
   your project folder. Quick check: press Ctrl+P and type `minit.lua`.
3. **Build** or **Build HTML5** to play. Audio unlocks on the first tap.
4. **The name** is in Project Settings → **Title** (`Defold Bouncy Ball 🏐🌱`).
   Players see it.
5. **The game** is in `main/game.script`.
6. **The description** is in `meta.json`.
   - `controls` and `logic` appear under **How to play** in the ⓘ sheet under the game.
   - `description` sits behind **Show more**.
   - minit.studio reads them on the **first upload only**; edit them there later.
   - The title is not in `meta.json`.
   - This sample's texts are a worked example of good store copy.
   - References: [meta.json reference](https://minit.studio/docs/meta-json-reference),
     [Writing your description](https://minit.studio/docs/writing-your-description),
     [Limits & Constraints](https://minit.studio/docs/limits-and-constraints).
7. **Project → Minit: Package for Upload.** It validates the metadata and bundle
   structure and lists what is missing. It does not check gameplay or whether
   audio is audible, so play-test that yourself. It writes `dist/Defold Bouncy Ball.zip` (emoji are dropped from the
   file name, kept in the uploaded title).
8. **Upload** that ZIP at minit.studio.
9. **Test on your phone:** on the game's page in minit.studio, click the QR button in the Live preview panel, then Generate preview link, and scan the code. With the Minit app installed, the game opens in the app; without it, it plays on a web page with links to download the app. The link works for 15 minutes. See [Testing Your Game on Device](https://minit.studio/docs/sharing-a-preview-link).

## Don't edit `minit_platform/`

- It makes the game behave inside the Minit app: audio, the viewport layout, touch mapping.
- Files: `audio.lua`, `layout.lua`, `minit.render`, `minit.render_script`,
  `game.input_binding`.
- Game code goes in `main/` and `modules/`.

## What the game shows you

| Where | What it demonstrates |
|---|---|
| `main/game.script` → `read_config()` | Config values, with defaults and clamping. Booleans follow the backend's coercion: anything but `"true"` is false. |
| `main/game.script` → `init()` | `minit.loading_done()` at the end, once the scene is built and placed. |
| `main/game.script` → `finish()` | `minit.report_result()` **exactly once**, with `flavor_text`, a persisted `user_data`, and a `delay` so the outro is seen. |
| `main/game.script` → `init()` | `minit.get_user_data()` read back as the player's previous best. |
| `minit_platform/audio.lua` | The audio rules that matter inside the app. Read this before touching sound. |
| `minit_platform/layout.lua` | No design resolution: every number derived from the live viewport. |
| `modules/ui.lua` | Small helpers over the quad / nine-slice / text factories. |
| `meta.json` | The three config keys the game reads, plus store copy and credits. |
| `minit/editor/minit_package.lua` (SDK dependency) | The packaging menu item. Also callable over the editor's HTTP `/eval` with `return require("minit.editor.minit_package").run().ok` — see its header. |

## Audio inside the app

Sound is the one thing that behaves differently inside the Minit app than in a
browser, and it fails **silently in both directions**. `minit_platform/` and the
SDK's HTML shell (`minit/minit.html`) already handle it; the pieces are load-bearing.

- **The host's volume message can be discarded.**
  - The app sends volume with `window.postMessage(payload, window.location.origin)`.
  - On iOS the game is served from a custom scheme with an opaque origin, so
    `location.origin` is `"null"` and `postMessage` throws.
  - The game then only sees the volume seeded at mount time, often `0`: healthy but inaudible.
  - `minit/minit.html` retries a rejected same-window message with `'*'`, so the host's
    real intent (including a deliberate mute) flows through.
- **Defold discards audio while its context is suspended.**
  - A loop started too early does not queue: its opening is gone.
  - `audio.music_start()` refuses until the context reports running; `game.script`
    retries four times a second until it takes.
- **The host owns the output gain.**
  - It fades a mute gain up; if that fade does not land, the game is healthy and inaudible.
  - `minit/minit.html` resumes a suspended context on gestures, visibility/focus
    changes and a watchdog, and re-applies the host's own target volume.
  - It never touches the gain when the host has deliberately muted or ducked.
- **Nothing plays before the game starts.** `audio.set_active(true)` runs on the
  first tap, which is also the gesture that resumes the context.

## Two more things that are not obvious

- **There is no design resolution.**
  - The render script projects `0..window_width` by `0..window_height`; a world
    unit is a backbuffer pixel, and `layout.lua` rebuilds from `window.get_size()` on change.
  - The app's game slot is roughly **2:3**, far wider relative to its height than
    a phone screen (it sits between the app's header and toolbar).
  - A game drawn to a fixed design width leaves a band down one side, invisible
    in a desktop browser at a phone viewport.
  - This game's own geometry (`field`, `measure()` in `main/game.script`) is derived
    from `layout.width/height/unit`.
- **Touch coordinates are not backbuffer pixels.**
  - `window.get_size()` returns **backbuffer** pixels (1170×2532, not 390×844).
  - `action.x/y` and every `action.touch[i].x/y` are the normalised position times
    the `game.project` display size: a stretch mapping that ignores the real aspect ratio.
  - Touch entries carry only `x`/`y`; there is no per-finger `screen_x`.
  - `layout.to_world()` does the conversion; `on_input` handles a touch list and a mouse.

## Platform notes

- `[input] use_accelerometer` is off: the platform forbids device hardware, and it defaults to on.
- `[display] update_frequency = 60` caps the frame rate.
- A custom HTML5 shell **must** keep an element with id `app-container`;
  `dmloader.js`'s resize callback sets `.style` on it and throws otherwise.
- The bundle loads its own `archive/` payload over `XMLHttpRequest` from inside
  the ZIP. The game makes no network calls and touches no web storage.

## Assets

- Generated by `tools/gen-art.mjs`, `tools/gen-audio.mjs` and `tools/gen-music.mjs`
  (helper: `tools/painter.mjs`).
- Maintainer-only: needs Node 22+, and is not part of building or packaging.
  The committed files are what ships.
- The music is the one third-party asset (a CC0 chiptune).
- Credits and licenses: `THIRD-PARTY-NOTICES.txt`.
