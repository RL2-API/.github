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

This mirrors the attribute with which the `ModLoader.Initialize` method is marked.
```cpp
[RuntimeInitializeOnLoadMethod(RuntimeInitializeLoadType.AfterAssembliesLoaded)]
public static void Initialize() { }
```

This makes the Unity engine call the method, beginning the mod loading cycle.

## Loading mods
The `ModLoader.Initialize` method looks like this.

![ModLoader.Initialize](https://raw.githubusercontent.com/RL2-API/.github/refs/heads/main/articles/how-does-rl2-modloader-work/3.png)