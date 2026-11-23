# Alpha voice assistant

Alpha is a Windows-first, voice-only desktop assistant. Its interface is a
small top-center Dynamic Island that stays completely hidden while dormant. It
appears as a translucent island after a wake phrase and remains visible while a
conversation is active.

The runtime is modular and event-driven: audio, STT, AI, automation, memory,
TTS, health monitoring, and UI rendering run as independent services. See
[`ARCHITECTURE.md`](ARCHITECTURE.md) for the service contracts, streaming
pipeline, fallback behavior, and performance diagnostics.

There is one desktop interface and no text-command box, full-screen legacy HUD,
typed CLI, or required BAT launcher.

## Conversation flow

1. Alpha starts silently when you sign in to Windows.
2. While dormant, the only accepted wake word is exactly **Alpha**:

   ```text
   Alpha, open Calculator
   ```

   You may also say **Alpha**, wait for the listening state, and then speak the
   request.
3. After each reply or completed task, Alpha keeps the island visible and
   listens for another request for **10 seconds**. During this active window,
   just speak naturally without another wake phrase:

   ```text
   Open Camera
   ```

4. While Alpha is speaking, start saying a new request to interrupt it. Alpha
   rejects likely loudspeaker echoes, stops the old response, and treats clear
   user speech as the next conversational turn. Say **stop** to silence the
   response while keeping the active conversation open.
5. After 10 seconds with no speech, the island hides. The next dormant
   activation must again begin with exact **Alpha**:

   ```text
   Alpha, open Camera
   ```

Say **goodbye Alpha** or **that's all** to end the active conversation
immediately. The microphone remains in lightweight local wake-phrase mode while
Alpha is running. Any person can activate it by saying exact **Alpha**.
**Hi Alpha**, **Hey Alpha**, **Hello Alpha**, **Wake up Alpha**, embedded
matches, and phonetic approximations are rejected.

## Windows installation

Open PowerShell in the project folder once and run:

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\install.ps1
```

The installer:

- creates the virtual environment when needed;
- installs Alpha and its voice dependencies;
- installs a verified current-user Windows Startup shortcut;
- launches Alpha silently with `pythonw.exe`;
- uses a watchdog to restart Alpha after an unexpected crash.

It does not require Administrator access. After installation, no VS Code,
PowerShell, Windows Terminal, folder opening, or BAT file is needed for normal
use.

If dependencies are already installed and only sign-in startup needs repair:

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\install-autostart.ps1
```

