# keyboard.wtf

**Stop typing. Say it.**

> Built with AI for the Devpost "Build with AI: Basics" Hackathon  
> A voice-first Windows desktop app that turns speech into action through AI-powered transcription, smart writing, and an intelligent assistant.

## 🏆 Hackathon Submission

This project was developed using the **Devpost Learn Skill Pack** following an AI-first development approach:
- ✅ **Scoped** with clear problem definition and boundaries ([scope.md](scope.md))
- ✅ **Planned** with comprehensive product requirements ([prd.md](prd.md))
- ✅ **Spec'd** with detailed technical architecture ([spec.md](spec.md))
- ✅ **Built** with iterative AI assistance and structured implementation

## 💡 The Problem

Modern computer work is dominated by typing, which is:
- **Slow**: Even fast typists max out at 70-100 WPM vs 150+ WPM for speech
- **Disruptive**: Switching between thinking and typing breaks concentration  
- **Physically demanding**: Hours of typing cause strain and fatigue
- **Error-prone**: Typos, formatting issues, and constant copy-paste

Existing voice solutions fall short:
- Traditional dictation requires rigid phrasing and lacks AI cleanup
- Voice assistants (Alexa, Siri) can't type into desktop apps
- AI chatbots require manual copy-paste between tools

## 🎯 The Solution

keyboard.wtf makes voice the primary input method through **three specialized modes**:

### 1️⃣ Dictation Mode (`Ctrl+Alt+D`)
Fast local transcription with no AI rewriting — perfect for note-taking and forms.
- 100% offline using Vosk speech recognition
- Types exactly what you say
- Real-time partial transcripts

### 2️⃣ Smart Writing Mode (`Ctrl+Alt+K`)  
AI-polished writing that removes fillers, adds punctuation, and fixes grammar.
- Removes "um", "uh", "like" automatically
- Honors self-corrections ("no wait, I mean...")
- Maintains your meaning and tone
- Perfect for emails, documents, messages

### 3️⃣ Jarvis Mode (`Ctrl+Alt+Q`)
Conversational AI assistant with safe computer control.
- Open apps, files, and folders by voice
- Control browser tabs (Chrome, Edge, Firefox, Brave, Opera, Vivaldi, Arc)
- Compose email drafts (Gmail)
- Search YouTube, Spotify, Amazon
- Take screenshots, camera photos
- Manage system settings
- Create reusable workflows
- **Safety-first**: Allowlisted actions, confirmation gates, no auto-submission

## ✨ Key Features

- **🎤 Three distinct voice modes** via global hotkeys
- **👁️ Non-intrusive voice orb UI** showing real-time status without stealing focus
- **🔒 Offline speech recognition** with Vosk (fast) and Whisper (accurate)
- **🤖 Gemini Live conversation** with bidirectional voice and interruption support
- **🛡️ Safety boundaries**: Ask vs Auto-execute modes, confirmation gates, no arbitrary command execution
- **🧠 Intent memory**: Jarvis remembers your preferences (20-entry cap)
- **⚡ Workflow automation**: Save multi-step routines and trigger by voice
- **🔐 Local-first privacy**: API keys encrypted with Windows DPAPI, no telemetry
- **📝 Multiple destinations**: Email, Slack, Discord, Teams, WhatsApp, Notion, Telegram
- **🚀 Start with Windows**: Background tray app with instant global hotkey access

## 🎮 Default Hotkeys

| Hotkey | Action |
|--------|--------|
| `Ctrl+Alt+K` | Smart writing: clean spoken thoughts and type polished text |
| `Ctrl+Alt+D` | Dictation: type recognized words without AI rewriting |
| `Ctrl+Alt+Q` | Jarvis mode: chat, open apps, prepare drafts, run safe actions |
| `Ctrl+Alt+X` | Cancel listening, transcription, or conversation |
| `Ctrl+Alt+,` | Open settings |

## 🚀 Quick Start

### For Users

