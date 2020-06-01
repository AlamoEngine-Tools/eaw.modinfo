# eaw.modinfo Definition - v1.2.0

A standard definition for Star Wars: Empire at War mod info files.

The info files defined herein allow mod makers and tool makers to specify metainformation about a given Empire at War mod.

### Filename

The metainformation has to be saved in a `JSON` file called `modinfo.json`.

### File Position

The `modinfo.json` has to be located at the top level of the mod folder next to the `data` folder.

**Added for v1.2:**

### File Content

The following sections specify the required and optional content for `eaw.modinfo` in Version 1.2.0.

### Exemplary Content

```json
{
  "name": "The mod's name",
  "summary": "A short summary about the mod in Steam-flavoured BBCode.\nNice, eh?",
  "icon": "relative/path/to/icon.ico",
  "version": "1.0.0.0",
  "dependencies": [
    {
	  "modtype": 0,
	  "identifier": "relative/or/absolute/path"		
	},
	{
	  "modtype": 1,
	  "identifier": "STEAMID"		
	}	
  ],
  "steamdata": {
    "publishedfileid": "xxxxxxxxxx",
    "contentfolder": "folder",
    "visibility": 1,
    "metadata": "",
    "tags":[
      "Multiplayer",
      "Land",
      "Space",
      "FOC",
      "Singleplayer"
    ]
  },
  "custom":{
    "my_custom_key": "My important additional value."
  }
}
```

## The `"modinfo"` Type

### The `"name"` Tag [REQUIRED]

This tag specifies the fully qualified mod name, e.g. "Republic at War", "Thrawn's Revenge: Imperial Civil War", "Awakening of the Rebellion" or "Yuuzhan Vong at War"

### The `"summary"` Tag [OPTIONAL]

This tag allows you to include a short summary about the mod supporting Steam-flavoured BBCode.

### The `"icon"` Tag [OPTIONAL]

The relative path to the mod's icon file.

### The `"version"` Tag [OPTIONAL]

The mod's version according to the extended semantic versioning: [Semantic Versioning](https://semver.org/) that also supports a fourth digits for build increments, etc. and suffixes (e.g. `"-rc1"`).

### The `"dependencies"` Tag [OPTIONAL]

The `dependencies` container holds an ordered list of [`"mod_reference type"`](#the-mod_reference-type)s this mod relies on.

The list is either absent from the `modinfo.json` or contains at least one item.

**Changes for v1.2:**
The first mod in the list resembles the mod you actually want to launch. Every `n+1` mod is another mod the current one relies on. 
If a dependency has dependencies itself these must not be added to the list.

**The mod of the this modinfo.json file must not be listed here because that would result in a cycle!**

```
Here is an example what it means when the list contains two mod references:
Imagine this mod is called `A`. 
The dependencies list contains the mods `[C, D, B]`. 

In Java Syntax this is equivalent to: A extends C, D, B

The mod dependency graph looks as follows:

    C
  /
 /
A --- D
 \
  \
    B
```


### The `"steamdata"` Tag [OPTIONAL]

The [`"steamdata type"`](#the-steamdata-type) container holds additional info that is required for the Steam Version of the game.
The container is either absent from the `modinfo.json` or it is fully required.

### The `"custom"` Tag [OPTIONAL]

The `custom` tag allows arbitrary extensions to the format. Programs implementing the core format should always be able to serialize the whole object, but the custom data wrapped within the extension object has to be accounted for only where applicable and/or needed.

---

## The `"mod_reference"` Type

#### The `"mod_reference.modtype"` Tag

The modtype enumeration:

| Value | Meaning |
|:--:|:--|
|`0`|any normal mod inside the Mods/ directory|
|`1`|a Steam Workshops mod|
|`2`|a "virtual" mod. (Currently unsupported)|

*Reasonable: The current mod does NOT contain a `modtype` property because the mod should not know it's owen type. Otherwise sharing this file across steam and disk mods would not be possible. A `mod_reference` requests this data, meaning tool support to determine the actual `modtype` is necessary. This design coice was made because mod linking should always be considered for Steam Workshop mods. For convenient test for mod creators we keep the possibility to reference to local mods. 

Becuase a workshop mod in theroy also is a `default` mod the number values are chooses the way they are. *

**Added for v1.2:** 

`2 : Virtual Mod Types`. A virtual mod does not exists on disk but is only a logical container that holds dependency information. This is currently not supported and only acts as a placeholder in this version of the specification. 

#### The `"mod_reference.identifier"` Tag

This property either contains an absolute or relative path of the parent mod or holds the STEAMID for workshop mods.

---

## The `"steamdata"` Type

### The `"steamdata.publishedfileid"` Tag

The Steam Workshop ID.

### The `"steamdata.contentfolder"` Tag

The content folder's name as specified by the Steam Uploader.

### The `"steamdata.visibility"` Tag

The visibility enumeration:

| Value | Meaning |
|:--:|:--|
|`0`|hidden|
|`1`|friends only|
|`2`|public|


### The `"steamdata.metadata"` Tag

Arbitrary metadata as string.

### The `"steamdata.tags"` Tag

Steam Tags as specified by the Steam Uploader.

At least either `EAW` or `FOC` is required to determine the game the mod shows up for.
