# Developer Settings

## Storing Developer Settings in Config File

This section is about how to have custom developer settings that can be set in Config/DefaultGame.ini file.
Helpful in scenario where you need to be able to modify certain settings via the **config file** instead of going through 
blueprints or C++ to change variables.

Let's see an example where I need to have variables like `MaxPlayersInSession` and
`MaxNumOfTeams` in a config file, so I can modify them easily instead of going through
the complex project


### Step 1 :
- First Step is to create a class derived from `UDeveloperSettings`
- Make sure to add the Config keyword in the `UPROPERTY` of the variable you wish to
get values from config file
- Add the following Params to the `UCLASS` macro based on your requirement

```
PREPROCESSOR PARAMS
* Config - Game(Modify the Config/DefaultGame.ini), MyCustomConfig(For custom config files, not sure if works properly in all builds)
* DefaultConfig - Add this if use Config=Game
* Blueprintable, BlueprintType - Add this if you want the settings to be exposed to blueprint

```
#### Example Code :

``` C++
UCLASS(Config=Game, DefaultConfig,  Blueprintable, BlueprintType)
class MULTIPLAYERSHOOTER_API UMultiplayerSessionsDevSettings : public UDeveloperSettings
{
	GENERATED_BODY()

public:

  // Config Keyword is important
	UPROPERTY(Config, EditDefaultsOnly, BlueprintReadOnly)
	uint8 MaxPlayerInSession;

	UPROPERTY(Config, EditDefaultsOnly, BlueprintReadOnly)
	uint8 MaxTeamCount;
	
};
```
<br>

### Step 2 :

- Add the required values to the config files
- When adding the class name make sure to not add the typical `U` or `A` prefix
of the class names

```
[/Script/ProjectName.ClassNameWithout(A)(U)Prefix]
Var1 = 100
Var2 = 10
```

#### Example 
```
[/Script/MultiplayerShooter.MultiplayerSessionsDevSettings]
MaxPlayerInSession = 100 
MaxTeamCount = 35
```

<br>

### Step 3 :
- It's `MORBIN TIME`, wait no, I mean it's loading time.
- Load the variables in BeginPlay or Init
- `Class Default Object` is already automatically instanced for us and can be 
accessed using `GetDefault<T>()`;


#### Example Code :
```
if (const UMultiplayerSessionsDevSettings* MultiplayerSessionsDevSettings = GetDefault<UMultiplayerSessionsDevSettings>())
{
	MaxPlayersInSession = MultiplayerSessionsDevSettings->MaxPlayers;
  MaxTeams = MultiplayerSessionsDevSettings->MaxTeamCount;
} 
```

There you go, now you have developer settings set in config file loaded into your game

### References
- <https://forums.unrealengine.com/t/how-to-store-variables-to-a-custom-ini-file/330274/6>
- <https://www.tomlooman.com/unreal-engine-developer-settings/>