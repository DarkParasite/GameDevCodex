# Setup

There are two ways you can set up Steam SDK, 

- METHOD - 1 : Download the SDK and link it to your project 
- METHOD - 2 :  Use the one that comes with Unreal Engine 

## Downloading and linking - METHOD 1

In this method, we download the Steam SDK from the Steam website and link it to the project. 

### Step 1 : Download the SDK from SteamWorks

You can download the SDK from here : [Steam SDK](https://partner.steamgames.com/doc/sdk)  
You would have to log in with your **Steam Account** to download it

### Step 2 : Adding and linking to your project

#### Placing the headers and libraries 

- After downloading the SDK, extract the folder.
- You will be able to find a folder called `steam` with all the header files in 
`sdk/public/`.
- The best directory to put this `steam` folder would be in your project's source directory,
so put it under `Source/ThirdParty/`, it goes by the Unreal Engine standard.
- Now we need to get the `steam_api64.dll` which is in the `sdk/redistributable_bin/win64`
and add it to your `Binaries/Win64` in your game project folder, but it might be a hassle 
to put this everytime you rebuild your project, so I have a tip for that, just follow along.
- Now copy the `lib` and `dll` file in the sdk/redistributable_bin/win64 and put it in the
`Source/ThirdParty/steam/lib/` folder.

#### Including the headers and lib to the project using Build.cs file

As I said, it might be hassle to copy the `dll` file all the time, so I have included the
`Build.cs` code that automatically put the `dll` file in `steam/lib` to the `Binaries` folder. 

!!! info "Build.cs File"
    This is the configuration file that Unreal Engine uses to build your project, it can be found
    in your project's `Source/Project_Name/` folder with the name `Project_Name.Build.cs`.

```C#
using System.IO;
using UnrealBuildTool;

public class ThirdPersonTemplate : ModuleRules
{
	public ThirdPersonTemplate(ReadOnlyTargetRules Target) : base(Target)
	{
		PCHUsage = PCHUsageMode.UseExplicitOrSharedPCHs;
	
		PublicDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine", "InputCore", "Sockets", "Networking", "Http", "Json", "JsonUtilities", "MoviePlayer", "UMG", "Slate", "SlateCore" });

		PrivateDependencyModuleNames.AddRange(new string[] {  });

		if (Target.Platform == UnrealTargetPlatform.Win64)
      			PublicAdditionalLibraries.Add(Path.Combine(ModuleDirectory, "Steam", "lib", "steam_api64.lib"));
		else if (Target.Platform == UnrealTargetPlatform.Linux)
			PublicAdditionalLibraries.Add(Path.Combine(ModuleDirectory, "Steam", "lib", "libsteam_api.so"));

		try
		{
			string SteamDLLFileName = "steam_api64.dll";
			string SteamDLLPath = Path.Combine(ModuleDirectory, "Steam", "lib", SteamDLLFileName);
			string ProjectPath = Directory.GetParent(ModuleDirectory).Parent.ToString();
			string BinariesDir = Path.Combine(ProjectPath, "Binaries", Target.Platform.ToString());

			if (!Directory.Exists(BinariesDir))
				Directory.CreateDirectory(BinariesDir);

			File.Copy(SteamDLLPath, Path.Combine(BinariesDir, SteamDLLFileName), true);
		} catch { }
	}
}
```

I got this code from this GitHub Gist : [Build Code Repo](https://gist.github.com/astutejoe/b3502dfe2bd594424657b86f69cc7d53)  

#### What It Does?
This code links the `lib` and `dll` file to the project. 

### Step 3: Creating the steam_appid.txt file

If you don't have a DevAppID for your game, then create a file called `steam_appid.txt` and put
it in your root folder. In the file just add `480` in the first line, this is the Steam AppID
that Steam provides for developers to test. The Steam API requires this file otherwise it won't
work.

### Step 4 : Including in Project and Testing

#### Including

If you had followed along and put the `steam` folder wit the header file, then you can include like
this, otherwise give the correct include path.

You can see that, some warnings are disabled while including the `steam_api.h` and then reset, this
is because, the `steam_api.h` header files gives off some compiler warning, I'm pretty sure they
are harmless and probably are better faster method that Steam developers came up with.

```C++
#pragma warning(push) // Disable warnings
#pragma warning(disable : 4996)
#include "ThirdParty/steam/steam_api.h"
#pragma warning(pop)
```

#### Time for testing!

Add this following code to the BeginPlay and run your project, if you have set up everything correctly
then you will be able to see your SteamID on your output log.

!!! info "Steam must be running"
    Make sure to have Steam running obviously otherwise it won't work

```C++
	if (SteamAPI_Init())
	{
		if (SteamUser() && SteamFriends())
		{
			const CSteamID PlayerID = SteamUser()->GetSteamID();
			UE_LOG(SteamHelperSubsystem, Warning, TEXT("MySteamID : %llu"), PlayerID.ConvertToUint64());
		}
		else
		{
			UE_LOG(SteamHelperSubsystem, Warning, TEXT("Steam not running"));
		}
	}
```

## Use the one that comes with Unreal Engine - METHOD 2

If you check the `Engine/Source/ThirdParty` in your Unreal Engine installation folder
you will be able to find a folder called `SteamWorks`, which actually has SteamSDK. So
Unreal Engine actually has SteamSDK since the SteamSubsystem uses it.

### Why would you use that?
Well, there are some advantages

- If you use SteamSubsystem in your project then using different versions might cause
bugs, so using the same headers and libs that Unreal Engine uses will prevent those bugs
- You don't need to download and setup, linking the one that is in Unreal Engine is easier

### How to link it?

Just add this to your project's `Build.cs` file. If you notice that the path just start as ThirdParty
this is because, by default, Unreal Engine's `Source` folder is considered as the root, so I have
combined the rest of the path here to the library file.

!!! warning "Version Name"
    Different version of Unreal Engine has different SteamSDK version, so change the `Steamv153`
    with the appropriate version you have in your Unreal Engine installation folder.

```C#
PublicAdditionalLibraries.Add(Path.Combine("ThirdParty", "Steamworks", "Steamv153", "sdk", "redistributable_bin", "win64", "steam_api64.lib"));
```

### Should I put the steam_api64.dll file to the Binaries folder?

Well, it's not required if you have `SteamSubsystem` enabled, as Unreal Engine automatically
puts it for you when you build your project. Otherwise, you might have to.

### Including and Testing

You can include it in your project like this

!!! warning "Version Name"
    Different version of Unreal Engine has different SteamSDK version, so change the `Steamv153`
    with the appropriate version you have in your Unreal Engine installation folder.

```C++
#pragma warning(push) // Disable warnings
#pragma warning(disable : 4996)
#include "Steamworks/Steamv153/sdk/public/steam/steam_api.h"
#pragma warning(pop)
```

Test it with the code I have included in the first method : [Testing](Setup.md#time-for-testing)







