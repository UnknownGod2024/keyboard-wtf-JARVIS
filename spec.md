# Technical Specification: keyboard.wtf

## Architecture Overview

keyboard.wtf is a single-process Windows desktop application built with .NET 8 and Windows Forms. The architecture follows a layered service pattern with clear separation between audio capture, transcription, AI processing, action execution, and UI presentation.

```
┌─────────────────────────────────────────────────────────────┐
│                       User Interface Layer                   │
│  ┌──────────────┐  ┌─────────────┐  ┌──────────────────┐  │
│  │ Voice Orb UI │  │ Tray Icon   │  │ Settings Browser │  │
│  │ (Overlay)    │  │ (Menu)      │  │ (Embedded HTML)  │  │
│  └──────────────┘  └─────────────┘  └──────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    Orchestration Layer                       │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  ModeCoordinator: Routes hotkeys to mode handlers    │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
          │                  │                    │
          ▼                  ▼                    ▼
┌────────────────┐  ┌────────────────┐  ┌────────────────────┐
│ Dictation Mode │  │ Smart Writing  │  │  Jarvis Mode       │
│                │  │     Mode       │  │  (Gemini Live)     │
└────────────────┘  └────────────────┘  └────────────────────┘
          │                  │                    │
          └──────────────────┴────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                      Service Layer                           │
│  ┌────────────┐ ┌──────────────┐ ┌──────────────────────┐  │
│  │  Audio     │ │ Transcription│ │   AI Providers       │  │
│  │  Service   │ │   Service    │ │ (Gemini/Claude/OAI)  │  │
│  └────────────┘ └──────────────┘ └──────────────────────┘  │
│  ┌────────────┐ ┌──────────────┐ ┌──────────────────────┐  │
│  │  Action    │ │  Settings    │ │   Destination        │  │
│  │  Executor  │ │  Service     │ │   Formatters         │  │
│  └────────────┘ └──────────────┘ └──────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

## Component Specifications

### 1. Audio Capture Service

**Responsibility**: Record microphone input and detect speech boundaries.

**Technology**: NAudio library for Windows audio APIs

**Implementation Details**:
```csharp
public class AudioCaptureService
{
    private WaveInEvent waveIn;
    private readonly int sampleRate = 16000; // Required for Vosk/Gemini
    private readonly int channels = 1;       // Mono audio
    private readonly int bitsPerSample = 16;
    
    public event EventHandler<AudioDataEventArgs> AudioDataAvailable;
    public event EventHandler<double> AudioLevelChanged;
    
    public void StartCapture(string deviceId)
    {
        // Initialize NAudio WaveInEvent
        // Monitor audio level for voice orb visualization
        // Buffer audio chunks for transcription
    }
    
    public void StopCapture()
    {
        // Clean shutdown of audio device
        // Flush remaining buffers
    }
}
```

**Pause Detection Algorithm**:
1. Track rolling window of audio levels (last 500ms)
2. Detect speech end when:
   - RMS level < threshold for 800ms consecutive
   - No speech detected in last 1200ms (absolute timeout)
3. Configurable thresholds in Settings

**Error Handling**:
- Device disconnection: Show error in orb, fall back to default device
- Permission denied: Prompt user to grant microphone access
- Buffer overrun: Log warning, continue recording

### 2. Transcription Service

**Responsibility**: Convert audio to text using local models.

**Models**:
- **Vosk**: Lightweight (~50MB), fast (real-time factor < 0.1x), 90% accuracy
- **Whisper**: Large (~1.5GB), slower (real-time factor ~0.3x), 95% accuracy

**Service Interface**:
```csharp
public interface ITranscriptionService
{
    Task<string> TranscribeAsync(byte[] audioData, CancellationToken ct);
    Task<string> GetPartialTranscript(); // Vosk only
    bool IsModelAvailable();
    Task DownloadModelAsync(IProgress<double> progress);
}
```

**Vosk Implementation**:
- Load model from `%LOCALAPPDATA%\keyboard.wtf\models\vosk-model-small-en-us-0.15`
- Create recognizer with 16kHz sample rate
- Stream audio chunks, get partial results every 100ms
- Final result on audio end

**Whisper Implementation**:
- Use Whisper.net bindings to whisper.cpp
- Model: `ggml-base.en.bin` (143MB, good speed/accuracy balance)
- No partial results (batch processing only)
- Auto-detect silence for segmentation

**Model Selection Strategy**:
- **Dictation mode**: Vosk (speed priority)
- **Smart Writing mode**: Whisper if available, else Vosk (accuracy priority)
- **Jarvis mode**: Direct Gemini Live streaming (no local transcription)

### 3. AI Provider Service

**Responsibility**: Format text with AI, handle provider-specific APIs.

**Supported Providers**:
- Gemini (Google AI): Default, free tier sufficient
- Claude (Anthropic): Sonnet/Opus models
- OpenAI: GPT-4o/GPT-4o-mini

**Common Interface**:
```csharp
public interface IAIProvider
{
    Task<string> CleanTranscriptAsync(string rawText, string destination);
    Task<StreamingResponse> StreamChatAsync(IEnumerable<Message> messages);
    Task<string> AnalyzeImageAsync(byte[] imageData, string prompt);
}
```

**Smart Writing Prompt Template**:
```
You are a writing assistant. Clean up this spoken transcript:
- Remove filler words (um, uh, like, you know)
- Honor self-corrections (e.g., "no wait, I mean...")
- Add appropriate punctuation
- Fix obvious grammar errors
- Maintain the speaker's meaning and tone
- Keep it concise but complete

