TSS AUDIO SLOT CONVERTER
=========================

Plain-language guide, technical overview, and security information

Version: TSSAudioSlotConverter_Toolkit_Fixed


1. WHAT THIS TOOL DOES
----------------------

This tool prepares music files for the Totentanz Sound System Standalone mod.
It accepts WAV and MP3 files, processes them with the same TSS audio processor
used by the original TSS builder, and creates numbered WAV files:

    track_001.wav
    track_002.wav
    track_003.wav
    ...

The generated files can then be copied to:

    red4ext\plugins\AudioXL\sounds\TotentanzSoundSystem\music\

The tool supports up to 30 tracks in one run.

The original music files are not edited. They remain in the folder with their
original names. The converter creates new processed copies instead.


2. THE SIMPLE USER WORKFLOW
---------------------------

1. Extract the ZIP to a normal working folder. Do not extract it directly into
a Windows system folder.

2. Put the original WAV and/or MP3 files beside the root-level file named:

       convert_audio.bat

   Example:

       My TSS Converter\
       |-- convert_audio.bat
       |-- My Song 01.mp3
       |-- My Song 02.wav
       |-- TSSAudioSlotConverter_Toolkit\

3. Double-click convert_audio.bat.

4. Wait until the command window says that processing has completed.

5. The converter creates track_001.wav, track_002.wav, and so on beside the
BAT file.

6. Copy those generated track files to the TSS Standalone music folder.

Do not run build_windows_exe.bat. That file is only for a developer who wants
to rebuild the optional Python version as a separate executable.


3. WHAT EACH FILE DOES
----------------------

convert_audio.bat

This is a Windows Batch file. It is the file intended for ordinary users to
run. It does not process audio by itself. It starts PowerShell and tells it to
run the converter script stored inside the organized toolkit folder.

The BAT also passes the working-folder path safely to PowerShell. It does not
send the music anywhere and does not use an internet connection by itself.


TSSAudioSlotConverter_Toolkit\convert_audio.ps1

This is the main orchestration script, written in PowerShell. PowerShell is a
standard Windows scripting environment. The script performs the following
steps:

1. Finds the folder containing the user's original music files.
2. Checks that TSSAudioProcessor.exe is present.
3. Searches only the working folder for WAV and MP3 files.
4. Ignores already-generated files whose names match track_001.wav through
   track_999.wav.
5. Sorts the input files alphabetically.
6. Uses at most the first 30 input files.
7. Downloads FFmpeg only if at least one MP3 file is present and no cached
   FFmpeg executable exists yet.
8. Creates a temporary _tss_work folder.
9. Converts MP3 files to WAV files when necessary.
10. Copies WAV files into the temporary TSS input folder.
11. Runs the original TSSAudioProcessor.exe.
12. Finds the processed audio produced by that processor.
13. Deletes old generated track_*.wav files in the working folder.
14. Copies the new processed files to track_001.wav, track_002.wav, and so on.
15. Deletes the temporary _tss_work folder.

The script uses ErrorActionPreference = Stop. If an important step fails, it
stops instead of silently continuing with an incomplete package.


TSSAudioSlotConverter_Toolkit\TSSAudioProcessor.exe

This is the original native Windows processor from the TSS Standalone Builder.
It is the component that performs the TSS compressor/limiter processing and
creates AudioXL-ready output. It is a precompiled Windows executable, not a
PowerShell script.

Because it is precompiled, its internal source code is not included in this
ZIP. This is the only component in the toolkit that is not plain-text source
code. Users who do not want to run a precompiled executable should not use the
converter and should instead process their files by another trusted method.

For verification, the SHA-256 hash of the distributed processor is:

    235e6d91a6bccc844ed24ab6854a088f825a1bd2d6f73d591124bd08c7aab6bc

A SHA-256 hash is a fingerprint for a file. If the hash of your copy is
 different, do not run it until the source of the difference is understood.


TSSAudioSlotConverter_Toolkit\TSSAudioSlotConverter.py

This is a readable Python reference implementation for the audio conversion
workflow. It is not used by convert_audio.bat. It is included for transparency
and for developers who want to inspect or adapt the logic.

The normal BAT workflow uses the PowerShell script and the original
TSSAudioProcessor.exe. Python is not required for ordinary use.


TSSAudioSlotConverter_Toolkit\build_windows_exe.bat

This is an optional developer helper. It can use PyInstaller to build the
Python reference implementation as a separate Windows executable. It is not
needed by users and is not part of the normal conversion workflow.


TSSAudioSlotConverter_Toolkit\requirements.txt

This is a small note for developers. The normal BAT workflow does not require
Python packages. The Python reference script calls an FFmpeg executable through
the command line.


TSSAudioSlotConverter_Toolkit\README.md

This is a short copy of the user instructions. This document contains the
longer explanation of the code and security model.


4. WHEN FFmpeg IS USED
----------------------

WAV-only workflow:

    WAV files -> TSSAudioProcessor.exe -> processed track_00x.wav files

No FFmpeg download is required for this workflow.

MP3 workflow:

    MP3 files -> FFmpeg -> temporary WAV files -> TSSAudioProcessor.exe
              -> processed track_00x.wav files

