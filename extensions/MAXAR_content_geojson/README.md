# MAXAR_content_geojson 

## Contributors

* Björn Blissing, Maxar, [@bjornblissing](https://twitter.com/bjornblissing)
* Erik Dahlström, Maxar, [@erikdahlstrom](https://twitter.com/erikdahlstrom)

## Status

Draft - **Version 0.1.0**, November 19, 2024

## Dependencies

Written against the 3D Tiles 1.1 spec.

## Contents

  - [Overview](#overview)
  - [Optional vs. Required](#optional-vs-required)
  - [Property Metadata](#property-metadata)
  - [Examples](#examples)

## Overview

This extension allows a tileset to use [GeoJSON](https://tools.ietf.org/html/rfc7946) directly as tile content.

It also allows for specifying a external schema to help interpret the feature properties stored in the GeoJSON layer. 
This schema can be marked with a high-level semantic identifier that define the meaning of the properties. Recommended best practice is to select a value from a controlled vocabulary or formal classification scheme.

Each field in the schema can also be marked with a semantic identifier, that describes how this property should be interpreted.

## Optional vs Required

This extension is required, meaning it should be placed in the tileset JSON top-level `extensionsRequired` and `extensionsUsed` lists.

## Property Metadata

A metadata node `metadataEntity.schema.json` can refer to a external properties metadata schema which specifies the content inside the GeoJSON layer.

The schema can be connected to a indexed 3D-tiles group. The usage of the property metadata schema have the following restrictions:
 - Only one type of Geometry(`Point`, `LineString`, `Polygon`, `MultiPoint`, `MultiLineString`, `MultiPolygon`) is allowed per GeoJSON-layer.
 - The refered GeoJSON layers cannot contain `GeometryCollection` nodes.

## Examples

**Usage without optional properties schema:**

```json
{
  "asset": {
    "version": "1.1"
  },
  "extensionsRequired": [
    "MAXAR_content_geojson"
  ],
  "extensionsUsed": [
    "MAXAR_content_geojson"
  ],
  "geometricError": 0,
  "root": {
    "boundingVolume": {
      "region": [
        -1.7095238193613294,
        0.54322210241748092,
        -1.7084717202817712,
        0.54416527589988806,
        243.45309697370976,
        360.09079237561673
      ]
    },
    "geometricError": 0,
    "refine": "REPLACE",
    "content": {
      "uri": "vegetation_tile.geojson"
    }
  }
}
```

**Usage *with* optional properties metadata schema:**

```json
{
  "asset": {
    "version": "1.1"
  },
  "extensionsRequired": [
    "MAXAR_content_geojson"
  ],
  "extensionsUsed": [
    "MAXAR_content_geojson"
  ],
  "metadata": {
    "class": "tileset",
    "properties": {
      "content_type": "VECTOR",
      "geometry_model": "OBJECTS",
      "name": "Vegetation Layer",
      "schema": "wff/15",
      "wff_version": "1.5"
    },
    "extensions": {
      "MAXAR_content_geojson": {
        "propertiesSchemaUri": "vegetation_schema.json"
      }
    }
  },
  "geometricError": 0,
  "root": {
    "boundingVolume": {
      "region": [
        -1.7095238193613294,
        0.54322210241748092,
        -1.7084717202817712,
        0.54416527589988806,
        243.45309697370976,
        360.09079237561673
      ]
    },
    "geometricError": 0,
    "refine": "REPLACE",
    "content": {
      "uri": "vegetation_tile.geojson"
    }
  }
}
```


**Example properties metadata schema:**

```json
{
  "name": "Vegetation Properties Schema",
  "semantic":"vegetation",
  "geometry": {
    "type":"Polygon",
    "dimensions": 2
  },
  "properties": [{
      "id": "areaId",
      "type": "Integer",
      "required": true
    }, {
      "id": "vegetationType",
      "type": "String",
      "default": "UNDEFINED"
    },{
      "id": "propertyValue",
      "type": "Float",
      "unit": "USD",
      "min" : 0,
    }
  ]
}
```