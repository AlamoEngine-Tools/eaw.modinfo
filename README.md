# eaw.modinfo Definition - v2.3

A standard definition for Star Wars: Empire at War mod info files.

The info files defined herein allow mod makers and tool makers to specify meta information about a given Empire at War mod.

The following sections specify the required and optional content for `eaw.modinfo` in Version 2.3

## Contents of the Specification:
- [Changes](#notable-changes)
- [I. Mods](#i-mods)
- [II. Modinfo JSON File](#ii-modinfo-json-file)
- [III. Data Structures](#iii-data-structure-definitions)
- [IV Mod Dependency Handling](#iv-mod-dependency-handling)

## Notable Changes

#### Legend: 
**Bold text indicates breaking changes.**

Breaking changes are any modifications that break backwards compatibility. This usually is the case when:
- renaming properties
- removing properties
- adding required properties
- adding restrictions to a property value
- changing data types
- re-assigning constant values

Not a breaking change are modifications like:
- adding an optional property
- adding an required property where default behavior is not inflicting the older version.

### Version History: 

- *v2.3*
  - The `dependencies` property was augmented to (optionally) support multiple kinds of resolve layouts. 
  - Added optional property `version-range` to `ModReference`
  - Virtual mods are fully specified.
  - overall specification improvements and restructuring.
- *v2.2.1:*
  - Added equality information for modinfo and `ModReference`
- *v2.2:*
  - **Re-added `steamdata.title` as required value**
  - Re-added `steamdata.description` as optional value
  - Re-added `steamdata.previewfile` as optional value
- *v2.1:* 
  - Added support to express language support.
  - **`version` property now only supports 3 digits.**
  - **`steamdata.visibility` values are changed.**
  - `steamdata.metadata` property is optional.

## Goals of this Specification

This specification introduces and defines a data model how to represent modifications, for now on called *Mods*, for the games *Star Wars: Empire at War* and *Star Wars: Empire at War: Forces of Corruption*. This data model shall provide necessary and optional metadata information for 3rd-party tools and applications to work with mods.

This specification is written for Mod Developers, who which to add such metadata to their products.

For End-Users this specification is nearly invisible, though they might highly profit from it, if a mod integrated the metadata information. 

### Contributing and Augmenting this Specification
For anyone who wished to alter the specification there are **3** important requirements each change must comply with:

1. **Error Resistance**: In order for a mod to be playable this specification MUST stay optional. A mod must not demand to have any information described here or a certain tool in oder to work. This also means that any malformed, corrupted or broken data this specification provides MUST NOT lead to a situation where a mod cannot be played.
2. **Usability impact for players**: Any new addition to this specification must provide a clear user story. Only if a Player (or a development team) benefits from additional features it shall get added to the specification. In any other case custom tools and applications shall define their own behavior. Also any changes to the specification MUST NOT alter the expected behavior from a player's point of view.
4. **Mod Developer Acceptance**: Avoid making changes which do require a mod developer to update their modinfo. Even if a change is considered to be a *breaking change* consider to specify a default behavior which stays backward compatible to the previous version of this specification. This way it can be ensured, this specification gets generally accepted and adapted by mod and tool developers.

---

# I Mods

A mod is an addition to the game which can be loaded by explicit user request to alter the game's mechanics, units, visuals, media, etc.  

Traditionally a mod was just a collection of file which represented an installed mod. However due to the addition of the game to the Steam platform, as well as other platforms, and the discovery of *linked mods* this plain file collection representation is not descriptive enough any more. 

The same mod may be installed multiple time on a user machine but on different locations. For mod developers versioning also often becomes relevant. Also mods don't require to have their whole file base bundled into one package, but they can be split into multiple mods. For proper tool support it is necessary a mod can be represented in a structured and sufficient way.

## I.1 Representing Mods

The following image illustrates the relationships of the data structures defined by this specification. As shown, there is a concrete mod installation called *MyMod* with the given attributes. *MyMod* references a *Modinfo File*. This modinfo file provides the information *Name*, *Version* and *Dependencies*. Both, the mod instance and the `ModInfo`, are an instance of the `ModIdentity` data. The mod instance's properties `Type`, `Identifier`, `Location` are defined by the mod itself. A `ModReference` instance may act as a pointer to the concrete instance by sharing the values of the mod's properties `Type`, and `Identifier`.
The next sections will explain these data (`ModIdentity`, `ModInfo` and `ModReference`) in more detail. 
![Mod Data Structure Relationship Diagram](/img/relationship.png)

*Implementation Detail: A concrete Mod instance can also derive from `ModReference`. In this case it would mean the instance points to itself.*

### I.1.1 ModIdentity
The `ModIdentity` data provides *only* the necessary data to uniquely identify a mod. Two mods cannot share the same identity without being considered to be *the same mod*. This information explicitly does not tell anything about the *location* of the mod. This way, the same mod may be installed multiple times (at different locations) on the same machine. 

See [ModIdentity Equality](#iii11-modidentity-and-equality) for detailed information when two ModIdentities are considered to be the same.

Mod developers SHOULD ensure their mods have a globally unique ModIdentity.

*Rationale: Because this specification itself is optional, it shall not force the `ModIdentity` to be globally unique. Meaning that, if there are mods available, which inconveniently share e.g. the same Name, they might end up having the same ModIdentity.*

*Implementation Note: Because of the rationale, an implementation of the specification MUST ensure two different instances of the same ModIdentity can be distinguished. The `ModReference` provides the means to accomplish this.*

### I.1.2 ModInfo

A `ModInfo` data is an instance of `ModIdentity`. Additionally it may provide metadata for a mod, which help tools and applications to handle mod instances or provide additional features to users and mod developers. [See partition III](#iii-data-structure-definitions) which pre-defined properties a ModInfo may have and how they shall behave. 

[Partition II](#ii-modinfo-json-file) describes the concrete JSON syntax of a modinfo and which rules apply when using a ModInfo for mod instances.

A ModInfo is an optional data for a mod instance.

*Implementation Note: Even if no ModInfo is explicitly provided to a mod instance, an implementation of this specification MUST ensure this mod instance still provides ModIdentity data*

### I.1.3 ModReference

A `ModReference` is pointer to an existing mod instance. It only contains the minimal information to describe where the instance can be found. 

*Rationale: This type was introduces because tools can work on the reference while not really processing the mod instance, would could lead to significant performance improvements. Also this way it was possible to represent mod dependencies without bloating the JSON file and keeping effort as low as possible for mod developers. Generally a `ModReference` can be thought of a pointer as known from C/C++.*

## I.2 Physical Mods

Physical mods exists on the file system and provide assets to alter the game. They can be played standalone or can be linked together into a set of virtual mods.

Physical mods can be categorized by either being a Steam Workshop Mod, or not being one. In the latter case the mod is considered to be a *default* or *ordinary* mod.

## I.3 Runtime-Mods

This specification also supports *Runtime Mods* or *Virtual Mods*. The key difference between a virtual mod and ordinary mod (e.g. a Steam Workshops Mod) is that a Virtual mod is not bound to an actual location on the file system. Virtual mods can be *composed* at a tool's runtime. Thus a *Modinfo* object shall also be able to exists only at runtime and must not be exclusively bound to a physical file. 

Virtual Mods require a `name` and respectively an `identifier` for a `ModReference` instance and `dependency` list. A Virtual Mod's dependency list may contain physical mods (such as Steam Workshop Mods) or other Virtual Mods. 
Because Virtual mods themselves do not contribute any physical assets for the game, each virtual mod must at least have one physical mod as a dependency.

The combination of the virtual mod's name and its dependency list represent the ModIdentity data for this instance.

---

# II Modinfo JSON File

This partition describes the syntax of a `ModInfo` file, its file constraints and lookup mechanics.

## II.1 Filename

The meta information must be saved to a `JSON` file. There are two ways for naming the file:

1. The file is called `modinfo.json`. This is considered as the *main file*.
2. The file is called `[Any_FS_compliant_letter]-modinfo.json`. This is considered as a *variant file.*


Option `2` can be used if you want to create different variants of a mod that share the same files. 

*Imagine if your mod is a Submod for not only one but two different mods. This way you only need to develop and upload the mod once but it can target both base mods simultaneously.* 

*Implementation Notes: In practice this will instantiate a new mod for each variant available.*

## II.2 File Position

The target directory is the top level of the mod's folder (next to where the mod's `data` folder is).

It may contain none, one or multiple files, as described in [Filename](#filename).

If there exists at least one variant file AND a main file the content from the main file gets merged into the variant file(-s) unless the variant overrides a property.  

If there are only variant files they each act as a main files on their own. 

*Implementation Notes: As soon as a mod folder contains modinfo variant files, only these should yield an instance of a mod. The main modinfo file (if existing) or just the directory itself should be ignored.*

## II.3 Exemplary Content

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
      "support": 1
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
      "key-2" : { }
    }
  ]
}
```

---

# III Data Structure Definitions

This partition defines the previously mentioned data types `ModIdentity`, `ModReference` and `ModIdentity` as well as their properties and other types which are introduced by this this specification.

## III.1 The `"modidentity"` Type

The `modidentity` type contains data which define a mod or enrich the mod's metadata information. The following sections describe the properties which can be expressed by this specification.

Therefore three properties are defined:
- **`name`**
- **`version`**
- **`dependencies`**

### III.1.1 ModIdentity and Equality

A mod identity can be used to fully qualify a mod definition. To compare the identity of two mods the three previously mentioned properties can be used.
The specification defines two common strategies how to use these properties for identity checking:


An Implementation of this specification must provide an identity checking where:
- Only `name` is considered. Comparison shall be case-sensitive.
- `name` AND `version` AND `dependencies` are considered.
  - The `name` comparison shall be case-sensitive.
  - The `version` comparison shall return "equals" when both version properties are not present OR both properties are present and their value is equal
  - The `dependencies` comparison shall return "equals" when both dependency lists ([See ModReference equality](#ModReference-Equality)):
    - have the same `resolve-layout` property,
    - have the exact same number of elements,
    - all elements match in value and position.

The latter strategy is considered to be de *default* identity checking strategy.

*Implementation Notes: It's up to an implementation of this specification to add more possible strategies. Runtime equality for different kinds of dependency lists could also be implemented.*

### III.1.2 Properties

#### The `"name"` Property

**Level:** **REQUIRED**

**Data Type**: `String`  

**Data Semantics**: Unique Text 

**Description:**

This property specifies the fully qualified mod name, e.g. "Republic at War", "Thrawn's Revenge: Imperial Civil War", "Awakening of the Rebellion" or "Yuuzhan Vong at War".

This property **must** also be present for variant modinfos that aim for overriding the main `modinfo.json`

#### The `"version"` Property

**Level:** *OPTIONAL*

**Data Type**: `String`  

**Data Semantics**: 3 Digit Semantic Version 

**Description:**

The mod's version according to the extended semantic versioning: [Semantic Versioning](https://semver.org/).

*Examples: `"1.0.0"`, `"1.0.0-rc1"`, `"1.2.3-ALPHA-1"`*

*Implementation Note: Implementations of this specification should consider handling malformed versions (e.g. four-digit or single-digit) to avoid potential parsing crashes. The behavior of this handling however shall be undefined by this specification.*

#### The `"dependencies"` Property

**Level:** *OPTIONAL*

**Data Type**: [`dependencyList`](#)

**Data Semantics**: Ordered List of Objects 

**Description:**

The `dependencyList` holds an ordered sequence of [`"modreference type"`](#the-modreference-type)s this mod relies on.

The list is either absent from the `modinfo.json` or contains at least one item.

The dependency list is strictly left-right ordered, where the first entry resembles the most closest ancestor and the *n*'th entry the least closet ancestor. 

The target mod itself must **not** be listed. Doing so would result in a dependency cycle!

This specification supports multiple *resolve layouts*. A resolve layout essentially is an enumeration `resolve-layout` which describes how the dependency collection shall get interpreted and processed by an implementation.

The JSON allows to specify the desired resolve layout by adding its name as a `string` value as the first element of the list. Example:

```json
{
  "name": "MyMod"
  "dependencies": 
  [
    "FullResolved", 
    {
	  "modtype": 0,
	  "identifier": "./Mods/BaseMod"		
    }	
  ]
}
```

If no resolve layout was specified in the JSON, the value `ResolveRecursive` shall be used as default.

An explanation of the supported layouts is shown the table below:

| Resolve Layout | Meaning |
|:--:|:--|
|`ResolveRecursive`|The list shall only contain the mods direct ancestors. Each entry may have their own dependencies which shall get resolved recursively.|
|`ResolveLastItem`|The list shall only contain the mods direct ancestors. Only the last item in the list shall be recursively resolved. There shall be no resolving for any of the previous entries. If the list only contains one element this shall also get recursively resolved.|
|`FullResolved`|The list shall contain all ancestors of the target mod. The entries in the list shall be interpreted *as is*. No dependency resolving shall be done by tool.|

[Partition IV](#iv-mod-dependency-handling) explains in detail the requirements how dependency resolving shall be implemented.


---

## III.2 The `"modreference"` Type

### III.2.1 ModReference vs. ModIdentity

A `ModReference` is distinguished by other properties than a `ModIdentity` in order to use it for dependency resolving. 

*Example: A mod `A` may have multiple mod references e.g. one is located in Steam-Workshops and the other in `FoC/Mods/A/`. Both references point to the same `ModIdentity` (Name: A, Version: null, Dependencies:Empty) however the references itself are not equal.*

### III.2.2 ModReference Equality

Two ModReferences equal when both properties `identifier` and `modtype` match. The `identifier` property is case-sensitive.
This strategy has to get applied when resolving dependencies.

*Implementation Notes: It's up to an implementation of this specification to add more possible strategies. If the `identifier` contains a local path, it's up to the implementation to normalize the path if necessary.*

### III.2.3 Properties

#### The `"modtype"` Property

**Level:** **REQUIRED**

**Data Type**: `integer`

**Data Semantics**: Enum

**Description:**

The modtype enumeration:

| Value | Meaning |
|:--:|:--|
|`0`|any normal mod inside the Mods/ directory|
|`1`|a Steam Workshops mod|
|`2`|a "virtual" mod.|

*Rationale: The current mod does NOT contain a `modtype` property because the mod should not have to know it's own type. Otherwise sharing this file across steam and disk mods would not be possible. A `modreference` requests this data, meaning tool support to determine the actual `modtype` is necessary. This design choice was made because mod linking should always be considered for Steam Workshop mods. The possibility to reference local mods is a convenience functionality intended to be used by mod developers for test setups and development.*

*Note: Since virtual mods have an unstable/unpredictable identifier, they should not be used in an modinfo.json to avoid tool specific errors for the user. However this specification shall not forbid it.*


#### The `"identifier"` Property

**Level:** **REQUIRED**

**Data Type**: `string`

**Data Semantics**: Unique identifier

**Description:**

This property either contains an absolute or relative path of the parent mod or holds the STEAM_ID for workshop mods.

#### The `"version-range"` Property

**Level:** *Optional*

**Data Type**: `string`

**Data Semantics**: Version Range

**Description:**

This property shall only get parsed and otherwise ignored completely by an implementation of this specification. It explicitly is not used for `modreference` equality matching and dependency resolution. It shall only be used to custom tools.

This specification shares the same syntax and semantics as used for [npm node dependency ranges](https://github.com/npm/node-semver#ranges). If the property is unset, the version range `*` (which means `>= 0.0.0`) shall be used.

*Rationale: This property can be used for entries in a mod's mod dependency list. Custom tools might want to consume the given version range and perform a custom mod matching.*

---

## III.3 The `"modinfo"` Type

### III.3.1 Properties

A `modinfo` is `modreference`, thus they share the same properties, with the same meaning.

#### The `"summary"` Property

**Level:** *OPTIONAL*

**Data Type**: `String`  

**Data Semantics**: Text 

**Description:**

This property allows you to include a short summary about the mod supporting Steam-flavoured BBCode.

#### The `"icon"` Property

**Level:** *OPTIONAL*

**Data Type**: `String`  

**Data Semantics**: File System Path 

**Description:**

The path to the mod's icon file **relative** to the mod's root directory or an **absolute** path.


#### The `"languages"` Property

**Level:** *OPTIONAL*

**Data Type**:  [`languageInfo`](#the-languageinfo-type)`[]`

**Data Semantics**: Collection of supported languages

**Description:**

This property holds a collection of [`language`](#the-language-type) objects. Each item indicates a language that is supported by the mod. 

The property is optional. When *NOT* present, the language **English** (`"en"`) is assumed to be default. However if the property is defined English *MUST* be included when supported by the mod, too.

*NOTE: If you are dealing with a situation where you have a variant modinfo file that shall get merged with a main modinfo file, the variant MUST explicitly set this property as well.* 

#### The `"steamdata"` Property

**Level:** *OPTIONAL*

**Data Type**: [`steamdata`](#the-steamdata-type)  

**Data Semantics**: Steam Workshops JSON

**Description:**

The [`"steamdata type"`](#the-steamdata-type) container holds additional info that is required for the Steam Version of the game.

#### The `"custom"` Property

**Level:** *OPTIONAL*

**Data Type**: `Dictionay<string, object>`

**Data Semantics**: Collection of custom objects, stored by keys

**Description:**

The `custom` property allows arbitrary extensions to the format. Programs implementing the core format should always be able to serialize the whole object, but the custom data wrapped within the extension object has to be accounted for only where applicable and/or needed.

---

## III.4 The `"languageInfo"` Type

### III.4.1 Properties

#### The `"code"` Property

**Level:** **REQUIRED**

**Data Type**: `string`

**Data Semantics**: Language Code

**Description:**

This property holds an [ISO 639-1](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) two letter language code.

#### The `"support"` Property

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

## III.5 The `"steamdata"` Type

### III.5.1 Properties

#### The `"publishedfileid"` Property

**Level:** **REQUIRED**

**Data Type**: `string`

**Data Semantics**: Unique identifier

**Description:**

The Steam Workshop ID.

#### The `"contentfolder"` Property

**Level:** **REQUIRED**

**Data Type**: `String`

**Data Semantics**: File System Path

**Description:**

The content folder's name as specified by the Steam Uploader.

#### The `"visibility"` Property

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

#### The `"title"` Property

**Level:** **REQUIRED**

**Data Type**: `string`

**Data Semantics**: Soft identifier

**Description:**

The display name of the mod in Steam Workshops.


#### The `"metadata"` Property

**Level:** *OPTIONAL*

**Data Type**: `String`

**Data Semantics**: Text

**Description:**

Arbitrary metadata as string.

*Implementation Notes: Even if this value is not present, tools should print this property with empty value `""` in the file. This behavior is recommended to increase compatibility with the Steam Workshops uploader.*

#### The `"tags"` Property

**Level:** **REQUIRED**

**Data Type**: `String[]`

**Data Semantics**: Collection of tags

**Description:**

Steam Tags as specified by the Steam Uploader.

At least either `EAW` or `FOC` is required to determine the game the mod shows up for.


#### The `"description"` Property

**Level:** *OPTIONAL*

**Data Type**: `String`

**Data Semantics**: Steam flavoured BB-Code description 

**Description:**

Optional description of the Mod in Steam flavoured BB-Code.

*Implementation Notes: Even if this value is not present, tools should print this property with empty value `""` in the file. This behavior is recommended to increase compatibility with the Steam Workshops uploader.*


#### The `"previewfile"` Property

**Level:** *OPTIONAL*

**Data Type**: `String`

**Data Semantics**: Relative Path

**Description:**

Relative path to an image file which holds the preview image

*Implementation Notes: Even if this value is not present, tools should print this property with empty value `""` in the file. This behavior is recommended to increase compatibility with the Steam Workshops uploader.*

---


# IV Mod Dependency Handling
The game supports chaining (could also be called *overriding* or *linked*) mods by queuing up the command line arguments `STEAMMOD` and/or `MODPATH`. 
While the Command Line options only can resemble a line (or more precisely a Queue), real mod dependencies can look like complex tree due to multiple inheritance. Thus there needs to be a deterministic logic to convert the real hierarchy into a CLI compatible representation. 

To describe relations between mods this specification introduces an optional [`dependencies`](#the-dependencies-property) property which is an ordered list.

The conversion from a tree structure to a line is called *flattening* or *traversing*. Since the `dependencies` list supports multiple resolve layouts there are different strategies defined how the conversion shall behave.

## IV.1 Dependency Resolving Algorithm

Each resolve layout implementations fullfil the following general requirements, in addition to its own requirements, to create a successful conversion:

1. The `dependencies` list gets processed from start to end.
2. The resulting list contains the target mod. It shall be the first element of the list.
2. The resulting list must not have duplicates.
3. The implementation must be able to recognize and respond to identified dependency cycles accordingly.


*Note: Converting a (non-binary) tree structure to a flatteded and duplicate-free line is a one-way operation. It also looses accuracy. However, since in some cases multiple outcomes are possible, the resolve layout `ResolveRecursive` might lead to unexpected results. The requirements above are created to ensure consistency across multiple implementations.*

*Advise: For making sure your mod always works as intended, it's recommended to keep the inheritance level to a minimum. multiple inheritances should be avoided.*


## IV.1.1 Resolving `ResolveRecursive`

Flattening shall be performed based on a Breadth-first search. This way the left-right order of the dependency list is kept.

For each entry to be resolved the algorithm has to honor the resolve layout of the entries dependency list. The means the algorithm shall not resolve everything recursively but only where the resolve layout wants it.

Because of multiple inheritance, and particularly [diamond inheritance](https://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem) it is possible an entry occurs multiple times during resolution. In this case it is not guaranteed a cycle is present. Duplicates, which are not a cycle must however be removed from the resulting list.

The algorithm must ensure the resulting list is [topological sorted](https://en.wikipedia.org/wiki/Topological_sorting) while still preserving a left-to-right order of direct ancestors.

To guide an implementation, this specification defines the following [test cases](#iv2-full-recursive-dependency-resolving-test-cases) a fully recursive resolving algorithm must pass.

*Implementation Note: Cycles detection works best on the directed dependency graph, rather than on a traversed list.*


## IV.1.2 Resolving `ResolveLastItem`
Only the last (or the only element) in the list shall be resolved as specified in [Resolving ResolveRecursive](#iv11-resolving-resolverecursive).

As soon as there exists a duplicate, a dependency cycle is present.

## IV.1.3 Resolving `FullResolved`
Since this layout indicates the dependency list shall be interpreted *as is*, the flattening algorithm shall return the list unmodified. 

If the list contains a duplicate, a dependency cycle is present.


## IV.2 Full Recursive Dependency Resolving Test Cases

Node `A` is always the mod that should be loaded. Every mod following after  `:` are direct dependencies of the mod, if multiple they are separated by a `,`.

Each dependency list has the option `ResolveRecursive` specified.

**Test-Case A:**
```
A : B, C
B : D
C : E

  B-D
 /
A
 \
  C-E

Expected list: A, B, C, D, E
```

**Test-Case B:**
```
A : C, B
B : D
C : E

  C-D
 /
A
 \
  B-E

Expected list: A, C, B, E, D

```

**Test-Case C:**
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

Expected list: A, B, C, D, E

```

**Test-Case D:**
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


Expected list: (A,) B, C, D, E

```

**Test-Case E:**
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

Expected list: A, B, C, E, D

```

**Test-Case F:**
```
A : B, C
B : D
C : E
E : D

  B-+
 /   \
A     D
 \   /
  C-E

Expected list: A, B, C, E, D

```

**Test-Case G:**
```
A : B, C, D, E, F, G

   +--B
   |--C
   |--D
 A-+
   |--E
   |--F
   +--G

Expected list: A, B, C, D, E, F, G

```

**Test-Case H:**
```
A : B, C
B : D
C : G
D : E, F
G : I
F : I

       E
      /
  B--D--F
 /        \
A          I
 \        /
  C--G---+

Expected list: A, B, C, D, G, E, F, I

```

**Test-Case I:**
```
A : C, B
C : D
B : E
E : X
X : D, F
D : F

  C-----D
 /      ^ \
A       |   F  
 \      | /
  B--E--X

Expected list: A, C, B, E, X, D, F

```

**Test-Case J:**
```
A : B, C, D
B : X
C : X, F
D : E
E : X, F

  B----X
 /    /| 
A----C-+---F
 \     | /
  D----E

Expected list: A, B, C, D, E, X, F

```

**Test-Case K (Cycle):**
```
 A : A

A--+
|  |
+--+

 Cycle!
```

**Test-Case L (Cycle):**
```
 A : B
 B : A

A--B
|  |
+--+

 Cycle!
```

**Test-Case M (Cycle):**
```
A : B
B : C, D
D : E
E : A

A-->B-->C
^   |
|   v
E<--D

Cycle!
``` 
