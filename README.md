# monody MCP server

[monody](https://monody.ai) is a DAW (digital audio workstation) with an MCP server built into the app. Claude Code, or another MCP client on the same computer, can read the session you have open and edit it: notes, sounds, devices, routing, automation and the mix.

Each `run_commands` call is one all-or-nothing transaction and one labeled undo step in the app, so ⌘Z walks back exactly what the model did.

This repository is documentation only. The server ships inside the monody desktop app, which is commercial and closed source. There is nothing to install from here.

- Setup guide: https://monody.ai/mcp
- Download: https://monody.ai/download
- Plans: https://monody.ai/pricing

## What you need

- **monody for Mac.** These steps were checked on the public Mac release (0.9.6) with Claude Code on the same computer. There is also an unsigned Windows test build; the MCP connection on Windows has not been verified.
- **A monody plan with MCP access:** Lifetime (one-time), Pro or Studio. Any client can connect and list the tools; tool calls need one of these plans.
- **An MCP client with its own model access.** Claude Code is the client we test with. Calls made over MCP use your client's model and do not spend monody's AI credits.

## How it connects

| | |
|---|---|
| Transport | Streamable HTTP, stateless |
| Endpoint | `http://127.0.0.1:<port>/mcp` (the app takes the first free port from 8710) |
| Protocol version | `2025-06-18` (`2025-03-26` is also accepted) |
| Auth | `Authorization: Bearer <key>`; the app creates the key |
| Capabilities | Tools |

The server listens only on your computer's loopback address. A client on another machine, or a web client, cannot reach it directly.

## Connect Claude Code

1. Open monody and sign in.
2. Go to **Settings → Account → Use monody from Claude**. It shows a command with your port and key:

   ```sh
   claude mcp add --transport http monody http://127.0.0.1:8710/mcp \
     --header "Authorization: Bearer <your key>"
   ```

3. Run the copied command in the folder where you use Claude Code, then start `claude` there.
4. Type `/mcp` in Claude Code to check the connection. "Added" only means the configuration was saved.

Keep monody open while you work. Keep the command private: it contains your key. **New key** in the same settings panel replaces the key and disconnects clients that use the old one.

## Tools

| Tool | What it does |
|---|---|
| `get_project` | Reads the whole session: tempo, tracks, clips, notes, device chains, automation, routing, transport, and the schema of every device |
| `get_selection` | Reads what you have selected in the app |
| `run_commands` | Applies a batch of monody commands as one labeled, undoable transaction; all or nothing |
| `action_history` | Groups several calls into one action, and lists, undoes or redoes those actions |
| `undo` / `redo` | Steps through history, yours and the model's |
| `transport` | Play, stop, seek, set the loop |
| `apply_sound_preset` | Puts one of monody's preset sounds, or a random one, on a MIDI track |
| `find_sounds` / `audition_sounds` | Searches the preset sounds by role and register, and renders up to three of them on your own notes to compare |
| `search_library` / `describe_library_sound` / `list_library_kits` | Searches the drum and sample library and lists the factory kits |
| `search_plugins` / `describe_plugin` / `search_plugin_presets` | Searches your installed AU, VST3 and CLAP plugins and reads their parameters and presets |
| `render_wav` | Renders the song to a WAV file in the project folder |
| `analyze_mix` | Renders the song and measures it for release: LUFS, true peak, balance, masking, sections |
| `analyze_range` | Measures one range of the song in a private render |
| `master_to_target` | Adjusts loudness toward a target as one undoable step and reports the measured result; on Mac, `mode: "tonal"` can also add one gentle EQ (at most 3 dB) |
| `separate_stems` | On Mac: separates one audio clip into vocals, drums, bass and other, then adds the parts you choose as new tracks in one undoable step |

The server lists more tools than this table. Your client's tool list shows the full set for your version of the app.

The command vocabulary ships with the server as MCP instructions, and `get_project` returns the schema of every device, so the model knows the commands and parameters before it changes anything.

## How edits behave

- **Checked before they apply.** Commands are validated first. If one command in a `run_commands` batch fails, nothing is applied and a structured error comes back for the model to fix.
- **One call, one undo step.** Each successful call is a single labeled step in the app's history, attributed to the AI. A request that takes several calls is several steps unless the client groups them with `action_history`.
- **Visible in the app.** The edits land in the session you have open. You can see them, play them and change them by hand.
- **Deterministic renders.** Offline renders are bit-exact, and the app's test suite checks them against a golden render.

## Try one change first

Work on a copy: **Project → Save a copy…**, then **Save and open copy**.

Ask the model to read without changing anything:

> Read the current project without changing anything. Tell me its name, the track names and their current volume settings, and which tracks have volume automation. Then wait.

Check the answer against the app, then ask for one small edit:

> Read the project again, then lower only "[track name]" by 3 dB in a single run_commands call. If the name is ambiguous, its volume is automated, or the change is refused, stop and explain. Report the before and after values, then wait.

Listen, and press ⌘Z in monody if you don't want it.

More requests once that works:

- "Route the drums to a new bus and put a bus compressor on it."
- "Add a tempo-synced delay to the lead, dotted eighth, mostly dry."
- "Analyze the mix for streaming and tell me the three biggest problems."

## Limits

- Verified on Mac with Claude Code on the same computer. Windows and other clients are not verified.
- Loopback only. A web client would need a tunnel with compatible authentication, which we have not tested.
- Tool calls need a Lifetime, Pro or Studio plan.
- `master_to_target` may miss its target. Read the measured LUFS and true peak, and listen before you keep the change.
- `separate_stems` needs a one-time model download that you start in the app, and separation is an estimate: sounds can spill between parts.
- A hosted plugin exposes its first 48 parameters.

## Other DAWs

This server drives monody only. For Ableton Live, REAPER and other DAWs, community-built MCP servers exist; monody's guides compare them: [Ableton MCP](https://monody.ai/blog/ableton-mcp), [REAPER and AI](https://monody.ai/blog/reaper-ai-workflows), [Claude for music production](https://monody.ai/blog/claude-for-music-production).

## Support

Email support@monody.ai. Please don't paste your key into an issue.

monody is not affiliated with Anthropic. Claude is a trademark of Anthropic.