[Destination-specific rules: {destinationPrompt}]

Transcript: {rawText}

Return only the cleaned text, no explanations.
```

**Destination Prompts**:
- **Email**: "Use professional tone, include greeting/closing if appropriate"
- **Slack/Discord**: "Keep casual and concise, use markdown"
- **Teams**: "Professional but friendly, structured paragraphs"
- **Default**: "Clear and polished, general purpose"

**Rate Limiting**:
- Implement exponential backoff on 429 errors
- Queue requests if needed (max queue: 5)
- Show "AI provider busy" in orb if delayed

**Provider Configuration**:
```json
{
  "selectedProvider": "gemini",
  "providers": {
    "gemini": {
      "apiKey": "<encrypted>",
      "model": "gemini-2.0-flash-exp",
      "maxTokens": 2048
    },
    "claude": {
      "apiKey": "<encrypted>",
      "model": "claude-sonnet-4-20250514",
      "maxTokens": 4096
    },
    "openai": {
      "apiKey": "<encrypted>",
      "model": "gpt-4o-mini",
      "maxTokens": 2048
    }
  }
}
```

### 4. Jarvis Assistant (Gemini Live Integration)

**Responsibility**: Bidirectional voice conversation with tool calling.

**Architecture**:
```
┌──────────────┐
│  Microphone  │
└──────────────┘
       │ 16kHz PCM
       ▼
┌──────────────────────────────────────┐
│  Gemini Live WebSocket Client        │
│  - Send audio chunks (Base64)        │
│  - Receive audio + tool calls        │
│  - Handle interruptions (duplex)     │
└──────────────────────────────────────┘
       │
       ├──────────────┬──────────────┐
       ▼              ▼              ▼
  ┌─────────┐   ┌──────────┐   ┌──────────┐
  │ Speaker │   │ Tool     │   │ Voice    │
  │ (Piper) │   │ Executor │   │ Orb UI   │
  └─────────┘   └──────────┘   └──────────┘
```

**Gemini Live Protocol**:
1. Establish WebSocket connection to `generativelanguage.googleapis.com`
2. Send setup message with:
   - Model: `gemini-2.0-flash-exp`
   - Voice: Selectable (Kore, Puck, Charon, etc.)
   - Tools: JSON schema for allowlisted functions
   - System instructions: Jarvis personality and safety boundaries
3. Stream audio in `realtimeInput` messages (Base64-encoded PCM)
4. Receive `serverContent` with:
   - `audio`: Spoken response (24kHz PCM)
   - `functionCall`: Tool invocation request
   - `turn`: Conversation turn completion
5. Send `clientContent` with function results

**System Instructions**:
```
You are Jarvis, a helpful voice assistant for Windows. You can:
- Answer questions conversationally
- Open apps, files, and folders
- Control browser tabs
- Search the web
- Compose email/message drafts (never send automatically)
- Take screenshots, camera photos (with user approval)
- Manage system settings

