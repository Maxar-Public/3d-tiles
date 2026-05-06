<!-- omit in toc -->
# MAXAR_grid
<!-- omit in toc -->
## Contributors

* Erik Dahlström, Maxar, [@erikdahlstrom](https://github.com/erikdahlstrom)
* Johan Borg, Maxar, [@jo-borg](https://github.com/jo-borg)
* Björn Blissing, Maxar, [@bjornblissing](https://github.com/bjornblissing)

<!-- omit in toc -->
## Status

**Version 1.0.0**, April 27, 2026

<!-- omit in toc -->
## Dependencies

Written against the 3D Tiles 1.1 spec.

The key words MUST, MUST NOT, SHOULD, SHOULD NOT, and MAY in this document are to be interpreted as described in RFC 2119.

## Contents

- [Overview](#overview)
- [Optional vs. Required](#optional-vs-required)
- [Schema](#schema)
- [Grid Types](#grid-types)
- [Tileset Properties](#tileset-properties)
- [Tile Properties](#tile-properties)
- [Examples](#examples)
- [Implementation Notes](#implementation-notes)
- [Limitations](#limitations)

## Overview

The MAXAR_grid extension describes the spatial indexing grid used for organizing and accessing 3D Tiles datasets. This extension enables efficient spatial queries, tile fetching, and navigation by providing explicit grid structure information including tile coordinates, levels, and spatial relationships.

The extension supports three grid types:
- **Quad grids**: Generic quadtree subdivision with custom spatial reference systems
- **S2 grids**: Google S2 spherical geometry-based subdivision
- **Geodetic grids**: Earth-based geodetic quadtree subdivision

By exposing grid structure, clients can implement optimized spatial indexing, predictable tile IDs for caching, and efficient neighbor tile discovery for streaming and prefetching.

## Optional vs. Required

This extension is optional, meaning it should be placed in the tileset JSON `extensionsUsed` list, but not in the `extensionsRequired` list.

## Schema

This extension modifies the 3D Tiles schema by adding properties to both tileset and tile levels:

- **Tileset Schema**: [`schema/tileset.MAXAR_grid.schema.json`](schema/tileset.MAXAR_grid.schema.json)
- **Tile Schema**: [`schema/tile.MAXAR_grid.schema.json`](schema/tile.MAXAR_grid.schema.json)
- **SRS Schema**: [`schema/srs.MAXAR_grid.schema.json`](schema/srs.MAXAR_grid.schema.json)

### Tileset Properties

| Property | Type | Description | Required |
|----------|------|-------------|----------|
| `type` | `string` | Grid type: `"quad"`, `"s2"`, or `"geod"` | Yes |
| `center` | `number[2]` | Center coordinates (quad grids only) | For quad |
| `size` | `number[2]` | Grid size dimensions (quad grids only) | For quad |
| `srs` | `object` | Spatial reference system (quad grids only) | For quad |

### Tile Properties

| Property | Type | Description | Required |
|----------|------|-------------|----------|
| `boundingBox` | `number[6]` | Tile bounding box `[xmin, ymin, xmax, ymax, zmin, zmax]` | Yes |
| `level` | `integer` | Grid level/depth of the tile | Yes |
| `index` | `integer[2]` | Grid coordinates `[x, y]` at the specified level | Yes |
| `face` | `integer` | Face identifier (S2: 0-5, Geodetic: 0-1) | For S2/Geodetic |
| `metersPerPixel` | `number` | Texture resolution in meters per pixel | No |

## Grid Types

This extension defines three spatial grid types for organizing tile hierarchies:

### Quad Grid (`"quad"`)

A generic quadtree subdivision system that allows custom spatial reference systems. Quad grids are defined by:

- **Center**: The center point of the root grid cell in the specified coordinate system
- **Size**: The width and height dimensions of the root grid cell
- **SRS**: Complete spatial reference system specification including coordinate system, datum, and elevation model

Quad grids provide maximum flexibility for datasets using projected coordinate systems like UTM, state plane, or custom projections.

### S2 Grid (`"s2"`)

Uses Google's S2 spherical geometry system for global coverage. S2 grids:

- **Face-based**: Utilizes 6 cube faces (0-5) projected onto a sphere
- **Hierarchical**: Each cell subdivides into 4 children at the next level
- **Global**: Provides seamless worldwide coverage without projection distortions
- **Reference frame**: ITRF2008 with epoch 2005.0 (implicit; the tileset extension does not carry an `srs` object for this grid type)

S2 grids are ideal for global datasets and applications requiring consistent angular resolution.

### Geodetic Grid (`"geod"`)

Earth-based geodetic quadtree using longitude/latitude subdivision:

- **Hemisphere-based**: Uses 2 root faces (0=west, 1=east hemisphere)
- **Geographic**: Direct longitude/latitude coordinate subdivision
- **Hierarchical**: Standard quadtree subdivision pattern
- **Reference frame**: ITRF2008 with epoch 2005.0 (implicit; the tileset extension does not carry an `srs` object for this grid type)

Geodetic grids work well for regional datasets and applications using geographic coordinates.

## Tileset Properties

The tileset-level extension properties define the overall grid structure and coordinate system:

### Grid Type (`type`)
Specifies which grid system is used: `"quad"`, `"s2"`, or `"geod"`.

### Quad Grid Properties
For quad grids, additional properties define the root grid cell:

- **`center`**: `[x, y]` coordinates of the grid center in the specified SRS
- **`size`**: `[width, height]` dimensions of the root grid cell
- **`srs`**: Spatial reference system object defining coordinate system, datum, epoch, and elevation model

#### SRS Object

The `srs` object is required for quad grids and describes the spatial reference system in which `center`, `size`, and tile `boundingBox` values are expressed. Its structure is defined by [`schema/srs.MAXAR_grid.schema.json`](schema/srs.MAXAR_grid.schema.json).

| Property | Type | Description | Required |
|----------|------|-------------|----------|
| `coordinateSystem` | `string` | Coordinate system identifier. One of: `"GEOD"`, `"ECEF"`, `"UTM<zz>N"`/`"UTM<zz>S"` (zones 01–60), or `"S2F<n>"` (faces 0–5). | Yes |
| `elevation` | `string` | Vertical datum: `"ELLIPSOID"` for ellipsoidal heights or `"EGM2008"` for geoid heights. | Yes |
| `referenceSystem` | `string` | Geodetic reference frame: `"ITRF2008"` or `"WGS84-G1762"`. | Conditional (see below) |
| `epoch` | `string` | Coordinate epoch as a decimal year between `1000` and `2999`, e.g. `"2005.0"`, `"2023.5"`. | Conditional (see below) |
| `axis` | `string` | Horizontal axis orientation: `"NED"` (North-East-Down) or `"ENU"` (East-North-Up). Default `"NED"`. | No |
| `unitHorizontal` | `string` | Unit for horizontal coordinates: `"METER"` or `"DEGREE"`. Default `"METER"`. | No |
| `unitVertical` | `string` | Unit for vertical coordinates: `"METER"`. Default `"METER"`. | No |

When `coordinateSystem` is `"GEOD"` or any `"S2F<n>"` value, `referenceSystem` defaults to `"ITRF2008"` and `epoch` defaults to `"2005.0"`; if specified, they MUST match those values. For all other coordinate systems (`"ECEF"`, `"UTM<zz>N"`, `"UTM<zz>S"`), `referenceSystem` and `epoch` are required and have no defaults.

`unitHorizontal` is normally `"DEGREE"` only when `coordinateSystem` is `"GEOD"`; for projected coordinate systems (`"UTM*"`, `"S2F*"`, `"ECEF"`) it SHOULD be `"METER"`.

Note: this rule applies only to the quad grid's `srs` object. The `"s2"` and `"geod"` grid types do not carry an `srs` object at all (see the corresponding entries in [Grid Types](#grid-types)).

### S2 and Geodetic Grid Properties
S2 and geodetic grids use predefined coordinate systems and do not require additional tileset properties.

## Tile Properties

Each tile in the hierarchy includes grid-specific properties for spatial indexing and navigation:

### Required Properties

#### Bounding Box (`boundingBox`)
Six-element array defining the tile's spatial extent: `[xmin, ymin, xmax, ymax, zmin, zmax]`. Coordinates are in the grid's coordinate system.

#### Level (`level`)
Integer indicating the tile's depth in the grid hierarchy. Root tiles are at level 0, with each subdivision increasing the level by 1.

#### Index (`index`)
Two-element array `[x, y]` specifying the tile's grid coordinates at its level. Coordinates start at `[0, 0]` for each level.

### Conditional Properties

#### Face (`face`)
Required for S2 and geodetic grids:
- **S2 grids**: Integer 0-5 identifying the cube face
- **Geodetic grids**: Integer 0-1 identifying hemisphere (0=west, 1=east)

This requirement is normative, but it cannot be enforced by `tile.MAXAR_grid.schema.json` alone because the grid type is defined at the tileset level. Implementations MUST validate the presence of `face` based on the tileset's `type`.

### Optional Properties

#### Meters Per Pixel (`metersPerPixel`)
Texture resolution expressed as meters per pixel. For vector datasets, this represents the resolution of source raster data used to generate the vectors.

To ensure proper indexing and mapping, each tile MUST provide complete specifications of its full index and bounding box. Additionally, the texture resolution SHOULD be included. In the case of vector datasets, the texture resolution MAY be used to depict the resolution of the source raster that was utilized in generating the vectors.

### Index 

The syntax for specifying the full index include `tile.MAXAR_grid.face` (for S2 tiles and Geodetic tiles), `tile.MAXAR_grid.level`, and `tile.MAXAR_grid.index`. Note that the `tile.MAXAR_grid.index` refers to the grid coordinate for a specific level.

### Bounding box

There is also an extra bounding box defined in `tile.MAXAR_grid.boundingBox` is defined by `[xmin, ymin, xmax, ymax, zmin, zmax]`.

### Texture resolution

There is also an optional texture resolution tag in `tile.MAXAR_grid.metersPerPixel` which is expressed as meters per pixel for the main texture, this can be seen as an alternative metric to geometricError used for each tile.

## Examples

_This section is non-normative_

### Quad grid

In the `tileset.extensions` node:

```json
"MAXAR_grid": {
  "type": "quad",
  "center": [3097202.3706942615, 500000.00000000122],
  "size": [2088960.0, 2088960.0],
  "srs": {
    "referenceSystem": "ITRF2008",
    "epoch": "2005.0",
    "coordinateSystem": "UTM14N",
    "elevation": "ELLIPSOID"
  }
}

```

In the `tile.extensions`:

```json
"MAXAR_grid": {
  "boundingBox": [
    3425906.396465421,
    500000.0000000012,
    3546738.395635767,
    694559.9877318252,
    -581.1745009114966,
    445.7444586344063
  ],
  "index": [
    5,
    4
  ],
  "level": 3
}
```

### S2 grid

Consists of up to six root nodes, one for each face in the S2 cube. A fake root node is inserted in the top `tileset.json` in order to carry the root nodes.
The keys defined in `tile.MAXAR_grid.face`, `tile.MAXAR_grid.level` and `tile.MAXAR_grid.index`.

In the `tileset.extensions` node:

```json
"MAXAR_grid": {
  "type": "s2"
}
```

In the `tile.extensions`:

```json
"MAXAR_grid": {
  "boundingBox": [
    -1.7095238193613294,
    0.54322210241748092,
    -1.7084717202817712,
    0.54416527589988806,
    243.45309697370976,
    360.09079237561673
  ],
  "face": 3,
  "level": 5,
  "index": [12, 8],
  "metersPerPixel": 2.5
}
```

### Geodetic grid

Consists of up to two root nodes, one for the west- and one for the east hemisphere. A fake root node is inserted in the top `tileset.json` in order to carry the root nodes.
The keys defined in `tile.MAXAR_grid.face`, `tile.MAXAR_grid.level` and `tile.MAXAR_grid.index`,  where face specifies the hemisphere (0 = west, 1 = east).


In the `tileset.extensions` node:

```json
"MAXAR_grid": {
  "type": "geod"
}
```

In the `tile.extensions`:

```json
"MAXAR_grid": {
  "boundingBox": [
    -180.0,
    -85.0,
    0.0,
    85.0,
    -1000.0,
    9000.0
  ],
  "face": 0,
  "level": 2,
  "index": [1, 2],
  "metersPerPixel": 1000.0
}
```


## Implementation Notes

### Performance Considerations
- **Spatial Indexing**: Grid information enables efficient spatial queries and tile discovery
- **Caching**: Predictable tile IDs based on grid coordinates improve cache hit rates
- **Prefetching**: Grid structure allows clients to predict and prefetch neighboring tiles
- **Level-of-Detail**: Grid levels provide natural LOD progression for streaming

### Coordinate System Considerations
- **ITRF2008**: Default for S2 and geodetic grids, provides high-precision global reference
- **Custom SRS**: Quad grids support any coordinate system via the SRS object
- **Elevation Models**: Support for both ellipsoidal heights and geoid-based elevations
- **Epoch**: Coordinate epoch specification handles tectonic plate motion

## Limitations

This extension has the following limitations:

- **Grid Consistency**: All tiles in a dataset MUST use the same grid type and parameters
- **Bounding Volume Relationship**: Grid bounding boxes SHOULD align with 3D Tiles bounding volumes
- **Face Constraints**: S2 face values MUST be 0-5, geodetic face values MUST be 0-1
- **Level Progression**: Grid levels SHOULD follow standard quadtree subdivision (4:1 ratio)
- **Coordinate Precision**: Grid coordinates are limited by floating-point precision
