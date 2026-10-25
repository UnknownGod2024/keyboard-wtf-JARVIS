# keyboard.wtf — Project Story

## 💡 Inspiration

For decades, computers have become faster, thinner, smarter, and more powerful.

The screens changed. The chips changed. The internet changed. AI changed almost overnight.

**But the way we use a computer? Not nearly enough.**

We still translate every thought into a chain of tiny mechanical actions: find the right app, click the right button, switch tabs, copy something, paste it somewhere else, rewrite it, search again, drag a window, and repeat.

In 2026, that started to feel absurd.

We now have AI that can reason, speak, see, write, understand context, and plan actions — yet our computers still wait for us to operate them one click at a time.

**That was the thought I could not get out of my head:**

> What if the computer understood intent instead of waiting for instructions?

What if I could simply say:

- "Open Chrome, Spotify, and my project folder."
- "Make this selected message sound professional."
- "What am I looking at on my screen?"
- "Open YouTube and search for the new MrBeast video."
- "Explain this error."

...and my computer actually **did** it?

Not inside another chatbot tab. Not by giving me six steps I still have to perform myself. But from wherever I am already working.

**That idea became keyboard.wtf.**

> Say it. Your computer does it.

The goal is ambitious but simple: **make the computer move closer to the speed of human thought.**

---

## 🎯 What keyboard.wtf Feels Like

keyboard.wtf is a voice-first AI action layer for your computer.

You don't have to open an AI website before every interaction. The assistant sits across the operating system and can be triggered from wherever you're working.

**Press Ctrl+Alt+Q, speak naturally, and Jarvis Mode wakes up.**

Imagine starting your day with:
> "Open Chrome, Spotify, VS Code, and my project folder."

Or being halfway through an email, highlighting an awkward paragraph, and saying:
> "Make this more professional."

Instead of copying it into an AI chatbot, explaining the context, copying the answer back, and fixing the formatting—keyboard.wtf already knows what "this" refers to.

Highlight something and ask:
- "Explain this."
- "Summarize this."
- "Translate this to French."
- "Make this shorter."

**The selected text becomes the context.**

Now imagine staring at a confusing error, chart, or webpage. Instead of taking a screenshot and uploading it somewhere, you can ask:
> "What does this error mean?"  
> "What am I looking at?"  
> "What should I do next?"

When you explicitly ask for screen understanding, keyboard.wtf can reason about what's visible and respond in the context of your current task.

**The idea is not merely voice + AI.**

It is: **voice + context + reasoning + real action**

---

## 🎨 Three Modes for Three Different Moments

One of our most important product decisions was realizing that **every voice interaction should not behave the same way.**

Sometimes I want the computer to think. Sometimes I just want it to write better. Sometimes I want it to type exactly what I said.

So keyboard.wtf separates those experiences.

### 🤖 Jarvis Mode — When I Want the Computer to Understand and Act

Jarvis is the actual assistant. It can reason about natural requests and use supported tools across apps, windows, files, selected text, screen context, and browsers.

You can say:
- "Open YouTube and search for MrBeast."
- "Open my Downloads folder."
- "Minimize this window."
- "Prepare an email draft."
- "Explain the paragraph I selected."
- "What is on my screen?"

The important part: **Jarvis doesn't merely describe what should happen**—where keyboard.wtf has a safe, supported action path, it actually performs it.

### ✍️ Smart Writing — When My Thoughts Are Faster Than My Typing

People don't speak in polished paragraphs. We say things like:

> "Hey um tell him I'll send it tomorrow — actually tomorrow evening — and make it sound polite because I'm late."

Smart Writing understands the false starts, corrections, intention, and tone, then turns that into:

> "Hi, I'll send the file by tomorrow evening. Thank you for your patience."

It removes filler words, understands corrections, fixes punctuation, improves structure, and writes the polished result **directly where you're already working.**

Emails. Messages. Notes. Reports. You speak roughly. keyboard.wtf writes clearly.

### 🎤 Dictation — When I Want Exactly What I Said

Sometimes AI should simply get out of the way.

Dictation captures your speech and types the transcript without trying to rewrite your ideas. No stylistic transformation. No unnecessary reasoning. Just fast speech-to-text.

**That gives keyboard.wtf a simple mental model:**
- **Dictation:** my exact words
- **Smart Writing:** better words  
- **Jarvis:** understand what I mean and help do it

---

## 🛠️ How We Built It

keyboard.wtf is not one giant prompt connected to a microphone. It's several systems working together.

### The Architecture

**Windows Desktop App** (.NET):
- Global hotkeys, microphone input, transcription
- Active-window context, selected text, clipboard interaction
- Applications, windows, files, screenshots, workflows
- Communication with browser extension

**Local Speech Recognition:**
- **Vosk**: Lightweight offline transcription (50MB, <2s latency, 90% accuracy)
- **Whisper**: High-accuracy local transcription (1.5GB, 95% accuracy)
- Speech remains local-first for privacy and speed

