# patran_geom_mesh_assoc_export
This PCL script exports the geometry-to-mesh relationship from the MSC Patran database for a meshed solid model. It identifies solid, surface, and edge entities, retrieves their associated node IDs, and writes the geometry hierarchy and node associations into structured CSV files.


# pat_db_geom_mesh_assoc_export_v1

`pat_db_geom_mesh_assoc_export_v1.pcl` is a PCL utility for **MSC Patran** that exports the geometry-to-mesh association stored in the Patran database for a meshed solid body.

The script starts from a root solid such as `Solid 1`, traverses the geometry hierarchy as:

- solid
- surfaces of the solid
- edges of each surface

and writes the geometry hierarchy and associated node IDs into CSV files.

## Purpose

This tool is intended for geometry-aware finite element automation. Instead of assigning loads, SPCs, or other boundary conditions to unstable raw node IDs, users can reference geometric entities such as surfaces and edges and then map those entities to the correct node sets.

This is especially useful when generating modified `.bdf` files, because remeshing or geometry changes may alter node numbering and make direct node-based assignments unreliable.

## Output Files

The script generates two CSV files:

### 1. `geometry_hierarchy.csv`

Contains the geometry tree.

Columns:

- `main_geometry_id`
- `parent_geometry_id`
- `geometry_id`
- `geometry_type`
- `level`

Example:

```csv
main_geometry_id,parent_geometry_id,geometry_id,geometry_type,level
1,,1,SOLID,0
1,1,1.1,SURFACE,1
1,1.1,1.1.1,EDGE,2
```

### 2. `geometry_nodes.csv`

Contains the node association for each geometry entity.

Columns:

- `main_geometry_id`
- `parent_geometry_id`
- `geometry_id`
- `geometry_type`
- `level`
- `node_picklist`

Example:

```csv
main_geometry_id,parent_geometry_id,geometry_id,geometry_type,level,node_picklist
1,,1,SOLID,0,1:150
1,1,1.4,SURFACE,1,58:72 105 106 108:135
1,1.4,1.4.1,EDGE,2,72 105 106
```

## Geometry Naming

The exported geometry labels are normalized for easier downstream processing:

- `Solid 1` -> `1`
- `Solid 1.2` -> `1.2`
- `Solid 1.2.3` -> `1.2.3`

The corresponding entity class is stored separately in the `geometry_type` field as:

- `SOLID`
- `SURFACE`
- `EDGE`

## Requirements

- **MSC Patran**
- access to a Patran `.db` model
- a meshed solid geometry

This version expects the root input to be a solid geometry label such as:

```text
Solid 1
```

## Main Function

The main callable function is:

```text
export_geom_mesh_assoc_tree(root_geometry, hierarchy_csv_path, nodes_csv_path)
```

Example:

```text
export_geom_mesh_assoc_tree("Solid 1","C:/pclexports/geometry_hierarchy.csv","C:/pclexports/geometry_nodes.csv")
```

## How to Use

Load the PCL file in Patran:

```text
!!INPUT C:/pcl/pat_db_geom_mesh_assoc_export_v1.pcl
```

Then run:

```text
export_geom_mesh_assoc_tree("Solid 1","C:/pclexports/geometry_hierarchy.csv","C:/pclexports/geometry_nodes.csv")
```

## Notes

- This version is intended for **solid-root traversal**
- node associations are exported as **Patran-style picklists**
- the script preserves geometry hierarchy in a compact, machine-friendly format
- the exported data can be used to drive robust geometry-based load and constraint assignment workflows