To remove automatic startup:

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\install-autostart.ps1 -Remove
```

## Interface

- Dormant: completely hidden so it does not cover any work.
- Active: a 94%-opacity island with state, waveform, and transcript preview.
- Follow-up: stays visible and listens for 10 seconds after each reply or task.
- Barge-in: clear speech can interrupt a spoken response and become the next turn.
- Details: click an active island to show time, CPU, RAM, and battery.
- Position: centered at the top of the active laptop screen.
- Behavior: always on top, frameless, and non-full-screen.
- Interrupt: say **Alpha stop**, or click the island while it is thinking/speaking.
- There is no manual show action; only exact **Alpha** may reveal the island.
- Tray control: **Stop current response** interrupts work without quitting.
- Exit: right-click Alpha's notification-area icon and choose **Quit Alpha**.
- Global launch/activate shortcut: press **Alt+F11** from any application.
- Global full-quit shortcut: press **Alt+F12**. The watchdog exits too.
- Alpha plays an original rising futuristic power-on cue when Alt+F11 makes
  listening ready, and a descending shutdown cue before Alt+F12 exits.

## Voice examples

| Say with `Alpha` while dormant, or directly during the 10-second active window | Result |
|---|---|
| `Alpha, open Brave` | Open Brave |
| `Alpha, open Brave and play some music on YouTube` | Play the first YouTube music result in Brave |
| `Alpha, open Notepad and type meeting notes` | Open Notepad and type into it |
| `Alpha, switch to Brave` | Focus the running Brave window |
| `Alpha, scroll down five` | Scroll the focused app |
| `Alpha, press Enter` | Press a key in the focused app |
| `Alpha, save` | Send Ctrl+S to the focused app |
| `Alpha, maximize this window` | Maximize the active window |
| `Alpha, remind me to stretch in ten minutes` | Save a local reminder |
| `Alpha, what are my tasks?` | Read pending tasks |
| `Alpha, what is on my screen?` | Describe the active app and important visible content |
| `Alpha, open Downloads` | Open Downloads |
| `Alpha, search YouTube for relaxing music` | Search YouTube |
| `Alpha, take screenshot` | Save a screenshot |
| `Alpha, volume down 20` | Lower system volume |
| `Alpha, system info` | Report system information |
| `Alpha, Chrome खोल।` | Open Chrome and answer in Nepali |
| `Alpha, VS Code open गर।` | Open VS Code and answer in Nepali |
| `Alpha, YouTube मा lo-fi music play गर।` | Play YouTube music from a mixed-language command |
| `Alpha, आजको काम के छ?` | Give the local briefing in Nepali |

Dangerous operations use spoken yes/no confirmation.
Typed secret phrases such as passwords, security codes, API keys, and tokens
are refused because voice commands may be present in local diagnostic logs.

Unknown natural-language automation requests can be planned by Alpha's AI manager using a
runtime catalog of Alpha's registered commands. Every plan is validated
against that local allowlist before execution; invented actions and parameters
are rejected, and dangerous registered actions still require confirmation.

## Persistent productivity and memory

Reminders, timers, tasks, notes, preferences, and conversation history are
stored locally under `%APPDATA%\ALPHA`. Personal memory fields are protected
with Fernet encryption when the installed configuration enables memory
encryption. Secret-bearing spoken commands are refused or redacted from command
history. Due reminders remain pending and silent while Alpha is dormant; ask
for reminders or a daily briefing after activation.

English, Nepali Unicode, and mixed Nepali-English command recognition are
automatic. Alpha answers in the latest language and switches when the user
switches. A local Nepali Windows voice is preferred. When Windows has none,
Alpha can use Microsoft's `ne-NP-SagarNeural` online voice; disable that text
transfer with `"nepali_online_tts": false` in `config.json`.

## Wake access

Alpha does not identify or restrict speakers. Anyone near the computer can
activate it with exact **Alpha**. Dangerous operations still require
spoken confirmation.

The complete product direction and honest implementation status are tracked in
[VISION_ROADMAP.md](VISION_ROADMAP.md).

## Microphone configuration

Alpha follows the Windows default input device unless `input_device` is set in:

```text
%APPDATA%\ALPHA\config.json
```

Use either a device index or part of its name:

```json
{
  "voice": {
    "input_device": "Microphone Array"
  }
}
```

Restart Alpha after changing the file. If Windows cannot open the microphone,
the island displays **Microphone unavailable** instead of claiming to be ready.

Also enable:

**Settings → Privacy & security → Microphone → Let desktop apps access your
microphone**

## Diagnostics

Normal use never needs a terminal. For a one-time foreground diagnostic only:

```powershell
.\.venv\Scripts\python.exe .\run.py
```

Logs are stored at:

```text
%APPDATA%\ALPHA\logs\alpha.log
%APPDATA%\ALPHA\logs\watchdog.log
```

The lightweight wake model loads at startup. The larger multilingual command
model loads only after the first exact activation, so that first request can
take several extra seconds before the island says **Listening**.

## Current source layout

```text
run.py
scripts/
├── install.ps1
├── install-autostart.ps1
└── alpha_watchdog.py
src/alpha/
├── app.py
├── ui/
│   ├── island_window.py
│   └── waveform.py
├── voice/
│   ├── engine.py
│   ├── wake_word.py
│   ├── stt.py
│   └── tts.py
└── tests/
```

## AI conversation routing

Alpha's assistant code makes every conversational request through
`AIManager.ask(...)`. A local intent classifier sends programming, debugging,
software-engineering, essay, story, and poetry requests to Claude. Greetings,
small talk, everyday questions, weather, time, translation, advice, travel,
music, and uncertain requests use OpenAI. If the selected service fails,
Alpha silently tries the other one without announcing a provider change.

Place one `OPENROUTER_API_KEY` in the project-root `.env` file. Both routes use
that shared key and `OPENROUTER_BASE_URL`; only `OPENROUTER_OPENAI_MODEL` or
`OPENROUTER_CLAUDE_MODEL` changes after intent classification. The file is ignored by Git,
credentials are never written to `config.json` or logs, and Alpha's own
session and encrypted long-term memory are passed to either model so context
remains intact when routing changes.

Natural Nepali neural speech uses a separate Microsoft online TTS connection
when enabled; it does not use the OpenRouter conversational AI credential.

## Screen awareness

Screen analysis is opt-in and runs only after an explicit voice command such as
`what is on my screen`, `read my screen`, or `summarize my screen`. Alpha
captures one temporary PNG, sends it to the configured Gemini model for image
understanding, and deletes the local image immediately after the response. Do
not use this command while passwords, authentication codes, payment details, or
other sensitive material are visible.
