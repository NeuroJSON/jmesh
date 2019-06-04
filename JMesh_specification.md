 JMesh - A versatile JSON data format for unstructured meshes and geometries
============================================================

- **Status of this document**: This document is current under development.
- **Copyright**: (C) Qianqian Fang (2011, 2019) <q.fang at neu.edu>
- **License**: Apache License, Version 2.0
- **Version**: 0.4
- **Abstract**:

> JMesh is a JSON and JData compliant file format for storage and interchange of
unstructured mesh and geometric data, such as triangular and tetrahedral meshes. 
Same as JData, a JMesh file utilizes the JavaScript Object Notation (JSON) 
[RFC4627] and Universal Binary JSON (UBJSON) constructs to serialize and encode
mesh data structures, therefore, it can be directly processed by most existing
JSON and UBJSON parsers. In this specification, we define a list of 
JData-compatible keywords to encode complex geometric shapes, inluding N-dimensional 
points, lines, surfaces, solids, shape primitives, their interactions and spatial
relations, together with their associated properties, such as numerical values, 
gray-scales, colors, and other properties related to scientific research, 3-D 
fabrication and computer graphics rendering.


## Table of Content

- [Introduction](#introduction)
    * [Background](#background)
    * [JMesh specification overview](#jmesh-specification-summary)
- [Grammar](#grammar)
    * [Text-based JData storage grammar](#text-based-jdata-storage-grammar)
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
in a (large set of incompatible)[https://en.wikipedia.org/wiki/List_of_file_formats#3D_graphics] 
file formats that are largely application specific and difficult 
to standardize.

For example, the widely used STL format (for 3-D printing) supports only 
triangular surfaces, and is not able to encode other shape constructs such as
tetrahedral or hexahedral solid objects. Similarly, the PLY (Polygon File 
Format), OFF (Object File Format) and the Wavefront OBJ file formats are 
also largely limited to triangular surfaces, with various degrees of 
metadata support such as triangle colors and node/surface norms. While 
parsing and interchanging these simple mesh data formats are 
straightforward and easy to implement, the limited functionalities and
lack of the ability to extend for more complex shapes and metadata makes
it difficult to standardize. On the other hand, more capable and flexible
shape storage formats are often associated with commercial 3-D CAD software,
such as 3DS, FBX, and DWG formats, with relatively large and complex 
parsers, which are often not widely available.

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
JSON/UBJSON constructs. 


### JMesh specification overview

JMesh is an extensible framework to store complex geometry and mesh related 
data using the JData representation, with a syntax compatible to the widely 
used JSON and UBJSON file specifications. JMesh adds a semantic layer to 
a JSON structure by attaching meanings and format specifications to a list of 
reserved "name" fields (or simply, JMesh keywords). These JMesh keywords allow
one to unambiguously store points, edges, surfaces and tetrahedral meshes
of 1-D, 2-D, 3-D or even higher dimensions.

The purpose of this document is to define the text and binary JMesh format 
specifications. This is achieved through the definition of a semantic layer 
over the JSON/UBJSON data storage syntax to map various types of mesh and
geometrical data. Such semantic layer includes

- a list of dedicated `"name"` fields, or keywords, that define the containers 
  of various nodes, elements, shape primitives
- a list of dedicated `"name"` fields and formats to facilitate the spatial
  organization, grouping and interaction of shapes
- a list of metadata keywords for enriched data annotation

In the following sections, we will clarify the basic JMesh grammar and define 
mesh data container formats to encode a wide-range of unstructured mesh and
shape data types, including points, lines, contours, shape primitives, 
triangular surfaces, tetrahedral meshes, and other surface and solid elements
In addition, we will define a set of structures to represent the interactions
and relationships between shape constructs, such as Boolean operations and 
grouping.
 

Grammar
------------------------

### JMesh Storage Grammar

The JMesh files are JData specification compliant. The same as JData, it has
both a text-based format based on JSON serialization and a binary-based format
based on the UBJSON serialization. Briefly, the text-based JMesh is a 
valid JSON file with the extension to support concatenated JSON objects; the
binary-format JMesh is a valid UBJSON with the grammar extension to support 
N-D array.



Recommended File Specifiers
------------------------------

For the text-based JMesh file, the recommended file suffix is **`".jmsh"`**; for 
the binary JMesh file, the recommended file suffix is **`".bmsh"`**.

The MIME type for the text-based JMesh document is 
**`"application/jmesh-text"`**; that for the binary JMesh document is 
**`"application/jmesh-binary"`**


Summary
----------