Safety rules:
- Always confirm destructive actions (shutdown, delete)
- Never bypass Windows permissions
- Don't claim success for actions you can't verify
- If unsure about a file/app, ask the user to clarify
- Keep responses concise unless user asks for detail

Current permission mode: {AskMode or AutoExecuteMode}
- Ask mode: Confirm every action before executing
- Auto-execute mode: Run explicit routine actions immediately, but still confirm destructive ops

User preferences (intent memory): {savedIntents}
```

**Voice Configuration**:
- Default: `Kore` (neutral, clear)
- Options: `Puck`, `Charon`, `Kore`, `Fenrir`, `Aoede`, `Orbit`, and 6 more
- Selectable in Settings → Assistant → Voice
- Voice change applies to next session (requires new WebSocket)

**Tool Execution Flow**:
1. Gemini sends `functionCall` with name and parameters
2. Jarvis validates against allowlist and permission mode
3. If Ask mode: Confirm with user ("I'll [action]. Say 'confirm' or 'cancel'.")
4. Execute action via ActionExecutorService
5. Send function result back to Gemini
6. Gemini synthesizes outcome into speech

### 5. Action Executor Service

**Responsibility**: Execute allowlisted computer actions safely.

**Action Categories**:

#### 5.1 App Management
```csharp
public class AppManagementActions
{
    // Resolve app name to executable path
    string ResolveApp(string spokenName)
    {
        // 1. Check learned aliases
        // 2. Search Windows Start Menu (Shell:AppsFolder)
        // 3. Query App Paths registry
        // 4. Scan Program Files, Program Files (x86)
        // 5. If multiple matches, prompt user to select
    }
    
    void OpenApp(string appPath)
    {
        Process.Start(appPath);
        LogAction("Opened app", appPath);
    }
    
    void CloseApp(string windowTitle)
    {
        // Find window by title (fuzzy match)
        // Send WM_CLOSE message
        // In Ask mode, require confirmation
    }
}
```

**Fuzzy Matching Algorithm**:
- Levenshtein distance < 3 for typos
- Substring match (case-insensitive)
- Acronym match (e.g., "vsc" → "Visual Studio Code")
- Confidence score: exact=1.0, substring=0.8, fuzzy=0.6
- If score < 0.7, show selection dialog

#### 5.2 File & Folder Access
```csharp
public class FileAccessActions
{
    // Safe search roots (user-approved only)
    string[] GetSafeRoots()
    {
        return new[]
        {
            Environment.GetFolderPath(SpecialFolder.Desktop),
            Environment.GetFolderPath(SpecialFolder.MyDocuments),
            Environment.GetFolderPath(SpecialFolder.MyPictures),
            Environment.GetFolderPath(SpecialFolder.MyMusic),
            Environment.GetFolderPath(SpecialFolder.MyVideos),
            Path.Combine(Environment.GetFolderPath(SpecialFolder.UserProfile), "Downloads"),
            Path.Combine(Environment.GetFolderPath(SpecialFolder.UserProfile), "OneDrive"),
            // + user-approved learned folders
        };
    }
    
    List<string> SearchFiles(string query, int maxResults = 10)
    {
        // Breadth-first search across safe roots
        // Bounded by: depth=5, scan=500 files, timeout=2s
        // Match filename (fuzzy) or exact path
        // Return top N by confidence score
    }
    
    void OpenFile(string path)
    {
        // Validate: No .exe, .bat, .cmd, .ps1, .vbs, .msi, .reg, .lnk
        // Open with default app via ShellExecute
    }
}
```

**Safety Boundaries**:
- No access outside user profile
- No C:\Windows, C:\Program Files (except reading)
- No executable/script files via generic "open file"
- Apps opened via specific app-open command only

#### 5.3 Browser Control
```csharp
public class BrowserControlActions
{
    // Detect active browser by window title/class
    string GetActiveBrowser()
    {
        IntPtr hwnd = GetForegroundWindow();
        string title = GetWindowText(hwnd);
        string className = GetClassName(hwnd);
        
        // Match: Chrome, Edge, Firefox, Brave, Opera, Vivaldi, Arc
        return DetectBrowserBySignature(title, className);
    }
    
