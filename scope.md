# Project Scope: keyboard.wtf

## Vision
Create a voice-first Windows desktop application that eliminates the friction of typing by turning speech into action through AI-powered transcription, smart writing cleanup, and a conversational assistant.

## Core Problem
Modern knowledge workers spend hours typing, copying, pasting, and manually formatting text. Voice input exists, but it's either too rigid (dictation software) or too disconnected from actual workflows. keyboard.wtf bridges this gap by making voice the primary input method while the keyboard becomes just a trigger.

## Target Users
- **Primary**: Knowledge workers, developers, and content creators who spend significant time typing
- **Secondary**: Users with accessibility needs or those who want to increase productivity
- **Specific Use Cases**: Email composition, documentation writing, note-taking, computer control

## In Scope

### Phase 1: Core Voice Modes (MVP)
- ✅ **Dictation Mode**: Raw speech-to-text with local offline transcription
- ✅ **Smart Writing Mode**: AI-cleaned text that removes fillers and adds punctuation
- ✅ **Global hotkeys**: Distinct shortcuts for each mode
- ✅ **Voice orb UI**: Non-intrusive overlay showing real-time status
- ✅ **Automatic pause detection**: Stop recording after speech ends

### Phase 2: AI Assistant (Jarvis)
- ✅ **Conversational AI**: Bidirectional voice conversation with Gemini Live
- ✅ **Safe computer actions**: Allowlisted commands for apps, files, and settings
- ✅ **Browser control**: Tab management for Chrome, Edge, Firefox, etc.
- ✅ **Context awareness**: Use clipboard, selected text, and active window
- ✅ **Permission modes**: Ask vs Auto-execute for different security levels

### Phase 3: Advanced Features
- ✅ **Local intent memory**: Remember user preferences (20-entry cap)
- ✅ **Workflow automation**: Save and trigger multi-step routines by voice
- ✅ **Multiple AI providers**: Gemini, Claude, OpenAI support
- ✅ **Destination formatting**: Email, Slack, Discord, Teams, etc.
- ✅ **Screen guidance**: One-shot vision analysis for troubleshooting
- ✅ **Windows integration**: Start with Windows, tray app, system controls

## Out of Scope

### Explicitly Not Included
- ❌ **macOS/Linux versions** (Windows-only for this phase)
- ❌ **Mobile apps** (desktop-first approach)
- ❌ **Cloud sync** (local-first privacy model)
- ❌ **Automatic email sending** (always require user review)
- ❌ **Continuous screen recording** (privacy boundary)
- ❌ **Generic keyboard/mouse automation** (security risk)
- ❌ **Browser extensions** (standalone app only)
- ❌ **Multi-user accounts** (single-user local settings)

### Future Considerations
- macOS version using Objective-C/Swift with similar architecture
- Browser extension for deeper web app integration
- Enterprise deployment with managed settings
- Custom wake word for hands-free activation
- Offline AI models for smart writing

## Success Criteria

### Technical
- ✅ Offline transcription works without internet
- ✅ End-to-end latency < 3 seconds for dictation
- ✅ API keys encrypted with Windows DPAPI
- ✅ Safe action boundaries prevent destructive operations
- ✅ App survives crashes and resumes state

### User Experience
- ✅ One-hand operation via global hotkeys
- ✅ Visual feedback doesn't steal focus
- ✅ Typing works in any Windows app (or clipboard fallback)
- ✅ Settings accessible via tray menu or hotkey
- ✅ Natural conversation flow with interruption support

### Business
- ✅ MIT license for open source distribution
- ✅ Self-contained installer (no complex dependencies)
- ✅ Free Windows download from keyboard-wtf.vercel.app
- ✅ Built for Name.com Domain Roulette challenge
- ✅ Demo-ready with clear value proposition

## Constraints

### Technical Limitations
- **Windows-only**: Uses Windows Forms, NAudio, DPAPI
- **Microphone required**: No keyboard-only operation mode
- **.NET 8 dependency**: Requires modern Windows (10+)
- **Local models**: Vosk ~50MB, Whisper ~1.5GB optional

### Privacy Boundaries
- **User approval gates**: Screen capture, camera, confirmations
- **No telemetry**: Completely local operation
- **Encrypted secrets**: API keys never stored in plain text
- **Bounded memory**: 20 intents, 50 history items, 80 aliases

### Safety Boundaries
- **Allowlisted actions only**: No arbitrary command execution
- **Confirmation for destructive ops**: Shutdown, restart, delete
- **No auto-submission**: Email, messages always reviewed
- **Browser-specific commands**: Won't substitute different browser

## Milestones

### ✅ Milestone 1: Basic Dictation (Week 1)
- Local Vosk transcription
- Global hotkeys
- Text typing simulation
- Voice orb UI

### ✅ Milestone 2: Smart Writing (Week 2)
- Gemini API integration
- Filler removal and cleanup
- Settings UI
- DPAPI encryption

### ✅ Milestone 3: Jarvis Mode (Week 3-4)
- Gemini Live voice integration
- Allowlisted tool calling
- App/browser control
- Clipboard/context awareness

### ✅ Milestone 4: Advanced Features (Week 5-6)
- Intent memory
- Workflow automation
- Multiple destinations
- Screen guidance
- Windows installer

### ✅ Milestone 5: Polish & Release (Week 7)
- Landing page
- Demo video
- Documentation
- Windows installer
- Public release

## Risk Mitigation

### Technical Risks
- **Windows API instability**: Fallback to clipboard if typing fails
- **Model download issues**: Graceful degradation, clear error messages
- **API rate limits**: Local transcription doesn't require cloud
- **Hotkey conflicts**: Auto-detection and fallback shortcuts

### Security Risks
- **Malicious prompts**: Allowlist approach, no arbitrary execution
- **Secret exposure**: DPAPI encryption, masked UI display
- **Unintended actions**: Confirmation gates, bounded action history
- **Permission escalation**: Never bypass Windows security

### User Experience Risks
- **Confusing permissions**: Clear Ask vs Auto-execute modes
- **Voice recognition errors**: Show transcript, allow manual retry
- **Focus stealing**: Non-modal overlay, tray-based settings
- **Unclear state**: Voice orb shows listening/processing/done

## Dependencies

### External Services (Optional)
- **Gemini API**: For smart writing and Jarvis (can use Claude/OpenAI)
- **Vercel**: For landing page hosting
- **GitHub**: For repository and releases

### Local Dependencies (Required)
- **.NET 8 SDK**: Runtime and build tools
- **Vosk models**: Lightweight local transcription
- **NAudio**: Windows audio recording
- **Whisper.net**: Optional high-accuracy transcription

## Measurement

### Key Metrics (Post-Launch)
- Installation completion rate
- Daily active usage (# of voice commands)
- Mode preference (Dictation vs Smart Writing vs Jarvis)
- Error/cancellation rate
- Average session length

### Qualitative Feedback
- User interviews about workflow integration
- Pain points with hotkeys or UI
- Feature requests and use cases
- Privacy/security concerns

---

**Document Version**: 1.0  
**Last Updated**: October 2026  
**Status**: ✅ Complete - All milestones delivered
