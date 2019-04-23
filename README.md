# eaw.modinfo Definition - v1.0.0

A standard definition for Star Wars: Empire at War mod info files.

The info files defined herein allow mod makers and tool makers to specify metainformation about a given Empire at War mod.

## Filename

The metainformation has to be saved in a `JSON` file called `modinfo.json`.

## File Position

The `modinfo.json` has to be located at the top level of the mod folder next to the `data` folder.

## File Content

The following sections specify the required and optional content for `eaw.modinfo` in Version 1.0.0.

### Exemplary Content

```json
{
  "name": "The mod's name",
  "description": "A generic description about the mod in Steam-flavoured BBCode.\nNice, eh?",
  "icon": "relative/path/to/icon.png",
  "version": "1.0.0.0",
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
  }
}
```

### The `"name"` Tag [REQUIRED]

This tag specifies the fully qualified mod name, e.g. "Republic at War", "Thrawn's Revenge: Imperial Civil War", "Awakening of the Rebellion" or "Yuuzhan Vong at War"

### The `"description"` Tag [OPTIONAL]

This tag allows you to include a description including Steam-flavoured BBCode.

### The `"icon"` Tag [OPTIONAL]

The relative path to the mod's preview image. This will also be used for the Steam Preview image.

**WARNING**: This tag is *REQUIRED* if `steamdata` is used.

### The `"version"` Tag [OPTIONAL]

The mod's version according to the extended semantic versioning: [Semantic Versioning](https://semver.org/) that also supports a fourth digits for build increments, etc. and suffixes (e.g. `"-rc1"`).

### The `"steamdata"` Tag [OPTIONAL]

The `steamdata` container holds additional info that is required for the Steam Version of the game.
The container is either absent from the `modinfo.json` or it is fully required.

#### The `"steamdata.publishedfileid"` Tag

The Steam Workshop ID.

#### The `"steamdata.contentfolder"` Tag

The content folder's name as specified by the Steam Uploader.

#### The `"steamdata.visibility"` Tag

The visibility enumeration:

| Value | Meaning |
|:--:|:--|
|0|hidden|
|1|friends only|
|2|public|


#### The `"steamdata.metadata"` Tag

Arbitrary metadata as string.

#### The `"steamdata.tags"` Tag

Steam Tags as specified by the Steam Uploader.

At least either `EAW` or `FOC` is required to determine the game the mod shows up for.
