# MAXAR_extent

## Contributors

* Erik Dahlström, Maxar, [@erikdahlstrom](https://github.com/erikdahlstrom)
* Johan Borg, Maxar, [@jo-borg]( https://github.com/jo-borg)
* Björn Blissing, Maxar, [@bjornblissing](https://github.com/bjornblissing)

## Status

**Version 1.0.0**, June 30, 2025

## Dependencies

Written against the 3D Tiles 1.1 spec.

## Contents

- [Overview](#overview)
- [Optional vs. Required](#optional-vs-required)
- [Schema](#schema)
- [Defining Extents](#defining-extents)
- [Examples](#examples)
  - [`tileset.json`](#tilesetjson)
  - [`extent.geojson`](#extentgeojson)

## Overview

This extension allows annotating one or more 2D regions (extents) in a tileset via a reference to a [GeoJSON](https://tools.ietf.org/html/rfc7946) file.

The extension is useful for a variety of scenarios, such as overlaying high-resolution geometry on top of low-level geometry, defining a 2D collision boundary, or clipping excess geometry inside a tileset.

## Optional vs. Required

This extension is optional, meaning it should be placed in the tileset JSON top-level `extensionsUsed` list but not in the `extensionsRequired` list.

## Schema

This extension modifies the 3D Tiles schema by adding properties to the tileset root:

- **JSON Schema**: [`schema/tileset.MAXAR_extent.schema.json`](schema/tileset.MAXAR_extent.schema.json)

The extension adds the following property to the tileset:

| Property | Type | Description | Required |
|----------|------|-------------|----------|
| `uri` | `string` | URI reference to an external GeoJSON file containing extent definitions | Yes |

The referenced GeoJSON file must contain only Polygon or MultiPolygon geometries that define the extent regions.

## Defining Extents

An extent is a collection of longitude and latitude coordinate pairs. Extents are two-dimensional in nature, but an optional third component can be specified for **each** extent coordinate to specify its height (in meters) above the WGS84 ellipsoid.

See the [GeoJSON](https://tools.ietf.org/html/rfc7946) specification for coordinate details.

The GeoJSON file referenced by this extension must meet the following requirements:

- **Geometry Types**: Must contain only Polygon or MultiPolygon geometries
- **Coordinate Count**: At least three coordinates must be provided for an extent to be valid
- **No Self-Intersection**: Individual polygons and multipolygons must not self-intersect. A polygon self-intersects when its boundary crosses or touches itself at any point other than at vertices shared between consecutive edges. Valid polygons must be simple polygons _(Jordan curves)_, where "simple" means injective (one-to-one mapping) except at the start/end closure point. Self-intersections create ambiguous regions and points with multiple boundary orientations, making it impossible to consistently determine interior vs exterior, leading to undefined behavior in geometric algorithms
- **Overlapping**: Multiple polygons and multipolygons may overlap each other (treated as boolean union)
- **Coordinate System**: Must follow the GeoJSON specification (WGS84 longitude/latitude). Coordinate order is [longitude, latitude] or [longitude, latitude, height].
- **Bounding Volume Constraint**: All extent coordinates must be confined within the bounding volume of the dataset root tile - no coordinate may lie outside this bounding volume
- **File Location**: The extent definition must be in a separate file referenced by URI

## Examples

#### `tileset.json`

```json
{
  "asset": {
    "version": "1.1"
  },
  "extensionsUsed": [
    "MAXAR_extent"
  ],
  "extensions": {
    "MAXAR_extent": {
      "uri": "extent.geojson"
    }
  },
  "geometricError": 100.0,
  "root": {}
}
```

#### `extent.geojson`

```json
{
  "features": [
    {
      "geometry": {
        "coordinates": [
          [
            [-116.61512629247555, 32.511166344475825],
            [-116.61385130521423, 32.514456825587125],
            [-116.61829305963617, 32.515638616140173],
            [-116.61944030086889, 32.513154268152434]
          ]
        ],
        "type": "Polygon"
      },
      "properties": {},
      "type": "Feature"
    }
  ],
  "type": "FeatureCollection"
}
```