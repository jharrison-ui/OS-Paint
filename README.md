# OS-Paint

A real-time collaborative whiteboard. Draw on a shared canvas with friends over a peer-to-peer connection — no accounts, no installs beyond the app itself, just a room code.

Built in Rust with [macroquad](https://github.com/not-fl3/macroquad) for rendering and input handling, and [matchbox](https://github.com/johanhelsing/matchbox) for WebRTC peer-to-peer networking.

## Features

- **Shared canvas** — A fixed-size (3200×2400) drawing surface that everyone in a room views identically, panned and zoomed independently per user.
- **Tools** — Pen, point eraser, and stroke eraser (removes entire strokes at once), each with an adjustable size.
- **Color picker** — A quick-pick palette (default presets plus your most recently used colors) and a full hue/saturation color wheel.
- **Undo/redo** — Up to 50 historical steps, synchronized across peers so everyone converges on the exact same canvas state.
- **Live peer cursors** — See where other participants in the room are pointing and the color they are currently using.
- **Lobbies** — Host a room to generate a random 6-digit code, or join an existing room with a shared code. No registration required.

## Controls

| Action | Input |
|---|---|
| Draw | Left-click drag (Pen tool) |
| Erase | Left-click drag (Eraser / Stroke Eraser tool) |
| Pan | Right-click drag |
| Zoom | Mouse wheel |
| Change tool size | Up / Down arrow keys |
| Switch to Pen | `D` |
| Switch to Eraser | `E` |
| Switch to Stroke Eraser | `R` |
| Undo | Ctrl+Z |
| Redo | Ctrl+Y |
| Open menu (host/join/leave lobby) | Esc |

## Running from source

Requires a [Rust toolchain](https://rustup.rs/). On Linux, you will also need system libraries for windowing, graphics, and audio:

```bash
sudo apt install pkg-config libx11-dev libxi-dev libgl1-mesa-dev libasound2-dev
```

Run the application with:

```bash
cargo run --release
```

By default, the app connects to a public signaling server (`wss://matchbox-8uwy.onrender.com`) for matchmaking. To use a custom signaling server instead, set the `OS_PAINT_SIGNALING_SERVER` environment variable before launching:

```bash
OS_PAINT_SIGNALING_SERVER=wss://your-signaling-server.example {your binary}
```

## Prebuilt binaries

Every push builds Windows, macOS, and Linux binaries via GitHub Actions (see [.github/workflows/build.yml](.github/workflows/build.yml)). You can download them directly from the workflow run artifacts or from the [latest release](../../releases/tag/latest), which is updated automatically on every push to `master`.

## Project layout

| File | Responsibility |
|---|---|
| [src/main.rs](src/main.rs) | Application loop: state, input dispatch, undo/redo, lobby lifecycle |
| [src/canvas.rs](src/canvas.rs) | Canvas dimensions, pan/zoom clamping, and screen↔canvas coordinate mapping |
| [src/stroke.rs](src/stroke.rs) | `Stroke` and `Tool` types |
| [src/input.rs](src/input.rs) | Mouse and keyboard handling for drawing, erasing, panning, and zooming |
| [src/render.rs](src/render.rs) | Rendering strokes, canvas borders, and peer cursors to the screen |
| [src/ui.rs](src/ui.rs) | Toolbar, size slider, color wheel, and UI icons |
| [src/menu.rs](src/menu.rs) | Pause menu: host, join, and leave lobbies |
| [src/network.rs](src/network.rs) | Packet definitions and WebRTC peer sync (strokes, erases, snapshots, cursors) |

## How networking works

Rooms consist of all peers connected to the signaling server with the same 6-digit code — there is no dedicated central game server storing state. Each peer:

- Broadcasts individual draw and erase actions to connected peers in real time.
- Sends a full canvas snapshot to any newly joined peer, as well as periodically (every 30 seconds) to all peers, ensuring late joiners and dropped packets converge on the same canvas state.
- Resolves snapshot conflicts by revision number, always keeping the newest state.

## License

MIT — see [LICENSE](LICENSE).
