<!-- omit in toc -->
# MAXAR_grid
<!-- omit in toc -->
## Contributors

* Erik Dahlström, [@erikdahlstrom](https://github.com/erikdahlstrom)
* Johan Borg, [@jo-borg](https://github.com/jo-borg)
* Björn Blissing, [@bjornblissing](https://github.com/bjornblissing)

<!-- omit in toc -->
## Status

Draft - **Version 0.0.1**, March 30, 2023

<!-- omit in toc -->
## Dependencies

Written against the 3D Tiles 1.1 spec.


## Contents

- [Overview](#overview)
- [Optional vs Required](#optional-vs-required)
- [Defining Grid](#defining-grid)
- [Examples](#examples)

## Overview

MAXAR_grid describes the grid used for the dataset, and for example says if a quadtree grid was used. The idea is to use the knowledge of the grid to set better tile ids (used for the spatial page index), and also make it easier to fetch neighboring tiles (also including the parent, and children).

## Optional vs Required

This extension is optional, meaning it should be placed in the tileset JSON `extensionsUsed` list, but not in the `extensionsRequired` list.

## Defining the Grid

For a given tile in the tree, there will be extension that provides the tree index. 

### Types

This extension defines three spatial types using `tileset.MAXAR_grid.type`:

* `quad` : Subdivision using generic quadtree specified with `center` and `size` as well the `srs` (Spatial Reference System).
* `s2`   : Subdivision using S2 bounding volumes
* `geod` : Subdivision using geodetic quadtree

By default, both geodetic grids and S2 grids utilize ITRF2008 with epoch 2005.0 as their reference system. However, for quad grids, the reference system must be explicitly specified.

#### Quad grid

A quad grid specifies a common quad tree grid which is defined by its center, size and spatial reference system. 
(`tileset.MAXAR_grid.center`, `tileset.MAXAR_grid.size` and `tileset.MAXAR_grid.srs`).

## Tile information

To ensure proper indexing and mapping, each tile must provide complete specifications of its full index and bounding box. Additionally, it is recommended to include the texture resolution. In the case of vector datasets, the texture resolution can be utilized to depict the resolution of the source raster that was utilized in generating the vectors.

### Index 

The syntax for specifying the full index include `tile.MAXAR_grid.face` (for S2 tiles and Geodetic tiles), `tile.MAXAR_grid.level`, and `tile.MAXAR_grid.index`. Note that the `tile.MAXAR_grid.index` refers to the grid coordinate for a specific level.

### Bounding box

There is also an extra bounding box defined in `tile.MAXAR_grid.boundingBox` is defined by `[xmin, ymin, xmax, ymax, zmin, zmax]`.

### Texture resolution

There is also an optional texture resolution tag in `tile.MAXAR_grid.metersPerPixel` which is expressed as meters per pixel for the main texture, this can be seen as an alternative metric to geometricError used for each tile. 

## Example:

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

### Geodetic grid

Consists of up to two root nodes, one for the west- and one for the east hemisphere. A fake root node is inserted in the top `tileset.json` in order to carry the root nodes.
The keys defined in `tile.MAXAR_grid.face`, `tile.MAXAR_grid.level` and `tile.MAXAR_grid.index`,  where face specifies the hemisphere (0 = west, 1 = east).


In the `tileset.extensions` node:

```json
"MAXAR_grid": {
  "type": "geod"
}
```


## Schema

* [tileset.MAXAR_grid.schema.json](schema/tileset.MAXAR_grid.schema.json).
* [tile.MAXAR_grid.schema.json](schema/tile.MAXAR_grid.schema.json).
* [srs.schema.json](schema/srs.schema.json).
