# RecordingAlert
This is a small OBS script that makes OBS play a sound and/or speak a message when you start, stop, pause or resume recording, as well as the wav files it needs. Windows only.

# How To Set It Up
- Download RecordingAlert.zip from here: https://github.com/hallowedthings/RecordingAlert/releases/tag/V1.1.0  
- Unzip it and then copy and paste the "Beep" folder to `C:\Program Files\obs-studio\obs-plugins`.  
- (Optional, skip this step if it doesn't apply to you) If your OBS installation is in a custom location or in a different drive from C:, or if you just want to use your own audio files, you'll need to edit the lua script to point to the right destination. Right click it and open it with a text editor like Notepad or an IDE like Visual Studio Code. Update these to match the names/locations of the audio files (The files **MUST** be `.wav` because this is what the Windows API's PlaySoundA function supports, so if they're `.mp3s` or something else you'll need to convert them first. The backslashes need to be escaped within the actual code, hence the double backslashes in the lua file):

```lua
local start_chime  = "C:\\Program Files\\obs-studio\\obs-plugins\\Beep\\start.wav"
local stop_chime   = "C:\\Program Files\\obs-studio\\obs-plugins\\Beep\\stop.wav"
local pause_chime  = "C:\\Program Files\\obs-studio\\obs-plugins\\Beep\\pause.wav"
local resume_chime = "C:\\Program Files\\obs-studio\\obs-plugins\\Beep\\resume.wav"
```

For example:

```lua
local start_chime  = "D:\\Program Files (x86)\\obs-studio\\obs-plugins\\Myownmusic\\naviheylisten.wav"
local stop_chime   = "E:\\Users\\me\\Music\\whatthedogdoin.wav"
local pause_chime  = "E:\\Users\\me\\Music\\pause.wav"
local resume_chime = "E:\\Users\\me\\Music\\resume.wav"
```

- **IMPORTANT: Configurable Toggles**  
  - I've added the option to toggle the chimes and Windows' inbuilt text to speech. At the top of the script you’ll find two booleans you can set to `true` or `false`. For example, if you only want the text to speech feature, you can turn the chimes off and turn the speech on. The speech is off by default:

```lua
local enable_chimes = true   -- set to false to disable .wav chimes
local enable_tts    = true   -- set to false to disable spoken announcements
```
  - You can also toggle the words by changing the text in quotes under "OBS event callback". For example, if you change "Recording started", Windows will say whatever you change it to when you start recording.
```lua
if event == obs.OBS_FRONTEND_EVENT_RECORDING_STARTED then
        if enable_chimes then play_sound(start_chime) end
        speak("Recording started")
```
With all that out of the way...

- Install the script from inside OBS:  
  - Open OBS  
  - Go to **Tools > Scripts**  
  - In the Scripts dialog, click the **+** button (add script)  
  - Locate `recording_alert.lua` and select it  
  - Close the Scripts window (the script should now be active)

- Voila, it should be working now, and should play a sound and/or speak a message when you start, stop, pause or resume recording. You can stop guessing whether you messed up now lol

# How It Works, For Those Who Care
- **LuaJIT and FFI**  
  OBS on Windows ships with LuaJIT, which lets Lua scripts call native Windows functions directly via the FFI (Foreign Function Interface).

- **winmm.dll**  
  The script explicitly loads `winmm.dll` (a standard Windows library) which contains the PlaySoundA function.

- **PlaySoundA Function**  
  PlaySoundA can play `.wav` files asynchronously without opening any external program.  
  The flags `SND_ASYNC`, `SND_NOWAIT`, etc. ensure there’s no pop-up or blocking behavior.

- **SAPI.SpVoice (TTS)**  
  We initialize COM and create a `SAPI.SpVoice` instance via `ole32`/`oleaut32` to speak simple phrases when `enable_tts` is true.

- **OBS Event Callback**  
  When OBS starts, stops, pauses or resumes recording, it triggers specific “frontend events.”  
  The script’s `on_event` function checks for:  
  - `OBS_FRONTEND_EVENT_RECORDING_STARTED`  
  - `OBS_FRONTEND_EVENT_RECORDING_STOPPED`  
  - `OBS_FRONTEND_EVENT_RECORDING_PAUSED`  
  - `OBS_FRONTEND_EVENT_RECORDING_UNPAUSED`

  For each event, it calls `play_sound()` with the appropriate chime (if `enable_chimes` is true) and `speak("...")` with the corresponding phrase (if `enable_tts` is true).  
  As a result, you get an audible and/or spoken notification whenever your recording status changes—all within OBS, with no external player or popup.

# Troubleshooting & Tips
- **No Sound?**  
  - Double-check the `.wav` file paths. Ensure the backslashes are escaped correctly (`\\`).  
  - Confirm your `.wav` files are valid by playing them in a media player.  
  - Check OBS’s Log (under **Help > Log Files > View Current Log**) for any Lua or path errors.

- **Wrong File Format**  
  - If your `.wav` files are actually compressed or incorrectly encoded, PlaySoundA may not play them. Use standard 16-bit PCM WAV or try a different conversion method.

- **Administrator Privileges**  
  - Not usually necessary. But if you run into permissions issues, try launching OBS as an Administrator.

- **Is FFI Missing or Disabled?**  
  - If OBS complains about `ffi` not being available, your OBS build may not support LuaJIT FFI by default. This is rare in official OBS releases for Windows but can happen in certain custom builds.

And that's about it.
