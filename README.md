# C Virtual Synthesizer 🎹
Small project for a software virtual analog synthesizer written in C for Linux and Windows as a standalone executable and a CLever Audio Plugin (CLAP) usable in REAPER.  
Tested on a Linux Mint Debian Edition machine, a Windows 11 machine and through WSL on Debian.

# Features 🎵
- 3 oscillator synthesizer
- Up to 6 note polyphony (arbitrary, just change the constant in the defs.h header )
- ADSR envelope
- Low pass filter with ADSR envelope
- Detune
- Distortion
- LFO that can modulate either filter cutoff, amplification or detune
- Keyboard input
- MIDI input
- Raygui graphical user interface for standalone mode
- Saving and loading presets configurations XML files
- Recording the synth output into a WAV file
- CLAP virtual instrument plugin usable in REAPER (not tested in other DAWs or plugin hosts, but if they offcially support CLAP plugins it should work, let me know if it doesn't)

# GUI 🖼️
## Standalone mode
The GUI shows all of the informations about the synth and let the user configure its parameters graphically :
- Waveform of the sound output
- ADSR envelope parameters level
- Filter ADSR envelope parameters level
- Filter cutoff level
- Amplification level
- Detune level
- Oscillators waveforms
- LFO waveform, modulated parameter and frequency
- Distortion level and overdrive
- Buttons for loading and saving presets into the preset folder
- Button for recording and stop recording into a WAV file
- Piano keyboard showing which keys are being pressed
## Plugin mode
- The GUI of the CLAP plugin is currently unfinished, but you can use the parameter view in REAPER to access every parameter graphically, which may be better than anything I could come up with.

\
![alt text](https://github.com/gpasques-gh/ALSA_raygui_Synthesizer/blob/main/synth.png "GUI screenshot")

# MIDI Input 🎹
The MIDI input should work with all USB MIDI keyboards, to use the synth with your keyboard just follow these two steps :
## Windows
- I did not find a command as simple as `amidi -l` on Linux to list Windows MIDI hardware IDs, so if you want to use MIDI on Windows, just run the synth this way `.\bin\synth -midi 0`. The program should print any available MIDI device and their IDs, then close the program and run it again with the correct ID `.\bin\synth -midi <ID>`.
## Linux
- Using the `amidi -l` command, get your MIDI device hardware id (should look something similar to `hw:1,0,0`)
- Then run the synth : `./bin/synth -midi <hardware id>`

# Keyboard input ⌨️
- The synthesizer can always be played with a computer keyboard, even when using MIDI mode.
- The keyboard simulates a piano keyboard, starting from `a` (`q` in AZERTY) to `j`.  
- The up arrow key move the keys up an octave and the down arrow key move the keys down an octave.
- This current implementation is more useful for testing than it is for playing, I'm planning on adding a second row of keys in the future.

# Dependencies 💻

## Cross-platform
These dependencies are automatically downloaded (and verified with a SHA256 checksum) by CMake with FetchContent at the first configure step, they are pinned to a specific commit :
- raylib
- raygui
- CLAP
- libxml2

stb_truetype is a single header vendored in the `third_party/` directory, no setup needed.

## Linux
All of these dependencies should be found in any Linux package manager, but they were only tested on APT :
- libasound-dev
- zenity

## Windows
This project use the GCC compiler so you would need to install it, I use the MinGW-64 one from scoop.
  
# Compilation 🛠️
To compile the projet as a standalone executable : `cmake -B build && cmake --build build`.
To compile the project as a CLAP plugin : `cmake -B build -DBUILD_CLAP_PLUGIN=ON && cmake --build build`.
To build both : `cmake -B build -DBUILD_CLAP_PLUGIN=ON -DBUILD_STANDALONE=ON && cmake --build build`.
The standalone executable is output to `build/synth` (or `synth.exe` on Windows), the plugin to `build/synth.clap`.
Don't forget to add the project directory to the CLAP plugin search path of your DAW.
Create the `presets/` and `audio/` directories in the base project folder in order to use the presets saving and audio recording functionnalities.
  
# Contribute & feedback
Don't hesitate to give feedback and contribute to the project!  
If you encounter any bug (and you most probably will), please report them to me if you have the time!
