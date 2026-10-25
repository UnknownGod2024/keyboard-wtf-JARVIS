# Product Requirements Document: keyboard.wtf

## Executive Summary

**Product Name**: keyboard.wtf  
**Tagline**: Stop typing. Say it.  
**Category**: Productivity, Voice-First Computing, AI Assistant  
**Platform**: Windows Desktop  
**License**: MIT Open Source  

keyboard.wtf is a voice-first Windows desktop application that replaces typing with speech through three distinct modes: raw dictation, AI-polished writing, and an intelligent assistant with safe computer control. Built with local-first speech recognition and AI provider flexibility, it prioritizes privacy, speed, and user control.

## Problem Statement

Modern computer work is dominated by typing, which is:
- **Slow**: Even fast typists max out at 70-100 WPM vs 150+ WPM for speech
- **Disruptive to flow**: Switching between thinking and typing breaks concentration
- **Physically demanding**: Hours of typing cause strain and fatigue
- **Error-prone**: Typos, formatting issues, and copy-paste mistakes

Existing solutions fall short:
- Traditional dictation software requires rigid phrasing and lacks AI cleanup
- Voice assistants (Alexa, Siri) can't type into desktop apps
- AI chatbots require manual copy-paste between tools
- Enterprise voice tools lack consumer-friendly UX

## Solution Overview

keyboard.wtf makes voice the primary input method through global hotkeys that activate three specialized modes:

1. **Dictation (Ctrl+Alt+D)**: Fast local transcription with no AI rewriting
2. **Smart Writing (Ctrl+Alt+K)**: AI removes fillers, adds punctuation, fixes grammar
3. **Jarvis (Ctrl+Alt+Q)**: Conversational AI that types, searches, opens apps, and assists

A non-intrusive voice orb shows real-time status, and all text appears directly in the active application or clipboard as fallback.

## Target Users

### Primary Personas

**1. The Knowledge Worker (Sarah, 32)**
- Role: Product Manager
- Pain: Writes 50+ emails/day, takes meeting notes, updates docs
- Goal: Compose professional communications 2x faster
- Use Case: Smart Writing for emails, Dictation for quick notes

**2. The Developer (Alex, 28)**
- Role: Software Engineer
- Pain: Context-switching between IDE, docs, and communication
- Goal: Control computer without leaving keyboard/voice flow
- Use Case: Jarvis opens apps/files, searches Stack Overflow, drafts messages

**3. The Content Creator (Jordan, 35)**
- Role: Technical Writer
- Pain: First drafts take too long, editing is tedious
- Goal: Get ideas down quickly, let AI handle grammar/flow
- Use Case: Dictation for rough drafts, Smart Writing for polished prose

### Secondary Personas

**4. Accessibility Users**
- Pain: Physical limitations make typing difficult
- Goal: Reliable voice input without fatigue
- Use Case: All three modes for daily computer use

**5. Remote Workers**
- Pain: Lots of chat messages, Zoom fatigue, async communication
- Goal: Faster Slack/email responses, hands-free when multitasking
- Use Case: Smart Writing for polished messages, Jarvis for quick searches

## User Stories

### Core Workflows

#### Story 1: Quick Email Response
> As a busy professional, I want to compose a polished email without typing, so I can respond faster while maintaining professionalism.

**Steps**:
1. Press `Ctrl+Alt+Q` (Jarvis)
2. Say: "Send a professional email to sarah@company.com saying I'll have the report ready by Friday afternoon"
3. Review pre-filled Gmail draft
4. Click Send

**Acceptance Criteria**:
- Gmail opens with recipient, subject, and body filled
- Message is grammatically correct and professional
- User manually clicks Send (no auto-submission)

#### Story 2: Dictation for Note-Taking
> As a developer in a meeting, I want to capture spoken ideas verbatim, so I don't miss important details.

**Steps**:
1. Open Notepad
2. Press `Ctrl+Alt+D`
3. Speak continuously for 30 seconds
4. Automatic pause detection stops recording
5. Text appears in Notepad

**Acceptance Criteria**:
- Transcription appears within 2 seconds
- Accurate word recognition (>90%)
- Works offline without internet
- No AI rewriting of spoken words

#### Story 3: App Control with Jarvis
> As a multitasker, I want to open apps and files by voice, so I don't lose focus by hunting through menus.

**Steps**:
1. Press `Ctrl+Alt+Q`
2. Say: "Open Visual Studio Code"
3. App launches
4. Say: "Open my project folder"
5. Explorer opens correct location

