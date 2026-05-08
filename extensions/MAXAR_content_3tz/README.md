# MAXAR_content_3tz

<!-- omit in toc -->
## Contributors

* Erik Dahlström, Maxar, [@erikdahlstrom](https://github.com/erikdahlstrom)
* Björn Blissing, Maxar, [@bjornblissing](https://github.com/bjornblissing)
* Johan Borg, Maxar, [@jo-borg](https://github.com/jo-borg)

<!-- omit in toc -->
## Status

**Version 1.1.0**, April 22, 2026

<!-- omit in toc -->
## Dependencies

Written against the [3D Tiles 1.1](https://github.com/CesiumGS/3d-tiles/tree/1.1/specification) spec.

The key words MUST, MUST NOT, SHOULD, SHOULD NOT, and MAY in this document are to be interpreted as described in RFC 2119.

Referenced 3TZ container files (`.3tz` or `.3dtiles.zip`) MUST follow the [3D Tiles Archive Format 1.4](https://github.com/Maxar-Public/3tz-specification/releases/download/v1.4/3D.Tiles.Archive.Format.v1.4.pdf) spec.

## Contents

  - [Overview](#overview)
  - [Path Resolver Algorithm](#path-resolver-algorithm)
  - [Optional vs. Required](#optional-vs-required)
  - [Examples](#examples)
  - [Schema](#schema)
  - [Frequently Asked Questions](#frequently-asked-questions)

## Overview

This extension allows a tileset to use a [3TZ container](https://github.com/Maxar-Public/3tz-specification/releases/download/v1.4/3D.Tiles.Archive.Format.v1.4.pdf) directly as tile content.

In this document, "3TZ container" refers to a 3D Tiles archive addressed through this extension, including `.3tz` and `.3dtiles.zip` files that follow the 3D Tiles Archive Format 1.4 specification.

When this extension is listed in `extensionsRequired` for a tileset, all URIs referenced by the tileset—both directly and indirectly—MUST be resolved using the 3TZ [Path Resolver algorithm](#path-resolver-algorithm). Note that this requirement includes any URIs inside tile contents, e.g., resources referenced by glTF.

The URI path syntax MUST be used when referring to a specific file inside a 3TZ container, treating the container as if it were a plain directory, e.g., `../../resources.3tz/tiles/0/0/0.glb`.

When a tileset outside any 3TZ container wants to use 3TZ URI resolution, it MUST explicitly list `MAXAR_content_3tz`. For a tileset JSON resource resolved from within a 3TZ container, `MAXAR_content_3tz` is implicitly in effect and remains so for subsequent URI references as long as resolution stays within one or more 3TZ containers. Descendant tilesets within a 3TZ container therefore do not need to explicitly list the extension. This implicit applicability ends once resolution reaches a resource outside all 3TZ containers. A later external tileset that wants to use 3TZ URI resolution MUST explicitly list `MAXAR_content_3tz`.

Any path segment whose basename ends in `.3tz` or `.3dtiles.zip` is interpreted as a 3TZ container, not as a regular directory.

Implementations MAY block resolved URIs whose base does not match the tileset requiring this extension.

Note that in URIs, any reserved characters (as defined in RFC 3986 Section 2.2 and RFC 3987 Section 2.2) MUST be percent-encoded.

Note that if this extension is used to tie the datasets to some underlying storage
or particular URI scheme, the data may become less portable as a result.

### Path Resolver Algorithm

1. If the URI is not a data URI and the URI path matches the regular expression `(.+(?:\.3tz|\.3dtiles\.zip))[\/]?(.*)`, then let `resolved 3TZ container path` be the first match group. The regular expression uses the grammar defined in [ECMAScript 262](https://262.ecma-international.org/5.1/#sec-15.10).

2. If the second match group is empty, then let the `resolved inner file path` be `tileset.json`; otherwise, let the `resolved inner file path` be the second match group.

3. Return the file contents of the `resolved inner file path` from the 3TZ container file that can be read from the `resolved 3TZ container path`.

    Implementations MAY return the data in a form that suits the consumer (e.g., the raw compressed data), as long as the consumer can decompress it if needed.

## Optional vs. Required

This extension is required. It MUST be listed in the tileset JSON top-level `extensionsRequired` and `extensionsUsed` arrays.

## Examples

```json
{
  "asset": {
    "version": "1.1"
  },
  "extensionsUsed": [
    "MAXAR_content_3tz"
  ],
  "extensionsRequired": [
    "MAXAR_content_3tz"
  ],
  "schema": {
    "id": "dataset_schema",
    "classes": {
      "dataset": {
        "properties": {
          "name": {
            "type": "STRING",
            "required": true
          },
          "revision": {
            "type": "STRING",
            "required": true
          }
        }
      }
    }
  },
  "metadata": {
    "class": "dataset",
    "properties": {
      "name": "Dataset with layers",
      "revision": "0.3"
    }
  },
  "groups": [
    {
      "class": "dataset",
      "properties": {
        "name": "Terrain",
        "revision": "0.4"
      }
    },
    {
      "class": "dataset",
      "properties": {
        "name": "Buildings",
        "revision": "0.2"
      }
    }
  ],
  "geometricError": 500,
  "root": {
    "boundingVolume": {
      "region": [
        -1.2419,
        0.7395,
        -1.2415,
        0.7396,
        0,
        20.4
      ]
    },
    "geometricError": 500,
    "refine": "REPLACE",
    "children": [
      {
        "boundingVolume": {
          "region": [
            -1.2419,
            0.7395,
            -1.2415,
            0.7396,
            0,
            20.4
          ]
        },
        "geometricError": 500,
        "content": {
          "uri": "terrain.3tz",
          "group": 0
        }
      },
      {
        "boundingVolume": {
          "region": [
            -1.2419,
            0.7395,
            -1.2415,
            0.7396,
            0,
            20.4
          ]
        },
        "geometricError": 500,
        "content": {
          "uri": "terrain.3tz/buildings.json",
          "group": 1
        }
      }
    ]
  }
}
```
## Schema

- [tileset.MAXAR_content_3tz.schema.json](schema/tileset.MAXAR_content_3tz.schema.json) - Schema for the tileset extension


## Frequently Asked Questions

### What happens when this extension is required by a tileset?

When `MAXAR_content_3tz` is listed in `extensionsRequired`, **all URIs** referenced by the tileset (both directly and indirectly) MUST be resolved using the 3TZ Path Resolver algorithm. This includes:

- URIs in the tileset JSON itself
- URIs inside tile contents (e.g., resources referenced by glTF files)
- Any other URI references throughout the entire tileset hierarchy

### Can I nest 3TZ containers inside other 3TZ containers?

**No.** According to the [3D Tiles Archive Format specification](https://github.com/Maxar-Public/3tz-specification/releases/download/v1.4/3D.Tiles.Archive.Format.v1.4.pdf), 3TZ containers MUST NOT be nested inside other 3TZ containers.

Additionally, filenames and directory name entries within a 3TZ container MUST NOT contain the substring `.3tz` or `.3dtiles.zip`. This restriction helps avoid confusion with actual container references. This restriction applies only to entry names within the container and does not apply to the container filename. It does not apply to URIs embedded in content stored inside the container.

### Can I reference other 3TZ containers from within a 3TZ container?

**Yes.** A tileset MAY reference other 3TZ containers from within a 3TZ container using URIs. However, be aware of these potential implementation considerations:

- **Best practice**: For maximum compatibility across different viewers, references SHOULD remain within the same distribution unit, that is, a known collection of related 3TZ container files, rather than pointing to external web locations.
- **Implementation variations**: Some viewer implementations may restrict or block references that don't originate from the same base location as the tileset requiring this extension.

### When is `MAXAR_content_3tz` implicit, and when must it be listed explicitly?

`MAXAR_content_3tz` MUST be explicitly listed by a tileset outside any 3TZ
container that uses 3TZ URI resolution.

For a tileset JSON resource resolved from within a 3TZ container,
`MAXAR_content_3tz` is implicitly in effect and remains so for subsequent URI
references as long as resolution stays within one or more 3TZ containers.
Once resolution reaches a resource outside all 3TZ containers, this implicit
applicability ends.

Examples:

- `archive.3tz/tileset.json` → `0/0/0.json` resolves to
  `archive.3tz/0/0/0.json`, and `MAXAR_content_3tz` remains implicitly in
  effect.

- `archive.3tz/tileset.json` → `../archive2.3tz/0/0/0.json` resolves into
  `archive2.3tz`, and `MAXAR_content_3tz` remains implicitly in effect.

- `archive.3tz/tileset.json` → `../external_tileset.json` resolves outside any
  3TZ container, so implicit applicability ends at `external_tileset.json`.

- If `external_tileset.json` later references `../archive2.3tz/0/0/0.json`,
  then `external_tileset.json` MUST explicitly list `MAXAR_content_3tz` in
  `extensionsUsed` and `extensionsRequired`.

- A reference such as `../dir.3tz/file.json` is interpreted as referring to the
  3TZ container `dir.3tz`, not to a regular directory named `dir.3tz`.

- A reference such as `../dir.3dtiles.zip/file.json` is interpreted as
  referring to the 3TZ container `dir.3dtiles.zip`, not to a regular
  directory named `dir.3dtiles.zip`.

### How do I reference files inside a 3TZ container from another 3TZ container?

When referencing files inside a 3TZ container from another 3TZ container or external tileset, the URI path syntax MUST be used, treating the container as if it were a directory:

```
../../resources.3tz/tiles/0/0/0.glb
```

The path resolver algorithm will automatically handle extracting the correct file from the 3TZ container.

### Can I mix 3TZ container and non-container content in the same tileset?

When this extension is **required** (listed in `extensionsRequired`), all URI resolution MUST go through the 3TZ Path Resolver algorithm. However, the algorithm is designed to handle both 3TZ container and non-container URIs appropriately - it only applies special 3TZ container processing to URIs that match the 3TZ container pattern `(.+(?:\.3tz|\.3dtiles\.zip))[\/]?(.*)`.
