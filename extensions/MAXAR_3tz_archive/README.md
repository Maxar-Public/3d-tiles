# MAXAR_3tz_archive 

## Contributors

Erik Dahlström, Maxar, [@erikdahlstrom](https://twitter.com/erikdahlstrom)

## Status

Draft

## Dependencies

This extension depends on 3DTILES_layers.

## Contents

  - [Overview](#overview)
  - [Schema Updates](#schema-updates)
  - [Examples](#examples)

## Overview

This extension allows a 3DTILES_layers to use 3tz archives as content, and to use content from inside 3tz archives as content.

When referencing a 3tz as content directly, the root tileset of the 3tz, as defined by the [3D Tiles Archive Format 1.0 specification](https://github.com/CesiumGS/3d-tiles/issues/422), must be used.

## Examples

```json
{
  "asset": {
    "version": "1.0"
  },
  "extensionsUsed": [
    "3DTILES_layers",
    "3DTILES_metadata",
    "MAXAR_3tz_archive"
  ],
  "extensionsRequired": [
    "3DTILES_layers",
    "3DTILES_metadata",
    "MAXAR_3tz_archive"
  ],
  "extensions": {
    "3DTILES_metadata": {
      "extras": {
        "draftVersion": "0.0.0"
      },
      "layers": {
        "3D_Terrain": {
          "name": "Terrain"
        },
        "3D_Buildings": {
          "name": "Buildings"
        }
      }
    }
  },
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
    "extensions": {
      "3DTILES_layers": {
        "contents": [
          {
            "layer": "3D_Terrain",
            "content": {
              "uri": "terrain.3tz",
              "type": "application/vnd.maxar.archive.3tz"
            }
          },
          {
            "layer": "3D_Buildings",
            "content": {
              "uri": "buildings.3tz",
              "type": "application/vnd.maxar.archive.3tz"
            }
          }
        ]
      }
    }
  }
}
```