**AI Layer** (Gemini):
- Smart Writing: Language cleanup and improvement
- Selected-text understanding and transformation
- Screen reasoning and visual context
- Command interpretation and planning
- **Gemini Live**: Real-time spoken conversations (bidirectional voice)

**Browser Integration:**
- Chrome extension for web app control
- Structured observation → planning → action → verification loop
- Safe, context-aware browser automation

### The Technical Stack

- **C# .NET 8** + **Windows Forms**: Fast, lightweight, native Windows
- **NAudio**: Real-time microphone capture (16kHz PCM)
- **Vosk** + **Whisper.net**: Offline speech recognition
- **Gemini Live**: Bidirectional voice with tool calling
- **Piper TTS**: Local text-to-speech
- **Windows DPAPI**: User-bound encryption for API keys

---

## 🎓 What We Learned

### 1. Plan Before You Code

Using the **Devpost Learn Skill Pack** methodology (scope.md → prd.md → spec.md), I documented everything before writing code. This prevented the classic AI-assisted trap: "rewrite everything later because you forgot a critical edge case."

**Example**: The skill pack asked, "What happens if the user's microphone disconnects mid-recording?" I hadn't thought of that. We designed graceful degradation—saved hours of debugging later.

**Lesson**: AI is great at writing code but terrible at remembering context across sessions. Writing it down first keeps you in control.

### 2. Voice UX Is Harder Than Expected

Early prototypes had silent recording indicators. Users didn't trust them—they kept tapping the hotkey multiple times.

**Solution**: Real-time feedback. Show waveforms, partial transcripts, AI thinking status, execution progress. Users need to **see and hear** what's happening.

**Lesson**: For voice interfaces, silence is terrifying. Feedback must be instant, visual, and audio.

### 3. Pause Detection Is an Art

I tried fixed thresholds (too aggressive or too slow), then voice activity detection (too many false positives).

**Final solution**: Adaptive threshold based on rolling window:
```
if (RMS_level < threshold for 800ms) AND (no speech in last 1200ms):
    pause_detected = true
```

Plus manual submit (press the same hotkey again).

**Lesson**: Voice interfaces need two escape hatches—automatic (pause detection) and manual (hotkey re-press).

### 4. AI Safety Requires Paranoia

During testing, I accidentally said "close all windows" instead of "close this window." It worked. **That was terrifying.**

**Changes made**:
- Added "close all" to destructive actions (always requires confirmation)
- Implemented Ask vs Auto-execute modes
- Bounded all resources (memory: 20 intents, history: 50 actions, aliases: 80 mappings)
- Action history for audit trail

**Lesson**: If your AI assistant *could* delete Production, it *will* eventually. Design for Murphy's Law.

### 5. Local-First Is Fast Enough

I was skeptical that local transcription could beat cloud APIs. I was wrong.

**Vosk performance**: 50MB model, <2s latency, 90% accuracy, works offline  
**Whisper performance**: 1.5GB model, ~4s latency, 95% accuracy

**Lesson**: Don't assume cloud = faster. Local models + privacy + offline capability is a huge UX win.

### 6. Fuzzy Matching Is Essential

Users don't say "Visual Studio Code." They say "VS Code," "vscode," "code," "that code editor."

**Solution**: Confidence-based matching:
- Exact match: confidence = 1.0
- Substring match: confidence = 0.8
- Levenshtein distance < 3: confidence = 0.6
- If confidence < 0.7: show selection dialog

Plus learned aliases (bounded to 80 entries).

**Lesson**: Natural language is messy. Design for typos, abbreviations, and synonyms.

### 7. Structured Iteration > One-Shot Prompts

The Devpost Learn Skill Pack's interview approach surfaced edge cases I wouldn't have thought of:
- Elevated apps blocking typing → clipboard fallback
- Hotkey conflicts → detect, warn, show fallbacks
- Model download failures → retry on restart, graceful degradation
- Permission balancing → Ask vs Auto-execute modes

**Lesson**: The time you spend planning is time you don't spend rewriting.

---

## 🏔️ Challenges We Faced

### Challenge 1: Gemini Live Audio Streaming

**Problem**: WebSocket with custom audio encoding, sparse documentation, bidirectional speech while handling tool calls mid-conversation.

**Solution**:
- `ClientWebSocket` for connection
- Base64-encoded 16kHz PCM streaming
- Parse `serverContent` for audio + tool calls
- Play 24kHz audio via NAudio
- Buffer audio playback while executing tools

**What made it hard**: Tool calls and audio responses are interleaved. Took 3 days of debugging.

### Challenge 2: Typing Into Elevated Apps

**Problem**: Windows blocks `SendInput` from non-elevated apps into elevated apps (Task Manager, Admin CMD).

**Attempted solutions**: Run elevated (breaks drag-drop), `SendKeys` (still blocked), accessibility APIs (requires permissions).

