# How does RL2.ModLoader work?
Today we want to bring closer to you the answer to this question, by analyzing the mod loader's source code, and explaining a few tricky parts.

|Table of contents      |
|-----------------------|
|1. Loading the loader  |
|2. Starting up         |
|3. Loading mods        |

## Loading the loader
Unity based games use two very important `.json` files, to determine which assmeblies (modules) to load. The first one of these files is `ScriptingAssemblies.json`. This file consists of a JSON object, containing two properties: `names` and `types`.  These files describe which assemblies to load (located in `GameInstallationPath/Game_Data/Managed`), and what their type is.

To load RL2.ModLoader, we start by adding our assembly to these lists.

![ScriptingAssemblies.json](https://raw.githubusercontent.com/RL2-API/.github/refs/heads/main/articles/how-does-rl2-modloader-work/1.png)

This now ensures that RL2.ModLoader is loaded when the game starts up.
The 16 in the `types` property signifies that the assmebly is an external thing, not provided by Unity (I think).

## Starting up

The second one of the important JSON files is `RuntimeInitializeOnLoads.json`. It contains metadata about the code entrypoint of an assembly.

![RuntimeInitializeOnLoads.json](https://raw.githubusercontent.com/RL2-API/.github/refs/heads/main/articles/how-does-rl2-modloader-work/2.png)

Here we add information about RL2.ModLoader's entrypoint - the name of the assembly, the namespace, the class, and the `static void` method that will execute our loading logic. We set `isUnityClass` to false (logically), and `loadTypes` to 2, which is equal to `RuntimeInitializeLoadType.AfterAssembliesLoaded`.

This makes the Unity engine call the method at application startup, beginning the mod loading cycle.

## Loading mods
The `ModLoader.Initialize` method looks like this.

![ModLoader.Initialize](https://raw.githubusercontent.com/RL2-API/.github/refs/heads/main/articles/how-does-rl2-modloader-work/3.png)

It invokes several methods, which we will go through now.

### 1.`EnsureModsDirectoryExists`
Self explanatory. We check whether the `GameInstallation/Rogue Legacy 2_Data/Mods` directory exists, and creates it if it doesn't.

![ModLoader.EnsureModsDirectoryExists](https://raw.githubusercontent.com/RL2-API/.github/refs/heads/main/articles/how-does-rl2-modloader-work/4.png)

### 2. `CommandManager.RegisterCommands`
This method registers the command from the provided assembly. Here we pass in the current executing one, which is the RL2.ModLoader assembly. This works by scanning the assembly for any `static` methods with return type of `void` marked with the `[Command("name")]` attribute, and [adding them to the registered commands list](https://github.com/RL2-API/RL2.ModLoader/blob/main/RL2.ModLoader/Console/CommandManager.cs#L22).

This loads the two [built-in commands](https://github.com/RL2-API/RL2.ModLoader/blob/main/RL2.ModLoader/BuiltinCommands.cs):

![ModLoader.BuiltinCommands](https://raw.githubusercontent.com/RL2-API/.github/refs/heads/main/articles/how-does-rl2-modloader-work/5.png)

### 3. `CreateModList`
This method is very straight forward. It ensures the `enabled.json` file exists and isn't malformed, and then proceeds to load it, creating an instance of the `ModList` class:

![ModList](https://raw.githubusercontent.com/RL2-API/.github/refs/heads/main/articles/how-does-rl2-modloader-work/6.png)

This object will be used in a further step.

### 4. `LoadModManifests`
In this step we search the previously created `Mods` directory for `.mod.json` files, and create `ModManifest` class instances...

![ModManifest](https://raw.githubusercontent.com/RL2-API/.github/refs/heads/main/articles/how-does-rl2-modloader-work/7.png)

... from each file we find, and add them to a list, including their path, for analysis and loading in the next step.