# VRICON_grid

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
- [Defining Grid](#defining-grid)
- [Examples](#examples)

## Overview

VRICON_grid describes the grid used for the dataset, and for example says if a quadtree grid was used. The idea is to use the knowledge of the grid to set better tile ids (used for the spatial page index), and also make it easier to fetch neighboring tiles (also including the parent, and children).

## Optional vs Required

This extension is optional, meaning it should be placed in the tileset JSON `extensionsUsed` list, but not in the `extensionsRequired` list.

## Defining Grid

For a given tile in the tree, there will be `extras.key` metadata that provides the quadtree index. The syntax for `extras.key` is `[face, [lev, x, y]]` and `[lev, x, y]`. The nested array syntax is in order to possibly be able to support `[lev,x,y,z]` later on, and tell it apart from the face variant. Note that face doesn't necessarily mean an S2 face, it can be an int to describe an N-headed grid.

There is also an extra bounding box defined in `extras.vricon.bbox` and is defined by `[xmin, ymin, xmax, ymax, zmin, zmax]`.

There is also an optional texture resolution tag in `extras.vricon.mpp` which is expressed as meters per pixel for the main texture, this can be seen as an alternative metric to geometricError used for each tile. When a `mpp` tag is missing
the reader should fallback to using `mpp = geometricError / 8` if a resolution value is required.

###  Types

This extension defines three spatial types:

* `quad`: Subdivision using quadtree specified with `center` and `size` as well the `pcs` (Projected Coordinate Systems).
* `s2`: Subdivision using S2 bounding volumes
* `geod`: Geodetic

## Quad grid
A quad grid specifies a common quad tree grid which is defined by its center, size and projection. 
The projection is defined using a string with the following syntax: `REFSYS[EPOCH]:PROJ,VERTCS`

 |*REFSYS*| 
 |----|
 | ITRF2008 |

 *EPOC* are expressed as a floating point number representing the decimal year of the epoch.

 |*PROJ*| Description | 
 |------|------|
 | GEOD | Geodetic projection (lat/long) |
 | UTMxxN | UTM North, example: UTM30N |
 | UTMxxS | UTM South, example: UTM30S |
 | S2Fn | S2 projection for a given face, example S2F2 |

 |*VERTCS*| Description |
 |----|----|
 | ELLIPSOID | |
 | EGM2008 | |

The keys defined in `extras.key` has the form `[lev,x,y]`.

Example:
```json
"VRICON_grid": {
  "type": "quad",
  "center": [ 3097202.3706942615, 500000.00000000122 ],
  "size": [ 2088960.0, 2088960.0 ],
  "pcs": "ITRF2008[2005.0]:UTM14N,ELLIPSOID"
}
```


## S2 grid
Consists of up to six root nodes, one for each face in the S2 cube. A fake root node is inserted in the top tileset.json in order to carry the root nodes.
The keys defined in `extras.key` has the form `[face,[lev,x,y]]`.

```json
"VRICON_grid": {
  "type": "s2"
}
```

## Geodetic grid
Consists of up to two root nodes, one for the west- and one for the east hemisphere. A fake root node is inserted in the top tileset.json in order to carry the root nodes.
The keys defined in `extras.key` has the form `[face,[lev,x,y]]` where face specifies the hemisphere (0 = west, 1 = east).

```json
"VRICON_grid": {
  "type": "geod"
}
```
