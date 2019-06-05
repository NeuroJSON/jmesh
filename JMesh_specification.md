 JMesh - A versatile JSON data format for unstructured meshes and geometries
============================================================

- **Status of this document**: This document is current under development.
- **Copyright**: (C) Qianqian Fang (2011, 2019) <q.fang at neu.edu>
- **License**: Apache License, Version 2.0
- **Version**: 0.4
- **Abstract**:

> JMesh is a portable and extensible file format for storage and interchange of
unstructured mesh and geometric data, including triangular and tetrahedral meshes. 
Built upon the JData specification, a JMesh file utilizes the JavaScript Object Notation (JSON) 
[RFC4627] and Universal Binary JSON (UBJSON) constructs to serialize and encode
mesh data structures, therefore, it can be directly processed by most existing
JSON and UBJSON parsers. In this specification, we define a list of 
JData-compatible keywords to encode complex geometric shapes, inluding N-dimensional 
points, curves, surfaces, solid elements, shape primitives, their interactions and spatial
relations, together with their associated properties, such as numerical values, 
gray-scales, colors, and other properties related to scientific research, 3-D 
fabrication and computer graphics rendering.


## Table of Content

- [Introduction](#introduction)
    * [Background](#background)
    * [JMesh specification overview](#jmesh-specification-summary)
- [Grammar](#grammar)
    * [Text-based  storage grammar](#text-based-jdata-storage-grammar)
    * [Binary JData storage grammar](#binary-jdata-storage-grammar)



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
metadata support such as triangle colors and node/surface norms. While 
parsing and interchanging these simple mesh data formats are 
straightforward and easy to implement, the limited functionalities and
lack of the ability to extend for more complex shapes and metadata makes
it difficult to capture other more sophisticated geometries. On the other 
hand, more capable and flexible shape storage formats are often associated 
with commercial 3-D CAD software, such as 3DS, FBX, and DWG formats, with 
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
data, faster file sizes and processing speed. The JData specification
provides the foundation for serializing complex hierarchical data using
JSON/UBJSON constructs. This permits us to define language and library-neutral
geometry and mesh descriptions using the simple and extensible constructs
from JSOB and UBJSON formats.


### JMesh specification overview

JMesh is an extensible framework to store complex geometry and mesh related 
data using the JData representation, with a syntax compatible to the widely 
used JSON and UBJSON file formats. JMesh adds a semantic layer to 
a JSON structure by attaching meanings and format specifications to a list of 
reserved "name" fields (or simply, JMesh keywords). These JMesh keywords allow
one to unambiguously store vertices, edges, surfaces and tetrahedral meshes
of 1-D, 2-D, 3-D or even higher dimensions.

The purpose of this document is to define the text and binary JMesh format 
specifications. This is achieved through the definition of a semantic layer 
over the JSON/UBJSON data storage syntax to map various types of mesh and
geometrical data. Such semantic layer includes

- a list of dedicated `"name"` fields, or keywords, that define the containers 
  of various nodes, elements, and shape primitives
- a list of dedicated `"name"` fields and formats to facilitate the spatial
  organization, grouping and interaction of geometric objects
- a list of metadata keywords for enriched data annotation

In the following sections, we will clarify the basic JMesh grammar and define 
mesh data container formats to encode a wide-range of unstructured mesh and
shape data types, including vertices, lines, contours, shape primitives, 
triangular surfaces, tetrahedral meshes, and other surface and solid elements
In addition, we will define a set of structures to represent the interactions
and relationships between shape constructs, such as Boolean operations and 
grouping.
 

Grammar
------------------------

All JMesh files are JData specification compliant. The same as JData, it has
both a text format based on JSON serialization and a binary format
based on the UBJSON serialization scheme. 

Briefly, the text-based JMesh is a valid JSON file with the extension to 
support concatenated JSON objects; the binary-format JMesh is a valid UBJSON 
with the extended syntax to support N-D array. Please refer to the JData 
specification for the definitions.

Nearly all supported mesh data containers (i.e. named JData nodes) can be defined 
using one of the two forms: an N-D array or a structure. For each JMesh data
keyword defined the following sections, we define the requirements on the data 
dimensions if one chooses the array form, and and the required and optional 
subfields if one chooses the structure form.

### Array form

For simple data, one can use the "array form" to store the data under a container 
keyword. In such case, the format of the data must follow the "N-Dimensional 
Array Storage Keyword" rules defined in the JData specification. For example, 
one can store a 1-D or 2-D array using the direct storage format as
```
 "jmesh_container_1d": [v1,v2,...,vn]
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
The direct storage format and the annotated storage format are equivallent. In the 
below sections, we use mostly the direct format to explain the data format, but
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
Here, only the `"Data"` subfield is required, and it must have an array-value that 
is the same as the "direct storage" form. The optional `"_DataInfo_"` is the JData 
construct for storing metadata associated with this structure. The other optional 
`"Properties"` subfield allows one to store additional data with this shape/mesh 
element. The `"Properties"` subfield can be an array or structure with 
additional subfields.


Mesh Data Keywords
------------------------
In this section, we define dedicated JSON `"name"` keywords that can be used to 
define discretized shapes objects (or meshes).

In most of the cases, the data container keywords defined in this section have a 
prefix of `"Mesh"`. Many of the keywords ends with a numerical value which typically
represents the column number of the data when stored in the array format.

All indices - integers mapping between data entries - shall have a start value of 
1. 

### Vertices

A vertex represents a discrete spatial location in the N-dimensional space.

For all vertex data objects, the "Properties" can store the below optional 
subfields:
```
"Norm": [...]
"Color": [...]
"Value": [...]
"Size": [...]
"Label": [...]
```

#### MeshVertex1
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

#### MeshVertex2
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
#### MeshVertex3
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

#### MeshVertex4
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

### Line segments and curves

For all line data objects, the "Properties" can store the below optional 
subfields:
```
"Color": []
"Value": []
"Size": []
"Label": []
```

#### MeshLine

`"MeshLine"` defines a set of line segments using an ordered 1-D list of node indices 
(starting from 1). It must be defined by an 1-by-N or N-by-1 vector of integers. 
If an index is 0, it marks the end of the current line segment and starts a new line 
segment from the next index.

```
"MeshLine": [N1, N2, N3, ... ]
```

#### MeshEdge

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


### Surfaces

For all surface objects, the "Properties" can store the below optional 
subfields:
```
"Color": []
"Value": []
"Size": []
"Label": []
```


#### MeshTri3
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

#### MeshQuad4
`"MeshQuad4"` defines a discretized surface made of quadrilateral with a quadruplet of node indices
in each row of the array. It must be defined by an N-by-4 integer array, where N is the 
total number of quadrilaterals.

```
"MeshQuad4": [
    [N11, N12, N13, N14],
    [N21, N22, N23, N24],
    [N31, N32, N33, N24],
    ...
]
```
#### MeshPoly
`"MeshPoly"` defines a discretized surface made of polygons of uniform or varied edge sizes. 
It must be defined by an array with elements of integer vectors of equal or varied lengths.

```
"MeshPoly": [
    [N11, N12, N13, ...],
    [N21, N22, N23, N24, ...],
    [N31, N32, N33, N34, ..., ...],
    ...
]
```

### Solid Elements

For all solid element objects, the "Properties" can store the below optional 
subfields:
```
"Color": []
"Value": []
"Label": []
```

#### MeshTet4
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

#### MeshHex8
`"MeshHex8"` defines a discretized volumetric domain made of 8-node hexahedral elements, with each
element specified by a 8-tuple node index. It must be defined by an N-by-8 integer array, where 
N is the total number of hexahedra.
```
"MeshHex8": [
    [N11, N12, N13, ..., N18],
    [N21, N22, N23, ..., N28],
    [N31, N32, N33, ..., N28],
    ...
]
```
#### MeshPyramid5
`"MeshHex8"` defines a discretized volumetric domain made of 5-node pyramid elements, with each
element specified by a 5-tuple node index. It must be defined by an N-by-5 integer array, where 
N is the total number of pyramid.
```
"MeshPyramid5": [
    [N11, N12, N13, ..., N15],
    [N21, N22, N23, ..., N25],
    [N31, N32, N33, ..., N25],
    ...
]
```

#### MeshTet10
`"MeshTet10"` defines a discretized volumetric domain made of 10-node straightline tetrahedral 
elements, with each element specified by a 10-tuple node index. It must be defined by an 
N-by-10 integer array, where N is the total number of 10-node tetrahedra. 

```
"MeshTet10": [
    [N11, N12, N13, ..., N1_10],
    [N21, N22, N23, ..., N2_10],
    [N31, N32, N33, ..., N2_10],
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



### Flexible mesh data containers

Flexible mesh data containers allows one to encode a wide range of mesh data using a simple 2-D array.

#### MeshNode
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

#### MeshSurf
`"MeshSurf"` defines a flexible container for the storage of surface patches and the associated 
properties. It must be defined by an N-by-M array, where N is the number of surface elements, and M is the 
number of vertices per element (K) plus the number of numerical properties (P) attached along each vertex, i.e.

`M = K + P`

The partition of `M` (into `K` and `P` columns) and the interpreations of the numerical 
property values are application dependent.

```
"MeshSurf": [
    [N11, N11, ..., N1K, ..., v11, v12, ..., v1P],
    [N21, N21, ..., N2K, ..., v21, v22, ..., v2P],
    [N31, N31, ..., N3K, ..., v31, v32, ..., v3P],
    ...
]
```

#### MeshElem
`"MeshElem"` defines a flexible container for the storage of volumetric elements and the associated 
properties. It must be defined by an N-by-M array, where N is the number of surface elements, and M is the 
number of vertices per element (K) plus the number of numerical properties (P) attached along each vertex, i.e.

`M = K + P`

The partition of `M` (into `K` and `P` columns) and the interpreations of the numerical 
property values are application dependent.

```
"MeshElem": [
    [N11, N11, ..., N1K, ..., v11, v12, ..., v1P],
    [N21, N21, ..., N2K, ..., v21, v22, ..., v2P],
    [N31, N31, ..., N3K, ..., v31, v32, ..., v3P],
    ...
]
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