1. **Download** the Windows installer: [keyboard-wtf-setup.exe](https://keyboard-wtf.vercel.app)
2. **Run** the installer (no admin required, per-user install)
3. **Launch** keyboard.wtf from the Start Menu or system tray
4. **Configure** your API key and microphone in Settings (`Ctrl+Alt+,`)
5. **Try it**:
   - Open Notepad
   - Press `Ctrl+Alt+D`, speak a sentence, and watch it appear
   - Press `Ctrl+Alt+K`, speak with fillers ("um, actually..."), and see polished text
   - Press `Ctrl+Alt+Q`, say "Open Notepad" or "What time is it?"

### For Developers

**Prerequisites:**
- Windows 10/11 (x64)
- .NET 8 SDK
- Microphone
- Optional: Gemini API key for AI features (free tier works)

**Build & Run:**
```powershell
git clone https://github.com/UnknownGod2011/keyboard.wtf-hackathon.git
cd keyboard.wtf-hackathon
dotnet restore
dotnet build KeyboardWtf.sln
dotnet run --project src/KeyboardWtf.csproj
```

The app starts in the Windows tray. Double-click the tray icon or press `Ctrl+Alt+,` to open settings.

**Optional: Set Gemini API key without committing secrets:**
```powershell
[Environment]::SetEnvironmentVariable("KEYBOARD_WTF_GEMINI_API_KEY", "your-key-here", "User")
```

On next launch, keyboard.wtf imports that key into encrypted local settings.

## 📐 Architecture

Built with **.NET 8** and **Windows Forms** for speed and simplicity:

- **Audio Capture**: NAudio (16kHz microphone recording)
- **Transcription**: Vosk (fast local) + Whisper.net (accurate local)
- **AI Providers**: Gemini (default), Claude, OpenAI
- **Voice Assistant**: Gemini Live WebSocket (bidirectional speech-to-speech)
- **TTS**: Piper (local) with Windows SAPI fallback
- **Security**: Windows DPAPI encryption for API keys
- **UI**: Custom voice orb overlay (always-on-top, click-through, transparent)
- **Actions**: Allowlisted Windows API calls (SendInput, Process.Start, ShellExecute)

See [spec.md](spec.md) for detailed technical architecture.

## 🛡️ Safety & Privacy

### Privacy Boundaries
- ✅ **100% offline dictation** (Vosk/Whisper) — no cloud API calls
- ✅ **Local-first storage** — all settings, memory, logs in `%LOCALAPPDATA%`
- ✅ **Encrypted secrets** — API keys protected with Windows DPAPI
- ✅ **No telemetry** — zero usage tracking or analytics
- ✅ **User-gated actions** — screen capture, camera access require explicit approval

### Safety Boundaries
- ✅ **Allowlisted actions only** — no arbitrary command execution
- ✅ **Confirmation gates** — destructive actions (shutdown, restart, delete) always require "confirm"
- ✅ **No auto-submission** — emails and messages always opened as drafts for review
- ✅ **Browser-specific commands** — tab control only works when that browser is foreground
- ✅ **Bounded resources** — memory (20 intents), history (50 actions), aliases (80 mappings)

### Permission Modes
- **Ask Mode**: Every action requires verbal confirmation ("Say 'confirm' or 'cancel'")
- **Auto-Execute Mode**: Explicit routine actions run immediately; destructive actions still gated

## 🎬 Demo Use Cases

### 1. Professional Email in Seconds
**Voice**: "Send a professional email to sarah@company.com saying I'll have the report ready by Friday afternoon"  
**Result**: Gmail opens with recipient, subject "Report Update," and polished body — ready to click Send

### 2. Hands-Free Note-Taking
Open Notepad → Press `Ctrl+Alt+D` → Speak continuously for 30 seconds → Text appears verbatim

### 3. Workflow Automation
**Voice**: "Whenever I say initialize my setup, open Chrome, VS Code, and my project folder"  
**Later**: "Initialize my set" → All three open in sequence

### 4. Browser Control
With Chrome active:  
**Voice**: "Open a new tab" → "Go to YouTube" → "Open my liked videos" → "Next tab" → "Close this tab"

### 5. Smart Writing for Messages
Press `Ctrl+Alt+K` → Say: "Um, I finished the prototype, actually the full demo, and I will send it soon"  
**Types**: "I finished the full demo prototype and will send it soon."

### 6. Context-Aware Assistance
Select text in any app → Press `Ctrl+Alt+Q` → Say: "Summarize this" or "Make this professional"  
**Jarvis**: Uses selected text as context, responds with summary or rewrite

## 📊 What I Learned

Building keyboard.wtf with AI assistance taught me:

1. **Planning prevents chaos**: Using the Devpost Learn skills to scope, PRD, and spec *before* coding prevented the "rewrite-everything-later" trap that plagues AI-assisted projects.

2. **Voice UX is hard**: Pause detection, real-time feedback, and error recovery took more iteration than the core transcription. Users need to see *and* hear what's happening.

3. **Safety boundaries are essential**: An AI that can "do anything" is terrifying. Allowlists, confirmation gates, and clear permission modes make Jarvis helpful instead of dangerous.

4. **Local-first works**: Offline Vosk transcription is fast enough for real-time dictation. You don't always need the cloud.

5. **Structured iteration beats "one-shot prompts"**: The skill pack's interview-driven approach surfaced edge cases (elevated apps, hotkey conflicts, model download failures) I wouldn't have thought of upfront.

## 🏗️ Project Structure

```
keyboard.wtf-hackathon/
├── scope.md                    # Project scope (what's in, what's out)
├── prd.md                      # Product requirements document
├── spec.md                     # Technical specification
├── README.md                   # This file (hackathon-focused)
├── LICENSE                     # MIT License
├── KeyboardWtf.sln            # Visual Studio solution
├── src/                        # Main application source
│   ├── Program.cs             # Entry point, single-instance guard
│   ├── VoiceOverlayForm.cs    # Voice orb UI
│   ├── KeyboardWtf.csproj     # .NET project file
│   ├── Services/              # Audio, transcription, AI, actions
│   ├── Destinations/          # Email, Slack, Discord formatters
│   ├── Helpers/               # Utilities, extensions
│   └── Models/                # Configuration, DTOs
├── installer/                  # Inno Setup script for Windows installer
├── scripts/                    # Build automation (PowerShell)
├── site/                       # Landing page (Vercel deployment)
├── tests/                      # Unit and integration tests
└── docs/                       # Additional documentation
```

## 🧰 Built With

- **.NET 8** — Cross-platform runtime (Windows desktop)
- **Windows Forms** — Lightweight UI framework
- **NAudio** — Audio recording and playback
- **Vosk** — Lightweight offline speech recognition
- **Whisper.net** — High-accuracy local transcription
- **Gemini Live** — Conversational AI with bidirectional voice
- **Piper TTS** — Local text-to-speech
- **OpenCvSharp** — Image processing for screen guidance
- **Windows DPAPI** — Encryption for API keys

## 📄 License

MIT License — Copyright (c) 2026 Tanush Shah

See [LICENSE](LICENSE) for full text.

## 🤝 Contributing

keyboard.wtf is open source! Contributions welcome:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 🙏 Acknowledgments

- **Devpost** for the "Build with AI: Basics" hackathon and Learn Skill Pack
- **Name.com** for the Domain Roulette challenge that inspired this project
- **Google** for Gemini API and Gemini Live
- **Vosk** and **Whisper** teams for open-source speech recognition
- **Piper** TTS project for local voice synthesis
- The open-source community for amazing .NET libraries

## 📞 Contact

**Tanush Shah**  
- GitHub: [@UnknownGod2011](https://github.com/UnknownGod2011)
- Email: unknowngod2024@gmail.com

---

**Built with AI. Designed for humans. Made for the Devpost "Build with AI: Basics" Hackathon.**
