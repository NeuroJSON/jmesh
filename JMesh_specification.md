JMesh - A versatile data format for unstructured meshes and geometries
============================================================

- **Status of this document**: This document is current under development.
- **Copyright**: (C) Qianqian Fang (2011, 2019) <q.fang at neu.edu>
- **License**: Apache License, Version 2.0
- **Version**: 1 (Draft 1)
- **Abstract**:

> JMesh is a portable and extensible file format for the storage and interchange of
unstructured geometric data, including discretized geometries such as triangular 
and tetrahedral meshes, parametric geometries such as NURBS curves and surfaces, and
constructive geometries such as constructive solid geometry (CGS) of shape primitives
and meshes. Built upon the JData specification, a JMesh file utilizes the JavaScript 
Object Notation (JSON) [RFC4627] and Universal Binary JSON (UBJSON) constructs to 
serialize and encode geometric data structures, therefore, it can be directly 
processed by most existing JSON and UBJSON parsers. In this specification, we define 
a list of JSON-compatible constructs to encode geometric data, including N-dimensional 
vertices, curves, surfaces, solid elements, shape primitives, their interactions and spatial
relations, together with their associated properties, such as numerical values, 
colors, normals, materials, textures and other properties related to graphics data
manipulation, 3-D fabrication and computer graphics rendering and animations.


## Table of Content

