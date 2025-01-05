# RecordingAlert
This is a small OBS script that makes OBS play a sound when you start or stop recording, as well as the wav files it needs. Windows only.

# How To Set It Up
- Download RecordingAlert.zip from here: https://github.com/hallowedthings/RecordingAlert/releases/tag/V1.0.0
- Unzip it and then copy and paste the "Beep" folder to C:\Program Files\obs-studio\obs-plugins.
- (Optional, skip this step if it doesn't apply to you) If your OBS installation is in a custom location or in a different drive from C:, or if you just want to use your own audio files, you'll need to edit the lua script to point to the right destination. Right click it and open it with a text editor like Notepad or an IDE like Visual Studio Code. Update these to match the names/locations of the audio files (The files MUST be wav because this is what the Windows API's PlaySoundA function supports, so if they're .mp3s or something else you'll need to convert them first. The backslashes need to be escaped within the actual code, hence the double backslashes in the lua file):

  _local start_chime = "C:\\Program Files\\obs-studio\\obs-plugins\\Beep\\start.wav"_
  
  _local stop_chime  = "C:\\Program Files\\obs-studio\\obs-plugins\\Beep\\stop.wav"_
  
  For example:
  
  _local start_chime = "D:\\Program Files (x86)\\obs-studio\\obs-plugins\\Myownmusic\\naviheylisten.wav"_
  
  _local stop_chime  = "E:\\Users\\me\\Music\\whatthedogdoin.wav"_
  
- Install the script from inside OBS:
  - Open OBS
  - Go to Tools > Scripts
  - In the Scripts dialog, click the + button (add script)
  - Locate recording_alert.lua and select it
  - Close the Scripts window (the script should now be active)
- Voila, it should be working now, and should play a sound when you start or stop recording. You can stop guessing whether you messed up now lol

# How It Works, For Those Who Care
- LuaJIT and FFI
  - OBS on Windows ships with LuaJIT, which lets Lua scripts call native Windows functions directly via the FFI (Foreign Function Interface).
- winmm.dll
  -	The script explicitly loads winmm.dll (a standard Windows library) which contains the PlaySoundA function.
- PlaySoundA Function
  - PlaySoundA can play .wav files asynchronously without opening any external program.
  - The flags SND_ASYNC, SND_NOWAIT, etc. ensure there’s no pop-up or blocking behavior.
- OBS Event Callback
  - When OBS starts or stops recording, it triggers specific “frontend events.”
  - The script’s on_event function checks whether the event is RECORDING_STARTED or RECORDING_STOPPED.
  - For each event, it calls the play_sound() function with either start_chime or stop_chime.
  - As a result, the audio file is played immediately by Windows in the background, giving you an audible notification whenever your recording status changes—all within OBS, with no external player or popup.

# Troubleshooting & Tips
- No Sound?
  - Double-check the .wav file paths. Ensure the backslashes are escaped correctly (\\).
  - Confirm your .wav files are valid by playing them in a media player.
  - Check OBS’s Log (under Help > Log Files > View Current Log) for any Lua or path errors.
- Wrong File Format
  - If your .wav files are actually compressed or incorrectly encoded, PlaySoundA may not play them. Use standard 16-bit PCM WAV or try a different conversion method.
- Administrator Privileges
  - Not usually necessary. But if you run into permissions issues, try launching OBS as an Administrator.
- Is FFI Missing or Disabled?
  - If OBS complains about ffi not being available, your OBS build may not support LuaJIT FFI by default. This is rare in official OBS releases for Windows but can happen in certain custom builds.

And that's about it.