**Acceptance Criteria**:
- Fuzzy matching handles "VS Code" vs "Visual Studio Code"
- Low-confidence matches present selection dialog
- File search covers user-approved safe locations
- Action history logs what was opened

## Features

### 1. Voice Input Modes

#### 1.1 Dictation Mode (`Ctrl+Alt+D`)
- **Purpose**: Type exactly what was said, no AI cleanup
- **Transcription**: Local Vosk (fast) or Whisper (accurate)
- **Output**: Raw transcript typed into active window
- **Use Cases**: Note-taking, forms, code comments
- **Privacy**: 100% offline, no cloud API calls

#### 1.2 Smart Writing Mode (`Ctrl+Alt+K`)
- **Purpose**: Clean spoken thoughts into polished writing
- **Processing**: AI removes fillers ("um", "uh"), fixes grammar, adds punctuation
- **Prompt Strategy**: "Remove filler words, honor corrections, maintain meaning"
- **Output**: Professional text typed into active window
- **Use Cases**: Emails, documents, messages, social posts

#### 1.3 Jarvis Mode (`Ctrl+Alt+Q`)
- **Purpose**: Conversational AI assistant with safe computer actions
- **Voice**: Gemini Live bidirectional speech (12 selectable voices)
- **Tools**: Allowlisted app/file/browser/productivity actions
- **Memory**: Bounded 20-entry intent memory for preferences
- **Safety**: Ask vs Auto-execute permission modes

### 2. Voice Orb UI

#### Visual Feedback States
- **Listening**: Animated waveform showing microphone level
- **Transcribing**: Partial transcript (Vosk) or "Processing..." (Whisper)
- **Thinking**: AI provider processing (Gemini/Claude/OpenAI)
- **Executing**: Action being performed
- **Speaking**: Jarvis audio output
- **Done**: Success checkmark, auto-dismiss after 2s
- **Cancelled**: Manual cancel via `Ctrl+Alt+X`
- **Error**: Red indicator with message

#### Design Principles
- Top-center screen position
- Always-on-top without stealing focus
- Semi-transparent when idle
- Keyboard-accessible (ESC dismisses)
- Color-coded by mode (blue=dictation, green=smart, purple=Jarvis)

### 3. Jarvis Assistant Capabilities

#### 3.1 App & Window Management
- Open installed apps (fuzzy name matching)
- Minimize, maximize, restore, close windows
- Switch between open windows
- Create, switch, close virtual desktops (with confirmation)

#### 3.2 Browser Control
- New tab, close tab, switch tabs, reopen closed
- Navigate back, forward, refresh
- Open downloads, history, find in page
- Supports Chrome, Edge, Firefox, Brave, Opera, Vivaldi, Arc
- Commands only work when browser is foreground (safety)

#### 3.3 File & Folder Access
- Fuzzy-search Desktop, Downloads, Documents, Pictures, Videos, Music
- Open folders in Explorer
- Open safe file types (no .exe, .bat, .msi, .reg)
- User-approved learned folder locations (bounded to 80 entries)
- Search limited by depth, scan count, timeout

#### 3.4 Content Awareness
- Use clipboard content: "Summarize this"
- Use selected text: "Make this professional"
- Active window context: "What app am I using?"
- Gmail draft composition (pre-filled, manual send)

#### 3.5 Productivity Actions
- Save notes (local JSON storage)
- Manage to-do list
- Set timers (in-app countdown)
- Search YouTube, Spotify, Amazon
- Open Liked Songs/Videos, subscriptions, history

#### 3.6 System Control
- Volume up/down, mute/unmute
- Open settings (Wi-Fi, Bluetooth, Display, Sound)
- Check battery, system info, uptime
- Screen capture, camera photo (one-shot, user-gated)
- Start/stop recording (sends Win+Alt+R to Game Bar)

#### 3.7 Workflows & Automation
- Create named routines: "Whenever I say [trigger], [actions]"
- Fuzzy phrase matching: "initialize my set" → "initialize my setup"
- Workflows run in sequence: apps, files, URLs
- Stored locally, manageable in Settings
- Bounded to reasonable complexity (no loops or conditionals)

### 4. Safety & Permissions

#### 4.1 Permission Modes
- **Ask Before Actions**: Every action requires verbal "confirm"
- **Auto-Execute Routine Actions**: Explicit requests (app open, file access) run immediately
- Destructive actions always gated: shutdown, restart, sleep, delete, Wi-Fi disable