- [Introduction](#introduction)
  * [Background](#background)
  * [JMesh specification overview](#jmesh-specification-overview)
- [Grammar](#grammar)
  * [Array form](#array-form)
  * [Structure form](#structure-form)
- [Mesh Data Keywords](#mesh-data-keywords)
  * [Discrete and parametric graphics](#discrete-and-parametric-graphics)
    + [Vertices](#vertices)
      - [MeshVertex1](#meshvertex1)
      - [MeshVertex2](#meshvertex2)
      - [MeshVertex3](#meshvertex3)
      - [MeshVertex4](#meshvertex4)
    + [Line segments and curves](#line-segments-and-curves)
      - [MeshLine](#meshline)
      - [MeshEdge](#meshedge)
      - [MeshBSpline2D](#meshbspline2d)
    + [Surfaces](#surfaces)
      - [MeshTri3](#meshtri3)
      - [MeshQuad4](#meshquad4)
      - [MeshPLC](#meshplc)
      - [MeshNURBS](#meshnurbs)
    + [Solid Elements](#solid-elements)
      - [MeshTet4](#meshtet4)
      - [MeshHex8](#meshhex8)
      - [MeshPyramid5](#meshpyramid5)
      - [MeshTet10](#meshtet10)
    + [Flexible mesh data containers](#flexible-mesh-data-containers)
      - [MeshNode](#meshnode)
      - [MeshSurf](#meshsurf)
      - [MeshPoly](#meshpoly)
      - [MeshElem](#meshelem)
  * [Constructive graphics and grouping](#constructive-graphics-and-grouping)
    + [Mesh grouping and partitioning](#mesh-grouping-and-partitioning)
    + [Constructive solid graphics (CSG)](#constructive-solid-graphics-csg)
  * [Textures](#textures)
    + [Texture1D](#texture1d)
    + [Texture2D](#texture2d)
    + [Texture3D](#texture3d)
  * [Shape primitives](#shape-primitives)
    + [2-D shapes](#2-d-shapes)
      - [ShapeBox2](#shapebox2)
      - [ShapeDisc2](#shapedisc2)
      - [ShapeEllipse](#shapeellipse)
      - [ShapeLine2](#shapeline2)
      - [ShapeArrow2](#shapearrow2)
      - [ShapeAnnulus](#shapeannulus)
      - [ShapeGrid2](#shapegrid2)
    + [3-D shapes](#3-d-shapes)
      - [ShapeLine3](#shapeline3)
      - [ShapePlane3](#shapeplane3)
      - [ShapeBox3](#shapebox3)
      - [ShapeDisc3](#shapedisc3)
      - [ShapeGrid3](#shapegrid3)
      - [ShapeSphere](#shapesphere)
      - [ShapeCylinder](#shapecylinder)
      - [ShapeEllipsoid](#shapeellipsoid)
      - [ShapeTorus](#shapetorus)
      - [ShapeCone](#shapecone)
      - [ShapeConeFrustum](#shapeconefrustum)
      - [ShapeSphereShell](#shapesphereshell)
      - [ShapeSphereSegment](#shapespheresegment)
    + [Shapes from transformation](#shapes-from-transformation)
      - [ShapeExtrude2D](#shapeextrude2d)
      - [ShapeExtrude3D](#shapeextrude3d)
      - [ShapeRevolve3D](#shaperevolve3d)
- [Recommended File Specifiers](#recommended-file-specifiers)
- [Summary](#summary)


Introduction
------------

### Background

Efficiently representing, displaying and processing 3-dimensional (3-D)
graphical objects, such as shape primitives (spheres, boxes, cylinders, etc),
triangular surfaces or tetrahedral meshes play a crucial role in today's
computer graphics and scientific research. Accurate and efficient 
descriptions of 3-D shapes are also the foundation for today's 
computer-aided-design (CAD), 3-D printing and digital fabrications.

A mesh typically represents a discretized geometry using groups of nodes, 
edges, surface patches and solid elements. Compared to rasterized 
arrays - also known as the "structured data", mesh and shape descriptors 
are often categorized under the "unstructured data" due to the irregular 
data dimensions and non-coalesced index-mapped data. Efficient storage 
of such unstructured and index-mapped data has been a challenge, resulting 
in a [large set of incompatible](https://en.wikipedia.org/wiki/List_of_file_formats#3D_graphics) 
file formats that are largely application specific and difficult 
to standardize.

For example, the widely used STL format for 3-D printing supports only 
triangular surfaces, and is not able to encode other shape constructs such as
tetrahedral or hexahedral solid objects. Similarly, the PLY (Polygon File 
Format), OFF (Object File Format) and the Wavefront OBJ file formats are 
also largely limited to triangular surfaces, with various degrees of 
metadata support such as triangle colors and node/surface normals. While 
parsing and interchanging these simple mesh data formats are 
straightforward and easy to implement, the limited functionalities and
lack of the ability to extend for more complex shapes and metadata makes
it difficult to capture other more sophisticated geometries. On the other 
hand, more capable and flexible shape storage formats are often associated 
with commercial 3-D CAD software, such as 3DS, FBX, and DWG formats, or with 
relatively large and complex parsers, which are often not widely available.

Due to the great diversity of geometric discretization schemes, the form 
and organization of mesh data may vary greatly from one application to 
another. An open protocol to store, transmit, modify and transform mesh-like 
data structures is of great importance to the simplification and 
implementation of applications that access mesh-based data.

Over the past few years, JavaScript Object Notation (JSON) became widely 
accepted among the Internet community due to its capability of storing 
complex data, excellent portability and human-readability. The proposals
and wide adoptions of binary JSON-like format, such as UBJSON and
MessagePack, also add complementary features such as support for typed 
data, smaller file sizes and faster processing speed. The JData specification
provides the foundation for serializing complex hierarchical data using
JSON/UBJSON constructs. This permits us to define language- and library-neutral
geometry and mesh representations using the simple and extensible constructs
from JSON and UBJSON syntax. 


### JMesh specification overview

JMesh is an extensible framework to store complex geometry and mesh related 
data using the JData representation, with a syntax compatible to the widely 
used JSON and UBJSON file formats. JMesh adds a semantic layer to 
a JSON structure by attaching meanings and format specifications to a list of 
reserved `"name"` fields (or simply, JMesh keywords). These JMesh keywords allow
one to unambiguously store vertices, edges, surfaces and tetrahedral meshes,
shape and parametric objects of 1-D, 2-D, 3-D or even higher dimensions.

The purpose of this document is to define the text and binary JMesh format 
specifications. This is achieved through the definition of a semantic layer 
over the JSON/UBJSON data storage syntax to map various types of mesh and
geometrical data. Such semantic layer includes

- a list of dedicated `"name"` fields, or keywords, that define the containers 
  of various nodes, elements, and shape primitives
- a list of dedicated `"name"` fields and formats to facilitate the spatial
  organization, grouping and interaction of geometric objects, 
  and timelines for dynamic shapes and animations
- a list of metadata keywords for enriched data annotation, including colors,
  materials, textures and other user-defined auxiliary data

In the following sections, we will clarify the basic JMesh grammar and define 
geometric data container formats to encode a wide-range of unstructured mesh and
shape data types, including vertices, lines, contours, shape primitives, 
triangular surfaces, tetrahedral meshes, and other surface and solid elements
In addition, we will define a set of structures to represent the interactions
and relationships between shape constructs, such as Boolean operations and 
grouping.
 

Grammar
------------------------

All JMesh files are JData specification compliant. The same as JData, it has
both a text format based on JSON serialization scheme and a binary format
based on the UBJSON serialization scheme. 

Briefly, the text-format JMesh is a valid JSON file with the extension to 
support concatenated JSON objects; the binary-format JMesh is a valid UBJSON 
with the extended syntax to support N-D array. Please refer to the JData 
specification for the definitions.

Nearly all supported mesh data containers (i.e. named JData nodes) can be defined 
using one of the two forms: an N-D array or a structure. For each JMesh data
keyword defined the following sections, we define the requirements by specifying 
the data dimensions if one chooses the array form, or define a list of required or
optional subfield names if one chooses the structure form.

### Array form

For simple data, one can use the "array form" to store the data under a JMesh 
keyword. In such case, the format of the data must follow the "N-Dimensional 
Array Storage Keyword" rules defined in the JData specification. For example, 
one can store a 1-D or 2-D array using the direct storage format as
```
 "jmesh_container_1d": [v1,v2,...,vn], 
 "jmesh_container_2d": [
    [v11,v12,...,v1n],
    [v21,v22,...,v2n],
    ...
    [vm1,vm2,...,vmn]
  ]
```
or using the "annotated storage" format as
```
 "jmesh_container_nd": {
       "_ArrayType_": "typename",
       "_ArraySize_": [N1,N2,N3,...],
       "_ArrayData_": [v1,v2,v3,...]
  }
```
Please note that the arrays that can be represented by the direct storage 
format is a subset of those supported by the "annotated format", as the 
latter provides additional features, such as defining data types or 
supporting sparse or complex-valued matrices. In the below sections, 
we use mostly the direct storage form to explain the data format, but 
one shall also store the data using the annotated format.

### Structure form

One can also use a JData structure to store the primary data as well as to support 
additional metadata associated with the container. For example, a structure-based
container may have the below subfields:
```
 "jmesh_container_struct": {
      "_DataInfo_":{
          ...
      },
      "Data":[
          ...
      ],
      "Properties": [
          ...
      ]
 }
```
Here, only the `"Data"` subfield is required, and it must have the same data 
stored in the "array form" (either in direct or annotated format) as shown above. 

The optional `"Properties"` subfield allows one to store additional data with this 
shape/mesh element. The `"Properties"` subfield can be an array or structure with 
additional subfields.

The optional `"_DataInfo_"` is the JData construct for storing metadata associated 
with this structure. It can be used to store simple metadata, such as data acquisition
date, operator name, or version number. The strategies how to split the metadata between
`_DataInfo_` and `Properties` is user-depedent.


Mesh Data Keywords
------------------------
In this section, we define dedicated JSON `"name"` keywords that can be used to 
define discretized shapes objects (or meshes), parametric graphics, or constructive
geometries.

Most of the data container keywords associated with discretized geometries have a
prefix of `"Mesh"`; keywords associated with parametric shape constructs have a prefix
of `"Shape"`; keywords associated with constructive solid geometries have a prefix `"CSG"`. 
Many of the keywords ends with a numerical value which typically
represents the column number of the data when stored in the array format.

All indices - integers mapping between data entries - shall have a start value of 1.

In the below section, we use notation "`N_i, N_ij` or `N_ijk` (`i=1,2,...`, `j=1,2,...`, 
`k=1,2,...`) to denote node indices or array dimensions; `x_i, y_i, z_i` or 
`x_ij, y_ij, z_ij`  to denote coordinates, `w_i` to denote additional coordinate
(in 4-D) or scaler values (such as weights), `v_i` or `v_ij` to denote numerical values
associated with the geometric object; `t_i`, `t_ij` or `t_ijk` to represent texture data in
various dimensions.

Below is a short summary of the JMesh data annotation/storage keywords to be introduced

* **Mesh grouping**: `MeshGroup`, `MeshObject`, `MeshPart`
* **Vertices**: `MeshVertex1`,`MeshVertex2`,`MeshVertex3`,`MeshVertex4`
* **Lines**: `MeshLine`,`MeshEdge`,`MeshBSpline2D`
* **Surfaces**: `MeshTri3`,`MeshQuad4`,`MeshPLC`,`MeshNURBS`
* **Solid**: `MeshTet4`,`MeshHex8`,`MeshPyramid5`,`MeshTet10`
* **Flexible containers**: `MeshNode`,`MeshSurf`,`MeshPoly`,`MeshElem`
* **CSG operators**: `CSGObject`,`CSGUnion`,`CSGIntersect`,`CSGSubtract`
* **Texture**: `Texture1D`,`Texture2D`,`Texture3D`
* **2-D shape primitives**: `ShapeBox2`,`ShapeDisc2`,`ShapeEllipse`,`ShapeLine2`,
  `ShapeArrow2`,`ShapeAnnulus`,`ShapeGrid2`
* **3-D shape primitives**: `ShapeLine3`,`ShapePlane3`,`ShapeBox3`,`ShapeDisc3`,
  `ShapeGrid3`,`ShapeSphere`,`ShapeCylinder`,`ShapeEllipsoid`,`ShapeTorus`,`ShapeCone`,
  `ShapeConeFrustum`,`ShapeSphereShell`,`ShapeSphereSegment`
* **Extrusion and revolving**: `ShapeExtrude2D`,`ShapeExtrude3D`,`ShapeRevolve3D`
* **Properties**: `Color`,`Normal`,`Size`,`Tag`,`Value`,`Texture`

### Common geometry properties

If a JMesh container is a structure, it can contain an optional element named `"Properties"`.
In this section, we define a set of common properties and their formats that are shared among
many JMesh keywords. 

#### Normal
The `Normal` property defines the normal vector(s) or orientation of a line or surface
object. It can take one of 3 values

* if the value is a single scalar, a value of 1 indicates the object has a clock-wise node order; 
  a value of -1 indicates a counter-clock-wise node order
* if the value is a single row-vector, it defines a common normal vector assuming the object 
  is a planar geometry
* if the value is an N-by-2 (for 2-D geometries) or N-by-3 (for 3-D geometries) matrix, each
  row defines a normal vector for each line-segment or surface patch; the row number `N` must 
  match the line-segment or face patch count in the parent object.

#### Color
The `Color` property defines the color of the entire object or the at each entry (a vertex, or a 
surface patch, or a solid element) of the parent object.

It can take one of 2 values

* if the value is a single row vector
  * if the vector contains a single scalar, it defines a gray-scale value
  * if the vector contains 3 scalars, it defines a color in the RGB (red-green-blue) format
  * if the vector contains 4 scalars, it defines a color in the RGBA (red-green-blue-alpha) format
  * if the vector contains N>4 scalars, it defines the gray-scale values at each entry of the parent object
* if the value is an N-by-3 or N-by-4 array, it defines the colors, in RGB or RGBA format, respectively, 
  at each entry of the parent object

#### Tag
The `Tag` property defines a label for the entire object or the at each entry (a vertex, or a 
surface patch, or a solid element) of the parent object.

It can take one of 3 values
* if it is a single integer or a string, the tag is associated with the entire parent object
* if it is an N-by-1 or 1-by-N vector with N matching the length of the entries in the parent object, 
  it defines the tags for each entry of the parent object.

#### Value
The `Value` property associates a single numerical value to the entire parent object or at each 
entry (a vertex, or a surface patch, or a solid element) of the parent object.

It can take one of 3 values
* if it is a single numerical value or a string, or a structure, the value is associated with 
  the entire parent object
* if it is an N-by-M matrix with N matching the length of the entries in the parent object, 
  it defines a set of M-tuple numerical properties associated with each entry in the parent object

#### Size
The `Size` property defines the size for the entire object or the at each entry (a vertex, or a 
surface patch, or a solid element) of the parent object.

It can take one of 3 values
* if it is a single numerical value, the size is uniform across all entries of the parent object
* if it is an N-by-1 or 1-by-N vector with N matching the length of the entries in the parent object, 
  it defines the size for each entry of the parent object.

### Discrete and parametric graphics

#### Vertices

A vertex represents a discrete spatial location in the N-dimensional space.

For all vertex data objects, the `"Properties"` can store the below optional 
subfields: `"Normal", "Color", "Value", "Size", "Tag"`.

##### MeshVertex1
`"MeshVertex1"` defines a 1-D position vector. It must be defined as an N-by-1 or
1-by-N numerical vector, where N equals to the total number of vertices. The values
in this vector shall be the coordinates of the 1-D vertices.

```
"MeshVertex1": [x1,x2,x3,...]
```
or
```
"MeshVertex1": [
   x1,
   x2,
   x3,
   ...
]
```

##### MeshVertex2
`"MeshVertex2"` defines a 2-D position vector. It must be defined as an N-by-2 
numerical array where N is the total number of vertices. Each row of this array
contains the coordinates of the vertex.

```
"MeshVertex2": [
    [x1,y1],
    [x2,y2],
    [x3,y3],
    ...
]
```
##### MeshVertex3
`"MeshVertex3"` defines a 3-D position vector. Similar to MeshVertex2, it must 
be defined as an N-by-3 numerical array.

```
"MeshVertex3": [
    [x1,y1,z1],
    [x2,y2,z2],
    [x3,y3,z3],
    ...
]
```

##### MeshVertex4
`"MeshVertex4"` defines a 4-D position vector. Similar to MeshVertex2, it must 
be defined as an N-by-4 numerical array.

```
"MeshVertex4": [
    [x1,y1,z1,w1],
    [x2,y2,z2,w2],
    [x3,y3,z3,w3],
    ...
]
```

#### Line segments and curves

For all line and curve objects, the `"Properties"` can store the below optional 
subfields: `"Normal", "Color", "Value", "Size", "Tag"`.

##### MeshPolyLine

`"MeshPolyLine"` defines a set of line segments using an ordered 1-D list of node indices 
(starting from 1). It must be defined by an 1-by-N or N-by-1 vector of integers. 
If an index is 0, it marks the end of the current line segment and starts a new line 
segment from the next index.

```
"MeshPolyLine": [N1, N2, N3, ... ]
```

##### MeshEdge

`"MeshEdge"` defines a set of line segments using a 2-D array with a pair of node indices
in each row of the array. It must be defined by an N-by-2 integer array, where N is the
total number of edges.

```
"MeshEdge": [
    [N11,N12],
    [N21,N22],
    [N31,N32],
    ...
]
```

##### MeshBSpline2D

`"MeshBSpline2D"` defines a 2-dimensional B-spline curve. It must be defined by an N-by-3 
numerical array, where N is the total number of edges. The first 2 columns must be the
2-D coordinates (`x`,`y`) of the control points of the B-spline, and the 3rd column defines 
the weight (`w`) at each control point.

```
"MeshBSpline2D": [
    [x1,y1,w1],
    [x2,y2,w2],
    [x3,y3,w3],
    ...
]
```

#### Surfaces

For allsurface objects, the `"Properties"` can store the below optional 
subfields: `"Normal", "Color", "Value", "Size", "Tag", "Texture"`.


##### MeshTri3
`"MeshTri3"` defines a discretized surface made of triangles with a triplet of node indices
in each row of the array. It must be defined by an N-by-3 integer array, where N is the 
total number of triangles.

```
"MeshTri3": [
    [N11, N12, N13],
    [N21, N22, N23],
    [N31, N32, N33],
    ...
]
```

##### MeshQuad4
`"MeshQuad4"` defines a discretized surface made of quadrilateral with a quadruplet of node indices
in each row of the array. It must be defined by an N-by-4 integer array, where N is the 
total number of quadrilaterals.

```
"MeshQuad4": [
    [N11, N12, N13, N14],
    [N21, N22, N23, N24],
    [N31, N32, N33, N34],
    ...
]
```

##### MeshPLC
`"MeshPLC"` defines a discretized surface made of polygons (piecewise linear complex) of 
uniform or varied edge sizes. It must be defined by an array with elements of integer 
vectors of equal or varied lengths.

```
"MeshPLC": [
    [N11, N12, N13, ...],
    [N21, N22, N23, N24, ...],
    [N31, N32, N33, N34, ..., ...],
    ...
]
```

##### MeshNURBS

`"MeshNURBS"` defines a 3-dimensional non-uniform rational B-spline surface (NIRBS). 
It must be defined by an Nx-by-Ny-by-4 3-D numerical array, where Nx is the number 
of control points along one of the directions and Ny is the number of control points
along the other direction. The first 3 planes of the last dimension must be the
3-D coordinates (`x`,`y`,`z`) of the control points of the NURBS, and the last plane 
defines the weight (`w`) at each control point.

```
"MeshNURBS": [
  [
    [
      [x11,y11,z11,w11],
      [x12,y12,z12,w12],
      [x13,y13,z13,w13],
       ...
    ],
    [
      [x21,y21,z21,w21],
      [x22,y22,z22,w22],
      [x23,y23,z23,w23],
       ...
    ],
    ...
  ],
  [
     ...
  ],
  ...
]
```

#### Solid Elements

For all solid element  objects, the `"Properties"` can store the below optional 
subfields: `"Color", "Value", "Tag", "Texture"`.

##### MeshTet4
`"MeshTet4"` defines a discretized volumetric domain made of tetrahedral elements, with each
element made of a quadruplet of node indices. It must be defined by an N-by-4 integer array, where 
N is the total number of tetrahedra.
```
"MeshTet4": [
    [N11, N12, N13, N14],
    [N21, N22, N23, N24],
    [N31, N32, N33, N34],
    ...
]
```

##### MeshHex8
`"MeshHex8"` defines a discretized volumetric domain made of 8-node hexahedral elements, with each
element specified by a 8-tuple node index. It must be defined by an N-by-8 integer array, where 
N is the total number of hexahedra.
```
"MeshHex8": [
    [N11, N12, N13, ..., N18],
    [N21, N22, N23, ..., N28],
    [N31, N32, N33, ..., N38],
    ...
]
```
##### MeshPyramid5
`"MeshPyramid5"` defines a discretized volumetric domain made of 5-node pyramid elements, with each
element specified by a 5-tuple node index. It must be defined by an N-by-5 integer array, where 
N is the total number of pyramid.
```
"MeshPyramid5": [
    [N11, N12, N13, ..., N15],
    [N21, N22, N23, ..., N25],
    [N31, N32, N33, ..., N35],
    ...
]
```

##### MeshTet10
`"MeshTet10"` defines a discretized volumetric domain made of 10-node straight-line tetrahedral 
elements, with each element specified by a 10-tuple node index. It must be defined by an 
N-by-10 integer array, where N is the total number of 10-node tetrahedra. 

```
"MeshTet10": [
    [N11, N12, N13, ..., N1_10],
    [N21, N22, N23, ..., N2_10],
    [N31, N32, N33, ..., N3_10],
    ...
]
```

The first 4 columns of the array specifies the indices of the 4 vertices of the tetrahedron, 
identical to `"MeshTet4"`, and the last 6 columns of the array specifies the mid-edge node indices, 
sorted in the below order:

```
  [N1, N2, N3, N4, N12, N13, N14, N22, N23, N34]
```
where `N12` is the global index of the node located at the middle of the edge between node N1 and N2;
`N13` is the index for the node between `N1` and `N3`, and so on.



#### Flexible mesh data containers

Flexible mesh data containers allows one to encode a wide range of mesh data using a simple 2-D array.

##### MeshNode
`"MeshNode"` defines a flexible container for the storage of vertex coordinates and the associated 
properties. It must be defined by an N-by-M array, where N is the number of vertices, and M is the 
number of coordinates (D) plus the number of numerical properties (P) attached along each vertex, i.e.

`M = D + P`

The partition of `M` (into `D` and `P` columns) and the interpreations of the numerical 
property values are application dependent.

```
"MeshNode": [
    [x11, y11, z11, ..., w1D, ..., v11, v12, ..., v1P],
    [x21, y21, z21, ..., w2D, ..., v21, v22, ..., v2P],
    [x31, y31, z31, ..., w3D, ..., v31, v32, ..., v3P],
    ...
]
```

##### MeshSurf
`"MeshSurf"` defines a flexible container for the storage of fix-node-length surface patches and 
the associated properties. It must be defined by an N-by-M array, where N is the number of surface 
elements, and M is the number of vertices per element (K) plus the number of numerical properties (P) 
attached along each vertex, i.e.

`M = K + P`

The partition of `M` (into `K` and `P` columns) and the interpretations of the numerical 
property values are application dependent.

```
"MeshSurf": [
    [N11, N11, ..., N1K, ..., v11, v12, ..., v1P],
    [N21, N21, ..., N2K, ..., v21, v22, ..., v2P],
    [N31, N31, ..., N3K, ..., v31, v32, ..., v3P],
    ...
]
```

##### MeshPoly
`"MeshPoly"` defines a flexible container for the storage of variable-node-length surface patches and 
the associated properties. Similar to `"MeshPLC"`, it must be defined by an array with elements of 
integer vectors, but it can contain additional metadata in each element. For each vector representing
a surface patch, the first non-numerical entry, for example, a structure or sub-array, of the vector marks 
the start of the property data.

```
"MeshPoly": [
    [N11, N12, N13, ..., {properties}],
    [N21, N22, N23, N24, ...],
    [N31, N32, N33, N34, ..., ..., [properties] ],
    ...
]
```

##### MeshElem
`"MeshElem"` defines a flexible container for the storage of volumetric elements and the associated 
properties. It must be defined by an N-by-M array, where N is the number of surface elements, and M is the 
number of vertices per element (K) plus the number of numerical properties (P) attached along each vertex, i.e.

`M = K + P`

The partition of `M` (into `K` and `P` columns) and the interpretations of the numerical 
property values are application dependent.

```
"MeshElem": [
    [N11, N11, ..., N1K, ..., v11, v12, ..., v1P],
    [N21, N21, ..., N2K, ..., v21, v22, ..., v2P],
    [N31, N31, ..., N3K, ..., v31, v32, ..., v3P],
    ...
]
```

### Constructive graphics and grouping

#### Mesh grouping and partitioning

To facilitate the organization of unstructured graphics data, JMesh supports data grouping mechanisms
such as those defined in the JData specification. 

Here we define the below keywords for grouping of mesh and shape objects: **"MeshGroup", "MeshObject"** 
and **"MeshPart"**. They are equivallent to the **`"_DataGroup_"`**, **`"_DataSet_"`**, and **`"_DataRecord_"`**
constructs, respectively, as defined in the JData specification, but are specifically applicable to 
graphics objects. The format of "MeshGroup", "MeshObject" and "MeshPart" are identical to JData data 
grouping tags, i.e, they can be either an array or structure, with an optional unique name (within 
the current document) via `"MeshGroup(unique name)"`, `"MeshObject(unique name)"`, and 
`"MeshPart(unique name)"`.

For example, the below JMesh snippet defines two mesh objects as a group

```
{
    "MeshGroup(scene)": [
       {
           "MeshObject(box)": {
               "MeshVertex3":[
                   ...
               ],
               "MeshTri3":{
                   "Data": [
                       ...
                   ],
                   "Properties": {
                       "Value": [
                          ...
                       ]
                   }
               }
           }
       },
       {
           "MeshObject(sphere)": {
               "MeshVertex3":[
                   ...
               ],
               "MeshTri3":[
                   ...
               ]
           }
       }
    ]
}
```

#### Constructive solid graphics (CSG)

Constructive solid graphics allows one to combine simple graphics into complex graphical 
objects via a sequence of Boolean operations. It is typically represented by a tree-like
data/operator data structure. Such data structure is represented in JMesh using the below
construct

* **"CSGObject": [root]**: the root of a CSG object, an array containing a single mesh object 
  or empty, an optional name can be added as "CGSObject(name)",
* **"CSGUnion": [obj1, obj2]**: the union operator of the two objects, "obj1" and "obj2", 
  must be defined as an array of 2 non-empty elements (3 if `{"_DataInfo_":...}` presents)
* **"CSGIntersect": [obj1, obj2]**: the intersection operator of the two objects, "obj1" and "obj2", 
  must be defined as an array of 2 non-empty elements (3 if `{"_DataInfo_":...}` presents)
* **"CSGSubtract": [obj1, obj2]**: the subtraction of obj2 from obj1, i.e. "obj1 - obj2"
  must be defined as an array of 2 non-empty elements (3 if `{"_DataInfo_":...}` presents).

If either (or both) of the operand, "obj1" or "obj2", is a string, it must be treated as a unique 
name, with which one must locate and retrieve the operand within this current document before 
performing the CSG operation.

For example, the below CSG tree represents the CSG object `"scene" = ("obj1" ∩ "obj2") ∪ "obj3"`
```
    "CSGObject(scene)": [
        {
            "CSGUnion": [
                {
                    "CSGIntersect":[
                        "MeshObject(obj1)":{
                            ...
                        },
                        "MeshObject(obj2)":{
                            ...
                        }
                     ]
                },
                {
                    "MeshObject(obj3)": {
                        ....
                    }
                }
            ]
        }
    }
```
The above CSG object can also be written as
```
    "MeshObject(obj1)":{
        ...
    },
    "MeshObject(obj2)":{
        ...
    },
    "MeshObject(obj3)": {
        ....
    },
    "CSGObject(scene)": [
        {
	      "CSGUnion":[
	           {
		       "CSGIntersect":["obj1","obj2"]
		   }, 
		   "obj3"
              ]
	}
    ]
```

### Textures

JMesh permits the definition and binding of textures (rasterized color or intensity values) with any 
mesh or shape objects. The texture data shall be defined using an N-D array using JData array constructs.

#### Texture1D

A 1-D texture can be defined as an N-by-1 or 1-by-N 1-D numerical array or N-by-C 2-D numerical array, 
where N is the number of pixels/voxels of the texture, and C is the number of color components (RGB, RGBA, etc)

```
"Texture1D": [t1,t2,t3,...,tN]
```
or

```
"Texture1D": [
   [R1,G1,B1,A1],
   [R2,G2,B2,A2],
   [R3,G3,B3,A3],
   ...
]
```

#### Texture2D

A 2-D texture can be defined as an Nx-by-Ny 2-D numerical array or Nx-by-Ny-by-C 3-D numerical array, 
where Nx and Ny are the number of pixels/voxels of the texture along x/y directions, and C is 
the number of color components (3 for RGB, 4 for RGBA, etc). Please be aware that the "annotated 
format" for array syntax can be used interchangeably with the direct form.

```
"Texture2D": [
    [t11,t12,t13,...,t1Ny],
    [t21,t22,t23,...,t2Ny],
    ...
    [tNx1,tNx2,tNx3,...,t1NxNy]
]
```
or
```
"Texture2D": [
   [ [R11,G11,B11,A11], [R12,G12,B12,A12], [R13,G13,B13,A13], ... ], 
   [ [R21,G21,B21,A21], [R22,G22,B22,A22], [R23,G23,B23,A23], ... ], 
   ...
]
```


#### Texture3D

A 3-D texture can be defined as an Nx-by-Ny-by-Nz 3-D numerical array or Nx-by-Ny-by-Nz-by-C 
4-D numerical array, where Nx, Ny and Nz are the number of pixels/voxels of the texture along
the x/y/z direction, respectively, and C is the number of color components (3 for RGB, 4 for 
RGBA, etc).

```
"Texture3D": [
   [ [t111,t112,t113,...,t11Nz], [t121,t122,t123,...,t12Nz], ... ], 
   [ [t211,t212,t213,...,t21Nz], [t221,t222,t223,...,t22Nz], ... ], 
   ...
]
```

### Shape primitives

All shape primitives are defined in the form of an object. Their optional properties can be 
directly inserted as a subfield in the shape object. The supported properties include: 
`"Color", "Value", "Tag", "Texture", "Segment"`.

All shape primitive objects supports an optional parameter `"Segment": [...]`, defining
the number of steps for the discretization of the shape components (lines, curves, surfaces).

For 1-D manifold (line objects), `Segment` contains a single integer; for 2-D manifold, it contains
two integers to specify the segments along the two axes spanning the surface; for 3-D manifolds, it
contains 3 integers.

#### 2-D shapes

##### ShapeBox2

A 2-D axis-aligned-bounding-box (AABB) defined by the two diagonal points (`"O"` as one end and `"P"` as the other end)

`"ShapeBox2": {"O":[x0,y0], "P": [x1,y1]}`

##### ShapeDisc2

A 2-D disc defined by the center (`"O"`) and radius `"r"`

`"ShapeDisc2": {"O":[x0,y0], "R": r}`

##### ShapeEllipse

A 2-D ellipse defined by the center (`"O"`), x and y axes in the un-rotated coordinate system (`"rx"` and `"ry"`), and rotation angle `"theta0"`

`"ShapeEllipse": {"O":[x0,y0], "R": [r1,r2], "Angle": theta0}`

##### ShapeLine2

A 2-D line defined by one point along the line (`"O"`), and a 2-D vector (`"V"`) aligns with the direction of the line

`"ShapeLine2": {"O":[x0,y0], "V": [v1,v2]}`

##### ShapeArrow2

A 2-D arrow object defined by one point along the line (`"O"`), and a 2-D vector (`"V"`)  aligns with 
the direction of the line, and the arrow-head size is indicated by scalar `"s"`

`"ShapeArrow2": {"O":[x0,y0], "V": [v1,v2], "Size": s}`

##### ShapeAnnulus

A 2-D annulus defined by the center (`"O"`), and the outer radius (`r1`) and inner radius (`r2`)

`"ShapeAnnulus": {"O":[x0,y0], "R": [r1,r2]}`

##### ShapeGrid2

A 2-D grid defined by the one end of the diagonal line (`"O"`), and the other end (`"P"`); the numbers of
segments along the first and 2nd coordinates are indicated by `"Nx"` and `"Ny"`, respectively

`"ShapeGrid2": {"O":[x0,y0], "P": [x1,y1], "Step":[Nx,Ny]}`

#### 3-D shapes

##### ShapeLine3

A 3-D line defined by one point along the line (`"O"`), and a 3-D vector (`"V"`) aligns with 
the direction of the line

`"ShapeLine3": {"O":[x0,y0,z0], "V": [v1,v2,v3]}`

##### ShapePlane3

A 3-D plane defined by one point along the line (`"O"`), and a 3-D vector (`"N"`) pointing to the 
normal direction of the plane

`"ShapePlane3": {"O":[x0,y0,z0], "N": [v1,v2,v3]}`

##### ShapeBox3

A 3-D axis-aligned-bounding-box (AABB) defined by the two diagonal points (`"O"` as one end and `"P"` as the other end)

`"ShapeBox3": {"O":[x0,y0,z0], "P": [x1,y1,z1]}`

##### ShapeDisc3

A 2-D disc defined by the center (`"O"`) and radius `"R"`; the normal direction of the disc is specified in `"N"`

`"ShapeDisc3": {"O":[x0,y0,z0], "R": r, "N": [v1,v2,v3]}`

##### ShapeGrid3

A 3-D grid defined by the one end of the diagonal line (`"O"`), and the other end (`"P"`); the numbers of
segments along the x/y/z coordinates are indicated by `"Nx"`, `"Ny"` and `"Nz"`, respectively

`"ShapeGrid3": {"O":[x0,y0,z0], "P": [x1,y1,z1], "Step":[Nx,Ny,Nz]}`

##### ShapeSphere

A 3-D sphere defined by the center (`"O"`) and radius `"r"`

`"ShapeSphere": {"O":[x0,y0,z0], "R": r}`

##### ShapeCylinder

A 3-D cylinder defined by one end of the cylindrical axis (`"O"`) and the other end of the axis `"P"`;
the radius of the cylinder is specified in `"r"`

`"ShapeCylinder": {"O":[x0,y0,z0], "P":[x1,y1,z1], "R": r}`

##### ShapeEllipsoid

A 3-D ellipsoid defined by the center position (`"O"`), x, y and z axes in the pre-rotated coordinate system
(`rx, ry, rz`), and the azimuthal rotation angle `theta0` and zenith rotation angle `phi0`

`"ShapeEllipsoid": {"O":[x0,y0,z0], "R": [rx,ry,rz], "Angle":[theta0, phi0]}`

##### ShapeTorus

A 3-D torus defined by the center of the torus (`"O"`), the radius of the torus (`r1`) and the tube radius 
(`r2`) and the normal direction vector `"N"`

`"ShapeTorus": {"O":[x0,y0,z0], "R": r1, "Rtube": r2, "N":[v1,v2,v3]}`

##### ShapeCone

A 3-D cone defined by the center of the bottom circle of the cone (`"O"`), the radius of the bottom 
circle (`r`) and the tip position `"P"`

`"ShapeCone": {"O":[x0,y0,z0], "P": [x1,y1,z1], "R": r}`

##### ShapeConeFrustum

A 3-D conical frustum defined by the center of the bottom circle of the frustum (`"O"`), the center of
the top circle of the frustum and the radii of the bottom and top circles (`r1` and `r2` respectively)

`"ShapeConeFrustum": {"O":[x0,y0,z0], "P": [x1,y1,z1], "R": [r1, r2]}`

##### ShapeSphereShell

A 3-D spherical shell defined by the center of the shell (`"O"`), the outer radius `r1` and the inner
radius `r2`

`"ShapeSphereShell": {"O":[x0,y0,z0], "R": [r1, r2]}`

##### ShapeSphereSegment

A 3-D spherical segment defined by the center of the sphere (`"O"`), the radius of the sphere 
the normal direction of the bottom/top plane `"N"` and the height of the segment `h1` for the 
starting plane, and `h2` at the ending plane

`"ShapeSphereSegment": {"O":[x0,y0,z0], "R": r, "N": [v1,v2,v3], "Height": [h1, h2]}`

#### Shapes from transformation

JMesh supports mesh constructs created via transformations, such as extrusion or revolution. 
These operators requires two curve objects as operands, one as the "source", the other as the 
guide (in extrusion) or axis (when revolving). The permitted curve objects can be found in the
"Line segments and curves" section, i.e. `"MeshLine"`, `"MeshEdge"`, and `"MeshBSpline2D"`.

##### ShapeExtrude2D

A 2-D surface generated by extruding a 2-D curve object "source_curve" along another 2-D curve 
object "guide_curve" defined within the same plane.

```
"ShapeExtrude2D": {
    "source_curve": [
        ...
    ],
    "guide_curve": [
        ...
    ]
}
```

##### ShapeExtrude3D

A 3-D surface generated by extruding a 3-D curve object "source_curve" along another 3-D curve 
object "guide_curve".

```
"ShapeExtrude3D": {
    "source_curve": [
        ...
    ],
    "guide_curve": [
        ...
    ]
}
```

##### ShapeRevolve3D

A 3-D surface generated by revolving a 3-D curve object "source_curve" using a straightline 
object as the rotation axis defined using the `"ShapeLine3"` keyword

```
"ShapeRevolve3D": {
    "source_curve": [
        ...
    ],
    "ShapeLine3": {
        ...
    }
}
```

Recommended File Specifiers
------------------------------

For the text-based JMesh file, the recommended file suffix is **`".jmsh"`**; for 
the binary JMesh file, the recommended file suffix is **`".bmsh"`**.

The MIME type for the text-based JMesh document is 
**`"application/jmesh-text"`**; that for the binary JMesh document is 
**`"application/jmesh-binary"`**


Summary
----------

