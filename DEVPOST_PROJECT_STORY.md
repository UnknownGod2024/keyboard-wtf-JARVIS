# keyboard.wtf - Project Story

## 💡 Inspiration

As a developer and student, I spend 6-8 hours a day typing—emails, code comments, documentation, messages, notes. Even at 80 WPM, typing is **slow** compared to speech (150+ WPM), **disruptive** to my flow state, and **exhausting** after hours of work. I kept asking: why am I still typing in 2026?

Existing solutions disappointed me:
- **Traditional dictation software** (Dragon) is rigid and expensive ($300)
- **Windows Voice Typing** is basic and can't control my PC
- **Voice assistants** (Alexa, Siri) can't type into desktop apps
- **AI chatbots** (ChatGPT) require constant copy-paste

I wanted something different: **one app, three modes, instant access via hotkeys**—raw dictation when I need verbatim text, AI-cleaned writing when I want polish, and an intelligent assistant that can actually *do* things on my computer without being dangerous.

keyboard.wtf was born from the question: **"What if voice became the primary input, and the keyboard was just the trigger?"**

---

## 🛠️ How I Built It

### The Tech Stack
- **C# .NET 8** + **Windows Forms**: Fast, lightweight, native Windows integration
- **NAudio**: Real-time microphone capture (16kHz PCM)
- **Vosk** (offline): Lightweight local transcription (~50MB, <2s latency)
- **Whisper.net**: High-accuracy local transcription for smart writing mode
- **Gemini Live**: Bidirectional voice conversation with tool calling
- **Piper TTS**: Local text-to-speech (fallback to Windows SAPI)
- **Windows DPAPI**: Military-grade encryption for API keys (user-bound)

### Architecture Overview
```
User presses hotkey
    ↓
Hotkey Service captures global shortcut
    ↓
Audio Capture Service (NAudio) starts recording
    ↓
[Branch by mode]
    ├─ Dictation: Vosk transcription → Type directly
    ├─ Smart Writing: Vosk/Whisper → Gemini cleanup → Type
    └─ Jarvis: Stream to Gemini Live → Tool calls → Actions
    ↓
Voice Orb UI shows real-time status
    ↓
Output: SendInput typing or clipboard fallback
```

### The Three Modes

**1. Dictation Mode (Ctrl+Alt+D)**
- Pure offline transcription with Vosk
- Shows live partial transcripts in the voice orb
- Auto-pause detection (800ms silence threshold)
- Types exactly what you say—no AI rewriting
- Perfect for: forms, code comments, notes

**2. Smart Writing Mode (Ctrl+Alt+K)**
- Transcribes locally, then sends to Gemini for cleanup
- Prompt engineering: "Remove fillers, honor corrections, maintain meaning"
- Removes "um," "uh," "like," false starts
- Adds punctuation and fixes grammar
- Types polished professional text
- Perfect for: emails, documents, messages

**3. Jarvis Mode (Ctrl+Alt+Q)**
- Full Gemini Live integration (bidirectional speech)
- 100+ allowlisted safe actions:
  - Open apps (fuzzy matching: "vscode" → "Visual Studio Code")
  - File search (breadth-first across safe folders)
  - Browser control (tabs, navigation—only when foreground)
  - Email drafts (Gmail compose, never auto-send)
  - System info, volume, settings
  - Workflow automation ("whenever I say X, do Y")
- **Ask vs Auto-execute modes**: User controls permission level
- Action history (50 entries), intent memory (20 entries)
- Perfect for: hands-free productivity, multitasking

### Safety-First Design Philosophy

I built Jarvis with **safety boundaries** from day one:

**Allowlist Approach**:
- No arbitrary command execution
- No generic keyboard/mouse automation
- No executables/scripts via "open file"
- Browser commands only work when that browser is foreground

**Confirmation Gates**:
- Destructive actions always require "confirm" (shutdown, restart, delete)
- In Ask mode, every action requires verbal approval
- Screen capture and camera access are one-shot, user-gated

