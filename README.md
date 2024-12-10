# eaw.modinfo Definition - v4.0.0
A standard format for Star Wars: Empire at War mod information files.

These files enable mod creators and tool developers to specify metadata about a given Empire at War mod.

The sections below detail the required and optional content for eaw.modinfo in Version 4.0.0.

## Contents of the Specification:
[Changes](#notable-changes)

**Partition I**
- [I. Mods](#i-mods)
  - [I.1 Representing Mods](#i1-representing-mods)
    - [I.1.1 ModIdentity](#i11-modidentity)
    - [I.1.2 ModInfo](#i12-modinfo)
    - [I.1.3 ModReference](#i13-modreference)
  - [I.2 Physical Mods](#i2-physical-mods)
  - [I.3 Runtime-Mods](#i3-runtime-mods)
  - [I.4 Instanciating Mods](#i4-instaciating-mods)
  
**Partition II**
- [II. Modinfo JSON File](#ii-modinfo-json-file)
  - [II.1 File Syntax](#ii1-file-syntax)
  - [II.2 Filename](#ii2-filename)
  - [II.3 File Position](#ii3-file-position)
  - [II.4 Exemplary Content](#ii4-exemplary-content)

**Partition III**
- [III. Data Structures](#iii-data-structure-definitions)
  - [III.1 The "modidentity" Type](#iii1-the-modidentity-type)
    - [III.1.1 ModIdentity and Equality](#iii11-modidentity-and-equality)
    - [III.1.2 Properties](#iii12-properties)
  - [III.2 The "modreference" Type](#iii2-the-modreference-type)
    - [III.2.1 ModReference vs. ModIdentity](#iii21-modreference-vs-modidentity)
    - [III.2.2 ModReference and Equality](#iii22-modreference-equality)
    - [III.2.3 Properties](#iii23-properties)
    - [III.2.4 Creating Identifiers](#iii24-creating-identifiers)
  - [III.3 The "modinfo" Type](#iii3-the-modinfo-type)
    - [III.3.1 Properties](#iii31-properties) 
    - [III.3.1 Merge Behavior](#iii32-merge-behavior)
  - [III.4 The "languageInfo" Type](#iii4-the-languageinfo-type)
    - [III.4.1 Properties](#iii41-properties)
  - [III.5 The "steamdata" Type](#iii5-the-steamdata-type)
    - [III.5.1 Properties](#iii51-properties)

**Partition IV**
- [IV Mod Dependency Handling](#iv-mod-dependency-handling)
  - [IV.1 Dependency Resolving Algorithm](#iv1-dependency-resolving-algorithm)
    - [IV.1.1 Resolving ResolveRecursive](#iv11-resolving-resolverecursive)
    - [IV.1.2 Resolving ResolveLastItem](#iv12-resolving-resolvelastitem)
    - [IV.1.3 Resolving FullResolved](#iv13-resolving-fullresolved)
  - [IV.2 Full Recursive Dependency Resolving Test Cases](#iv2-full-recursive-dependency-resolving-test-cases)

## Notable Changes

#### Legend: 
**Bold text indicates breaking changes.**

Breaking changes are any modifications that break backward compatibility. This usually occurs when:
- Renaming properties
- Removing properties
- Adding required properties
- Adding restrictions to a property value
- Changing data types
- Re-assigning constant values

Not a breaking change are modifications like:
- Adding an optional property
- Adding a required property when default behavior is backward-compatible

### Version History: 

*v4.0.0*
  - **`modreference.identifier` again specifies its concrete content 
but now supports the scenario to uniquely identify two mods at the same location.**
  - **Specified how to instanciate mods in the presence of this specification**
  - `modreference.identifer` and `modidentity.name` is not case-insensitive for equality matching

*v3.0.2*
  - `modreference.identifier` does not specify its data concrete content anymore.
  - `steamdata.publishedfileid` shall be able to parse into a `ulong`.

*v3.0.1*
  - Specified that the optional lists `modidentity.dependencies`, `modinfo.languages` and `modinfo.custom` are not `nullable`.

*v3.0.0*
  - **Changed ID domain**
  - **Added versioning to JSON schema ID**
  - **Fixes schema definition for `dependencies`, `custom` and `languages`**
  - Added explicit merge behavior for `modinfo` data type.
  - `modinfo.languages` is now a set
  - `modinfo.name` cannot be null or an empty string.
  - Added equality information for `languageInfo`
  - Split schemas into multiple files for easier reuse
  - `modreference.identifier` must not be null or empty
  - `steamdata.tags` entries have a new specification and rules
  - Changed intended behavior in `II.3` for mod instanciation from multiple variant files together with a main modinfo. 

*v2.3.2*
  - Allow trailling commas and JSON comments.

*v2.3.1*
  - Added a requirement on virtual mods for the dependency resolve algorithm.

*v2.3*
  - The `dependencies` property was augmented to (optionally) support multiple kinds of resolve layouts. 
  - Added optional property `version-range` to `ModReference`
  - Virtual mods are fully specified.
  - overall specification improvements and restructuring.

*v2.2.1:*
  - Added equality information for modinfo and `ModReference`

*v2.2:*
  - **Re-added `steamdata.title` as required value**
  - Re-added `steamdata.description` as optional value
  - Re-added `steamdata.previewfile` as optional value

*v2.1:* 
  - Added support to express language support.
  - **`version` property now only supports 3 digits.**
  - **`steamdata.visibility` values are changed.**
  - `steamdata.metadata` property is optional.

## Goals of this Specification

This specification introduces and defines a data model to represent modifications, hereafter referred to as Mods, for the games Star Wars: Empire at War and Star Wars: Empire at War: Forces of Corruption. This data model provides necessary and optional metadata information that third-party tools and applications can use to work with mods.

This specification is intended for Mod Developers who wish to add such metadata to their products.

For End-Users, this specification is nearly invisible, although they may benefit greatly if a mod includes this metadata.

### Contributing and Augmenting this Specification
For anyone wishing to alter the specification, there are 3 important requirements each change must meet:

1. **Error Resilience**: In order for a mod to remain playable, this specification MUST stay optional. A mod must not require any of the information described here or any specific tool to work. Additionally, any malformed, corrupted, or broken data provided by this specification MUST NOT lead to a situation where a mod cannot be played.
2. **layer Usability Impact**: Any addition to this specification must provide a clear benefit to the player experience. Only if players (or a development team) benefit from added features should it be incorporated. In any other case, custom tools and applications should define their own behavior. Moreover, any changes to the specification MUST NOT alter the expected behavior from the player’s perspective.
4. **Mod Developer Acceptance**: Avoid changes that require a mod developer to update their modinfo. Even if a change is considered a breaking change, consider defining a default behavior that remains backward-compatible with the previous version of this specification. This approach ensures broad acceptance and adoption by mod and tool developers.
---

# I Mods

A mod is an addition to the game that a user can load to modify the game’s mechanics, units, visuals, media, etc.

Traditionally, a mod was simply a collection of files representing an installed mod. However, with the addition of the game to platforms such as Steam, along with the introduction of linked mods, this plain file collection representation is no longer sufficient.

The same mod may be installed multiple times on a user’s machine in different locations. Versioning has also become relevant for mod developers. Furthermore, mods do not need to bundle all files into one package; they can be split across multiple mods. For proper tool support, a mod must be represented in a structured and comprehensive way.

## I.1 Representing Mods

The following diagram illustrates the relationships between the data structures defined in this specification. As shown, there is a specific mod installation called *MyMod* with the listed attributes. *MyMod* references a *Modinfo File*, which provides information on *Name*, *Version* and *Dependencies*. Both the mod instance and the `ModInfo`, are an instance of the `ModIdentity` data. The mod instance's properties `Type`, `Identifier`, `Location` are defined by the mod itself. A `ModReference` instance may act as a pointer to the concrete instance by sharing the values of the mod's properties `Type`, and `Identifier`.
The next sections will explain these data (`ModIdentity`, `ModInfo` and `ModReference`) in more detail. 
![Mod Data Structure Relationship Diagram](/img/relationship.png)

*Implementation Detail: In cases where a concrete mod instance derives from `ModReference`, the instance essentially self-references.*

### I.1.1 ModIdentity
The `ModIdentity` data provides *only* the necessary data to uniquely identify a mod.  Two mods cannot share the same identity without being considered the same mod. This information explicitly does not indicate the location of the mod, so the same mod may be installed multiple times (in different locations) on the same machine.

See [ModIdentity Equality](#iii11-modidentity-and-equality) for detailed information when two ModIdentities are considered to be the same.

Mod developers SHOULD ensure their mods have a globally unique ModIdentity.

> *Rationale: Since this specification itself is optional, it does not mandate `ModIdentity` to be globally unique. Therefore, if mods inadvertently share the same Name, they may end up with the same ModIdentity.*

> *Implementation Note: Due to the above rationale, an implementation of this specification MUST distinguish between two different instances with the same ModIdentity. `ModReference` provides the means to achieve this.*

### I.1.2 ModInfo

A `ModInfo` is an instance of `ModIdentity` and may provide additional metadata to help tools and applications handle mod instances or provide extra features for users and mod developers. [See partition III](#iii-data-structure-definitions) for predefined properties of `ModInfo` and their expected behavior.

[Partition II](#ii-modinfo-json-file) describes the concrete JSON syntax of a modinfo and which rules apply when using a ModInfo for mod instances.

Including a ModInfo is optional for a mod instance.

> *Implementation Note: Even if no ModInfo is explicitly provided to a mod instance, implementation of this specification MUST ensure that this mod instance still provides `ModIdentity` data.*

### I.1.3 ModReference

`ModReference` is a pointer to an existing mod instance and contains only the minimal information required to locate the instance.

> *Rationale: This type was introduced to allow tools to work with a reference rather than a full mod instance, potentially leading to performance improvements. Additionally, it enables mod dependencies to be represented without bloating the JSON file, simplifying the effort for mod developers. Generally, `ModReference` functions similarly to a pointer in C/C++.*

## I.2 Physical Mods

Physical mods exist on the file system and provide assets to modify the game. They can be standalone or linked together into a set of virtual mods.

Physical mods can be categorized as either Steam Workshop Mods or non-Steam mods (default or ordinary mods).

## I.3 Runtime-Mods

This specification also supports Runtime Mods or Virtual Mods. Unlike ordinary mods (e.g., Steam Workshop Mods), a Virtual Mod is not bound to a specific location on the file system. Virtual mods can be composed at a tool’s runtime. As such, a Modinfo object may exist only at runtime and does not need to be exclusively bound to a physical file.

Virtual Mods require a `name` and respectively an `identifier` for a `ModReference` instance and `dependency` list. A Virtual Mod’s dependency list may contain physical mods (such as Steam Workshop Mods) or other Virtual Mods. Since Virtual Mods do not provide physical assets to the game, each Virtual Mod must have at least one physical mod as a dependency.

The combination of the virtual mod’s name and its dependency list constitutes the `ModIdentity` data for this instance.

## I.4 Instantiating Mods

An instance of a mod must have its `mod-reference.identifier` set according to the rules specified in [III.2.4 Creating Identifiers](#iii24-creating-identifiers).

### I.4.1 Mod Types

A mod's type is determined by its installation location.

- If the mod is located in a Steam Workshop directory, and the mod's root folder name is a valid Steam Workshop identifier (convertible to a `unsigned long`), the resulting `modtype` is `1` (Steam Workshop).
  
- In all other cases, the type is `0` (Default).

> *Note: Virtual mods are explicitly not supported. It is up to third-party developers to determine how to instantiate virtual mods.*

### I.4.2 Modinfo Files

The following rules apply when instantiating mods in the presence of modinfo files, as specified in [II.3 File Position](#ii3-file-position): 

- If no modinfo files are present, the mod is instantiated normally.

- If only a main modinfo file exists, one mod instance is created.

- If only variant modinfo files exist, one mod instance is created for each variant file.

- If both a main modinfo file and one or more variant modinfo files exist, one mod instance is created for the main modinfo file, and one mod instance is created for each variant modinfo file.

- If the main modinfo file is malformed, the mod is instantiated normally.

- If a variant modinfo file is malformed, that specific variant mod is not instantiated. If all variant files are malformed, it must be ensured that at least one mod is instantiated. This can be derived either from the main modinfo file (if present) or by falling back to normal instantiation as if no modinfo file were present.
---

# II. Modinfo JSON File

This section describes the syntax of a `ModInfo` file, its file constraints, and lookup mechanics.

## II.1 File Syntax

The metadata must be saved in a `JSON` file. To increase compatibility and reduce errors, the following allowances are made:

1. It is permitted to include unofficial JSON comments. Supported formats are single-line `// Comment` and multi-line `/* Comment */` comments. Tool implementers may choose whether to parse or ignore the content of these comments.
2. A trailing comma `,` is allowed after lists, values, or objects.


## II.2 Filename

There are two naming conventions for the file:

1. **Main file:** Named `modinfo.json`.
2. **Variant file:** Named `[Any_FS_compliant_name]-modinfo.json`.

Option `2` can be used to create different variants of a mod that share the same files.

*Example: If your mod is a submod for two different base mods, this setup allows you to develop and upload the mod once, while targeting both base mods simultaneously.*

## II.3 File Position

The target directory is the top level of the mod's folder (where the mod's `data` folder is located).

It may contain no files, one file, or multiple files, as described in [Filename](#ii1-filename).

If there is both a main file and at least one variant file, the main file’s content is merged into each variant file, unless the variant overrides a property.

Merging is described in [III.3.2](#iii32-merge-behavior).

If only variant files are present, each acts as a standalone main file.

[I.4 Instanciating Mods](#i4-instantiating-mods) specifies how to create mods are created form Modinfo files.

## II.4 Exemplary Content

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

# III. Data Structure Definitions

This section defines the `ModIdentity`, `ModReference`, and `ModInfo` data types, as well as their properties and other types introduced by this specification.

## III.1 The `"modidentity"` Type

The `modidentity` type contains data that defines a mod or enriches the mod's metadata. The following sections describe the properties that can be expressed within this specification.

The `modidentity` type defines three main properties:
- **`name`**
- **`version`**
- **`dependencies`**

### III.1.1 ModIdentity and Equality

A mod identity is used to fully qualify a mod definition. To compare the identities of two mods, these three properties can be used. The specification defines two standard strategies for using these properties to verify identity:

Implementations of this specification must provide an identity check based on two different strategies:

1. Only the `name` is considered, with a case-insensitive comparison.
2. The `name`, `version`, and `dependencies` are all considered:
   - The `name` comparison is case-insensitive.
   - The `version` comparison returns "equal" if both versions are either absent or both are present with the same value.
   - The `dependencies` comparison returns "equal" if both dependency lists (see [ModReference Equality](#iii22-modreference-equality)):
     - Have the same `resolve-layout` property AND,
     - Contain the exact same number of elements AND,
     - Have all elements matching in value and position.

The second strategy is the *default* identity-checking strategy.

> *Implementation Note: Implementations can add additional strategies if needed. Runtime equality for different kinds of dependency lists could also be implemented.*
  

### III.1.2 Properties

#### The `"name"` Property

**Level:** **REQUIRED**

**Data Type**: `String`  

**Data Semantics**: Unique Text 

**Description:**

This property specifies the fully qualified mod name, e.g., "Republic at War," "Thrawn's Revenge: Imperial Civil War," "Awakening of the Rebellion," or "Yuuzhan Vong at War." The value cannot be null or an empty string.

This property **must** also be present for variant modinfo files, even if they override all other properties from the main `modinfo.json`.

#### The `"version"` Property

**Level:** *OPTIONAL*

**Data Type**: `String`  

**Data Semantics**: 3 Digit Semantic Version 

**Description:**

The mod's version according to the extended semantic versioning: [Semantic Versioning](https://semver.org/).

*Examples: `"1.0.0"`, `"1.0.0-rc1"`, `"1.2.3-ALPHA-1"`*

> *Implementation Note: Implementations of this specification should consider handling malformed versions (e.g., four-digit or single-digit) to avoid potential parsing crashes. The behavior of this handling, however, shall be undefined by this specification.*

#### The `"dependencies"` Property

**Level:** *OPTIONAL*

**Data Type**: `dependencyList`

**Data Semantics**: Ordered List of Objects 

**Description:**

The `dependencyList` holds an ordered sequence of [`"modreference"` types](#the-modreference-type) that this mod relies on.

The list is either absent from the `modinfo.json` or contains at least one item. The list is not nullable.

The dependency list is strictly left-right ordered, where the first entry resembles the closest ancestor and the *n*'th entry the least close ancestor.

The target mod itself must **not** be listed. Doing so would result in a dependency cycle!

This specification supports multiple *resolve layouts*. A resolve layout is essentially an enumeration `resolve-layout` that describes how the dependency collection should be interpreted and processed by an implementation.

In JSON, the desired resolve layout can be specified by adding its name as a `string` value as the first element of the list. Example:

```json
{
  "name": "MyMod",
  "dependencies": [
    "FullResolved", 
    {
      "modtype": 0,
      "identifier": "./Mods/BaseMod"		
    }	
  ]
}
```

If no resolve layout is specified in the JSON, the value `ResolveRecursive` shall be used as the default.

An explanation of the supported layouts is shown in the table below:

| Resolve Layout | Meaning |
|:--:|:--|
|`ResolveRecursive`|TThe list shall only contain the mod's direct ancestors. Each entry may have its own dependencies, which shall be resolved recursively.|
|`ResolveLastItem`|The list shall only contain the mod's direct ancestors. Only the last item in the list shall be recursively resolved. There shall be no resolving for previous entries. If the list contains only one element, this element shall also be recursively resolved.|
|`FullResolved`|The list shall contain all ancestors of the target mod. The entries in the list shall be interpreted *as is*. No dependency resolving shall be performed by the tool.
|

[Partition IV](#iv-mod-dependency-handling) explains in detail the requirements how dependency resolving shall be implemented.


---

## III.2 The `"modreference"` Type

### III.2.1 ModReference vs. ModIdentity

A `ModReference` is distinguished by properties other than those of a `ModIdentity`, allowing it to be used for dependency resolving.

*Example: A mod `A` may have multiple mod references, e.g., one located in Steam Workshop and another in `FoC/Mods/A/`. Both references point to the same `ModIdentity` (Name: A, Version: null, Dependencies: Empty); however, the references themselves are not equal.*

### III.2.2 ModReference Equality

Two `ModReferences` are considered equal when both properties, `identifier` and `modtype`, match. The `identifier` property is case-insensitive. This strategy must be applied when resolving dependencies.

> *Implementation Notes: It is up to an implementation of this specification to add more possible strategies. If the `identifier` contains a local path, it is the responsibility of the implementation to normalize the path if necessary.*

### III.2.3 Properties

#### The `"modtype"` Property

**Level:** **REQUIRED**

**Data Type**: `integer`

**Data Semantics**: Enum

**Description:**

The `modtype` enumeration:

| Value | Meaning |
|:--:|:--|
|`0`|any normal mod inside the `Mods/` directory|
|`1`|a Steam Workshop mod|
|`2`|a "virtual" mod.|

*Rationale: The current mod does NOT contain a `modtype` property because the mod should not have to know its own type. Otherwise, sharing this file across Steam and disk mods would not be possible. A `ModReference` requests this data, meaning tool support to determine the actual `modtype` is necessary. This design choice was made because mod linking should always be considered for Steam Workshop mods. The possibility to reference local mods is a convenience functionality intended to be used by mod developers for test setups and development.*

> *Note: Since virtual mods have an unstable/unpredictable identifier, they should not be used in a `modinfo.json` to avoid tool-specific errors for the user. However, this specification shall not forbid it.*



#### The `"identifier"` Property

**Level:** **REQUIRED**

**Data Type**: `string`

**Data Semantics**: Unique identifier

**Description:**

Uniquely and predictably identifies a mod reference. Two mod references with the same identifier are considered equal.  
The content must be predictable so it can be used to locally identify mods.  

The identifier cannot be `null` or an empty string.  

The rules for creating the identifier are described in [III.2.4 Creating Identifiers](#iii24-creating-identifiers).  

> *Note: The identifier should only be used for comparing two references. It is not intended for deserialization to extract information, such as Steam Workshop IDs, file paths or a mod names.*

> *Rationale: The identifier is only locally unique and not globally unique because file paths are tied to the current system. For Steam Workshop items, the ID is inherently globally unique.*

> *Security Note: Deserializing the identifier, if really necessary, should only occur after proper data validation or sanitization to prevent security risks.*

#### The `"version-range"` Property

**Level:** *Optional*

**Data Type**: `string`

**Data Semantics**: Version Range

**Description:**

This property shall only be parsed and otherwise ignored completely by an implementation of this specification. It explicitly is not used for `modreference` equality matching and dependency resolution. It shall only be used by custom tools.

This specification shares the same syntax and semantics as those used for [npm node dependency ranges](https://github.com/npm/node-semver#ranges). If the property is unset, the version range `*` (which means `>= 0.0.0`) shall be used.

> *Rationale: This property can be used for entries in a mod's dependency list. 3rd party tools might want to consume the given version range and perform custom mod matching.*


## III.2.4 Creating Identifiers

The following rules apply to create a identifier: 

#### Default Mods
For default mods, the identifier is the mod's install directory path.

In the case where the mod's location produces multiple mod instances due to variant modinfo files, append the variant's name to the path.

For mods installed in the game's Mods directory, use the relative path to the `GAME_DIR/Mods/` directory. This effectively means using just the mod's folder name.

For mods installed elsewhere, use the absolute path.

**`Identifier := PATH (':' MOD_NAME)?`**

*Example (Mod installed in Mods folder)*  
`Identifier = "mod-folder-name"`

*Example (Variant Mod installed in Mods folder)*  
`Identifier = "mod-folder-name:Variant1"`

*Example (Variant Mod installed elsewhere)*  
`Identifier = "C:\mod-folder-name:Variant1"`

> *Notes and Rationale:*
> - The identifier is compared in a case-insensitive manner, aligning with the Windows file system. While this may cause collisions on Linux systems, this is an acceptable trade-off to make manual creation of modinfo files more error tolerant.
Therefore, the identifier should not be parsed.  
> - Absolute paths identifiers are intended solely for development purposes to work locally on a developer's system. Absolute paths vary across operating systems (e.g., Linux uses `'/'` as a separator, while Windows uses `'\'` by default). No additional guarantees are provided for absolute paths. 
> - Appending the mod's name to the path, with the syntax specified here, effectively makes the path invalid for Windows due to the `':'` character being prohibited in file names. 

#### Steam Workshops Mods

For Steam Workshop mods, the identifier is the mod's Steam Workshop ID. 

In the case the mod's location produces multiple mod instances due to variant modinfo files, append the variant's name to the Steam Workshops ID.
**`Identifier := STEAM_WS_ID (':' MOD_NAME)?`**

*Example (Workshop Mod)* 
`Identifier = "1234567890"`

*Example (Variant Workshop Mod)* 
`Identifier = "1234567890:Variant1"`


#### Virtual Mods

For virtual mods, the identifier is the mod identity data in JSON format. 

**`Identifier := MOD_IDENTITY_JSON`**

*Example:* 
```
Identifier = "{
  "name": "Virtual Mod Name",
  "version": "1.0.0",
  "dependencies": [
    "FullResolved", 
    {
      "modtype": 0,
      "identifier": "./Mods/BaseMod"		
    }	
  ]
}"`
```

> *Implementation Note: Avoid deserialization of the identifier to check equality. 
Instead a library should be used, that procudes stable JSON data, so the identifier can be compared on string level.*


---

## III.3 The `"modinfo"` Type

### III.3.1 Properties

A `modinfo` is a `modreference`, thus they share the same properties with the same meanings.

#### The `"summary"` Property

**Level:** *OPTIONAL*

**Data Type**: `String`  

**Data Semantics**: Text 

**Description:**

This property allows you to include a short summary about the mod, supporting Steam-flavored BBCode.

#### The `"icon"` Property

**Level:** *OPTIONAL*

**Data Type**: `String`  

**Data Semantics**: File System Path 

**Description:**

The path to the mod's icon file, **relative** to the mod's root directory or an **absolute** path.


#### The `"languages"` Property

**Level:** *OPTIONAL*

**Data Type**:  [`languageInfo`](#iii4-the-languageinfo-type)`[]`

**Data Semantics**: Set of supported languages

**Description:**

This property holds a unique set of [`language`](#iii4-the-languageinfo-type) objects. Each item indicates a language that is supported by the mod. 

The property is optional. When *NOT* present, the language **English** (`"en"`) is assumed to be default. However, if the property is defined, English *MUST* be included when supported by the mod, too.

If the array is empty, the property is considered as unset.

The value is not nullable.

#### The `"steamdata"` Property

**Level:** *OPTIONAL*

**Data Type**: [`steamdata`](#iii5-the-steamdata-type)  

**Data Semantics**: Steam Workshops JSON

**Description:**

The [`"steamdata type"`](#iii5-the-steamdata-type) container holds additional info that is required for the Steam Version of the game.

#### The `"custom"` Property

**Level:** *OPTIONAL*

**Data Type**: `Dictionay<string, any>`

**Data Semantics**: Collection of arbitrary data, stored by keys

**Description:**

The `custom` property allows arbitrary extensions to the format using a collection of key/value paris. Key shall be unique strings. The value can be of any value. 

The property is not nullable.

> *Note: Because the custom proptery exists only for 3rd party tools, it shall therefore be unspecified whether `key` is case-sensitive or insensitive.* 

> *Implementation Note: 3rd party tools parsing the custom propertie should support the case where the JSON data contains elements of the same key wihtout crashing. It's up to the 3rd party tool which value(s) to use in those occasions.*

> *Security Note: Parsing the JSON as `Dictionary<string, any>` while guessing the conrect type of the value is potentially insecure. [Check out why](https://www.blackhat.com/docs/us-17/thursday/us-17-Munoz-Friday-The-13th-JSON-Attacks-wp.pdf). It is therefore recommented for 3rd party tools to deserialize `any` into the native JSON representation your JSON parser provides. E.g., for .NET this is `JsonElement`.*

### III.3.2 Merge Behavior

It is possible to merge the values of one modinfo into another. This is used when there exists a main modinfo file and one or many variants. The variant can reuse properties of the main modinfo file, simply by not specifying a property. This allows the variant files to be as short as possible. 

*The variant file is hereby called `target`. The main modinfo file is hereby called `base`.*

In general, properties of `target` overwrite the properties from `base`. In other words, `target` is merged into `base`.

The following rules apply when properties get overwritten:

- `name` can never be overwritten because it is a required property for every modinfo file. 

- `language` only gets overwritten if the property was explitily set by `target`.

- `custom` is merged per key/value pair. In the case `base` and `target` contain the same `key`, the `value` of the `target` is used.

- all other properties are overwritten by-reference. This means there shall be no merging of subsequent properties/items for objects and array data types such as `steamdata` or `dependencies`. 

> *Implementation Note: An implementation of this specification must be aware whether a modinfo explicity set the `language` property the default value (English - FullLocalized) was applied implicitly.*

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
|`1`| **Text:** A `mastertextfile_xxx.dat` is available in this language. |
|`2`| **Speech:** Speech event files are in their own language folder. (Important for Movies, Missions, and Holograms) |
|`4`| **SFX:** Sound effects, such as unit actions, are localized. |
|`7`| **Fully localized:** Combines `1`, `2`, and `4`. |

When the property is omitted for a `language` object, the value `7` (fully localized) is applied.


### III.4.2 LanguageInfo Equality
A `LanguageInfo` is uniquely identified by its `code` property. The language code is case-insensitive.

*Note: Validation shall not fail if multiple languages exist in the JSON modinfo. The behavior regarding which language is selected in such a case shall be undefined by this specification as long as any of the duplicate languages is selected.*

*Rationale: JSON Schema does not support validating uniqueness on single properties. Neither does the JSON specification define how multiple keys shall be handled. Thus, this specification also leaves this undefined.*

> *Implementation Note: Tools may offer additional equality strategies, which also include the `support` property. The contract defined above must be the default.*

---

## III.5 The `"steamdata"` Type

### III.5.1 Properties

#### The `"publishedfileid"` Property

**Level:** **REQUIRED**

**Data Type**: `string`

**Data Semantics**: Unique identifier

**Description:**

The Steam Workshop ID, which shall be able to parse into an `unsigned long`.

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

Arbitrary metadata as a string.

> *Implementation Notes: Even if this value is not present, tools should print this property with an empty value `""` in the file. This behavior is recommended to increase compatibility with the Steam Workshop uploader.*


#### The `"tags"` Property

**Level:** **REQUIRED**

**Data Type**: `Tag[]`

**Data Semantics**: Set of tags

**Description:**

Steam tags as defined by the [Steam tag documentation](https://partner.steamgames.com/doc/api/ISteamUGC#SetItemTags). Tags are limited to 255 characters each, must only contain ASCII printable characters, and cannot include the comma `,` character. Extended ASCII characters (`> '\u007F'`) are not supported. Tags are case-sensitive, and each tag must be unique.

At least either `EAW` or `FOC` is required to determine the game for which the mod is displayed. It is permissible to support both game tags.


#### The `"description"` Property

**Level:** *OPTIONAL*

**Data Type**: `String`

**Data Semantics**: Steam flavoured BB-Code description 

**Description:**

Optional description of the Mod in Steam-flavored BBCode.

> *Implementation Notes: Even if this value is not present, tools should print this property with an empty value `""` in the file. This behavior is recommended to increase compatibility with the Steam Workshop uploader.*



#### The `"previewfile"` Property

**Level:** *OPTIONAL*

**Data Type**: `String`

**Data Semantics**: Relative Path

**Description:**

Relative path to an image file that holds the preview image.

> *Implementation Notes: Even if this value is not present, tools should print this property with an empty value `""` in the file. This behavior is recommended to increase compatibility with the Steam Workshop uploader.*


---


# IV Mod Dependency Handling

The game supports chaining (which could also be called *overriding* or *linked* mods) by queuing up the command line arguments `STEAMMOD` and/or `MODPATH`. While the command line options can only resemble a line (or, more precisely, a queue), real mod dependencies can appear as a complex tree due to multiple inheritance. Thus, there needs to be a deterministic logic to convert the real hierarchy into a CLI-compatible representation. 

To describe relations between mods, this specification introduces an optional [`dependencies`](#the-dependencies-property) property, which is an ordered list.

The conversion from a tree structure to a line is called *flattening* or *traversing*. Since the `dependencies` list supports multiple resolve layouts, different strategies are defined to dictate how the conversion shall behave.

## IV.1 Dependency Resolving Algorithm

Each resolve layout implementation fulfills the following general requirements, in addition to its own requirements, to create a successful conversion:

1. The `dependencies` list is processed from start to end.
2. The resulting list contains the target mod, which shall be the first element of the list.
3. The resulting list must not have duplicates.
4. The implementation must be able to recognize and respond to identified dependency cycles accordingly.
5. Virtual mods must be preserved in the resulting list.

*Rationale: While virtual mods cannot be represented by a command line (which means they are practically invisible), they still need to exist in the resulting list so tools always have the most precise data to deal with. Removing virtual mods from the flattened result is a tool-specific implementation detail.*

*Note: Converting a (non-binary) tree structure to a flattened and duplicate-free line is a one-way operation, which also loses accuracy. However, since in some cases multiple outcomes are possible, the resolve layout `ResolveRecursive` might lead to unexpected results. The requirements above are created to ensure consistency across multiple implementations.*

*Advice: To ensure your mod always works as intended, it is recommended to keep the inheritance level to a minimum, and multiple inheritances should be avoided.*



## IV.1.1 Resolving `ResolveRecursive`

Flattening shall be performed based on a breadth-first search. This approach ensures that the left-right order of the dependency list is preserved.

For each entry to be resolved, the algorithm must honor the resolve layout of the entry's dependency list. This means the algorithm shall not resolve everything recursively but only where the resolve layout specifies.

Due to multiple inheritance, particularly in the case of [diamond inheritance](https://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem), it is possible for an entry to occur multiple times during resolution. In such cases, it is not guaranteed that a cycle is present. However, duplicates that are not part of a cycle must be removed from the resulting list.

The algorithm must ensure that the resulting list is [topologically sorted](https://en.wikipedia.org/wiki/Topological_sorting) while still preserving the left-to-right order of direct ancestors.

To guide an implementation, this specification defines the following [test cases](#iv2-full-recursive-dependency-resolving-test-cases) that a fully recursive resolving algorithm must pass.

*Implementation Note: Cycle detection works best on the directed dependency graph rather than on a traversed list.*



## IV.1.2 Resolving `ResolveLastItem`

Only the last (or the only) element in the list shall be resolved, as specified in [Resolving ResolveRecursive](#iv11-resolving-resolverecursive).

As soon as there exists a duplicate, a dependency cycle is present.

## IV.1.3 Resolving `FullResolved`

Since this layout indicates that the dependency list shall be interpreted *as is*, the flattening algorithm shall return the list unmodified.

If the list contains a duplicate, a dependency cycle is present.

## IV.2 Full Recursive Dependency Resolving Test Cases

Node `A` is always the mod that should be loaded. Every mod following `:` are direct dependencies of the mod; if there are multiple, they are separated by a `,`.

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


Expected list: A, B, C, D, E

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
