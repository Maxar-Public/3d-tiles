# VRICON_class

## Contributors

* Erik Dahlström, [@erikdahlstrom](https://github.com/erikdahlstrom)
* Johan Borg, [@jo-borg](https://github.com/jo-borg)
* Björn Blissing, [@bjornblissing](https://github.com/bjornblissing)

## Status

Draft

## Dependencies

Written against the 3D Tiles 1.0 spec.


## Contents

- [Overview](#overview)
- [Optional vs Required](#optional-vs-required)
- [Defining Class](#defining-class)
- [Examples](#examples)

## Overview

VRICON_class extension is used to associate a label to the class attributes used in the glTF tile content.

## Optional vs Required

This extension is optional, meaning it should be placed in the tileset JSON `extensionsUsed` list, but not in the `extensionsRequired` list.

## Defining Class

A class is defined with at `name` string and a `class` int.

### Geometry classes

 |*Type*     |*Id*  | 
 |:----------|-----:|
 | DEFAULT   | 0x00 |
 | DSM       | 0x01 |
 | DTM       | 0x02 |
 | DHM       | 0x03 |
 | DRAPE     | 0x20 |
 | FILLER    | 0x30 |
 | FOREST    | 0x05 |
 | BUILDING  | 0x06 |
 | BRIDGE    | 0x07 |

### Texture classes

 |*Type*             |*Id*    |
 |:------------------|-------:|
 | DEFAULT           | 0x0000 |
 | TEXTURE           | 0x0001 |
 | UNCERTAINTY_CE90  | 0x0031 |
 | UNCERTAINTY_LE90  | 0x0032 |
 | UNCERTAINTY_CE90R | 0x0033 |
 | UNCERTAINTY_LE90R | 0x0034 |
 | LANDCOVER         | 0x1100 |

 
 

## Examples

```json
"VRICON_class": {
  "geometry": [
  {
    "name": "dsm",
    "class": 1
  }
  ],
  "texture": [
  {
    "name": "r3dm::cdm_difference",
    "class": 18
  },
  {
    "name": "r3dm::date_avg",
    "class": 98
  },
  {
    "name": "r3dm::date_first",
    "class": 96
  },
  {
    "name": "r3dm::date_last",
    "class": 97
  },
  {
    "name": "r3dm::image_count",
    "class": 32
  },
  {
    "name": "r3dm::ndvi",
    "class": 2314
  },
  {
    "name": "r3dm::ndvi_cnt",
    "class": 2317
  },
  {
    "name": "r3dm::ndvi_max",
    "class": 2315
  },
  {
    "name": "r3dm::ndvi_min",
    "class": 2316
  },
  {
    "name": "r3dm::security",
    "class": 48
  },
  {
    "name": "r3dm::uncertainty_ce90",
    "class": 49
  },
  {
    "name": "r3dm::uncertainty_le90",
    "class": 50
  },
  {
    "name": "r3dm::universal_mix",
    "class": 86
  },
  {
    "name": "texture",
    "class": 1
  }
  ]
},
```

## glTF
In the glTF files each mesh and image is tagged which the appropriate class id under extras.vricon_class

Example of a glTF DTM mesh with a Landcover texture:

```json
{
  ...
  images: [
    {
        ...
        "extras": {
          "vricon_class": 145
        }
    }
  ],
  meshes: [
    {
      ...
      "extras": {
        "vricon_class": 2
      }
    }
  ]
}
```