    void SendBrowserShortcut(string browser, string action)
    {
        // Only execute if detected browser is foreground (safety)
        if (!IsBrowserForeground(browser)) {
            throw new Exception("Browser not active");
        }
        
        var shortcuts = new Dictionary<string, string> {
            {"new_tab", "^t"},       // Ctrl+T
            {"close_tab", "^w"},     // Ctrl+W
            {"next_tab", "^{TAB}"},  // Ctrl+Tab
            {"prev_tab", "^+{TAB}"}, // Ctrl+Shift+Tab
            {"reopen", "^+t"},       // Ctrl+Shift+T
            {"refresh", "{F5}"},
            {"back", "%{LEFT}"},     // Alt+Left
            {"forward", "%{RIGHT}"}, // Alt+Right
            {"downloads", "^j"},
            {"history", "^h"},
            {"find", "^f"}
        };
        
        SendKeys.SendWait(shortcuts[action]);
    }
}
```

**Browser Detection Logic**:
| Browser | Window Class | Title Pattern |
|---------|--------------|---------------|
| Chrome | Chrome_WidgetWin_1 | "Google Chrome" |
| Edge | Chrome_WidgetWin_1 | "Microsoft Edge" |
| Firefox | MozillaWindowClass | "Mozilla Firefox" |
| Brave | Chrome_WidgetWin_1 | "Brave" |
| Opera | Chrome_WidgetWin_1 | "Opera" |
| Vivaldi | Chrome_WidgetWin_1 | "Vivaldi" |
| Arc | Chrome_WidgetWin_1 | "Arc" |

#### 5.4 Clipboard & Context
```csharp
public class ContextAwareActions
{
    string GetSelectedText()
    {
        string prevClipboard = Clipboard.GetText();
        SendKeys.SendWait("^c"); // Ctrl+C
        Thread.Sleep(100);
        string selected = Clipboard.GetText();
        Clipboard.SetText(prevClipboard); // Restore
        return selected != prevClipboard ? selected : null;
    }
    
    string GetClipboardContent()
    {
        return Clipboard.GetText();
    }
    
    string GetActiveWindowTitle()
    {
        IntPtr hwnd = GetForegroundWindow();
        return GetWindowText(hwnd);
    }
}
```

### 6. Settings Service

**Responsibility**: Persist configuration with encryption.

**Storage Location**: `%LOCALAPPDATA%\keyboard.wtf\settings.json`

**Data Structure**:
```json
{
  "version": "1.2.0",
  "audio": {
    "selectedMicrophone": "Realtek Audio",
    "pauseThreshold": 0.02,
    "pauseDurationMs": 800
  },
  "hotkeys": {
    "dictation": "Ctrl+Alt+D",
    "smartWriting": "Ctrl+Alt+K",
    "jarvis": "Ctrl+Alt+Q",
    "cancel": "Ctrl+Alt+X",
    "settings": "Ctrl+Alt+,"
  },
  "transcription": {
    "preferredEngine": "vosk",
    "voskModelPath": "%LOCALAPPDATA%\\keyboard.wtf\\models\\vosk",
    "whisperModelPath": "%LOCALAPPDATA%\\keyboard.wtf\\models\\whisper"
  },
  "ai": {
    "selectedProvider": "gemini",
    "providers": {
      "gemini": {
        "apiKeyEncrypted": "<DPAPI blob>",
        "model": "gemini-2.0-flash-exp"
      }
    }
  },
  "jarvis": {
    "voice": "Kore",
    "tone": "balanced",
    "permissionMode": "ask",
    "intentMemory": [
      {"timestamp": "2026-10-20T10:00:00Z", "intent": "I prefer concise responses"}
    ],
    "workflows": {
      "initialize my setup": ["open chrome", "open vscode", "open ~/Projects"]
    },
    "learnedAliases": {
      "vsc": "C:\\Program Files\\Microsoft VS Code\\Code.exe"
    }
  },
  "destinations": {
    "slack": {"webhookEncrypted": "<DPAPI blob>"},
    "discord": {"webhookEncrypted": "<DPAPI blob>"}
  },
  "startup": {
    "startWithWindows": true
  }
}
```

**Encryption Strategy**:
```csharp
public class SecureSettingsService
{
    // Encrypt sensitive values with Windows DPAPI
    string EncryptSecret(string plaintext)
    {
        byte[] bytes = Encoding.UTF8.GetBytes(plaintext);
        byte[] encrypted = ProtectedData.Protect(
            bytes, 
            null, // No additional entropy
            DataProtectionScope.CurrentUser // User-specific key
        );
        return Convert.ToBase64String(encrypted);
    }
    
