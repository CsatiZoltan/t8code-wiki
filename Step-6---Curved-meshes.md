## Step 5 - Curved meshes

`t8code` has the functionality to read in a mesh file along with the geometry file used to generate the mesh. With this parallel read in, we can generate curved adaptive meshes, as we call them. The curved adaptive meshes then are refined along the original geometry. This is shown in the following figure.

### Requirements

1. `t8code` has to be linked to [`OpenCASCADE`](https://dev.opencascade.org/doc/overview/html/index.html). This is done via the configure option `--with-occ`.
To use this feature, we have to fulfill some requirements:
2. The used mesh has to be written in the [`Gmsh`'s `.msh` 4.X format](https://gmsh.info/doc/texinfo/gmsh.html#MSH-file-format). Moreover, the parametric export option from Gmsh has to be enabled.
3. The geometry file has to be written in [`OpenCASCADE`'s `.brep` format](https://dev.opencascade.org/doc/overview/html/specification__brep_format.html).
4. The mesh has to be hex-only.