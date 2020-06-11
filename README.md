# eaw.modinfo Definition - v2.0

A standard definition for Star Wars: Empire at War mod info files.

The info files defined herein allow mod makers and tool makers to specify meta information about a given Empire at War mod.

The following sections specify the required and optional content for `eaw.modinfo` in Version 2.0.

## Contents of the Specification:
1. [Allowed File Names](#filename)
2. [File Position](#file-position)
3. [Exemplary Content](#exemplary-content)
4. [`modinfo` Type Specification](#the-modinfo-type)
5. [`modreference` Type Specification](#the-modreference-type)
6. [`steamdata` Type Specification](#the-steamdata-type)
7. [Dependency Resolving](#dependency-resolving)
8. [Dendepency Test Cases](#dependency-resolving-test-cases)

## Filename

The meta information must be saved to a `JSON` file. There are two ways for naming the file:

1. The file is called `modinfo.json`. This is considered as the *main file*.
2. The file is called `[Any_FS_compliant_letter]-modinfo.json`. This is considered as a *variant file.*


Option `2` can be used if you want to create different variants of a mod that share the same files. 

*Imagine if your mod is a submod for not only one but two different mods. This way you only need to develop and upload the mod once but it can target both base mods simultaneously.* 

*Implementation Notes: In practice this will instanciate a new mod for each variant available.*

## File Position

The target directory is the top level of the mod's folder (next to where the mod's `data` folder is).

It may contain none, one or multiple files, as described in [Filename](#filename).

If there exists variant files AND a main file the content from the main file gets merged into the variant file unless the variant overrides a property.  

If there are only variant files they each act as a main files on their own. 

*Implementation Notes: As soon as a mod folder contains modinfo variant files, only these should yield an instance of a mod. The main modinfo file (if existing) or just the directry itself should get ignored.*

## Exemplary Content

```json
{
  "name": "The mod's name",
  "summary": "A short summary about the mod in Steam-flavoured BBCode.\nNice, eh?",
  "icon": "relative/or/absolute/path/to/icon.ico",
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
  "custom": [
    {
      "my_custom_key": "My important additional value."
    }
  ]
}
```

## The `"modinfo"` Type

### The `"name"` Property

**Level:** **REQUIRED**

**Data Type**: `String`  

**Data Semantics**: Unique Text 

**Description:**

This property specifies the fully qualified mod name, e.g. "Republic at War", "Thrawn's Revenge: Imperial Civil War", "Awakening of the Rebellion" or "Yuuzhan Vong at War".

This property **must** also be present for variant modinfos that aim for overriding the main `modinfo.json`

### The `"summary"` Property

**Level:** *OPTIONAL*

**Data Type**: `String`  

**Data Semantics**: Text 

**Description:**

This property allows you to include a short summary about the mod supporting Steam-flavoured BBCode.

### The `"icon"` Property

**Level:** *OPTIONAL*

**Data Type**: `String`  

**Data Semantics**: File System Path 

**Description:**

The path to the mod's icon file **relative** to the mod's root directory or an **absolute** path.

### The `"version"` Property

**Level:** *OPTIONAL*

**Data Type**: `String`  

**Data Semantics**: 4 Digit Semantic Version 

**Description:**

The mod's version according to the extended semantic versioning: [Semantic Versioning](https://semver.org/) that also supports a fourth digits for build increments, etc. and suffixes (e.g. `"-rc1"`).

### The `"dependencies"` Property

**Level:** *OPTIONAL*

**Data Type**: [`modreference`](#the-modreference-type)`[]`

**Data Semantics**: Ordered List of Objects 

**Description:**

The `dependencies` container holds an ordered list of [`"modreference type"`](#the-modreference-type)s this mod relies on.

The list is either absent from the `modinfo.json` or contains at least one item.

The first mod in the list resembles the closest dependeny. Every `n+1` item is another dependeny of the target mod.
If a dependency has dependencies itself these **must not** be added to the list.

**The mod of the this modinfo.json file must not be listed here because that would result in a cycle!**

Here is an example what it means when the list contains three mod references:
Imagine this mod is called `A`. 
The dependencies list contains the mods `[C, D, B]`. 

In Java Syntax this is equivalent to: `A extends C, D, B`. The mod dependency graph looks as follows:

```
  C
 /
A - D
 \
  B
```


### The `"steamdata"` Property

**Level:** *OPTIONAL*

**Data Type**: [`steamdata`](#tthe-steamdata-type)  

**Data Semantics**: Steam Workshops JSON

**Description:**

The [`"steamdata type"`](#the-steamdata-type) container holds additional info that is required for the Steam Version of the game.
The container is either absent from the `modinfo.json` or it is fully required.

### The `"custom"` Property

**Level:** *OPTIONAL*

**Data Type**: `Object[]`

**Data Semantics**: A list of anything

**Description:**


The `custom` property allows arbitrary extensions to the format. Programs implementing the core format should always be able to serialize the whole object, but the custom data wrapped within the extension object has to be accounted for only where applicable and/or needed.

---

## The `"modreference"` Type

#### The `"modreference.modtype"` Property

**Level:** **REQUIRED**

**Data Type**: `integer`

**Data Semantics**: Enum

**Description:**

The modtype enumeration:

| Value | Meaning |
|:--:|:--|
|`0`|any normal mod inside the Mods/ directory|
|`1`|a Steam Workshops mod|
|`2`|a "virtual" mod. (Currently unsupported)|

*Reasonable: The current mod does NOT contain a `modtype` property because the mod should not know it's owen type. Otherwise sharing this file across steam and disk mods would not be possible. A `modreference` requests this data, meaning tool support to determine the actual `modtype` is necessary. This design coice was made because mod linking should always be considered for Steam Workshop mods. For convenient test for mod creators we keep the possibility to reference to local mods.*

*Becuase a workshop mod in theroy also is a `default` mod the number values are chooses the way they are.*

`2 : Virtual Mod Types`. A virtual mod does not exists on disk but is only a logical container that holds dependency information. This is currently not supported and only acts as a placeholder in this version of the specification. 

#### The `"modreference.identifier"` Property

**Level:** **REQUIRED**

**Data Type**: `string`

**Data Semantics**: Unique identifier

**Description:**

This property either contains an absolute or relative path of the parent mod or holds the STEAMID for workshop mods.

---

## The `"steamdata"` Type

### The `"steamdata.publishedfileid"` Property

**Level:** **REQUIRED**

**Data Type**: `string`

**Data Semantics**: Unique identifier

**Description:**

The Steam Workshop ID.

### The `"steamdata.contentfolder"` Property

**Level:** **REQUIRED**

**Data Type**: `String`

**Data Semantics**: File System Path

**Description:**

The content folder's name as specified by the Steam Uploader.

### The `"steamdata.visibility"` Property

**Level:** **REQUIRED**

**Data Type**: `integer`

**Data Semantics**: Enum

**Description:**

The visibility enumeration:

| Value | Meaning |
|:--:|:--|
|`0`|hidden|
|`1`|friends only|
|`2`|public|


### The `"steamdata.metadata"` Property

**Level:** **REQUIRED**

**Data Type**: `String`

**Data Semantics**: Text

**Description:**

Arbitrary metadata as string.

### The `"steamdata.tags"` Property

**Level:** **REQUIRED**

**Data Type**: `String[]`

**Data Semantics**: Collection of tags

**Description:**

Steam Tags as specified by the Steam Uploader.

At least either `EAW` or `FOC` is required to determine the game the mod shows up for.

## Dependency Resolving
The game supports chaining (could also be called *overriding* or *linked*) mods by queuing up the command line arguments `STEAMMOD` and/or `MODPATH`. 
While the Command Line options only can resemble a line (or more precisely a Queue), real mod dependencies can look like complex tree due to multiple inheritance. Thus there needs to be a deterministic logic to convert the real hierachy into a CLI compatible representation. 

To describe relations between mods this specification introduces an optional [`dependency`](#the-dependencies-property) property which is an ordered list. Each item in the list branches from current mod, which fullfils the multiple inheritance logic.

The conversion from a tree structure to a line is called *flattening*.
Implementations of this standard must fullfill the following requirements to create a sucessfull conversion:

1. The `dependencies` list MUST be processed from start to end.
2. Before processing dependiencies of `n+1` (where n is the current processed mod) all dependencies of `n` must be processed first.
3. Duplicates MUST get removed. *Having duplicates that are not direct neighbours is a strong indicator you have a dependency cycle!*
4. The implementation MUST be able to recognice and react on dependency cycles. 
5. The provided [test cases](#dependency-resolving-test-cases) MUST pass:
6. The result of the *flattening* MAY get reversed depending on the requirements of the consumer.

*Note: Converting a (non-binary) tree structure to a flatteded and duplicate-free line is a one-way operation. It also looses accuracy. The requirements above are created to ensure consistency across multiple implementations.*

*Advise: For making sure you mod always work as intended, we recommend to keep the inheritance level to a minimum. Also try to avoid multiple inheritances.*

### Dependency Resolving Test Cases

Node `A` is always the mod that should get executed. Everything behind  `:` means these are the direct dependencies.

```
A : B, C
B : D
C : E

  B-D
 /
A
 \
  C-E

Expected list: (A,) B, C, D, E
```

```
A : C, B
B : D
C : E

  C-D
 /
A
 \
  B-E

Expected list: (A,) C, B, E, D

```

```
A : B, C
B : D
C : D
D : E

  B
 / \
A   D-E
 \ /
  C

Actual List: (A,) B, C, D, D, E, E
Reduced Expected list: (A,) B, C, D, E

```

```
A : B, C, D
B : E
C : E
D : E

  B
 / \
A-C-E
 \ /
  D


Actual List: (A,) B, C, D, E, E, E
Reduced Expected list: (A,) B, C, D, E

```

```
A : B, C
B : E
C : D
E : D

  B-E
 /   \
A     D
 \   /
  C-+

Actual List: (A,) B, C, E, D, D
Reduced Expected list: (A,) B, C, E, D

```

```
 A : A

A--+
|  |
+--+

 Cycle!
```

```
 A : B
 B : A

A--B
|  |
+--+

 Cycle!
```

```
A : B
B : C, D
D : E
E : A

A-B->C
| |
E-D

Cycle!
``` 