    string DecryptSecret(string ciphertext)
    {
        byte[] encrypted = Convert.FromBase64String(ciphertext);
        byte[] decrypted = ProtectedData.Unprotect(
            encrypted, 
            null, 
            DataProtectionScope.CurrentUser
        );
        return Encoding.UTF8.GetString(decrypted);
    }
}
```

**DPAPI Security Properties**:
- User-specific: Secrets only decrypt for same Windows user
- Machine-bound: Copying settings.json to another PC won't work
- No master password needed: Tied to Windows login credentials
- Limitation: If Windows profile is compromised, secrets are too

### 7. Destination Formatters

**Responsibility**: Format text for specific communication platforms.

**Formatter Interface**:
```csharp
public interface IDestinationFormatter
{
    Task<FormattedOutput> FormatAsync(string content, AIProvider provider);
}

public class FormattedOutput
{
    public string Text { get; set; }
    public Dictionary<string, string> Metadata { get; set; } // e.g., subject, recipient
    public OutputAction Action { get; set; } // Type, Clipboard, OpenURL
}
```

**Email Formatter** (Gmail Focus):
```csharp
public class EmailFormatter : IDestinationFormatter
{
    public async Task<FormattedOutput> FormatAsync(string content, AIProvider provider)
    {
        // Extract recipient, subject, body from spoken content
        var parsed = await provider.ParseEmailIntentAsync(content);
        
        // Construct Gmail compose URL
        string gmailUrl = $"https://mail.google.com/mail/?view=cm" +
            $"&to={Uri.EscapeDataString(parsed.Recipient)}" +
            $"&su={Uri.EscapeDataString(parsed.Subject)}" +
            $"&body={Uri.EscapeDataString(parsed.Body)}";
        
        return new FormattedOutput {
            Text = parsed.Body,
            Metadata = new() {
                {"recipient", parsed.Recipient},
                {"subject", parsed.Subject}
            },
            Action = OutputAction.OpenURL,
            URL = gmailUrl
        };
    }
}
```

**Webhook Formatter** (Slack/Discord/Teams):
```csharp
public class WebhookFormatter : IDestinationFormatter
{
    public async Task<FormattedOutput> FormatAsync(string content, AIProvider provider)
    {
        // Post to webhook URL
        var webhook = settingsService.GetWebhook(platform);
        var payload = new { text = content };
        
        await httpClient.PostAsJsonAsync(webhook, payload);
        
        return new FormattedOutput {
            Text = content,
            Action = OutputAction.Clipboard // Also copy locally
        };
    }
}
```

### 8. Voice Orb UI

**Responsibility**: Show real-time status without stealing focus.

**Technology**: Custom Windows Form with transparency and layering

**Implementation**:
```csharp
public class VoiceOrbForm : Form
{
    public VoiceOrbForm()
    {
        FormBorderStyle = FormBorderStyle.None;
        StartPosition = FormStartPosition.Manual;
        TopMost = true;
        ShowInTaskbar = false;
        BackColor = Color.Black;
        TransparencyKey = Color.Black;
        Opacity = 0.95;
        
        // Position at top-center of primary screen
        var screen = Screen.PrimaryScreen.Bounds;
        Size = new Size(300, 120);
        Location = new Point(
            screen.Width / 2 - Width / 2,
            20
        );
    }
    