**Final solution**: Clipboard fallback
```csharp
try {
    SendInput(text);
} catch {
    Clipboard.SetText(text);
    Notify("Copied to clipboard (elevated app detected)");
}
```

**Lesson**: You can't bypass Windows security. Design graceful degradation.

### Challenge 3: Pause Detection False Positives

**Problem**: Ambient noise (keyboard clicks, fans) triggered false pauses, cutting off users mid-sentence.

**Solution**: Multi-stage detection:
1. Calculate RMS audio level
2. Track rolling 500ms window
3. Require RMS < threshold for 800ms AND no speech in absolute 1200ms window
4. Configurable threshold in Settings

**Lesson**: Build in user controls for environment tuning.

### Challenge 4: Hotkey Conflicts

**Problem**: Chosen hotkeys might conflict with users' existing shortcuts.

**Solution**:
- Register with `RegisterHotKey` API, detect failures
- Offer fallback shortcuts (Ctrl+Alt+J for Jarvis, Ctrl+Alt+Escape for cancel)
- Settings UI shows active shortcuts
- Full customization support

**Lesson**: Never assume your shortcuts are available. Always provide escape hatches.

### Challenge 5: Model Storage & Downloads

**Problem**: Vosk (50MB), Whisper (1.5GB), Piper (100MB) too large to bundle—but users won't manually download them.

**Solution**:
- Auto-download on first use with progress bar
- Store in `%LOCALAPPDATA%\keyboard.wtf\models\`
- Retry incomplete downloads on restart
- Graceful degradation if models fail

**Lesson**: Make setup automatic but resilient. Users won't read install instructions.

---

## 🚀 What's Next

**Short-term:**
- macOS version (Swift/Objective-C, same architecture)
- Custom wake word ("Hey Jarvis" for hands-free)
- Browser extension improvements (deeper web app control)
- Better contact/email lookup

**Long-term:**
- Offline local AI models (Llama/Mistral) for Smart Writing
- Cloud sync for workflows/memory (opt-in, encrypted)
- Mobile companion app (view history, remote commands)
- Linux support

---

## 💭 The Hardest Challenge: Power Without Fear

The magical version of an AI computer assistant is obvious: You say something. The computer does it.

**The dangerous version is also obvious**: The computer misunderstands you... and still does it.

That tension shaped every design decision.

**Our safety principles:**
- Jarvis can **prepare** a Gmail draft, but never secretly press Send
- Selected-text transformations return or copy results instead of destroying originals
- Screen reasoning happens only when explicitly requested—no silent watching
- Browser automation verifies state instead of assuming every click worked
- Dedicated Cancel path cuts through everything: listening, thinking, audio, pending actions

**Trust is not a policy page. Trust is product behavior.**

---

## 🎯 What We're Proud Of

The thing I'm most proud of isn't one individual feature.

It's that keyboard.wtf increasingly makes the entire computer feel like **one intelligent surface.**

When the system works, users stop thinking:
- "Which AI website should I open?"
- "Where do I paste this?"
- "Which menu contains that?"
- "How do I phrase this perfectly?"

They simply express the intention:
- "Make this better."
- "Explain this."
- "Open my project."
- "What is wrong on my screen?"

**The system behind those sentences is complicated. The sentence itself should not be.**

That is exactly the point.

---

## 🏆 Final Thoughts

Building keyboard.wtf taught me that the future of agents isn't just about making the model smarter.

**A useful computer agent needs:**
- **Context** — What is the user referring to?
- **Capability** — Can the system genuinely perform the requested action?
- **Verification** — Did the action actually happen?
- **Control** — Can the user interrupt or correct it?
- **Trust** — What should the agent refuse to do?

Remove any one of those and the experience starts to break.

**The most important lesson:**

> The future interface is not just chat. It is intent connected to action.

---

This was my first time following a **structured AI-first development process** using the Devpost Learn Skill Pack, and it completely changed how I work with AI coding assistants.

**Before**: "AI, build this feature" → messy code, forgotten edge cases, endless debugging

**After**: Interview myself with the skill pack → write scope/PRD/spec → *then* build → clean, maintainable code

keyboard.wtf took **7 weeks** from idea to working app. Dictation, Smart Writing, Jarvis, 100+ actions, offline models, encryption, installer, documentation—all by one person working nights and weekends.

**I couldn't have done this without AI—but AI couldn't have done this without a plan.**

---

**Thanks for reading!** If you made it this far, try keyboard.wtf. Press `Ctrl+Alt+K` and say this sentence with all the "ums" and "uhs" you want. Watch it type back clean, polished text.

That's the magic.

— **Tanush Shah**  
GitHub: [@UnknownGod2011](https://github.com/UnknownGod2011)  
Repository: [keyboard-wtf-JARVIS](https://github.com/UnknownGod2024/keyboard-wtf-JARVIS)