**Privacy Boundaries**:
- Screen guidance: in-memory only (not saved to disk)
- Camera: single photo, explicit permission
- No continuous recording or screen watching
- Zero telemetry or analytics

**Why this matters**: An AI that can "do anything" is terrifying. Allowlists and confirmation gates make Jarvis *helpful* instead of *dangerous*.

### The Voice Orb UI

The hardest UX challenge was: **how do you show voice status without stealing focus?**

Solution: A semi-transparent, always-on-top, click-through overlay that shows:
- **Listening**: Animated waveform (live audio level)
- **Transcribing**: Partial text from Vosk
- **Thinking**: AI provider icon with spinner
- **Executing**: Action being performed
- **Speaking**: Gemini audio output (animated)
- **Done**: Checkmark (auto-dismiss 2s)
- **Cancelled**: Manual stop via Ctrl+Alt+X

Tech: Custom Windows Form with `WS_EX_LAYERED`, `WS_EX_TRANSPARENT`, and GDI+ rendering at 60 FPS. Positioned top-center, color-coded by mode (blue/green/purple).

### Encryption & Privacy

**All secrets encrypted with Windows DPAPI**:
```csharp
byte[] encrypted = ProtectedData.Protect(
    Encoding.UTF8.GetBytes(apiKey),
    null, // No additional entropy
    DataProtectionScope.CurrentUser // User-specific key
);
```

This means:
- API keys only decrypt for the same Windows user
- Copying `settings.json` to another PC won't work
- No master password needed (tied to Windows login)
- If Windows profile is compromised, secrets are too (acceptable tradeoff)

**Local-first architecture**:
- Vosk models: 100% offline, no cloud calls
- Settings, memory, logs: `%LOCALAPPDATA%\keyboard.wtf\`
- No telemetry, no analytics, no phone-home

---

## 🎓 What I Learned

### 1. **Plan Before You Code (The Devpost Learn Way)**

Using the Devpost Learn Skill Pack's interview-driven approach, I created `scope.md`, `prd.md`, and `spec.md` **before writing a single line of code**. This prevented the classic AI-assisted coding trap: "rewrite everything later because you forgot a critical edge case."

**Example**: The skill pack asked, "What happens if the user's microphone disconnects mid-recording?" I hadn't thought of that. We designed graceful degradation and fallback to default device—saved me hours of debugging later.

**Lesson**: AI is great at writing code, but terrible at remembering context across sessions. Writing it down first keeps *you* in control.

### 2. **Voice UX Is Harder Than I Expected**

Early prototypes had a silent recording indicator. Users didn't trust it—they kept tapping the hotkey multiple times, thinking it wasn't working.

**Solution**: Real-time feedback in the voice orb. Show the waveform, partial transcripts, AI thinking, execution status. Users need to **see and hear** what's happening.

**Lesson**: For voice interfaces, *silence is terrifying*. Feedback must be instant, visual, and audio.

### 3. **Pause Detection Is an Art, Not a Science**

I tried:
- Fixed 500ms silence → Too aggressive (cut off sentences)
- Fixed 2000ms silence → Too slow (felt laggy)
- Voice activity detection → Too complex (false positives)

**Final solution**: Adaptive threshold based on rolling window:
```
if (RMS_level < threshold for 800ms) AND (no speech in last 1200ms):
    pause_detected = true