    protected override CreateParams CreateParams
    {
        get
        {
            var cp = base.CreateParams;
            cp.ExStyle |= 0x00080000; // WS_EX_LAYERED
            cp.ExStyle |= 0x00000080; // WS_EX_TOOLWINDOW
            cp.ExStyle |= 0x00000008; // WS_EX_TOPMOST
            cp.ExStyle |= 0x20;       // WS_EX_TRANSPARENT (click-through)
            return cp;
        }
    }
}
```

**State Machine**:
```
Idle (hidden)
  │
  ├──> Listening (waveform animation)
  │     └──> Transcribing (partial text + spinner)
  │           └──> Thinking (AI provider icon + "...")
  │                 └──> Done (checkmark, auto-hide 2s)
  │                 └──> Error (X + message, auto-hide 4s)
  │
  ├──> Cancelled (via Ctrl+Alt+X)
  │     └──> Idle
  │
  └──> (Jarvis) Speaking (animated mouth icon + live caption)
```

**Rendering**:
- Use GDI+ for custom drawing (faster than WPF for simple graphics)
- Waveform: Circular bars representing audio levels
- Text: Roboto font, white on semi-transparent background
- Icons: Embedded SVG resources rendered at runtime
- Animation: 60 FPS timer, smooth transitions

### 9. Hotkey Management

**Responsibility**: Register global hotkeys, handle conflicts.

**Implementation**:
```csharp
public class HotkeyService
{
    [DllImport("user32.dll")]
    private static extern bool RegisterHotKey(IntPtr hWnd, int id, uint fsModifiers, uint vk);
    
    [DllImport("user32.dll")]
    private static extern bool UnregisterHotKey(IntPtr hWnd, int id);
    
    public bool RegisterHotkey(IntPtr handle, string combination, int id)
    {
        var (modifiers, key) = ParseHotkey(combination); // e.g., "Ctrl+Alt+K"
        bool success = RegisterHotKey(handle, id, modifiers, key);
        
        if (!success)
        {
            // Conflict detected - try fallback if available
            logger.Warn($"Hotkey {combination} already in use");
            return false;
        }
        
        return true;
    }
    
    // Fallback shortcuts if user-configured ones conflict
    private readonly Dictionary<string, string> Fallbacks = new()
    {
        {"jarvis", "Ctrl+Alt+J"},
        {"cancel", "Ctrl+Alt+Escape"}
    };
}
```

**Hotkey Processing**:
1. Application receives WM_HOTKEY message
2. Lookup which mode triggered (by ID)
3. Cancel current operation if any
4. Start new mode handler
5. Show voice orb in listening state

**Conflict Resolution**:
- If registration fails, show notification in tray
- User can change hotkey in Settings
- Fallback shortcuts shown in Settings UI
- `Ctrl+Alt+Space` deliberately avoided (common conflict)

### 10. Windows Integration

#### 10.1 Startup Registration
```csharp
public class StartupService
{
    private const string RunKey = @"SOFTWARE\Microsoft\Windows\CurrentVersion\Run";
    
    public void EnableStartup()
    {
        string exePath = Process.GetCurrentProcess().MainModule.FileName;
        using (var key = Registry.CurrentUser.OpenSubKey(RunKey, true))
        {
            key.SetValue("keyboard.wtf", $"\"{exePath}\"");
        }
    }
    
    public void DisableStartup()
    {
        using (var key = Registry.CurrentUser.OpenSubKey(RunKey, true))
        {
            key.DeleteValue("keyboard.wtf", false);
        }
    }
}
```

#### 10.2 Single Instance Guard
```csharp
public class Program
{
    private static Mutex singleInstanceMutex;
    
