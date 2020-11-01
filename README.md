# eaw.modinfo Definition - v2.2

A standard definition for Star Wars: Empire at War mod info files.

The info files defined herein allow mod makers and tool makers to specify meta information about a given Empire at War mod.

The following sections specify the required and optional content for `eaw.modinfo` in Version 2.2.

## Contents of the Specification:
1. [Changes](#notable-changes)
2. [Allowed File Names](#filename)
3. [File Position](#file-position)
4. [Exemplary Content](#exemplary-content)
5. [`modinfo` Type Specification](#the-modinfo-type)
6. [`modreference` Type Specification](#the-modreference-type)
7. [`language` Type Specification](#the-language-type)
8. [`steamdata` Type Specification](#the-steamdata-type)
9. [Dependency Resolving](#dependency-resolving)
10. [Dendepency Test Cases](#dependency-resolving-test-cases)

## Notable Changes

### Leged: 
**Bold text indicates breaking changes.**

Breaking changes are any modifications that break backwards compatibility. This usualy is the case when:
- renaming properties
- removing properties
- adding required properties
- adding restrictions to a property value
- changing data types
- re-assigning constant values

Not a breaking change are modifications like:
- adding an optional property

### Version History: 

- *v2.1:* 
  - Added support to express language support.
  - **`version` property now only supports 3 digits.**
  - **`steamdata.visibility` values are changed.**
  - `steamdata.metadata` property is optional.
- *v2.2:*
  - **Re-added `steamdata.title` as required value**
  - Re-added `steamdata.description` as optional value
  - Re-added `steamdata.previewfile` as optional value

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

If there exists at least one variant file AND a main file the content from the main file gets merged into the variant file(-s) unless the variant overrides a property.  

If there are only variant files they each act as a main files on their own. 

*Implementation Notes: As soon as a mod folder contains modinfo variant files, only these should yield an instance of a mod. The main modinfo file (if existing) or just the directry itself should be ignored.*

## Exemplary Content

```json
{
  "name": "The mod's name",
  "summary": "A short summary about the mod in Steam-flavoured BBCode.\nNice, eh?",
  "icon": "relative/or/absolute/path/to/icon.ico",
  "version": "1.0.0",
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
  "languages": [
    {
      "code": "en"
    },
    {
      "code": "de",
      "support": 0
    },
    {
      "code": "es",
      "support": 1
    }
  ],
  "steamdata": {
    "publishedfileid": "xxxxxxxxxx",
    "contentfolder": "folder",
    "visibility": 1,
    "title": "Your Mod Name",
    "metadata": "",
    "tags":[
      "Multiplayer",
      "Land",
      "Space",
      "FOC",
      "Singleplayer"
    ],
    "previewfile": "my\\path\\splash.png",
    "description": "Some Description Test"
  },
  "custom": [
    {
      "key-1": "someData",
      "key-2" : { } // Some Object
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

**Data Semantics**: 3 Digit Semantic Version 

**Description:**

The mod's version according to the extended semantic versioning: [Semantic Versioning](https://semver.org/).

*Examples: `"1.0.0"`, `"1.0.0-rc1"`, `"1.2.3-ALPHA-1"`*

### The `"dependencies"` Property

**Level:** *OPTIONAL*

**Data Type**: [`modreference`](#the-modreference-type)`[]`

**Data Semantics**: Ordered List of Objects 

**Description:**

The `dependencies` container holds an ordered list of [`"modreference type"`](#the-modreference-type)s this mod relies on.

The list is either absent from the `modinfo.json` or contains at least one item.

The first mod in the list resembles the closest dependeny. Every `n+1` item is another dependeny of the target mod.
If a dependency has dependencies itself these **must not** be added to the list.

**The mod described by the current `modinfo.json` file must not be listed here, including it would result in a cycle!**

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
### The `"languages"` Property

**Level:** *OPTIONAL*

**Data Type**:  [`language`](#the-language-type)`[]`

**Data Semantics**: Collection of supported languages

**Description:**

This property holds a collection of [`language`](#the-language-type) objects. Each item indicates a language that is supported by the mod. 

The property is optional. When *NOT* present, the language **English** (`"en"`) is assumed to be default. However if the property is defined English *MUST* be inclued when supported by the mod, too.

*NOTE: If you are dealing with a situation where you have a variant modinfo file that shall get merged with a main modinfo file, the variant MUST explicitly set this property as well.* 

### The `"steamdata"` Property

**Level:** *OPTIONAL*

**Data Type**: [`steamdata`](#the-steamdata-type)  

**Data Semantics**: Steam Workshops JSON

**Description:**

The [`"steamdata type"`](#the-steamdata-type) container holds additional info that is required for the Steam Version of the game.

### The `"custom"` Property

**Level:** *OPTIONAL*

**Data Type**: `Dictionay<string, object>`

**Data Semantics**: Collection of custom objects, stored by keys

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

*Rationale: The current mod does NOT contain a `modtype` property because the mod should not have to know it's own type. Otherwise sharing this file across steam and disk mods would not be possible. A `modreference` requests this data, meaning tool support to determine the actual `modtype` is necessary. This design coice was made because mod linking should always be considered for Steam Workshop mods. The possibility to reference local mods is a convenience functionality intended to be used by mod developers for test setups and development.*

~~*Because a workshop mod in theroy also is the `default` mod type the numeric enumeration values are choosen the way they are.*~~

`2 : Virtual Mod Types`. A virtual mod does not exists on disk but is only a logical container that holds dependency information. This is currently not supported and only acts as a placeholder in this version of the specification. 

#### The `"modreference.identifier"` Property

**Level:** **REQUIRED**

**Data Type**: `string`

**Data Semantics**: Unique identifier

**Description:**

This property either contains an absolute or relative path of the parent mod or holds the STEAMID for workshop mods.

---

## The `"language"` Type

#### The `"language.code"` Property

**Level:** **REQUIRED**

**Data Type**: `string`

**Data Semantics**: Language Code

**Description:**

This property holds an [ISO 639-1](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) two letter language code.

#### The `"language.support"` Property

**Level:** *OPTIONAL*

**Data Type**: Integer

**Data Semantics**: Level of language support

**Description:**

The language support enumeration acts as a bit flag and is defined as follows:

| Value | Meaning |
|:--:|:--|
|`1`| **Text:** A `mastertextfile_xxx.dat` is available in this language.|
|`2`|**Speech**: Speech event files are in their own language folder. (Important for Movies, Missions and Holograms)|
|`4`|**SFX** Sound effects, such as unit actions, are localized. |
|`7`|**Fully localized:** Combines `1`, `2`, `4`|

When the property was omitted for a `language` object, value `7` (fully localized) is applied.

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

The visibility enumeration [(based on Steam API)](https://partner.steamgames.com/doc/api/ISteamRemoteStorage#ERemoteStoragePublishedFileVisibility):

| Value | Meaning |
|:--:|:--|
|`0`|public|
|`1`|friends only|
|`2`|private|
|`3`|unlisted|

*Note: Value `3` (unlisted) currently is not documented by Valve and should thus not get used for now.*

### The `"steamdata.title"` Property

**Level:** **REQUIRED**

**Data Type**: `string`

**Data Semantics**: Soft identifier

**Description:**

The display name of the mod in Steam Workshops.


### The `"steamdata.metadata"` Property

**Level:** *OPTIONAL*

**Data Type**: `String`

**Data Semantics**: Text

**Description:**

Arbitrary metadata as string.

*Implementation Notes: Even if this value is not present, tools should print this property with empty value `""` in the file. This behaviour is recommended to increase compatiblity with the Steam Worshops uploader.*

### The `"steamdata.tags"` Property

**Level:** **REQUIRED**

**Data Type**: `String[]`

**Data Semantics**: Collection of tags

**Description:**

Steam Tags as specified by the Steam Uploader.

At least either `EAW` or `FOC` is required to determine the game the mod shows up for.


### The `"steamdata.description"` Property

**Level:** *OPTIONAL*

**Data Type**: `String`

**Data Semantics**: Steam flavoured BB-Code description 

**Description:**

Optional description of the Mod in Steam flavoured BB-Code.

*Implementation Notes: Even if this value is not present, tools should print this property with empty value `""` in the file. This behaviour is recommended to increase compatiblity with the Steam Worshops uploader.*


### The `"steamdata.previewfile"` Property

**Level:** *OPTIONAL*

**Data Type**: `String`

**Data Semantics**: Relative Path

**Description:**

Relative path to an image file which holds the preview image

*Implementation Notes: Even if this value is not present, tools should print this property with empty value `""` in the file. This behaviour is recommended to increase compatiblity with the Steam Worshops uploader.*


## Dependency Resolving
The game supports chaining (could also be called *overriding* or *linked*) mods by queuing up the command line arguments `STEAMMOD` and/or `MODPATH`. 
While the Command Line options only can resemble a line (or more precisely a Queue), real mod dependencies can look like complex tree due to multiple inheritance. Thus there needs to be a deterministic logic to convert the real hierachy into a CLI compatible representation. 

To describe relations between mods this specification introduces an optional [`dependency`](#the-dependencies-property) property which is an ordered list. Each item in the list branches from current mod, which fullfils the multiple inheritance logic.

The conversion from a tree structure to a line is called *flattening*.
Implementations of this standard must fullfill the following requirements to create a sucessfull conversion:

1. The `dependencies` list MUST be processed from start to end.
2. Before processing dependiencies of `n+1` (where n is the current processed mod) all dependencies of `n` must be processed first.
3. Duplicates MUST be removed. *Having duplicates that are not direct neighbours is a strong indicator for a dependency cycle!*
4. The implementation MUST be able to recognise and respond to identified dependency cycles accordingly. 
5. The provided [test cases](#dependency-resolving-test-cases) MUST pass.
6. The result of the *flattening* MAY get reversed depending on the requirements of the consumer.

*Note: Converting a (non-binary) tree structure to a flatteded and duplicate-free line is a one-way operation. It also looses accuracy. The requirements above are created to ensure consistency across multiple implementations.*

*Advise: For making sure your mod always works as intended, we recommend to keep the inheritance level to a minimum. Also try to avoid multiple inheritances.*

### Dependency Resolving Test Cases

Node `A` is always the mod that should be loaded. Every mod following after  `:` are direct dependencies of the mod, if multiple they are separated by a comma.

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