```

Plus manual submit (press the same hotkey again).

**Lesson**: Voice interfaces need **two escape hatches**: automatic (pause detection) and manual (hotkey re-press).

### 4. **AI Safety Requires Paranoia**

I originally had Jarvis with broader permissions. During testing, I accidentally said "close all windows" instead of "close this window." It worked. That was *terrifying*.

**Changes made**:
- Added "close all" to destructive actions (always requires confirmation)
- Implemented Ask vs Auto-execute modes
- Bounded all resources (memory, history, aliases)
- Action history for audit trail

**Lesson**: If your AI assistant *could* delete Production, it *will* eventually. Design for Murphy's Law.

### 5. **Local-First Is Fast Enough**

I was skeptical that local Vosk transcription could beat cloud APIs. I was wrong.

**Vosk performance**:
- Model size: 50MB (fits in RAM)
- Latency: <2 seconds for 10s audio
- Accuracy: ~90% (good enough for most use cases)
- **Works offline** (no internet = no problem)

**Whisper (accurate mode)**:
- Model size: 1.5GB
- Latency: ~4 seconds for 10s audio
- Accuracy: ~95% (excellent)

**Lesson**: Don't assume cloud = faster. Local models have come incredibly far, and **privacy + offline capability** is a huge UX win.

### 6. **Fuzzy Matching Is Essential**

Users don't say "Visual Studio Code." They say "VS Code," "vscode," "code," "that code editor."

**Solution**: Fuzzy matching with confidence scoring:
```csharp
- Exact match: confidence = 1.0
- Substring match: confidence = 0.8
- Levenshtein distance < 3: confidence = 0.6
- If confidence < 0.7: show selection dialog
```

Also: learned aliases. First time Jarvis resolves "vsc" → "Visual Studio Code," it saves that mapping locally (bounded to 80 entries).

**Lesson**: Natural language is messy. Design for typos, abbreviations, and synonyms.

### 7. **Structured Iteration > One-Shot Prompts**

The Devpost Learn Skill Pack's workflow (scope → PRD → spec → build) forced me to think through:
- Edge cases (elevated apps blocking typing → clipboard fallback)
- Hotkey conflicts (detect, warn, show fallback shortcuts)
- Model download failures (retry on restart, graceful degradation)
- Permission modes (Ask vs Auto-execute balancing)

**Without planning**: I would have discovered these issues *after* writing 5,000 lines of code, requiring massive refactors.

**Lesson**: The time you spend planning is time you don't spend rewriting.

---

## 🏔️ Challenges I Faced

### Challenge 1: **Gemini Live Audio Streaming**

**Problem**: Gemini Live uses WebSocket with custom audio encoding. Documentation was sparse. I needed bidirectional speech (user → AI → speaker) while handling tool calls mid-conversation.

**Solution**:
- Used `System.Net.WebSockets.ClientWebSocket` for connection
- Base64-encoded 16kHz PCM audio chunks (streaming)
- Parsed `serverContent` messages for audio + tool calls
- Played 24kHz audio response via NAudio `WaveOutEvent`
- Implemented interruption support (send new audio while receiving)

**What made it hard**: Tool calls and audio responses are *interleaved*. I had to buffer audio playback while executing tools, then resume. Took 3 days of debugging.

### Challenge 2: **Typing into Elevated Apps**

**Problem**: Windows security blocks `SendInput` from non-elevated apps into elevated apps (e.g., Task Manager, Admin Command Prompt).

**Attempted solutions**:
1. Run keyboard.wtf elevated → *Bad*: breaks drag-and-drop from Explorer
2. Use `SendKeys.SendWait()` → *Still blocked* by UAC
3. Use accessibility APIs → *Requires screen reader permissions*

**Final solution**: **Clipboard fallback**
```csharp
try {
    SendInput(text); // Try typing
} catch {
    Clipboard.SetText(text); // Fall back to clipboard
    NotificationService.Show("Copied to clipboard (elevated app detected)");
}
```

**Lesson**: You can't bypass Windows security. Design graceful degradation instead.

### Challenge 3: **Pause Detection False Positives**

**Problem**: Early versions cut off users mid-sentence because ambient noise (keyboard clicks, fans) triggered false pauses.

**Solution**: Multi-stage detection:
1. Calculate RMS audio level (root mean square)
2. Track rolling 500ms window
3. Require *both*:
   - RMS < threshold for 800ms consecutive
   - No speech detected in absolute 1200ms window
4. Configurable threshold in Settings (for noisy environments)

**Lesson**: Voice activity detection is hard. Build in user controls for tuning.

### Challenge 4: **Hotkey Conflicts**

**Problem**: Users have different apps. My chosen hotkeys (`Ctrl+Alt+K`, `Ctrl+Alt+Q`) might conflict with their existing shortcuts.

**Solution**:
- Register hotkeys with `RegisterHotKey` API
- Detect failure → show notification in tray
- Offer fallback shortcuts (`Ctrl+Alt+J` for Jarvis, `Ctrl+Alt+Escape` for cancel)
- Settings UI shows which shortcuts are active
- User can customize all hotkeys

**Lesson**: Never assume your shortcuts are available. Always provide escape hatches.

### Challenge 5: **Model Download & Storage**

**Problem**: Vosk (50MB), Whisper (1.5GB), Piper voices (100MB) are too large to bundle. But users won't manually download them.

**Solution**:
- Auto-download on first use (with progress bar)
- Store in `%LOCALAPPDATA%\keyboard.wtf\models\`
- Retry incomplete downloads on app restart
- Graceful degradation if models fail (show error, allow retry)

**Code snippet**:
```csharp
if (!File.Exists(voskModelPath)) {
    await DownloadModelAsync("vosk", voskModelUrl, progress);
}
```

**Lesson**: Make setup *automatic* but *resilient*. Users won't read install instructions.

---

## 🚀 What's Next

**Short-term** (if I win / if people use it):
- macOS version (Swift/Objective-C with similar architecture)
- Custom wake word ("Hey Jarvis" for hands-free activation)
- Browser extension for deeper web app integration
- Improved contact/email lookup (current limitation: no Outlook integration)

**Long-term** (dream features):
- Offline local AI models (Llama, Mistral) for Smart Writing (no API key needed)
- Team/enterprise licensing with managed settings
- Cloud sync for workflows/memory (opt-in, end-to-end encrypted)
- Mobile companion app (view history, send voice commands remotely)

---

## 🎯 Why This Matters

**keyboard.wtf isn't just a dictation tool.** It's a proof of concept for a bigger idea:

> **Voice should be the primary input method for knowledge work. The keyboard should be optional.**

We've had the technology for years:
- Speech recognition is 95%+ accurate (Whisper)
- AI can clean up messy speech (GPT/Gemini)
- Voice assistants exist (Alexa, Siri)

But **nobody combined them into one cohesive desktop experience** with:
✅ Offline privacy  
✅ Global hotkey access  
✅ Safety-first AI actions  
✅ Clean, non-intrusive UI  

keyboard.wtf proves it's possible. And it's open source (MIT), so anyone can build on it.

---

## 🏆 Final Thoughts

This was my first time following a **structured AI-first development process** (Devpost Learn Skill Pack), and it completely changed how I work with AI coding assistants.

**Before**: "AI, build this feature" → messy code, forgotten edge cases, endless debugging.

**After**: Interview myself with the skill pack → write scope/PRD/spec → *then* build → clean, maintainable code with fewer surprises.

I'll never go back to "yolo coding" with AI. Planning first is the difference between a prototype and a product.

keyboard.wtf took **7 weeks** from idea to working app. Dictation, smart writing, Jarvis, 100+ actions, offline models, encryption, installer, landing page, documentation—all by one person (me) working nights and weekends.

**I couldn't have done this without AI**—but AI couldn't have done this without a *plan*.

---

**Thanks for reading!** If you made it this far, go try keyboard.wtf. Press `Ctrl+Alt+K` and say this sentence with all the "ums" and "uhs" you want. Watch it type back clean, polished text. That's the magic.

— Tanush Shah  
GitHub: [@UnknownGod2011](https://github.com/UnknownGod2011)  
Repository: [keyboard-wtf-JARVIS](https://github.com/UnknownGod2024/keyboard-wtf-JARVIS)