#### 4.2 Allowlist Philosophy
- No arbitrary command execution
- No generic keyboard/mouse simulation beyond typing
- No auto-clicking or form submission
- No silent browser changes
- No access outside approved safe paths

#### 4.3 Confirmation Gates
- Two-turn confirmation for sensitive actions
- Preview what will happen before executing
- "Are you sure?" for irreversible operations
- Action history capped at 50 entries for audit

#### 4.4 Privacy Boundaries
- Screen guidance: one-shot, in-memory only (not saved)
- Camera: one-shot, explicit permission, saved locally
- No continuous recording or screen watching
- No telemetry or usage analytics
- API keys encrypted with Windows DPAPI

### 5. Settings & Configuration

#### 5.1 Accessible Via
- Tray icon menu
- Global hotkey: `Ctrl+Alt+,`
- Voice command: "Open settings"

#### 5.2 Configuration Options
- **Audio**: Microphone selection, volume levels, pause threshold
- **Hotkeys**: Customize all shortcuts, view active bindings
- **Transcription**: Choose Vosk vs Whisper, model downloads
- **AI Provider**: Gemini (default), Claude, OpenAI, DeepSeek (placeholder), Perplexity (placeholder)
- **API Keys**: Encrypted storage, masked display
- **Jarvis Voice**: 12 Gemini Live voices (Kore default)
- **Jarvis Tone**: Concise, Balanced, Detailed, Casual
- **Permission Mode**: Ask vs Auto-execute
- **Memory**: View/delete saved intents (20 max)
- **Workflows**: View/edit/delete routines
- **Aliases**: View/delete learned mappings (80 max)
- **Destinations**: Configure webhooks for Slack, Discord, Teams
- **Startup**: Start with Windows (enabled by default)

### 6. Installation & Setup

#### 6.1 End User Installation
1. Download from `https://keyboard-wtf.vercel.app`
2. Run `keyboard-wtf-setup.exe` (per-user, no admin)
3. Installer extracts to `%LOCALAPPDATA%\Programs\keyboard.wtf`
4. App auto-starts, opens setup page if no API key
5. User configures API key, microphone, hotkeys
6. Registered to start with Windows

#### 6.2 Developer Setup
1. Install .NET 8 SDK
2. Clone repository
3. `dotnet restore && dotnet build KeyboardWtf.sln`
4. `dotnet run --project src\KeyboardWtf.csproj`
5. Optional: Set `KEYBOARD_WTF_GEMINI_API_KEY` environment variable

#### 6.3 Model Management
- Vosk model: Auto-downloaded to `%LOCALAPPDATA%\keyboard.wtf\models\vosk`
- Whisper model: Optional download via Settings (~1.5 GB)
- Piper TTS voices: Bundled with installer
- Graceful degradation if models missing

### 7. Destinations & Formatting

#### 7.1 Raw Destinations
- **Clipboard**: Copy transcript (no typing)
- **Type Out**: Simulate keystrokes in active window

#### 7.2 AI Destinations
- **Email**: Format for Gmail draft
- **Slack**: Use casual tone, shorter paragraphs
- **Discord**: Similar to Slack, emoji-friendly
- **Teams**: Professional, structured
- **Calendar**: Parse dates/times (future feature)
- **WhatsApp**: Informal, conversational
- **Notion**: Markdown formatting
- **Telegram**: Similar to Slack

#### 7.3 Destination Configuration
- Webhook URLs for Slack/Discord/Teams
- DPAPI-encrypted storage
- Optional custom prompts per destination
- Fallback to clipboard if typing fails

## Non-Functional Requirements

### Performance
- **Latency**: Dictation transcript within 2 seconds
- **Responsiveness**: Voice orb updates <100ms
- **Memory**: <200MB RAM idle, <500MB during Jarvis conversation
- **Disk**: ~50MB app + ~50MB Vosk + optional 1.5GB Whisper

### Reliability
- **Graceful Degradation**: Missing models don't crash app
- **Fallback Paths**: Clipboard if typing fails, SAPI if Piper missing
- **Error Handling**: User-friendly messages, never silent failure
- **Single Instance**: Prevent multiple app instances

### Security
- **Encryption**: DPAPI for API keys and webhooks
- **No Plain Text Secrets**: Keys masked in UI
- **Bounded Resources**: Memory/history/aliases capped
- **No Telemetry**: Zero data sent to developer

