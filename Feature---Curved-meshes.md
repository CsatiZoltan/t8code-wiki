## Feature - Curved meshes

`t8code` has the functionality to read in a mesh file along with the geometry file used to generate the mesh. With this parallel read in, we can generate curved adaptive meshes, as we call them. The curved adaptive meshes then are refined along the original geometry. This is shown in the following figure.

### Requirements

To use this feature, we have to fulfill some requirements:  
1. `t8code` has to be linked to [`OpenCASCADE`](https://dev.opencascade.org/doc/overview/html/index.html). This is done via the configure option `--with-occ`.  
2. The used mesh has to be written in the [`Gmsh`'s `.msh` 4.X format](https://gmsh.info/doc/texinfo/gmsh.html#MSH-file-format). Moreover, the parametric export option from `Gmsh` has to be enabled.  
3. The geometry file has to be written in [`OpenCASCADE`'s `.brep` format](https://dev.opencascade.org/doc/overview/html/specification__brep_format.html).  
4. The mesh has to be hex-only.  

### Generating a geometry

Generally, we can use every Software, which supports the `.brep` file format: [`FreeCAD`](https://www.freecadweb.org/), [`Gmsh`](https://gmsh.info), [`TiGL`](https://dlr-sc.github.io/tigl/) (all based on `OpenCASCADE`) to name a few. But we can also use `OpenCASCADE` itself.
  
Note: _If you are using `Gmsh` for the generation of geometries, make sure to export the geometry into a `.brep` file and import it again before meshing. `Gmsh` has an internal geometry indexing system based on the order of geometry generation. In the resulting `.brep` file, the geometries have different indices than in `Gmsh` itself. This can result in different `.msh` files in which the mesh nodes are attributed to the wrong geometries. By reloading the `.brep` file into `Gmsh`, `Gmsh` adapts the indexing of the `.brep` file._  

Henceforth, we are going to generate a geometry and mesh with `Gmsh`. For this, we use the scripting language `.geo` from `Gmsh`. If we take a look at the `t8_features_curved_meshes_generate_cmesh.geo` file in the tutorials directory, we can see how the geometry and mesh are created.  
After the two-dimensional definition of the NACA geometry, it is extruded and meshed hex-only. Note, that the `.brep` file is reloaded before the meshing, as discussed before.