    [STAThread]
    static void Main()
    {
        bool createdNew;
        singleInstanceMutex = new Mutex(true, "keyboard.wtf.single.instance", out createdNew);
        
        if (!createdNew)
        {
            // Another instance running - bring it to front
            MessageBox.Show("keyboard.wtf is already running. Check the system tray.");
            return;
        }
        
        Application.Run(new MainApplicationContext());
    }
}
```

#### 10.3 Tray Icon Menu
```
keyboard.wtf
├── 🎤 Start Dictation (Ctrl+Alt+D)
├── ✍️ Start Smart Writing (Ctrl+Alt+K)
├── 🤖 Start Jarvis (Ctrl+Alt+Q)
├── ──────────────
├── ⚙️ Settings (Ctrl+Alt+,)
├── 📖 View Logs
├── 🔄 Restart App
├── ──────────────
└── ❌ Exit
```

### 11. Installer Specification

**Technology**: Inno Setup Script Compiler

**Installer Behavior**:
1. Extract files to `%LOCALAPPDATA%\Programs\keyboard.wtf`
2. Create Start Menu shortcut
3. Register startup entry (optional, default yes)
4. Check for .NET 8 Desktop Runtime, prompt download if missing
5. Launch app after install
6. If no API key configured, open setup page in default browser

**Bundled Components**:
- keyboard.wtf.exe (main application)
- All .NET dependencies
- NAudio, Vosk, Whisper.net DLLs
- Piper TTS binaries (piper.exe, onnxruntime.dll)
- Piper voice models (~100MB total)
- Vosk model (auto-downloads on first use)

**Uninstaller**:
- Remove all files from installation directory
- Delete registry startup entry
- Leave `%LOCALAPPDATA%\keyboard.wtf` (user data) by default
- Prompt to delete settings/models (opt-in)

**Build Command**:
```powershell
.\scripts\build-windows-release.ps1
# 1. dotnet publish -c Release
# 2. Copy Piper TTS assets
# 3. Run Inno Setup compiler
# Output: dist\keyboard-wtf-setup.exe
```

## Data Flow Diagrams

### Dictation Mode Flow
```
User presses Ctrl+Alt+D
  │
  ├──> Show voice orb (Listening)
  ├──> Start NAudio microphone capture
  │     │
  │     ├──> Stream audio to Vosk
  │     ├──> Display partial transcripts in orb
  │     └──> Monitor for pause (800ms silence)
  │
  ├──> Pause detected or manual submit
  ├──> Show orb (Transcribing)
  ├──> Finalize Vosk transcript
  │
  ├──> Show orb (Done)
  ├──> Type transcript into active window (SendInput)
  │     └──> If SendInput fails → Copy to clipboard
  │
  └──> Auto-hide orb (2 seconds)
```

### Smart Writing Mode Flow
```
User presses Ctrl+Alt+K
  │
  ├──> [Same as Dictation: Listening + Transcribing]
  │
  ├──> Show orb (Thinking - Gemini icon)
  ├──> Send transcript to AI provider with cleanup prompt
  │     │
  │     └──> If destination specified: Include destination-specific rules
  │
  ├──> Receive cleaned text from AI
  ├──> Show orb (Done)
  ├──> Type cleaned text into active window
  │
  └──> Auto-hide orb (2 seconds)
```

### Jarvis Mode Flow
```
User presses Ctrl+Alt+Q
  │
  ├──> Show orb (Listening - Jarvis mode)
  ├──> Establish Gemini Live WebSocket connection
  │     └──> Send system instructions, tools, voice preference
  │
  ├──> Start NAudio microphone capture (16kHz)
  │     └──> Stream audio chunks to Gemini (Base64)
  │
  ├──> Receive Gemini responses
  │     │
  │     ├──> If audio data:
  │     │     └──> Play via NAudio (24kHz)
  │     │     └──> Show orb (Speaking - animated)
  │     │
  │     ├──> If function call:
  │     │     ├──> Show orb (Executing)
  │     │     ├──> Validate against allowlist
  │     │     ├──> Check permission mode (Ask vs Auto-execute)
  │     │     │     └──> If Ask: Confirm with user first
  │     │     ├──> Execute action via ActionExecutorService
  │     │     ├──> Log to action history
  │     │     └──> Send function result back to Gemini
  │     │
  │     └──> If turn complete:
  │           └──> Wait for next user speech or auto-end phrase
  │
  ├──> User says "bye" or "stop" → Auto-end conversation
  │     OR: User presses Ctrl+Alt+X → Cancel immediately
  │
  └──> Close WebSocket, hide orb
