
# Unity Debugging
These are my personal notes of how to get Unity Debugging to work.

The commonly suggested doc is the dnSpy version [here](https://github.com/dnSpy/dnSpy), but it talks about mono and hacked builds, which is not longer correct.

These instructions are for .NET x64 games, which is basically all Unity games these days.
# Visual Studio
Install Visual Studio with Unity Engine option.

Note - I believe that there is an issue where one of the .dll's aren't installed with VS2022.  I think this is the one that I had to install VS2019 just for that .dll.


# BepinEx 
If the game is using BepinEx and the version is 5.4.23.1 or higher, it is considerably easier to debug.  That BepinEx version uses Doorstop 4.

Otherwise, see [Instructions for Games Without BepinEx or BepinEx Prior to 5.4.23.1](#instructions-for-games-without-bepinex-or-bepinex-prior-to-54231)

## Steps

Recompile and deploy the game's DLL as per the [Game's Debug .dll](#games-debug-dll) section

In the game's root directory, open the doorstop_config.ini and change the debug_enabled to true as shown below.
```ini
# If true, Mono debugger server will be enabled
debug_enabled = true
```

Run the game.

Visual Studio should still be open from the previous compile.  Place a breakpoint on a line in the code that is expected to be hit. 
An Update() function is usually called frequently.  

In the Visual Studio menu, go to Debug > Attach Unity Debugger.
In the "Select Unity Instance" dialog, click the "Input IP" button.
In the "Custom IP endpoint" dialog, change the IP to 127.0.0.1:10000.  This is the same value as the debug_address in the doorstop_config.ini.
Click OK.


Assuming the line with the breakpoint is invoked, Visual Studio should now break on that breakpoint.

## Stopping Before Game Load.
Sometimes it is necessary to debug code that is run very early in the game's startup.
In this case, set debug_suspend to true in the doorstop_config.ini

```ini
# If true and debug_enabled is true, Mono debugger server will suspend the game execution until a debugger is attached
debug_suspend = true
```

Note that when running the game in this mode, it will appear that the game is not running at all.  Once the debugger is attached, the game will proceed as normal.

# Instructions for Games Without BepinEx or BepinEx Prior to 5.4.23.1

## Find the Unity Version
* Go to the .exe for the game and go to properties.

![](./media/GamesEditorVersion.png) 

* Install the Unity Editor for that version.  Note, the "Unity Hub" is the app that has to be installed.  When that app is run, there are options to install the editors.
	* https://learn.unity.com/tutorial/install-the-unity-hub-and-editor#

* Go to the editor's Playback directory: 
```
C:\Program Files\Unity\Hub\Editor\2022.3.4f1\Editor\Data\PlaybackEngines\windowsstandalonesupport\Variations\win64_player_development_mono
```
In this example, the Unit version is 2022.3.4.f1

Copy the files:
```
WindowsPlayer.exe
WinPixEventRuntime.dll
UnityPlayer.dll
```

To the Game's directory.
Rename `WindowsPlayer.exe` to the game's .exe (overwriting)


# boot.config


  Add the following to the game's *_Data/boot.config file.



```
player-connection-mode=Listen
player-connection-debug=1
player-connection-wait-timeout=-1
wait-for-managed-debugger=0
```

### Wait for debugger at start

To attach a debugger to the game before Unity actually starts, add 
`wait=for-managed-debugger=1`
to the boot.config

This is useful if the code is at the very start of the game.

# Game's Debug .dll

Decompile the game's .DLL (Usually `Assembly-CSharp.dll` in the game's `/*Data/Managed/` folder).
ILSpy is probably the best tool for this. See [Using ILSpy to decompile the .dll](#using-ilspy-to-decompile-the-dll) below for the use.

If you have winget, the package is ``icsharpcode.ILSpy``

Otherwise, the offical page is [here](https://github.com/icsharpcode/ILSpy)

Open the .csproj file in Visual Studio and compile it in debug mode. It is common to need to add additional library references to the project during the compile. 
All the necessary .dll's will be in the Game's `/*Data/Managed/` directory.

The project's .NET target in the project's properties page may also need to set to a higher version.


Copy the .dll and the .pdb to the game's Managed directory and overwrite the .dll.



# Using ILSpy to decompile the .dll
Open ILSpy and find the game's .dll

Make sure to right click the .dll and click "Load Dependencies" before decompiling.
![Load Dependencies](media/LoadDependencies.png)

Then use the menu to "Save Code", which will ask for a directory.  The file will be a .csproj file and the directory will contain all the code.

# Attach Debugger

Run the game.  When the game is running, attach the debugger:
![](media/UnityDebuggerAttach.png)



There should be an Instance window with an entry in the list:
![](media/AttachUnity.png)

Select the entry and hit ok.

## In Game indicator
Also, when in the game (even without attaching the debugger), there should be a "Development Build" indicator in the lower right, and the Dev console on the left.

![](media/DevelopmentBuildExample.png)



# Testing
Put a breakpoint on some function that is called.  Like the World Manager or GameScreen, etc.  Usually the Update or LateUpdate functions are good choices since they are called every frame.

The debugger should break on that point.

![](/media/DebuggerBreakpointExample.png)


# Game Updates
Every time the game is updated, this install will need to happen again as any changed files will be overwritten by the installer.
Also, the code for the .dll has to be decompiled since it may have changed.

# Restoring the Game
To restore the game back to normal, use your game providers "Verify Files" functionality.  

# Issues

## Steam Launcher

For steam games, launch from the launcher and not the game's exe.  Steam's DRM will often prevent execution.

The Shortcuts created by steam are a decent option.

## Conditional breakpoints

Visual Studio conditional breakpoints generally don't work.  Usually Unity and Visual Studio will just lock up.


## Debugger not working
### Connection Refused / not showing up.
If for some reason debugging is not working or showing up, try completely exiting Steam and starting the game again.

If that doesn't work, sometimes it requires an entire system restart.  I'm not sure why, but this fixes it.  Just signing off and on doesn't work.



