# Pandora Behaviour Engine+

<div>
    <a href="https://github.com/Monitor144hz/Pandora-Behaviour-Engine-Plus/actions/workflows/build.yaml" target="_blank">
        <img src="https://github.com/Monitor144hz/Pandora-Behaviour-Engine-Plus/actions/workflows/build.yaml/badge.svg" alt="Build">
    </a>
    <a href="https://discord.gg/8nUQCWMn3w" target="_blank">
        <img src="https://img.shields.io/discord/1218265541106466826?logo=discord&logoColor=white&color=7289da" alt="Discord">
    </a>
</div>

<br/>

A modular and lightweight behavior engine for TES Skyrim SE, for creatures and humanoids.  

Built with backwards compatibility in mind for [Nemesis Unlimited Behavior Engine](https://github.com/ShikyoKira/Project-New-Reign---Nemesis-Main) and FNIS, Pandora is an alternative engine streamlines both the author and user experience through a simplified UI, robust logging, intuitive formats, and fast patching times. Pandora runs on Windows, Linux, and MacOS.

<br/>
<br/>

## Navigation
* [For Users](#for-users)
  * [Quickstart](#quickstart)
  * [Troubleshooting](#troubleshooting)
  * [Patch Order](#patch-order)
  * [Mod Cache](#mod-cache)
  * [Startup Arguments](#startup-arguments)
* [For Mod Authors](#for-mod-authors)
  * [Patch Format](#patch-format)
  * [File Targeting](#file-targeting)
    * [Unique Identifiers](#unique-identifiers)
    * [Indirect Identifiers](#indirect-identifiers)
  * [AnimData](#animdata)
    * [Manual Addition](#manual-addition)
  * [AnimSetData](#animsetdata)
    * [Adding AnimSetData Animations](#adding-animsetdata-animations)
  * [Graph Injection](#graph-injection)
  * [Custom Projects](#custom-projects)
  * [Verbose Logging](#verbose-logging)
    * [Severity](#severity)
    * [Component](#component)
    * [Input](#input)
* [For Developers](#for-developers)
  * [Build Requirements](#build-requirements) 

<br/>
<br/>

## For Users

### Quickstart
Install [.NET 8 Desktop Runtime](https://dotnet.microsoft.com/en-us/download/dotnet/8.0) if you do not have it installed.  

<br/>

<details>

<summary>MO2 Users</summary>
<br/>

1. Install Pandora Behaviour Engine as a mod* and make sure it is active.
2. Add Pandora as an [application for MO2](https://stepmodifications.org/wiki/Guide:Mod_Organizer#General_Application_Setup)
    * Having a dedicated output mod is recommended to keep the files that the tool generates in one place.
3. Run the program from within MO2, using the menu on the right.
4. Tick the patches you want and click Launch. 


*<sub>Alternatively, you may install Pandora outside of the mod manager and set its output path to the Skyrim installation data folder using the `-o:"{path}"` startup argument.<sub/>

</details>
<br/>
<details>

<summary>Vortex Users</summary>
<br/>

1. Install Pandora Behaviour Engine outside of the mods folder.
2. Add it to the tools dashboard.
  2.a) It is highly recommended you set a custom output mod folder using `-o:{path}` in the 'Command Line' field.
  2.b) Ensure that the 'Start In' field is set to the Skyrim data directory.
4. Run the program either manually or after adding it to the tools dashboard.
5. Tick the patches you want and click Launch. 

</details>

<br/>

### Patch Order
Pandora has a drag and drop priority system. Higher priority mods will overwrite conflicting changes from lower priority mods. Direct behaviour conflicts are rare, so manual resolving is almost never needed, but the option is there just in case. 


### Mod Cache
Pandora saves the active mods to an external cache file after the engine successfully finishes its patching process. When the cache is loaded, all active mods are shown at the top with relative priority preserved, for better readability. To clear the cache, delete `Pandora_Engine/ActiveMods.txt`. 



### Troubleshooting

**Something's gone wrong and I'm crashing/bugging out with the output generated by Pandora!**

In case of any problems with the output, `Engine.log` names patches with failed edits for easier troubleshooting. You may want to try again with the mods that failed most frequently in the log disabled.

It's recommended to pass the log on to the relevant mod author if a specific mod was the issue, as well as the engine developer.

<br/>

**My animation has no movement!**

One of the mods does not have motion data. That is not a Pandora issue, it is a mod issue. 

<br/>

**It says 0 animations added or fails to output files!**

Run the engine as administrator or move the engine install out of a protected location.

<br/>

**There are a lot of warnings in Engine.log!**

Warnings are not a major issue unless they noticeably interfere with the in-game behavior in some shape or form. Please don't report warnings coming from other mods as a bug.

**Pandora doesn't read FNIS mods!***

Use the `-tesv:` argument if your game path is different from the registry path, or if you have multiple installs. Otherwise, ensure your registry path is correct. Read the [startup arguments](#startup-arguments) section for more information.

<br/>
<br/>

## Startup Arguments
Pandora has a variety of startup arguments to support customizability. 

- `-autorun`: runs the engine using the same active mods as cached from the last successful run.
- `-autoclose`: closes the engine automatically upon finishing a single launch.
- `-skyrimDebug64`: produces debug `.xml` files alongside normal `.hkx` output. Only for authors that know what they're doing.
- `-o:{output path}`: sets a custom output path.
- `-tesv:{game path}`: sets the path to the game directory (the root folder containing the exe, not the data folder!). Intended for users with Wabbajack "[Stock Game](https://github.com/LivelyDismay/Learn-To-Mod/blob/main/lessons/Setting%20up%20Stock%20Game%20for%20Skyrim%20SE.md)" setup or with multiple installations.

## For Mod Authors
This section exists to inform current behavior authors of the key differences and features of Pandora, it's not a guide for making behavior mods.
### Patch Format
In addition to supporting almost all of Nemesis patch format, Pandora uses its own format which is more efficient and fault-tolerant.

Patches in the new Pandora format do not use multiple text files per behavior graph, only a single xml file per graph with all edits self-contained in the below format.
```xml

<patch>
    <replace>
    <edit path="#xxxx\...\..."><!-- content --></edit>
    </replace>
    <insert>
    <edit path="#xxxx\...\..."><!-- content --></edit>
    </insert>
    <append>
    <edit path="#xxxx\...\..."><!-- content --></edit>
    </append>
    <loose>
    <edit path="#xxxx\...\..."><!--content --></edit>
    </loose>
</patch>
```
Edits have the following format:
```xml
<edit path="#xxxx\...\..."><!-- XText xor XElement --></edit>
```

An example of a patch file:

```xml
<patch>
	<replace>
		<edit path="#0885/legs/Element0/maxAnkleHeightMS">
			<hkparam name="maxAnkleHeightMS">0.700000</hkparam>
		</edit>
		<edit path="#0885/legs/Element0/hipIndex">
			<hkparam name="hipIndex">12</hkparam>
		</edit>
		<edit path="#0885/legs/Element0/kneeIndex">
			<hkparam name="kneeIndex">13</hkparam>
		</edit>
		<edit path="#0885/legs/Element0/ankleIndex">
			<hkparam name="ankleIndex">14</hkparam>
		</edit>
		<edit path="#0885/legs/Element0/isPlantedMS">
			<hkparam name="isPlantedMS">false</hkparam>
		</edit>
	</replace>
</patch>
```

### File Targeting

#### Unique Identifiers
Pandora supports the same [patch file format](https://github.com/ShikyoKira/Project-New-Reign---Nemesis-Sub-tool) as Nemesis, but the folder system is expanded on to provide creature compatibility.  Patch folders can use the short name or their full unique name to be recognized by the engine.  
<br/>
The full identifying name is `[ProjectName]~[FileName]`. For example:
<br/>

* `0_master` is also recognized as `defaultmale~0_master`.
    * `_1stperson~0_master` targets a different file in the 1st person project.

<br/>

* `horsebehavior` is also recognized as `defaultmale~horsebehavior`
    * `horseproject~horsebehavior` targets a different file in the horse project.
  
<br/>
<br/>
Note that using the full name does not separate files that are already shared between projects, it is only there to resolve naming conflicts.
<br/>
<br/>

#### Indirect Identifiers
Skeleton and character files can be targeted using either short or full names. Alternatively they can also use:
<br/>
<br/>
* `[ProjectName]_character` for targeting the character file of the project.

<br/>

* `[ProjectName]_skeleton` for targeting the skeleton file of the project.
<br/>

### AnimData

#### Manual Addition
Mods need to register their clip generators manually if they want motion.  

<br/>

To do so, create a folder named `animdata` and create a `[ProjectName].txt` file with each line containing the name of one clip generator. Don't worry about repetition as the engine will automatically discard duplicate clip generator names (case sensitive).  

As of v0.3.0-alpha Pandora generates a compatible format for existing animsingledatafiles in the Nemesis format. 

<br/>
<br/>

### AnimSetData
AnimSetData is a unique case that Pandora does not automatically generate, because it's not necessary for all new animations and would be a waste of time to find and automatically generate for every new animation file. 
<br/>

It's only needed for adding paired animations and other edge cases this section can be skipped if it's not relevant.
<br/>
Currently Pandora only supports adding anim info, for paired animations. 
<br/>

#### Adding AnimSetData Animations
To make additions to AnimSetData, authors must define animation paths in a separate file with this folder structure.
<br/>
```
[ModFolder]\animationsetdatasinglefile\[ProjectName]\[SetName].txt
```
<br/>
<br/>

Each file of the project folder should have paths with the animation relative to the data folder. For example, in path:
```
..\testmod\animationsetdatasinglefile\DefaultMale\H2HDual.txt
```

or 

```
..\testmod\animationsetdatasinglefile\DefaultMale.txt
```

The second example adds the animation to all the sets automatically under that project, it's intended to save time for mod authors that want to register animations for the entire project easily without a lot of copy pasting.

<br/>
There would be something like:
<br/>

```
meshes\actors\character\animations\killmove1.hkx
meshes\actors\character\animations\killmove2.hkx
meshes\actors\character\animations\killmove3.hkx
```

These animations are then parsed, encoded with the right format, and added to animsetdata.
<br/>

Formerly, authors would have to encode the file and paths themselves and add it to the copied set file between two comments indicating the operation. Now it's as easy as writing a single line in a new file with the same name!


<br/>
<br/>

### Graph Injection

Graph injection is the process of injecting properties from modded graphs into the vanilla behavior graphs. 
A graph is an hkx file that is specialized for behavior and contains a "graph" of nodes. 


FNIS creates custom graphs by reading the animlist files when creating mod behavior through `GenerateFNISforModders.exe`.

First, the custom graph must be unpacked into a readable xml, using something like [hkxconv](https://github.com/ret2end/hkxconv/releases) for 64 bit(SE/AE) files or [hkxcmd](https://www.nexusmods.com/skyrim/mods/1797) for 32 bit(LE) files. 

To inject a custom graph reference, including its variables and animations into a specific graph, make a subfolder named `inject` in the identifying folder of the behavior graph that you want to inject into. Then, in the `inject` folder, make another folder that has the same name as the `hkbStateMachine` that you want to inject under. Then place the custom graph under this folder.

To inject a custom graph's animations (skimmed from all hkbClipGenerators) into a specific character file, make a subfolder name `inject` in the identifying folder of the character file that you want to inject into. Then place the custom graph under this folder.

Graph injection is an experimental feature and should only be made by authors that know what they are doing.

More information on the [wiki page](https://github.com/Monitor144hz/Pandora-Behaviour-Engine-Plus/wiki/Graph-Injection)

<br/>
<br/>

### Custom Projects

Not yet implemented.

<br/>
<br/>

### Verbose Logging
Log messages are kept in a separate file called `Engine.log`. 

Messages loosely follow this format:
<br/>

`[Severity]: [Component] > [Data] > [Operation] > [Input] > [Status]`

<br/>

Most of these are self explanatory but some of the more obscure formatting is explained below.
<br/>

#### Severity: 

* `INFO` means it's just there as a notification. Nothing has gone wrong.
* `WARN` means something is unexpected and could be a potential issue. 
* `ERROR` means the subject prevented the engine from performing some part of its work and is likely to be an issue.
* `FATAL` means the engine has failed completely because of a significant fault, usually on export. These should be reported immediately to the engine developer(s).

#### Component:

* The `Assembler` is responsible for parsing the patch format to derive an operation for each edit. Usually the name is preceded by the format that it parses.
  * The assembler responsible for parsing Nemesis patches is the `Nemesis Assembler`.
* The `Dispatcher` is responsible for saving the patch edits and applying them to the target file.
* The `Validator` is responsible for validating all the edits made by the Dispatcher after it is run.

#### Input:

The input is usually a parameter of the operation that is significant for manual debugging. The most common input are xml paths which denote the path to the area of the xml file where an edit failed.
<br/>

Can you see why this edit failed?
<br/>

```


Dispatcher > "ExampleMod" > defaultfemale~1hm_behavior > Replace > Element > #2521/event/Element0/id > FAILED
```

```xml
<hkobject name="#2521" class="BSRagdollContactListenerModifier" signature="0x8003d8ce">
  <hkparam name="variableBindingSet">null</hkparam>
  <hkparam name="userData">2</hkparam>
  <hkparam name="name">VictimState_RagdollListener</hkparam>
  <hkparam name="enable">true</hkparam>
  <hkparam name="contactEvent">
    <hkobject>
      <hkparam name="id">74</hkparam>
      <hkparam name="payload">null</hkparam>
    </hkobject>
  </hkparam>
  <hkparam name="bones">#2520</hkparam>
</hkobject>
```

<details>
  <summary>
    Answer:
  </summary>
  
  The path should be `#2521/contactEvent/Element0/id`, not `#2521/event/Element0/id`. In this case, it should be fixed by the author, or reported to the author if found by a user.
</details>

## For Developers 
### Build Requirements
* [Nito HashAlgorithms CRC](https://www.nuget.org/packages/Nito.HashAlgorithms.CRC)
* [NLog](https://www.nuget.org/packages/NLog/)
* [XmlCake](https://github.com/Monitor221hz/XML.Cake.NET)
* [HKX2](https://github.com/Monitor221hz/HKX2-Enhanced-Library) 