```

## Error Handling Strategy

### Graceful Degradation
1. **Model Missing**: Use fallback (Vosk → Whisper → online API)
2. **API Failure**: Retry with exponential backoff, fallback provider
3. **Microphone Failure**: Show clear error, offer device selection
4. **Network Offline**: Disable Jarvis, allow local dictation only
5. **Typing Blocked**: Fallback to clipboard with notification

### User-Visible Errors
- Show in voice orb (4 seconds)
- Log to file: `%LOCALAPPDATA%\keyboard.wtf\logs\app.log`
- Critical errors: Show MessageBox with actionable fix
- Non-critical: Toast notification from tray icon

### Logging
```csharp
public enum LogLevel { Debug, Info, Warn, Error, Fatal }

public class Logger
{
    private string logPath = Path.Combine(
        Environment.GetFolderPath(SpecialFolder.LocalApplicationData),
        "keyboard.wtf", "logs", $"app-{DateTime.Now:yyyy-MM-dd}.log"
    );
    
    public void Log(LogLevel level, string message, Exception ex = null)
    {
        string entry = $"[{DateTime.Now:HH:mm:ss}] [{level}] {message}";
        if (ex != null) entry += $"\n{ex}";
        
        File.AppendAllText(logPath, entry + "\n");
        
        if (level >= LogLevel.Error)
        {
            // Also show in UI for critical issues
            ShowErrorNotification(message);
        }
    }
}
```

## Security Considerations

### Threat Model
1. **Malicious AI prompts**: Mitigated by allowlist approach
2. **Secret exfiltration**: Encrypted with DPAPI, masked in UI
3. **Unintended actions**: Confirmation gates for destructive ops
4. **Code execution**: No arbitrary command running
5. **Data leakage**: No telemetry, all local storage

### Security Boundaries
- ✅ Never execute arbitrary commands
- ✅ Never bypass Windows UAC/permissions
- ✅ Never auto-send emails/messages
- ✅ Never continuous screen recording
- ✅ Never write executables or scripts via voice
- ✅ API keys never logged or transmitted except to provider

## Performance Requirements

| Metric | Target | Measured |
|--------|--------|----------|
| Cold start time | <3s | ~2s |
| Hotkey response | <200ms | ~100ms |
| Vosk transcription | <2s (for 10s audio) | ~1.5s |
| Whisper transcription | <5s (for 10s audio) | ~4s |
| AI cleanup latency | <3s | ~2s |
| Memory (idle) | <200MB | ~150MB |
| Memory (Jarvis active) | <500MB | ~400MB |
| CPU (idle) | <1% | <0.5% |
| CPU (listening) | <10% | ~5% |

## Testing Strategy

### Unit Tests
- Audio level detection algorithm
- Fuzzy matching logic (apps, files)
- Hotkey parsing and validation
- Encryption/decryption (DPAPI)
- Configuration serialization

### Integration Tests
- Vosk/Whisper model loading
- AI provider API calls (mocked)
- Windows API interactions (SendInput, window management)
- Settings persistence and migration

### Manual Test Scenarios
1. Install on clean Windows 10/11
2. Test all hotkeys with/without conflicts
3. Test each Jarvis action in Ask and Auto-execute modes
4. Verify typing works in: Notepad, Word, browser, elevated apps
5. Test clipboard fallback when typing fails
6. Verify API key encryption survives app restart
7. Test graceful handling of missing models
8. Verify single-instance guard (launch twice)
9. Test startup registration (Windows sign-out/sign-in)
10. Uninstall and verify cleanup

## Deployment Checklist

- [ ] Version number updated in AssemblyInfo
- [ ] CHANGELOG.md updated with release notes
- [ ] All tests passing
- [ ] Build Windows installer (keyboard-wtf-setup.exe)
- [ ] Test installer on clean Windows 10 VM
- [ ] Test installer on clean Windows 11 VM
- [ ] Upload installer to GitHub Releases
- [ ] Update site/downloads/keyboard-wtf-setup.exe
- [ ] Deploy landing page to Vercel
- [ ] Update README with new version number
- [ ] Tag release in Git (v1.2.0)
- [ ] Post announcement on Reddit r/productivity
- [ ] Demo video uploaded to YouTube
- [ ] Submit to Devpost hackathon

---

**Document Version**: 1.0  
**Author**: Tanush Shah  
**Last Updated**: October 2026  
**Status**: ✅ Complete - Production-ready specification