If at least one MP3 is detected, the PowerShell script downloads an FFmpeg
Windows essentials ZIP only when FFmpeg has not already been cached. The
current download address in the script is:

    https://www.gyan.dev/ffmpeg/builds/packages/ffmpeg-8.1.2-essentials_build.zip

The download is saved temporarily inside:

    TSSAudioSlotConverter_Toolkit\_ffmpeg\

The ZIP is extracted, ffmpeg.exe is located, and the temporary downloaded ZIP
and extraction folder are deleted. The cached ffmpeg.exe remains there so the
same download is not needed on the next MP3 conversion.

The FFmpeg download is the only automatic network operation in the normal
workflow. WAV-only processing does not need an internet connection.

The FFmpeg build is obtained from gyan.dev, one of the Windows build sources
linked from FFmpeg's official download page. Users may independently inspect
or replace that dependency if desired.


5. PRIVACY AND NETWORK BEHAVIOR
-------------------------------

The converter is designed for local processing.

- Music files are read from the local folder.
- Music files are not uploaded to GitHub, Nexus, a TSS server, or any other
  service.
- The original audio files are not modified.
- The processor is run locally on the user's Windows computer.
- WAV-only processing does not contact the internet.
- MP3 processing contacts the FFmpeg download URL only when FFmpeg is missing.
- No account, password, token, telemetry service, or analytics service is used
  by the BAT or PowerShell workflow.

The separate FFmpeg download is not a music upload. It downloads the FFmpeg
program needed to decode MP3 files.


6. FILES THAT ARE CREATED OR REMOVED
------------------------------------

Created beside the BAT:

    track_001.wav
    track_002.wav
    ...

These are the processed output copies.

Created temporarily:

    _tss_work\

This folder contains temporary copies and processor output. It is deleted at
the end of a successful run.

Possible cached dependency folder:

    TSSAudioSlotConverter_Toolkit\_ffmpeg\ffmpeg.exe

This remains after an MP3 conversion so FFmpeg does not have to be downloaded
again.

Before writing new output, the script deletes files in the working folder
matching:

    track_*.wav

This is intentional. It prevents old tracks from remaining in the output set.
It does not delete the user's original files unless they were already named
track_*.wav and located in the converter's working folder. For that reason,
do not place original personal files named track_001.wav, track_002.wav, etc.
beside the BAT unless they are intended to be replaced.


7. AUDIO FORMAT AND ORDER
-------------------------

For MP3 input, FFmpeg creates an intermediate WAV using these settings:

- Stereo: 2 channels
- Sample rate: 48,000 Hz
- Sample format: signed 16-bit PCM
- Video streams ignored

The original TSSAudioProcessor then performs its normal TSS compression and
limiting process.

Input files are sorted alphabetically before being assigned numbers. For
predictable order, use names such as:

    01 - Intro.mp3
    02 - Track A.mp3
    03 - Track B.wav

The converter does not use artist metadata to determine order.


8. SECURITY INFORMATION AND LIMITATIONS
----------------------------------------

No software can honestly be described as guaranteed harmless merely because
it is distributed in a ZIP file. The toolkit contains one precompiled
executable, TSSAudioProcessor.exe. Users should make their own decision about
running it.

The transparency measures provided here are:

- The BAT file is plain text and can be opened in Notepad.
- The PowerShell file is plain text and can be inspected before running.
- The Python reference file is plain text.
- The PowerShell script shows the exact FFmpeg URL.
- The processor is kept in a separate named file rather than hidden in a
  self-extracting installer.
- A SHA-256 hash is provided for the processor.
- The converter does not request administrator privileges.
- The converter does not install a Windows service.
- The converter does not modify the Windows registry.
- The converter does not modify the game installation automatically.
- The converter does not delete the original input music files.

Windows Defender or another antivirus program may still display a warning for
an unknown executable. That warning is not proof of malware, but it should not
be ignored. Users can scan the ZIP and the processor with their preferred
security software before running them.

For maximum caution, users can:

1. Download the archive only from the author's trusted GitHub release.
2. Compare the processor hash with the hash published by the author.
3. Open the BAT and PowerShell files in Notepad.
4. Run the converter in a separate working folder.
5. Block network access temporarily if processing WAV files only.
6. Inspect the generated output before copying it into the game folder.


9. REQUIREMENTS
---------------

- Windows 10 or later is recommended.
- PowerShell must be available. Windows PowerShell is included with supported
  Windows installations.
- Internet access is required only for the first MP3 conversion if FFmpeg is
  not already cached.
- Python is not required for the normal BAT workflow.
- The converter does not need to be installed into Cyberpunk 2077.


10. LICENSE AND THIRD-PARTY COMPONENTS
--------------------------------------

This toolkit includes the TSSAudioProcessor.exe from the TSS Standalone Builder
and downloads an FFmpeg Windows build when MP3 support is needed. Users should
review the license and redistribution terms that apply to each third-party
component before redistributing modified copies.

FFmpeg information:

    https://ffmpeg.org/

Windows build source used by the script:

    https://www.gyan.dev/ffmpeg/builds/

The TSS converter itself is a utility for preparing audio files for the
Totentanz Sound System Standalone mod. It is not affiliated with Microsoft,
FFmpeg, GitHub, Nexus Mods, AudioXL, or CD PROJEKT RED.
