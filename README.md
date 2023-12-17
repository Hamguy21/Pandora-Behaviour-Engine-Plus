# Pandora Behaviour Engine+

A modular and lightweight behavior engine for TES Skyrim SE, for creatures and humanoids.  

Built with backwards compatibility in mind for [Nemesis Unlimited Behavior Engine](https://github.com/ShikyoKira/Project-New-Reign---Nemesis-Main) and FNIS, Pandora is an alternative engine streamlines both the author and user experience through a simplified UI, robust logging, intuitive formats, and fast patching times.


<br/>
<br/>

## Navigation
* [For Users](#for-users)
  * [Quickstart](#quickstart)
  * [Troubleshooting](#troubleshooting)
  * [Patch Order](#patch-order)
  * [Mod Cache](#mod-cache)
  * [Patch Hub](#patch-hub)
* [For Mod Authors](#for-mod-authors)
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
Install [.NET 7 Desktop Runtime](https://dotnet.microsoft.com/en-us/download/dotnet/7.0) if you do not have it installed.  

<br/>

<details>

<summary>MO2 Users</summary>
<br/>

1. Install Pandora Behaviour Engine as a mod and make sure it is active.
2. Add Pandora as an [application for MO2](https://stepmodifications.org/wiki/Guide:Mod_Organizer#General_Application_Setup)
    * Having a dedicated output mod is recommended to keep the files that the tool generates in one place.
3. Run the program from within MO2, using the menu on the right.
4. Tick the patches you want and click Launch. 


</details>
<br/>
<details>

<summary>Vortex Users</summary>
<br/>

1. Install Pandora Behaviour Engine as a mod and make sure it is active.
2. Run the program either manually or after adding it to the tools dashboard.
3. Tick the patches you want and click Launch. 

</details>

<br/>

### Patch Order
Pandora has a drag and drop priority system. Higher priority mods will overwrite conflicting changes from lower priority mods. Direct behaviour conflicts are rare, so manual resolving is almost never needed, but the option is there just in case. 


### Mod Cache
Pandora saves the active mods to an external cache file after the engine successfully finishes its patching process. When the cache is loaded, all active mods are shown at the top with relative priority preserved, for better readability. To clear the cache, delete `Pandora_Engine/ActiveMods.txt`. 

### Patch Hub

Patches for FNIS posers and other mods [available here](https://modding-guild.com/mod/pandora-behaviour-engine-patch-hub/) and also in the releases tab.

Note that hidden mods are not active by default if there was data loaded from a cache.

### Troubleshooting

**Something's gone wrong and I'm crashing/bugging out with the output generated by Pandora!**

<br/>

In case of any problems with the output, `Engine.log` names patches with failed edits for easier troubleshooting. You may want to try again with the mods that failed most frequently in the log disabled.

It's recommended to pass the log on to the relevant mod author if a specific mod was the issue, as well as the engine developer.

<br/>

**My animation has no movement!"**

One of the mods does not have motion data. That is not a Pandora issue, it is a mod issue. 

<br/>
<br/>

## For Mod Authors
This section exists to inform current behavior authors of the key differences and features of Pandora, it's not a guide for making behavior mods.
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
[ModFolder]\animsetdatasinglefile\[ProjectName]\[SetName].txt
```
<br/>
<br/>

Each file of the project folder should have paths with the animation relative to the data folder. For example, in path:
```
..\testmod\animsetdatasinglefile\DefaultMale\H2HDual.txt
```

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
* [XmlCake](https://github.com/Monitor144hz/XML-Cake)
* [HKX2](https://github.com/Monitor144hz/HKX2-Library/tree/wpf) 