### Compatibility
- **Windows**: 10, 11 (x64)
- **.NET**: 8.0 or later
- **Browsers**: Chrome, Edge, Firefox, Brave, Opera, Vivaldi, Arc
- **Microphone**: Any Windows-compatible audio input

### Accessibility
- **Keyboard-Only**: All features via hotkeys, no mouse required
- **Screen Reader**: Settings UI follows Windows accessibility patterns
- **Visual Feedback**: Color + text + animation redundancy
- **Customizable**: Hotkeys, pause thresholds, text size

## Success Metrics

### Adoption (Month 1)
- 1,000+ downloads
- 300+ active weekly users
- 70%+ installation completion rate

### Engagement
- Average 20+ voice commands per active user per day
- 60%+ return next day after first use
- <10% error/cancellation rate

### Mode Distribution
- Dictation: 30-40% of usage
- Smart Writing: 40-50% of usage
- Jarvis: 10-20% of usage

### Qualitative
- Net Promoter Score >40
- <5% negative feedback on privacy/security
- Top feature request: macOS version

## Release Strategy

### Phase 1: Soft Launch (Week 1)
- Name.com Domain Roulette demo submission
- GitHub repository public
- Landing page live at keyboard-wtf.vercel.app
- Reddit r/productivity post

### Phase 2: Community Feedback (Week 2-4)
- Hacker News Show HN post
- Product Hunt launch
- YouTube demo video
- Iterate based on bug reports

### Phase 3: Broader Distribution (Month 2+)
- Windows Store submission (optional)
- Scoop/Chocolatey package managers
- Technology blog outreach
- Document use cases and workflows

## Future Roadmap (Out of Scope for MVP)

### Short Term (3-6 months)
- macOS version (Swift/Objective-C)
- Custom wake word for hands-free activation
- Browser extension for web app integration
- Improved contact/email address lookup

### Medium Term (6-12 months)
- Linux version (GTK/Qt)
- Team/enterprise licensing
- Cloud sync for settings/workflows (opt-in)
- Mobile companion app (view history, send commands)

### Long Term (12+ months)
- Offline local AI models (Llama, Mistral)
- Video guidance (not just screenshots)
- Calendar/meeting automation
- IDE plugins for code dictation

## Competitive Analysis

| Feature | keyboard.wtf | Dragon NaturallySpeaking | Windows Voice Typing | Talon Voice | AI Chatbots |
|---------|--------------|--------------------------|----------------------|-------------|-------------|
| Offline dictation | ✅ | ✅ | ✅ | ✅ | ❌ |
| AI cleanup | ✅ | ❌ | ❌ | ❌ | ✅ |
| Computer control | ✅ (safe) | ✅ (full) | ❌ | ✅ (full) | ❌ |
| Conversational AI | ✅ | ❌ | ❌ | ❌ | ✅ |
| Global hotkeys | ✅ | ✅ | ❌ | ✅ | ❌ |
| Free & open source | ✅ | ❌ ($300) | ✅ | ✅ | Mixed |
| Easy setup | ✅ | ❌ | ✅ | ❌ (complex) | ✅ |

**Differentiation**: keyboard.wtf uniquely combines offline dictation, AI-polished writing, and a safety-bounded conversational assistant in one tool with minimal setup.

## Technical Architecture (High-Level)

- **Language**: C# .NET 8
- **UI**: Windows Forms (lightweight, fast startup)
- **Audio**: NAudio (microphone capture)
- **Transcription**: Vosk (fast), Whisper.net (accurate)
- **AI**: HTTP clients for Gemini/Claude/OpenAI
- **Voice**: Gemini Live WebSocket, Piper TTS
- **Encryption**: Windows DPAPI
- **Storage**: Local JSON files in `%LOCALAPPDATA%`
- **Typing**: SendInput API with clipboard fallback
- **Installer**: Inno Setup (single .exe, per-user)

## Open Questions & Decisions

### Resolved
- ✅ Should we support macOS at launch? **No, Windows-only MVP**
- ✅ Which AI provider as default? **Gemini (free tier sufficient)**
- ✅ Allow custom AI prompts per destination? **Yes, in settings**
- ✅ Should Jarvis auto-send emails? **No, always require review**

### Future Decisions
- Should we add a paid tier for cloud sync? (Post-MVP)
- What's the right balance for offline AI models vs cloud quality? (User feedback)
- Enterprise licensing model? (Demand-dependent)

---

**Document Version**: 1.0  
**Author**: Tanush Shah  
**Last Updated**: October 2026  
**Status**: ✅ Approved - All features implemented
