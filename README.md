ParaView and VTK

ParaView stands for "Parallel View".  It is a scientific visualization program developed especially for viewing large 3-D scientific data sets.  The primary interface is a GUI, although scripting is possible using Python.

In contrast, MATLAB was designed to be used by engineers and their common data types (time series and basic 2-D plots).  Although MATLAB has 3-D capabilities, these capabilities are limited in comparison to programs like ParaView.

These notes describe how to create VTK (Visualization Tool Kit) dataset files that can be read by ParaView.  They also describe the terminology used by ParaView and VTK.  

<div style="background-color:yellow">
Prior to reading these notes, work through the activities in the [[#ParaView Tutorial]].  This tutorial shows how the ParaView GUI works and has examples of its capabilities.  
</div>

<div style="background-color:yellow">
While reading these notes, you are encouraged to use ParaView to read each file, modify the view, and read the information tab.  In addition, for each example, make small modifications to the file and predict what will happen before opening the modified file.
</div>

# References 

The following set of notes apply to ParaView and VTK.  They were developed based on: 

* File format specification for VTK: [http://www.vtk.org/VTK/img/file-formats.pdf]
* VisIt description of ASCII VTK files: [http://www.visitusers.org/index.php?title=ASCII_VTK_Files]
* VTK User's Guide: [https://vtk.org/vtk-users-guide/]
* VTK description of Geometry and Topology: [http://www.vtk.org/Wiki/VTK/Tutorials/GeometryTopology]
* ParaView description of VTK Data Model: [http://www.paraview.org/Wiki/ParaView/Users_Guide/VTK_Data_Model]
* ParaView Manual: [http://www.paraview.org/files/v4.0/ParaViewManual.v4.0.pdf]
* ParaView User's Guide: [http://www.paraview.org/paraview-downloads/download.php?submit=Download&version=v4.3&type=data&os=all&downloadFile=TheParaViewGuide-v4.3-CC-Edition.pdf]
* Generate in ParaView, render in Blender: [http://www.personal.psu.edu/dab143/OFW6/Training/cragun_slides.pdf]

None of the above references are very helpful to beginners (based on my experience).  The set of notes that follow attempt to help the reader get to the point where these references are useful.

Other useful links:
* ParaView workshop activities [http://www.hpc.mcgill.ca/downloads/paraview_workshop/]
* ParaView Tutorials [http://www.paraview.org/Wiki/SNL_ParaView_4_Tutorials]
* Sample VTK files: [http://people.sc.fsu.edu/~jburkardt/data/vtk/vtk.html] [http://www.gris.uni-tuebingen.de/edu/areas/scivis/volren/datasets/new.html]

## Terminology 

The following terms are used in this set of notes.  The usage of these terms in these notes are consistent, but may be inconsistent with definitions used in the [[#References|references]] (and in many cases, the references have conflicting or differing definitions).

* VTK - Visualization Tool Kit; a software library for creating computer graphics.
* ParaView - An application that uses VTK to allow for display of and interaction with computer graphics.
* Data model - How data are described, grouped, connected.   A key part of the data model of MATLAB is the matrix, which represents a group of numbers in tabular form.  Each datum in the table is identified (described) by specifying the row and column in which it is located. 
* Coordinate - The x, y, and z values of a point in space.
* Vertex - A point in space that has zero or more lines connected to it.  "Points" and "Vertex" are often used interchangeably, and a vertex with zero lines connected to it is often referred to as simply a point.
* Dataset -  A collection of coordinate values (that correspond vertices in a 3-dimensional space) and connections of these points.  Dataset types include Structured Points, Rectilinear Grid, Structured Grid, and Unstructured Grids.
* Dataset attributes - Each part of a dataset (points and cells) may have color, scalar, vector, normal, tensor, texture, and field attributes.
* Geometry - A list of x, y, and z coordinate values of vertices
* Topology - A description of how a geometry is interconnected to form cells (which may be vertices, lines, surfaces, and volumes)
* Cell - A basic unit of topology.  Vertices are connected to form cells. 
* Surface graphics primitives - vertices, lines, polygons, and triangle strips.  Each primitive corresponds to a cell and has a front and back surface.
* Polygon - A set of points connected with lines.
* Grid - A set of coordinates (locations of vertices).
* Mesh - A set of cells and vertices; defined by topology and geometry) ([http://www.paraview.org/files/v4.0/ParaViewManual.v4.0.pdf pg 13])
* Connectivity - The mapping from cells to vertices ([http://www.paraview.org/files/v4.0/ParaViewManual.v4.0.pdf pg 13])

## Introduction 

In these notes, VTK data files are used to show how VTK and ParaView describe data and to familiarize the user with the terminology used by ParaView.  The first three lines of a VTK data files (sometimes referred to as a legacy VTK file) will always have the form

```
# vtk DataFile Version 3.0
Descriptive Text
ASCII
```

where `Descriptive Text` will change depending on the content of the file.  The next two parts of the datafile
contain a description of a dataset and an optional set of dataset attributes.  In the examples in this section, only the dataset description is given in the file.  See [[#Dataset Attributes]] for examples of files containing both dataset descriptions and dataset attributes.

The following examples illustrate the VTK file format.  Additional details are given in [[#Datasets]].


> If you have a typo in the VTK file and attempt to load it, you will get a message <b>A reader could not be found</b> or an error message.  When creating VTK files based on these examples, make sure to copy the text exactly (including capitalization and spaces).


**Example**

To test this file in ParaView, open a text editor (e.g., Notepad, TextWrangler, or GEdit), copy the following text, save it as `demo.vtk` and then open the file in ParaView and click the Apply button.  You should see a diagonal line.

```
# vtk DataFile Version 3.0
A dataset with one line and no attributes
ASCII

DATASET POLYDATA
POINTS 2 float
0.0 0.0 0.0
1.0 1.0 0.0
LINES 1 3
2 0 1
```

This file describes the location of two points (the geometry) and how these points are connected (the topology).  The dataset has type `POLYDATA`, which is described in [[#Polygonal Data]].  The third line, containing the string `ASCII`, is used to indicate that all of the following content should be read using the ASCII version of the VTK file reader.  All files here should have `ASCII` as the third line.  (The other option is `BINARY` and is not covered here.)

The interpretation of `POINTS 2 float`, `LINES 1 3`, and `2 0 1` is:

* `POINTS 2 float`: Two rows of x,y,z values of points are to follow.  The values should be interpreted as floating point.
* `LINES 1 3`: `1` row of line connections are to follow; each row has `3` integers.  Note that this primitive (`LINES`) does not require a `dataType` to be specified (e.g., `float`).  The dataType is always an integer.
* `2 0 1`: `2` integers (that refer to points in the point list) are to follow and the first point (identified with `0`) in the list of points is connected to the second point (identified with `1`).

In this and the following examples, the keyword `float` appears.  This keyword identifies how the numbers that follow should be interpreted.   From [http://www.vtk.org/VTK/img/file-formats.pdf]:


> dataType is one of the types bit, unsigned_char, char, unsigned_short, short, unsigned_int, int, unsigned_long, long, float, or double. These keywords are used to describe the form of the data {numbers}, both for reading from file, as well as constructing the appropriate internal objects. Not all data types are supported for all classes {primitives}.

*Example*

This file connects three points with a single line and can be read by ParaView.

```
# vtk DataFile Version 3.0
A dataset with one polyline and no attributes
ASCII

DATASET POLYDATA
POINTS 3 float
0.0 0.0 0.0
1.0 0.0 0.0
1.0 1.0 0.0
LINES 1 4
3 0 1 2
```

In this example, three points are specified at the x,y,z values of `0,0,0`, `1,0,0`, and `1,1,0`.  The first point in the list is the 0th point (by definition).  The second is the 1st point, etc.  The statement `LINES 1 4` means "one line specification is to follow and the total number of numbers that specify each line is four".  The statement `3 0 1 2` means "three numbers are to follow that identify how the points are connected to form a line.  Connect point 0 to point 1 to point 2".

To close the triangle, you could modify the last two statements from
 LINES 1 4
 3 0 1 2
to
 LINES 1 5
 4 0 1 2 0
The last statement now means "4 values that specify a line are to follow.  To create the line, connect point 0 to 1, 1 to 2, and then 2 back to 0".

<img src="https://raw.githubusercontent.com/rweigel/cds301/master/vtk/LinesA.png" width=400px/>

*Example*

Three points can be connected with two individual lines using the following file.

The statement `LINES 2 6` means "two lines are to follow.  The total number of values used to describe the lines is 6."  The statement `2 0 1` means "2 values that specify a line are to follow.  This line should connect point 0 to point 1.  The statement `2 1 2` means "2 values that specify a line are to follow.  This line should connect point 1 to point 2".

The wireframe image looks the same as the previous example, but the Information tab will note that there are two cells (instead of one as in the previous example).

```
# vtk DataFile Version 3.0
A dataset with two lines and no attributes
ASCII

DATASET POLYDATA
POINTS 3 float
0.0 0.0 0.0
1.0 0.0 0.0
1.0 1.0 0.0
LINES 2 6
2 0 1
2 1 2
```

<img src="https://raw.githubusercontent.com/rweigel/cds301/master/vtk/LinesB.png" width=400px/>


*Example*

In this file, the lines are connected in a different way.

```
# vtk DataFile Version 3.0
A dataset with two lines and no attributes
ASCII

DATASET POLYDATA
POINTS 3 float
0.0 0.0 0.0
1.0 0.0 0.0
1.0 1.0 0.0
LINES 2 6
2 1 2
2 2 0
```

<img src="https://raw.githubusercontent.com/rweigel/cds301/master/vtk/LinesC.png" width=400px/>

# Datasets 

A dataset in VTK has two key components [http://www.vtk.org/Wiki/VTK/Tutorials/GeometryTopology]:

1. Geometry
2. Topology

*Geometry* is represented by a list of x,y,z coordinate values (corresponding to points in 3-dimensional space).

The way in which a list of coordinates are connected to form cells represents its *topology*.

There are many ways of representing topology in VTK.  Although a dataset can be specified without topology, most of the operations on data require a topology to be specified. 

A given geometry may have more than one topological representation.  For example, three points may be connected with

* two lines to form two separate lines (two cells)
* a single continuous line to form a polyline (one cell), and
* three lines to form a triangle (one cell).

In the case of the polyline, the cell consists of both lines.  If the two connected lines will always share the same attributes, they should be connected as a polyline.

The types of datasets include

1. [Structured Points](#structured-points)
2. [[#Rectilinear Grid]],
3. [[#Structured (Curvilinear) Grid]],
4. [[#Polygonal Grid]], and
5. [[#Unstructured Grid]].

For the first three types of datasets, only a geometry is specified and the topology is inferred.

An unstructured grid requires both geometry and topology to be specified.

## Structured Points

Structured points have a constant spacing in each spatial direction.  The number of points in each direction is specified along with the spacing between points in each direction.

The number of points in each direction, nx, ny, and nz, must be greater than or equal to 1 and each of their spacings, sx, sy, and sz, must be greater than 0. The syntax is

```
DATASET STRUCTURED_POINTS
DIMENSIONS nx ny nz
ORIGIN x y z
SPACING sx sy sz
```

where `nx ny nz` are integers and `x y z` and `sx sy sz` are floating point values.

*Example*

The statement

```
DIMENSIONS 2 4 8
```

specifies that there are 2 points in the x-direction, 4 points in the y-direction, and 8 points in the z-direction.
The statement

```
 SPACING 1.0 2.0 4.0
```

specifies that points should have a separation of 1.0 in the x-direction, 2.0 in the y-direction, and 4.0 in the z-direction.

```
# vtk DataFile Version 3.0
Structured Points Example
ASCII
 
DATASET STRUCTURED_POINTS
DIMENSIONS 2 4 8
ORIGIN 0.0 0.0 0.0
SPACING 1.0 2.0 4.0
```

In this example, only geometry is specified, but VTK infers the topology.  To see this, open the file in ParaView and select the "Surface with Edges" representation.  Also, select coloring by cellNormals in the x-direction to generate. The display should be similar to the second image.

Wireframe representation

<img src="https://raw.githubusercontent.com/rweigel/cds301/master/vtk/StructuredPointsA.png" width=400px/>

Surface with edges representation and coloring by cellNormals in x-direction

<img src="https://raw.githubusercontent.com/rweigel/cds301/master/vtk/StructuredPointsA_surface_with_edges_colored_by_cell_x_normals.png" width=400px/>

## Rectilinear Grid

A rectilinear grid is a dataset with

* a regular topology (all graphics primitives are the same - rectangles), and 
* semi-regular geometry along each axes. 

Regular topology mean that each cell has the same type (rectangle).

The geometry is specified with a list of monotonically increasing values.  A semi-regular geometry is one in which the spacing of the points in the direction of each axis is uniform, but the spacing is not the same for all three directions.

All cells in a rectilinear grid have the same type and the topology is inferred from the number of points along each axis. The type can be a vertex (corresponding to a 0-D dataset), a line (1-D), pixel (a square; 2-D) or voxel (a cube; 3-D).

```
DATASET RECTILINEAR_GRID
DIMENSIONS nx ny nz 
X_COORDINATES nx dataType
x0 x1 ... x(nx-1)
Y_COORDINATES ny dataType
y0 y1 ... y(ny-1)
Z_COORDINATES nz dataType
z0 z1 ... z(nz-1)
```

*Example*

In the following example, a rectilinear grid is specified.  Note that numbers in the line `DIMENSIONS 2 2 1` must be consistent with the numbers for each of the coordinate directions (2 x-coordinates, 2 y-coordinates, and 1 z-coordinate).

```
# vtk DataFile Version 3.0
Rectilinear Grid Example A
ASCII

DATASET RECTILINEAR_GRID
DIMENSIONS 2 2 1
X_COORDINATES 2 float
0.0 1.0
Y_COORDINATES 2 float
0.0 1.0
Z_COORDINATES 1 float
0.0
```

<img src="https://raw.githubusercontent.com/rweigel/cds301/master/vtk/RectilinearGridA.png" width=400px/>


*Example*

```
# vtk DataFile Version 3.0
Rectilinear Grid Example B
ASCII

DATASET RECTILINEAR_GRID
DIMENSIONS 3 4 1
X_COORDINATES 3 float
0.0 2.0 4.0
Y_COORDINATES 4 float
1.0 2.0 4.0 8.0
Z_COORDINATES 1 float
0.0 
```

<img src="https://raw.githubusercontent.com/rweigel/cds301/master/vtk/RectilinearGridB.png" width=400px/>


### Structured (Curvilinear) Grid

All cells in a curvilinear grid are of the same type. The type (and dimensionality) can be a vertex (0-D), a line (1-D), a quad (2-D) or a hexahedron (3-D).

```
DATASET STRUCTURED_GRID
DIMENSIONS nx ny nz
POINTS n dataType
p0x p0y p0z
p1x p1y p1z
...
p(n-1)x p(n-1)y p(n-1)z
```


**Example**

```
# vtk DataFile Version 3.0
Structured Grid Example A
ASCII

DATASET STRUCTURED_GRID
DIMENSIONS 2 2 1
POINTS 4 float
0 0 0
1 0 0
0 1 0
1 1 0
```

<img src="https://raw.githubusercontent.com/rweigel/cds301/master/vtk/StructuredGridA.png" width=400px"/>

'''Example (invalid)'''
{|style="border-top:1px solid black" cellpadding="2" width="100%" align="left"
|-
|style="vertical-align:top" width="50%" |
Note that the order in which the points are specified matters.  VTK uses the point ordering to infer the topology.

In this example, the surface color is black and coloring by cell normals does not work.  The reason is that the cell created is not valid.
```
# vtk DataFile Version 3.0
Structured Grid Example B
ASCII

DATASET STRUCTURED_GRID
DIMENSIONS 2 2 1
POINTS 4 float
0 0 0
1 1 0
0 1 0
1 0 0
```
|style="vertical-align:top"|
<imgc>url=https://raw.githubusercontent.com/rweigel/cds301/master/vtk/StructuredGridB.png|width=400px|display=none|expire=1</imgc>
|}


'''Example'''
{|style="border-top:1px solid black" cellpadding="2" width="100%" align="left"
|-
|style="vertical-align:top" width="50%" |
A structured grid may have a non-rectilinear shape.  (See also [http://www.paraview.org/Wiki/File:ParaView_UG_Curvilinear.png])
```
# vtk DataFile Version 3.0
Structured Grid Example C
ASCII
DATASET STRUCTURED_GRID
DIMENSIONS 3 2 1
POINTS 6 float
0 0 0
1 0 0
2 0 0
0 1 0
1 0.5 0
2 0.2 0
```

|style="vertical-align:top"|
<imgc>url=https://raw.githubusercontent.com/rweigel/cds301/master/vtk/StructuredGridC.png|width=400px|display=none|expire=1</imgc>
|}

### Polygonal Grid (Polydata)

From [http://www.paraview.org/Wiki/index.php?oldid=47855]

<blockquote>
A polydata such as [http://www.paraview.org/Wiki/File:ParaView_UG_Polydata.png Figure 3.9] is a specialized version of an unstructured grid designed for efficient rendering. It consists of 0D cells (vertices and polyvertices), 1D cells (lines and polylines) and 2D cells (polygons and triangle strips). Certain filters that generate only these cell types will generate a polydata. Examples include the Contour and Slice filters. An unstructured grid, as long as it has only 2D cells supported by polydata, can be converted to a polydata using the Extract Surface filter. A polydata can be converted to an unstructured grid using Clean to Grid.
</blockquote>

A polygonal dataset consists of

* [[#Vertices]] (and polyvertices)
* [[#Lines]] (and polylines),
* [[#Polygons]] (of various types),
* [[#Triangle Strips]], and
* [[#Combinations]] of the above.

A polygonal dataset requires the specification of points and then vertices, lines, polygons, triangle strips or combinations thereof.  A polygonal dataset always starts with

```
DATASET POLYDATA
POINTS n dataType
p0x p0y p0z
p1x p1y p1z
...
p(n-1)x p(n-1)y p(n-1)z
```

#### Vertices 

```
DATASET POLYDATA

POINTS n dataType 
p0x p0y p0z
p1x p1y p1z
...
p(n-1)x p(n-1)y p(n-1)z 

VERTICES n size
numPoints0, i0, j0, k0, ...
numPoints1, i1, j1, k1, ...
...
numPointsn-1, in-1, jn-1, kn-1, ...
```

'''Example'''
{|style="border-top:1px solid black" cellpadding="2" width="100%" align="left"
|-
|style="vertical-align:top" width="50%" |
Note that when this file is viewed in ParaView, the Wireframe and Surface With Edges options do not work because a topology has not been inferred.  To better show the vertices, `Filters->Alphabetical->Glyph` was selected.

Note that the number of cells = 3.  In this case each cell is a vertex.
```
# vtk DataFile Version 3.0
PolydataVertices Example 1
ASCII

DATASET POLYDATA

POINTS 3 float
0.0 0.0 0.0
1.0 0.0 0.0
1.0 1.0 0.0

VERTICES 3 6
1 0
1 1
1 2
```

|style="vertical-align:top"|
<imgc>url=https://raw.githubusercontent.com/rweigel/cds301/master/vtk/PolydataVertices.png|width=400px|display=none|expire=1</imgc>
|}

#### Lines 

```
DATASET POLYDATA

POINTS n dataType 
p0x p0y p0z
p1x p1y p1z
...
p(n-1)x p(n-1)y p(n-1)z

LINES n size
numPoints0, i0, j0, k0, ...
numPoints1, i1, j1, k1, ...
...
numPointsn-1, in-1, jn-1, kn-1, ...
```


'''Example'''
{|style="border-top:1px solid black" cellpadding="2" width="100%" align="left"
|-
|style="vertical-align:top" width="50%" |
Note that the number of cells = 2.  In this case a cell is a line.

```
# vtk DataFile Version 3.0
vtk output
ASCII
DATASET POLYDATA

POINTS 3 float
0.0 0.0 0.0
1.0 0.0 0.0
1.0 1.0 0.0

LINES 2 6
2 0 1
2 0 2
```
|style="vertical-align:top"|
<imgc>url=https://raw.githubusercontent.com/rweigel/cds301/master/vtk/PolydataLines.png|width=400px|display=none|expire=1</imgc>
|}

#### Polygons 

```
DATASET POLYDATA

POINTS n dataType 
p0x p0y p0z
p1x p1y p1z
...
p(n-1)x p(n-1)y p(n-1)z

POLYGONS n size
numPoints0, i0, j0, k0, ...
numPoints1, i1, j1, k1, ...
...
numPointsn-1, in-1, jn-1, kn-1, ...
```

'''Example'''
{|style="border-top:1px solid black" cellpadding="2" width="100%" align="left"
|-
|style="vertical-align:top" width="50%" |
```
# vtk DataFile Version 3.0
Polydata + Polygons Example A
ASCII

DATASET POLYDATA
POINTS 4 float
0.0 0.0 0.0
0.0 1.0 0.0
1.0 1.0 0.0
1.0 0.0 0.0

POLYGONS 1 5
4 0 1 2 3    
```
The interpretation of the line `POINTS 4 float` is that there are four points (specified by x,y, and z values), and the values should be interpreted as floating point.

The line `POLYGONS 1 5` indicates that one line will follow and the total number of integers on all following lines is `5`.

|style="vertical-align:top"|
The line `4 0 1 2 3` indicates how the points should be connected.  The `4` indicates that there are four integers to follow.  The polygon is to be drawn by connecting a line from point `0` (located at 0.0, 0.0, 0.0) to point `1` (0.0, 1.0, 0.0) to point `2` (1.0, 1.0, 0.0) to point `3` (1.0, 0.0, 0.0).

Note that in this example, if the POLOYGONS section was omitted, no wireframe or points can be drawn when the file is opened in ParaView.
<imgc>url=https://raw.githubusercontent.com/rweigel/cds301/master/vtk/PolydataPolygonsA.png|width=400px|display=none|expire=1</imgc>
|}

'''Example'''
{|style="border-top:1px solid black" cellpadding="2" width="100%" align="left"
|-
|style="vertical-align:top" width="50%" |
In the following example, the first polygon is created using the first four points as in the previous example.

The second polygon is drawn by connecting point `0` (0.0, 0.0, 0.0) to point `4` (0.0, 0.0, 1.0) to point `5` (0.0, 1.0, 1.0) to point `1` (0.0, 1.0, 0.0).

```
# vtk DataFile Version 3.0
Polydata + Polygons Example B
ASCII
DATASET POLYDATA
POINTS 6 float
0.0 0.0 0.0
0.0 1.0 0.0
1.0 1.0 0.0
1.0 0.0 0.0
0.0 0.0 1.0
0.0 1.0 1.0

POLYGONS 2 10
4 0 1 2 3
4 0 4 5 1
```
|style="vertical-align:top"|
<imgc>url=https://raw.githubusercontent.com/rweigel/cds301/master/vtk/PolydataPolygonsB.png|width=400px|display=none|expire=1</imgc>
|}

#### Triangle Strips 

```
DATASET POLYDATA

POINTS n dataType 
p0x p0y p0z
p1x p1y p1z
...
p(n-1)x p(n-1)y p(n-1)z

TRIANGLE_STRIPS n size
numPoints0, i0, j0, k0, ...
numPoints1, i1, j1, k1, ...
...
numPointsn-1, in-1, jn-1, kn-1, ...
```

'''Example'''
{|style="border-top:1px solid black" cellpadding="2" width="100%" align="left"
|-
|style="vertical-align:top" width="50%" |
```
# vtk DataFile Version 3.0
Triangle Strips Example A
ASCII
DATASET POLYDATA

POINTS 3 float
0.0 0.0 0.0
0.0 1.0 0.0
1.0 0.0 0.0

TRIANGLE_STRIPS 1 4
3 0 1 2
```
|style="vertical-align:top"|
<imgc>url=https://raw.githubusercontent.com/rweigel/cds301/master/vtk/PolydataTriangleStripsA.png|width=400px|display=none|expire=1</imgc>
|}

'''Example'''
{|style="border-top:1px solid black" cellpadding="2" width="100%" align="left"
|-
|style="vertical-align:top" width="50%" |
```
# vtk DataFile Version 3.0
Triangle Strips Example B
ASCII
DATASET POLYDATA

POINTS 5 float
0.0 0.0 0.0
0.0 1.0 0.0
1.0 0.0 0.0
1.0 0.9 0.0
2.0 0.3 0.0

TRIANGLE_STRIPS 1 6
5 0 1 2 3 4
```
|style="vertical-align:top"|
<imgc>url=https://raw.githubusercontent.com/rweigel/cds301/master/vtk/PolydataTriangleStripsB.png|width=400px|display=none|expire=1</imgc>
|}

#### Combinations 

One can re-use the points to create combinations of surface graphics primitives.

'''Example'''
{|style="border-top:1px solid black" cellpadding="2" width="100%" align="left"
|-
|style="vertical-align:top" width="50%" |

```
# vtk DataFile Version 3.0
vtk output
ASCII

DATASET POLYDATA

POINTS 3 float
0.0 0.0 0.0
1.0 0.0 0.0
1.0 1.0 0.0

LINES 2 6
2 0 1
2 0 2

VERTICES 3 6
1 0
1 1
1 2
```
|style="vertical-align:top"|
<imgc>url=https://raw.githubusercontent.com/rweigel/cds301/master/vtk/PolydataVertices_and_Lines.png|width=400px|display=none|expire=1</imgc>
Note that the number of cells is 5 (see the Information tab in ParaView).  There are two lines and three vertices.  Without additional attributes applied to them, they are difficult to distinguish using ParaView.
|}

### Unstructured Grid

All of the above datasets can be represented as an unstructured grid.

```
DATASET UNSTRUCTURED_GRID

POINTS n dataType
p0x p0y p0z
p1x p1y p1z
...
p(n-1)x p(n-1)y p(n-1)z

CELLS n size
numPoints0, i, j, k, l, ...
numPoints1, i, j, k, l, ...
numPoints2, i, j, k, l, ...
...
numPointsn-1, i, j, k, l, ...

CELL_TYPES n
type0
type1
type2
...
typen-1
```

From [http://www.paraview.org/Wiki/index.php?title=ParaView/Users_Guide/VTK_Data_Model]:
<blockquote>
An unstructured grid such as [http://www.paraview.org/Wiki/images/3/35/ParaView_UG_Unstructured.png Figure 3.8] is the most general primitive dataset type. It stores topology and point coordinates explicitly. Even though VTK uses a memory-efficient data structure to store the topology, an unstructured grid uses significantly more memory to represent its mesh. Therefore, use an unstructured grid only when you cannot represent your dataset as one of the above datasets. VTK supports a large number of cell types, all of which can exist (heterogeneously) within one unstructured grid. The full list of all cell types supported by VTK can be found in the file vtkCellType.h in the VTK source code.
</blockquote> 

From [http://www.vtk.org/VTK/img/file-formats.pdf]:
<imgc>url=https://raw.githubusercontent.com/rweigel/cds301/master/vtk/refs/file-formats-figure-2.png|width=500px|display=none|expire=1</imgc>

#### Polyline (CELL_TYPE = 4) 

'''Example'''
{|style="border-top:1px solid black" cellpadding="2" width="100%" align="left"
|-
|style="vertical-align:top" width="50%" |
```
# vtk DataFile Version 3.0
vtk output
ASCII
DATASET UNSTRUCTURED_GRID

POINTS 5 float
0.0 0.0 0.0
0.0 1.0 0.0
2.0 0.0 0.0
2.0 1.0 0.0
3.0 0.0 0.0

CELLS 1 6
5 0 1 2 3 4

CELL_TYPES 1
4
```
|style="vertical-align:top"|
<imgc>url=https://raw.githubusercontent.com/rweigel/cds301/master/vtk/UnstructuredGridPolyline.png|width=400px|display=none|expire=1</imgc>
|}

#### Polygon (CELL_TYPE = 7) 
    
'''Example'''
{|style="border-top:1px solid black" cellpadding="2" width="100%" align="left"
|-
|style="vertical-align:top" width="50%" |
```
# vtk DataFile Version 3.0
vtk output
ASCII
DATASET UNSTRUCTURED_GRID

POINTS 5 float
0.0 0.0 0.0
0.0 1.0 0.0
2.0 0.0 0.0
2.0 1.0 0.0
3.0 0.0 0.0

CELLS 1 6
5 0 2 4 3 1

CELL_TYPES 1
7
```
|style="vertical-align:top"|
Note that the same set of points is used as in the previous example (and are listed in the same order).

<imgc>url=https://raw.githubusercontent.com/rweigel/cds301/master/vtk/UnstructuredGridPolygon.png|width=400px|display=none|expire=1</imgc>
|}


#### Pyramid (CELL_TYPE = 14) 

'''Example'''
{|style="border-top:1px solid black" cellpadding="2" width="100%" align="left"
|-
|style="vertical-align:top" width="50%" |
```
# vtk DataFile Version 3.0
Unstructured Grid - Pyramid
ASCII
DATASET UNSTRUCTURED_GRID

POINTS 5 float
0.0 0.0 0.0
1.0 0.0 0.0
1.0 1.0 0.0
0.0 1.0 0.0
0.5 0.2 1.0

CELLS 1 6
5 0 1 2 3 4

CELL_TYPES 1
14
```
|style="vertical-align:top"|
Note that the same set of points is used as in the previous example (and are listed in the same order).

<imgc>url=https://raw.githubusercontent.com/rweigel/cds301/master/vtk/UnstructuredGridPyramid.png|width=400px|display=none|expire=1</imgc>
|}

### Field

From [http://www.vtk.org/wp-content/uploads/2015/04/file-formats.pdf]:

<blockquote>
Field data is a general format without topological and geometric structure, and without a particular dimensionality. Typically field data is associated with the points or cells of a dataset. However, if the FIELD type is specified as the dataset type (see Figure 1), then a general VTK data object is defined. Use the format described in the next section to define a field. Also see “Working With Field Data” on page 158 and the fourth example in this chapter “Examples” on page 7.
</blockquote>

The following VTK file will not load in ParaView.  See [http://www.paraview.org/Bug/view.php?id=5887] for an explanation.

```
# vtk DataFile Version 2.0
Field data example
ASCII
FIELD speed 2

TIME 1 5 float
0.0 1.0 2.0 3.0 4.0

SPEED 1 5 float
10.0 11.0 8.0 12.0 13.0
```

### Table

### AMR

### Multiblock

### Multipiece

## Dataset Attributes 

### Overview

Each part of a dataset (points and cells) may have an associated (type of attribute) scalar, vector, normal, tensor, texture coordinates, and field.

The text in this overview section is from [http://www.vtk.org/VTK/img/file-formats.pdf].  Examples are given in the sections following this overview.

The Visualization Toolkit supports the following dataset attributes: scalars (one to four components), vectors, normals, texture coordinates (1D, 2D, and 3D), 3 × 3 tensors, and field data. In addition, a lookup table
using the RGBA color specification, associated with the scalar data, can be defined as well. Dataset attributes are supported for both points and cells.

Each type of attribute data has a dataName associated with it. This is a character string (without embedded
whitespace) used to identify a particular data. The dataName is used by the VTK readers to extract data. As a result, more
than one attribute data of the same type can be included in a file. For example, two different scalar fields defined on the
dataset points, pressure and temperature, can be contained in the same file. (If the appropriate dataName is not specified in
the VTK reader, then the first data of that type is extracted from the file.)

'''Scalars'''

Scalar definition includes specification of a lookup table. The definition of a lookup table is optional. If not specified, the default VTK table will be used (and tableName should be “default”). Also note that the numComp variable is optional - by default the number of components is equal to one. (The parameter numComp must range between (1,4) inclusive; in versions of VTK prior to vtk 2.3 this parameter was not supported.)

```
SCALARS dataName dataType numComp
LOOKUP_TABLE tableName
s0
s1
...
s(n-1)
```

The definition of color scalars (i.e., unsigned char values directly mapped to color) varies depending upon the
number of values (nValues) per scalar. If the file format is ASCII, the color scalars are defined using nValues float
values between (0,1). If the file format is BINARY, the stream of data consists of nValues unsigned char values per
scalar value.

```
COLOR_SCALARS dataName nValues
c00 c01 ... c0(nValues-1)
c10 c11 ... c1(nValues-1)
...
c(n-1)0 c(n-1)1 ... c(n-1)(nValues-1)
```

'''Lookup Table'''

The tableName field is a character string (without imbedded white space) used to identify the lookup table. This label is used by the VTK reader to extract a specific table.  Each entry in the lookup table is a rgba[4] (red-green-blue-alpha) array (alpha is opacity where alpha=0 is transparent). If the file format is ASCII , the lookup table values must be float values between (0,1). If the file format is BINARY , the stream of data must be four unsigned char values per table entry.
```
LOOKUP_TABLE tableName size
r0 g0 b0 a0
r1 g1 b1 a1
...
r(size-1) g(size-1) b(size-1) a(size-1)
```

'''Vectors'''

```
VECTORS dataName dataType
v0x v0y v0z
v1x v1y v1z
...
v(n-1)x v(n-1)y v(n-1)z
```

'''Normals'''
Normals are assumed normalized |n| = 1.

```
NORMALS dataName dataType
n0x n0y n0z
n1x n1y n1z
...
n(n-1)x n(n-1)y n(n-1)z
```

'''Texture Coordinates'''

Texture coordinates of 1, 2, and 3 dimensions are supported.

```
TEXTURE_COORDINATES dataName dim dataType
t00 t01 ... t0(dim-1)
t10 t11 ... t1(dim-1)
...
t(n-1)0 t(n-1)1 ... t(n-1)(dim-1)
```

'''Tensors'''

Currently only 3 × 3 real-valued, symmetric tensors are supported.

```
TENSORS dataName dataType
t^0_00 t^0_01 t^0_02
t^0_10 t^0_11 t^0_12
t^0_20 t^0_21 t^0_22
t^1_00 t^1_01 t^1_02
t^1_10 t^1_11 t^1_12
t 1_20 t^1_21 t^1_22
...
t^n-1_00 t^n-1_01 t^n-1_02
t^n-1_10 t^n-1_11 t^n-1_12
t^n-1_20 t^n-1_21 t^n-1_22
```

'''Field Data'''

Field data is essentially an array of data arrays. Defining field data means giving a name to the field and specifying
the number of arrays it contains. Then, for each array, the name of the array arrayName(i), the number of components of the array, numComponents, the number of tuples in the array, numTuples, and the data type, dataType, are defined.

```
FIELD dataName numArrays
arrayName0 numComponents numTuples dataType
f00 f01 ... f0(numComponents-1)
f10 f11 ... f1(numComponents-1)
...
f(numTuples-1)0 f(numTuples-1)1 ... f(numTuples-1)(numComponents-1)
arrayName1 numComponents numTuples dataType
f00 f01 ... f0(numComponents-1)
f10 f11 ... f1(numComponents-1)
...
f(numTuples-1)0 f(numTuples-1)1 ... f(numTuples-1)(numComponents-1)
...
arrayName(numArrays-1) numComponents numTuples dataType
f00 f01 ... f0(numComponents-1)
f10 f11 ... f1(numComponents-1)
...
f(numTuples-1)0 f(numTuples-1)1 ... f(numTuples-1)(numComponents-1)
```

### POINT_DATA

'''Example - scalars and default lookup'''
{|style="border-top:1px solid black" cellpadding="2" width="100%" align="left"
|-
|style="vertical-align:top" width="50%" |
```
# vtk DataFile Version 3.0
Dataset attribute example
ASCII
DATASET STRUCTURED_GRID
DIMENSIONS 2 2 1
POINTS 4 float
0 0 0
1 0 0
0 1 0
1 1 0

POINT_DATA 4
SCALARS point_scalars int 1
LOOKUP_TABLE default
0
0
3
3
```

|style="vertical-align:top"|

Surface representation
<imgc>url=https://raw.githubusercontent.com/rweigel/cds301/master/vtk/StructuredGridA_point_data_scalar_default.png|width=400px|display=none|expire=1</imgc>

The following image was created by selecting a representation of "Points", `Filters->Alphabetical->Glyphs`, Glyph Type of Sphere, and a scale mode of scalar.  This scales the glyphs at each point according to their relative size.  In addition, the spheres should be color-coded and a colorbar should indicate the mapping from color to scalar value.

<imgc>url=https://raw.githubusercontent.com/rweigel/cds301/master/vtk/StructuredGridA_point_data_scalar_default_glyphs.png|width=400px|display=none|expire=1</imgc>

|}

'''Example - scalars and defined lookup table'''
{|style="border-top:1px solid black" cellpadding="2" width="100%" align="left"
|-
|style="vertical-align:top" width="50%" |
In this example, the points are explicitly assigned a color from a lookup table.  

A useful list of color tables is given at http://www.earthmodels.org/data-and-tools/color-tables.
```
# vtk DataFile Version 3.0
Dataset attribute example
ASCII
DATASET STRUCTURED_GRID
DIMENSIONS 2 2 1
POINTS 4 float
0 0 0
1 0 0
0 1 0
1 1 0

POINT_DATA 4
SCALARS point_scalars int 1
LOOKUP_TABLE my_table
0
0
1
1

LOOKUP_TABLE my_table 4
1.0 0.0 0.0 1.0
1.0 0.0 0.0 1.0
0.0 0.0 1.0 1.0
0.0 0.0 1.0 1.0
```

Note that in ParaView 4.2, the colorbar shown may have the wrong colors [https://raw.githubusercontent.com/rweigel/cds301/master/vtk/StructuredGridA_point_data_scalars_w_lookup_good_colorbar.png] - one needs to manually change the colormap for the colorbar (including the number of discrete colors) to match the view. In addition, the scalar values of the points must be between 0 and 1.
|style="vertical-align:top"|
Surface representation of file.
<imgc>url=https://raw.githubusercontent.com/rweigel/cds301/master/vtk/StructuredGridA_point_data_scalars_w_lookup_good_colorbar.png|width=400px|display=none|expire=1</imgc>
|}

'''Example - Vectors'''

<span style="background-color:yellow">In ParaView 5.0, coloring is different</span>

{|style="border-top:1px solid black" cellpadding="2" width="100%" align="left"
|-
|style="vertical-align:top" width="50%" |
```
# vtk DataFile Version 3.0
Dataset attribute example
ASCII
DATASET STRUCTURED_GRID
DIMENSIONS 2 2 1
POINTS 4 float
0 0 0
1 0 0
0 1 0
1 1 0

POINT_DATA 4
VECTORS point_vectors float
1.0 0.0 0.0
0.0 1.0 0.0
0.0 0.0 1.0
1.0 1.0 1.0
```

|style="vertical-align:top"|

The following image was generated by loading the file, selecting Apply, and selecting a representation of surface and coloring by point_vectors using the magnitude of the vector.

<imgc>url=https://raw.githubusercontent.com/rweigel/cds301/master/vtk/StructuredGridA_point_data_vectors.png|width=400px|display=none|expire=1</imgc>

The following image was generated by loading the file, selecting Apply, and selecting a representation of wireframe and then `Filters > Alphabetical > Glyph`

<imgc>url=https://raw.githubusercontent.com/rweigel/cds301/master/vtk/StructuredGridA_point_data_vectors2.png|width=400px|display=none|expire=1</imgc>

|}

'''Example - field'''
{|style="border-top:1px solid black" cellpadding="2" width="100%" align="left"
|-
|style="vertical-align:top" width="50%" |
In this example, each face has two associated field attributes.  The first is an ID (ParaView automatically assigns IDs, so this is not necessary).  Each face also has an an attribute with four values.  In ParaView, one can select the array element to color by (the default is the magnitude). 
```
# vtk DataFile Version 2.0
Cube example
ASCII
DATASET POLYDATA
POINTS 8 float
0.0 0.0 0.0
1.0 0.0 0.0
1.0 1.0 0.0
0.0 1.0 0.0
0.0 0.0 1.0
1.0 0.0 1.0
1.0 1.0 1.0
0.0 1.0 1.0

POLYGONS 6 30
4 0 1 2 3
4 4 5 6 7
4 0 1 5 4
4 2 3 7 6
4 0 4 7 3
4 1 2 6 5

CELL_DATA 6
SCALARS cell_scalars int 1
LOOKUP_TABLE default
0
1
2
3
4
5

FIELD FieldData 2
cellIds 1 6 int
0 1 2 3 4 5
faceAttributes 4 6 float
0.0 1.0 2.0 3.0
1.0 2.0 2.0 3.0
2.0 3.0 2.0 3.0
3.0 4.0 2.0 3.0
4.0 5.0 2.0 3.0
5.0 6.0 2.0 3.0
```

|style="vertical-align:top"|

|}

### CELL_DATA

'''Example'''
{|style="border-top:1px solid black" cellpadding="2" width="100%" align="left"
|-
|style="vertical-align:top" width="50%" |
```
# vtk DataFile Version 3.0
Polydata + Polygons Example B with Cell Data
ASCII

DATASET POLYDATA
POINTS 6 float
0.0 0.0 0.0
0.0 1.0 0.0
1.0 1.0 0.0
1.0 0.0 0.0
0.0 0.0 1.0
0.0 1.0 1.0

POLYGONS 2 10
4 0 1 2 3
4 0 4 5 1

CELL_DATA 2
SCALARS cell_scalars int 1
LOOKUP_TABLE default
1
2
```
|style="vertical-align:top"|
<imgc>url=https://raw.githubusercontent.com/rweigel/cds301/master/vtk/PolydataPolygonsB_cell_data_scalar_default.png|width=400px|display=none|expire=1</imgc>
|}

### POINT_DATA and CELL_DATA

Point data and cell data attributes may both appear in a file.  When viewed in ParaView, there will be an option to color by either one of these attributes.

'''Example'''
{|style="border-top:1px solid black" cellpadding="2" width="100%" align="left"
|-
|style="vertical-align:top" width="50%" |
```
# vtk DataFile Version 3.0
Polydata + Polygons Example B with Cell Data
ASCII

DATASET POLYDATA
POINTS 6 float
0.0 0.0 0.0
0.0 1.0 0.0
1.0 1.0 0.0
1.0 0.0 0.0
0.0 0.0 1.0
0.0 1.0 1.0

POLYGONS 2 10
4 0 1 2 3
4 0 4 5 1

CELL_DATA 2
SCALARS cell_scalars int 1
LOOKUP_TABLE default
1
2

POINT_DATA 6
SCALARS point_scalars int 1
LOOKUP_TABLE default
1
1
2
2
3
3
```

|style="vertical-align:top"|

Coloring by point_scalars selected
<imgc>url=https://raw.githubusercontent.com/rweigel/cds301/master/vtk/PolydataPolygonsB_cell_data_and_point_data_scalar_default_view_1.png|width=400px|display=none|expire=1</imgc>

Coloring by cell_scalars selected
<imgc>url=https://raw.githubusercontent.com/rweigel/cds301/master/vtk/PolydataPolygonsB_cell_data_and_point_data_scalar_default_view_2.png|width=400px|display=none|expire=1</imgc>

|}

## Problems - Datasets 

### Points and Lines I

Covered in [[#Introduction]].

Create a VTK file that specifies four points that form a square connected by four individual lines using the `POLYDATA` dataset type.  When your file is opened in ParaView and the wireframe view is selected, you should see a square formed by white lines displayed.  When you select the Information tab, you should see "Number of Cells: 4".

{| class="wikitable collapsible collapsed"
! align="left" |&nbsp;Answer
|-
|
```
# vtk DataFile Version 3.0
Square using 4 lines
ASCII
DATASET POLYDATA
POINTS 4 float
0 0 0
1 0 0
1 1 0
0 1 0

LINES 4 12
2 0 1
2 1 2
2 2 3
2 3 0
```
|}

### Points and Lines II

Covered in [[#Introduction]].

Create a VTK file that specifies four points that form a square connected by a single line using the `POLYDATA` dataset type.  When the file is opened in ParaView and the wireframe view is selected, you should see a square displayed.  When you select the Information tab, you should see "Number of Cells: 1".

{| class="wikitable collapsible collapsed"
! align="left" |&nbsp;Answer
|-
|
```
# vtk DataFile Version 2.0
Square using one line
ASCII
DATASET POLYDATA
POINTS 4 float
0 0 0
1 0 0
1 1 0
0 1 0

LINES 1 6
5 0 1 2 3 0
```

or

```
# vtk DataFile Version 3.0
Polydata + Polygon
ASCII

DATASET POLYDATA
POINTS 4 float
0.0 0.0 0.0
0.0 1.0 0.0
1.0 1.0 0.0
1.0 0.0 0.0

POLYGONS 1 5
4 0 1 2 3
```

Note that this could have been achieved using 5 points using the following.  The disadvantage of this approach is the redundant point and the fact that software may not recognize that the first and last points are actually identical (ParaView gives a point ID of 0 and 4 to the position 0,0,0).

```
# vtk DataFile Version 3.0
A dataset with one polyline and no attributes
ASCII

DATASET POLYDATA
POINTS 5 float
0.0 0.0 0.0
1.0 0.0 0.0
1.0 1.0 0.0
0.0 1.0 0.0 
0.0 0.0 0.0

LINES 1 6
5 0 1 2 3 4
```
|}

### Structured Points I

<span style="background-color:yellow">In ParaView 5.0, this does not work</span>

Covered in [[#Structured_Points]].

Create a VTK file that specifies four points that form a square using a dataset of type `STRUCTURED_POINTS`. When your file is opened in ParaView and viewed as a wireframe, you should see a square displayed.  When you select the Information tab, you should see "Number of Cells: 1".

(This is an alternative method of creating the same dataset as in the previous problem, which used a dataset of type `POLYDATA`.)

{| class="wikitable collapsible collapsed"
! align="left" |&nbsp;Answer
|-
|
```
# vtk DataFile Version 3.0
Square using structured points
ASCII

DATASET STRUCTURED_POINTS
DIMENSIONS 2 2 1
ORIGIN 0.0 0.0 0.0
SPACING 1.0 1.0 1.0
```
|}

### Structured Points II

Covered in [[#Structured_Points]].

{| border="0" cellpadding="2" width="100%" align="left"
|-
|style="vertical-align:top" width="50%" |
Create a VTK file that, when loaded in ParaView and viewed as a Wireframe, results in the image shown.

The displayed object is a cube with sides of length 1.0.

{| class="wikitable collapsible collapsed"
! align="left" |&nbsp;Answer
|-
|
```
# vtk DataFile Version 3.0
Structured Points Example B
ASCII

DATASET STRUCTURED_POINTS
DIMENSIONS 2 2 2
ORIGIN 0.0 0.0 0.0
SPACING 1.0 1.0 1.0
```
Note that in the Information tab, the number of cells is given as 1.0.  However, when coloring by cellNormals in various directions, it appears that there are six cells - the reason is that cellNormals is better labeled as "cell surface normals".
|}

|style="vertical-align:top"|
<imgc>url=https://raw.githubusercontent.com/rweigel/cds301/master/vtk/StructuredPointsB.png|width=400px|display=none|expire=1</imgc>
|}

### Structured Points III

Covered in [[#Structured_Points]].

{| border="0" cellpadding="2" width="100%" align="left"
|-
|style="vertical-align:top" width="50%" |
Create a VTK file that, when loaded in ParaView and viewed as a Wireframe, results in the image shown. The outer part of the object shown is a cube with sides of length 1.0.

{| class="wikitable collapsible collapsed"
! align="left" |&nbsp;Answer
|-
|
```
# vtk DataFile Version 3.0
Structured Points Example C
ASCII

DATASET STRUCTURED_POINTS
DIMENSIONS 2 2 5
ORIGIN 0.0 0.0 0.0
SPACING 1.0 1.0 0.2
```
Note that in the Information tab of ParaView, the number of cells is given as 5.
|}

|style="vertical-align:top"|
<imgc>url=https://raw.githubusercontent.com/rweigel/cds301/master/vtk/StructuredPointsC.png|width=400px|display=none|expire=1</imgc>
|}

### Structured Points IV

{| border="0" cellpadding="2" width="100%" align="left"
|-
|style="vertical-align:top" width="50%" |
Covered in [[#Structured_Points]].

Create a VTK file using `STRUCTURED_POINTS` that, when loaded in ParaView, shows the following when viewed as a wireframe. Note that the small cubes in the diagram all have sides of length 1.
|style="vertical-align:top"|
<imgc>url=https://raw.githubusercontent.com/rweigel/cds301/master/vtk/StructuredPointsIV.png|width=400px|display=none|expire=1</imgc>
|}


{| class="wikitable collapsible collapsed"
! align="left" |&nbsp;Answer
|-
|
```
# vtk DataFile Version 3.0
Structured Points Example
ASCII

DATASET STRUCTURED_POINTS
DIMENSIONS 4 4 4
ORIGIN 0.0 0.0 0.0
SPACING 1.0 1.0 1.0
```
|}

### Rectilinear Grid I

{| border="0" cellpadding="2" width="100%" align="left"
|-
|style="vertical-align:top" width="50%" |
Create a VTK file that, when loaded in ParaView and viewed as a wireframe, results in the image shown.

Note that if you have an error in your VTK file, ParaView may crash or show an error message (which is usually not helpful in identifying the problem).

{| class="wikitable collapsible collapsed"
! align="left" |&nbsp;Answer
|-
|
```
# vtk DataFile Version 3.0
Rectilinear Grid Example C
ASCII

DATASET RECTILINEAR_GRID
DIMENSIONS 5 4 1
X_COORDINATES 5 float
0 1 2 3 4
Y_COORDINATES 4 float
1 2 4 8
Z_COORDINATES 1 float
0
```
|}
|style="vertical-align:top"|
<imgc>url=https://raw.githubusercontent.com/rweigel/cds301/master/vtk/RectilinearGridC.png|width=400px|display=none|expire=1</imgc>
|}

### Rectilinear Grid II

{| border="0" cellpadding="2" width="100%" align="left"
|-
|style="vertical-align:top" width="50%" |
Create a VTK file that, when loaded in ParaView and viewed as a wireframe, results in the image shown.

Note that if you have an error in your VTK file, ParaView may crash or show an error message (which is usually not helpful in identifying the problem).

{| class="wikitable collapsible collapsed"
! align="left" |&nbsp;Answer
|-
|
```
# vtk DataFile Version 3.0
Rectilinear Grid Example D
ASCII

DATASET RECTILINEAR_GRID
DIMENSIONS 5 4 2
X_COORDINATES 5 float
0 1 2 3 4
Y_COORDINATES 4 float
1 2 4 8
Z_COORDINATES 2 float
0 1 
```
|}
|style="vertical-align:top"|
<imgc>url=https://raw.githubusercontent.com/rweigel/cds301/master/vtk/RectilinearGridD.png|width=400px|display=none|expire=1</imgc>
|}

### Triangle Strips I

{|style="border-top:1px solid black" cellpadding="2" width="100%" align="left"
|-
|style="vertical-align:top" width="50%" |

Create the following triangle strip (shown as a wireframe).

{| class="wikitable collapsible collapsed"
! align="left" |&nbsp;Answer
|-
|
```
# vtk DataFile Version 3.0
vtk output
ASCII
DATASET POLYDATA

POINTS 6 float
0.0 0.0 0.0
0.0 1.0 0.0
1.0 0.0 0.0
1.0 0.9 0.0
2.0 0.3 0.0
2.0 0.8 0.0

TRIANGLE_STRIPS 1 7
6 0 1 2 3 4 5
```
|}

|style="vertical-align:top"|
<imgc>url=https://raw.githubusercontent.com/rweigel/cds301/master/vtk/PolydataTriangleStripsC.png|width=400px|display=none|expire=1</imgc>
|}

## Problems - Dataset Attributes 

### POINT_DATA SCALARS

Covered in [[#POINT_DATA]].

The following VTK file specifies a grid along with scalars associated with each grid point.  
```
# vtk DataFile Version 3.0
Structured Points Example B
ASCII

DATASET STRUCTURED_POINTS
DIMENSIONS 2 2 2
ORIGIN 0.0 0.0 0.0
SPACING 1.0 1.0 1.0

POINT_DATA 8
SCALARS point_scalars int 1
LOOKUP_TABLE default
0
1
2
3
4
5
6
7
```

Use ParaView to determine how values are assigned to each of the corners.  That is, replace the `?` in the following with the appropriate scalar value.
```
Corner at (0,0,0), scalar value = ?
Corner at (1,0,0), scalar value = ?
Corner at (0,1,0), scalar value = ?
Corner at (0,0,1), scalar value = ?
Corner at (1,1,0), scalar value = ?
Corner at (0,1,1), scalar value = ?
Corner at (1,0,1), scalar value = ?
Corner at (1,1,1), scalar value = ?
```

{| class="wikitable collapsible collapsed"
! align="left" |&nbsp;Answer
|-
|

```
Corner at (0,0,0), scalar value = 0
Corner at (1,0,0), scalar value = 1
Corner at (0,1,0), scalar value = 2
Corner at (0,0,1), scalar value = 4
Corner at (1,1,0), scalar value = 3
Corner at (0,1,1), scalar value = 6
Corner at (1,0,1), scalar value = 5
Corner at (1,1,1), scalar value = 7
```

There are multiple ways of doing this.

1. Render as a surface, color by `point_scalars` and edit the color map to have 8 colors and use the colormap to determine the value of each corner.
2. Create a new tab, select SpreadSheet View, highlight all of the numbers, select `View > Selection Display Inspector` and then select `Cell Labels > ID`.

From this, you can infer that the way values are assigned to points is
```
 P = [0:7]
 for k in z
    for j in y
      for i in x
         Assign point at x(i),y(k),z(k) value P(p)
         p = p+1
      end
    end
 end 
```

## CELL_DATA SCALARS

Covered in [[#CELL_DATA]].

Provide a VTK file along with instructions on what to do in ParaView to show a cube with each face having a different color.

**Answer**

1. Load file in ParaView and select Apply
2. Select Representation of Surface
3. Select Color By `cell_scalars`
4. Select Rescale (may not be necessary)
5. If there are not 6 colors, select `Coloring > Edit` and change the number of table values to 6.

```
# vtk DataFile Version 3.0
Cube with different colored sides
ASCII

DATASET POLYDATA
POINTS 8 float
0 0 0
1 0 0
1 1 0
0 1 0
0 0 1
1 0 1
1 1 1
0 1 1

POLYGONS 6 30
4 0 1 2 3
4 0 3 7 4
4 0 1 5 4
4 4 5 6 7
4 3 2 6 7
4 1 2 6 5

CELL_DATA 6
SCALARS cell_scalars int 1
LOOKUP_TABLE default
1
2
3
4
5
6
```

## POINT_DATA VECTORS

Covered in [[#POINT_DATA]].  

Modify the following file so that the vector in the top of the image points in the -z direction (instead of in the +x direction).

Note that to create the image, Arrow Glyphs were chosen, the scale factor for the glyphs was set to 0.2, and the coloring was changed (using `Coloring > Edit`).

```
# vtk DataFile Version 3.0
Structured Points Example B
ASCII

DATASET STRUCTURED_POINTS
DIMENSIONS 2 2 2
ORIGIN 0.0 0.0 0.0
SPACING 1.0 1.0 1.0

POINT_DATA 8
VECTORS point_vectors float
0.0 0.0 1.0
0.0 0.0 1.0
0.0 0.0 1.0
0.0 0.0 1.0
1.0 0.0 0.0
1.0 0.0 0.0
1.0 0.0 0.0
1.0 0.0 0.0
```

|style="vertical-align:top"|
<imgc>url=https://raw.githubusercontent.com/rweigel/cds301/master/vtk/StructuredPointsB_vector_attributes.png|width=400px|display=none|expire=1</imgc>


**Answer**

```
# vtk DataFile Version 3.0
Structured Points with Vectors
ASCII

DATASET STRUCTURED_POINTS
DIMENSIONS 2 2 2
ORIGIN 0.0 0.0 0.0
SPACING 1.0 1.0 1.0

POINT_DATA 8
VECTORS point_vectors float
0.0 0.0 1.0
0.0 0.0 1.0
0.0 0.0 1.0
0.0 0.0 1.0
1.0 0.0 0.0
1.0 0.0 0.0
1.0 0.0 -1.0
1.0 0.0 0.0
```

## POINT_DATA VECTORS

The wind velocity was measured at eight positions as indicated in the following table.

```
x    y   z    vx  vy  vz
0.0 0.0 0.0  1.0 1.0 0.0
1.0 0.0 0.0  1.0 2.0 0.0
1.0 1.0 0.0  2.0 2.0 0.0
0.0 1.0 0.0  2.0 2.0 0.0
0.0 0.0 1.0  1.0 1.0 0.0
1.0 0.0 1.0  1.0 2.0 0.0
1.0 1.0 1.0  2.0 2.0 0.0
0.0 1.0 1.0  2.0 2.0 0.0
```

Create a VTK file that represents the data in these table.  Provide instructions for using ParaView for creating a visualization of these data.

**Answer**

Note that these vectors can be checked using the selection display inspector, which was used to create the image that follows.

```
# vtk DataFile Version 3.0
Velocity vectors based on table
ASCII

DATASET STRUCTURED_POINTS
DIMENSIONS 2 2 2
ORIGIN 0 0 0
SPACING 1 1 1

POINT_DATA 8
VECTORS point_vectors float
1.0 1.0 0.0
1.0 2.0 0.0
2.0 2.0 0.0
2.0 2.0 0.0
1.0 1.0 0.0
1.0 2.0 0.0
2.0 2.0 0.0
2.0 2.0 0.0
```

[[Image:tablewithvectors.png|400px]]

Note that some students answered this question using a STRUCUTRED_GRID with points specified in an order that leads to an invalid cell (to see this select a representation of surface - you'll see a black surface.  In addition, coloring by cell normals will not work). 

```
DATASET STRUCTURED_GRID
DIMENSIONS 2 2 2
POINTS 8 float
0.0 0.0 0.0
1.0 0.0 0.0
1.0 1.0 0.0
0.0 1.0 0.0
0.0 0.0 1.0
1.0 0.0 1.0
1.0 1.0 1.0
0.0 1.0 1.0

POINT_DATA 8
VECTORS point_vectors float
1.0 1.0 0.0
1.0 2.0 0.0
2.0 2.0 0.0
2.0 2.0 0.0
1.0 1.0 0.0
1.0 2.0 0.0
2.0 2.0 -1.0
2.0 2.0 0.0
```

|}

## Problems - Creating VTK Files 

### Creating VTK Files I

Create a VTK file such that when the following steps are followed, I see 1000 spheres with random colors between blue and red located at arbitrary 3-D positions.

1. Open file in ParaView
2. Select Apply
3. Select Representation of Points
4. Select `Filters > Alphabetical > Glyphs`
5. Select Sphere
6. Select Apply

Hint: See [[#POINT_DATA]].

**Answer**

```
clear;

% Each row is a (x,y,z) value for a point.
P = rand(1000,3);
N = size(P,1);

fid = fopen('HW8_1.vtk','w');

fprintf(fid,'# vtk DataFile Version 3.0\n');
fprintf(fid,'HW 8_1\n');
fprintf(fid,'ASCII\n\n');
fprintf(fid,'DATASET POLYDATA\n');
fprintf(fid,'POINTS %d float\n',N);
%Avoid for loop using this technique
%PT = P';
%fprintf(fid,'%f %f %f\n',PT(:));
for i = 1:N
  fprintf(fid,'%f %f %f\n',P(i,1),P(i,2),P(i,3));
end
fprintf(fid,'\nPOINT_DATA %d\n',N);
fprintf(fid,'SCALARS point_scalars float\n');
fprintf(fid,'LOOKUP_TABLE default\n');
% Row in S is sum of squares of row in P.  Will result in points
% with colors that depend on distance from origin.
S = sum(P.^2,2);
%fprintf(fid,'%f\n',S);
for i = 1:N
  fprintf(fid,'%f\n',S(i));
end
fclose(fid);
```

### Creating VTK Files II 

With a text editor, create a VTK file with only geometry for a cylinder approximated as an object with 8 sides, radius of 1.0, and height of 2.0.  Save your file as `HW8_2.vtk`.

**Answer**


Note - to create this file, drew in 3-D the points that I needed.  Then, I saved a file with these points and used the Selection Display Inspector to view the ID numbers.  Based on the ID numbers, I created the polygons, one for each panel on the cylinder.  Note how the IDs used for the polygons follow a pattern.  This pattern would be used if we were asked to create a cylinder with more sides.

* the first column goes from 0 to N/2.
* the second column goes from 1 to N/2

```
# vtk DataFile Version 3.0
Cylinder
ASCII
DATASET POLYDATA
POINTS 16 float
1.000000 0.000000 0.000000
0.707107 0.707107 0.000000
0.000000 1.000000 0.000000
-0.707107 0.707107 0.000000
-1.000000 0.000000 0.000000
-0.707107 -0.707107 0.000000
0.000000 -1.000000 0.000000
0.707107 -0.707107 0.000000
1.000000 0.000000 2.000000
0.707107 0.707107 2.000000
0.000000 1.000000 2.000000
-0.707107 0.707107 2.000000
-1.000000 0.000000 2.000000
-0.707107 -0.707107 2.000000
0.000000 -1.000000 2.000000
0.707107 -0.707107 2.000000

POLYGONS 8 40
4 0 1 9 8
4 1 2 10 9
4 2 3 11 10
4 3 4 12 11
4 4 5 13 12
4 5 6 14 13
4 6 7 15 14
4 7 0 8 15
```

## Generating VTK files with a program

### Background 
To generate a VTK file using MATLAB, the function `fprintf` can be used.  (Details on `fprintf` are given in [[IO]].)

Consider the file

```
# vtk DataFile Version 3.0
A dataset with one polyline and no attributes
ASCII

DATASET POLYDATA
POINTS 3 float
0.0 0.0 0.0
1.0 0.0 0.0
1.0 1.0 0.0

LINES 1 4
3 0 1 2
```

The above file can be generated using the following program

'''Method 1'''

```
clear;
outfile = 'simple_vtk_method_1.vtk';

% Create and open a file named outfile for writing ('w').
fid = fopen(outfile,'w');

fprintf(fid,'# vtk DataFile Version 3.0\n');
fprintf(fid,'A dataset with one polyline and no attributes\n');
fprintf(fid,'ASCII\n');
fprintf(fid,'\n');
fprintf(fid,'DATASET POLYDATA\n');
fprintf(fid,'POINTS 3 float\n');
fprintf(fid,'0.0 0.0 0.0\n');
fprintf(fid,'1.0 0.0 0.0\n');
fprintf(fid,'1.0 1.0 0.0\n');
fprintf(fid,'\n');
fprintf(fid,'LINES 1 4\n');
fprintf(fid,'3 0 1 2\n');

% Close the file
fclose(fid);

% Display the contents of outfile to screen
type(outfile);
```

'''Method 2''' - use format strings (%f, %d, etc.) in `fprintf`.
```
clear;
outfile = 'simple_vtk_method_2.vtk';

% Create and open a file named outfile for writing ('w').
fid = fopen(outfile,'w');

fprintf(fid,'# vtk DataFile Version 3.0\n');
fprintf(fid,'A dataset with one polyline and no attributes\n');
fprintf(fid,'ASCII\n');
fprintf(fid,'\n');
fprintf(fid,'DATASET POLYDATA\n');
fprintf(fid,'POINTS %d float\n',3);
fprintf(fid,'%f %f %f\n',0.0, 0.0, 0.0);
fprintf(fid,'%f %f %f\n',1.0, 0.0, 0.0);
fprintf(fid,'%f %f %f\n',1.0, 1.0, 0.0');
fprintf(fid,'LINES %d %d\n',1,4);
fprintf(fid,'%d %d %d %d\n',3, 0, 1, 2);
fclose(fid);
type(outfile);
```


'''Method 3''' - use format strings and variable references with `fprintf`.

```
clear;

% Each row is a (x,y,z) value for a point.
P = [0.0, 0.0, 0.0; 1.0, 0.0, 0.0; 1.0, 1.0, 0.0];
Npts = size(P,1);

outfile = 'simple_vtk_method_3.vtk';

% Create and open a file named outfile for writing ('w').
fid = fopen(outfile,'w');

fprintf(fid,'# vtk DataFile Version 3.0\n');
fprintf(fid,'A dataset with one polyline and no attributes\n');
fprintf(fid,'ASCII\n');
fprintf(fid,'\n');
fprintf(fid,'DATASET POLYDATA\n');
fprintf(fid,'POINTS %d float\n',Npts);
fprintf(fid,'%f %f %f\n',P(1,1),P(1,2),P(1,3));
fprintf(fid,'%f %f %f\n',P(2,1),P(2,2),P(2,3));
fprintf(fid,'%f %f %f\n',P(3,1),P(3,2),P(3,3));
fprintf(fid,'LINES %d %d\n',1,4);
fprintf(fid,'%d %d %d %d\n',3, 0, 1, 2);
fclose(fid);
type(outfile);
```

Note that the last two statements 
 fprintf(fid,'LINES %d %d\n',1,4);
 fprintf(fid,'%d %d %d %d\n',3, 0, 1, 2);
could have been written as
 lines = [0:2];
 fprintf(fid,'LINES %d %d\n',1,length(lines) + 1);
 fprintf(fid,'%d %d %d %d\n',length(lines), lines(1), lines(2), lines(3));

'''Method 4''' - use a loop to print out each point.

```
clear;

% Each row is a (x,y,z) value for a point.
P = [0.0, 0.0, 0.0; 1.0, 0.0, 0.0; 1.0, 1.0, 0.0];
Npts = size(P,1);

outfile = 'simple_vtk_method_4.vtk';

% Create and open a file named outfile for writing ('w').
fid = fopen(outfile,'w');

fprintf(fid,'# vtk DataFile Version 3.0\n');
fprintf(fid,'A dataset with one polyline and no attributes\n');
fprintf(fid,'ASCII\n');
fprintf(fid,'\n');
fprintf(fid,'DATASET POLYDATA\n');
fprintf(fid,'POINTS %d float\n',Npts);
for i = 1:Npts
  fprintf(fid,'%f %f %f\n',P(i,1),P(i,2),P(i,3));
end
fprintf(fid,'LINES %d %d\n',1,4);
fprintf(fid,'%d %d %d %d\n',3, 0, 1, 2);
fclose(fid);
type(outfile);
```

Note that the last two statements 
 fprintf(fid,'LINES %d %d\n',1,4);
 fprintf(fid,'%d %d %d %d\n',3, 0, 1, 2);
could have been written as
 lines = [0:2];
 fprintf(fid,'LINES %d %d\n',1,length(lines) + 1);
 fprintf(fid,'%d ',length(lines)); % Note no \n
 for i = 1:length(lines)
   fprintf(fid,'%d ', lines(i)); % Note no \n
 end
 fprintf(fid,'\n');

#### Problem I 

Write four MATLAB programs, one for each of the four methods described above, to create the following VTK file.

```
# vtk DataFile Version 3.0
A dataset with two lines and no attributes
ASCII

DATASET POLYDATA
POINTS 3 float
0.0 0.0 0.0
1.0 0.0 0.0
1.0 1.0 0.0
LINES 2 6
2 0 1
2 1 2
```

{| class="wikitable collapsible collapsed"
! align="left" |&nbsp;Answer
|-
|
'''Method 1'''
```
clear;
outfile = '1.8.2.2_1.vtk';

% Create and open a file named outfile for writing ('w').
fid = fopen(outfile,'w');

fprintf(fid,'# vtk DataFile Version 3.0\n');
fprintf(fid,'A dataset with two lines and no attributes\n');
fprintf(fid,'ASCII\n');
fprintf(fid,'\n');
fprintf(fid,'DATASET POLYDATA\n');
fprintf(fid,'POINTS 3 float\n');
fprintf(fid,'0.0 0.0 0.0\n');
fprintf(fid,'1.0 0.0 0.0\n');
fprintf(fid,'1.0 1.0 0.0\n');
fprintf(fid,'LINES 2 6\n');
fprintf(fid,'2 0 1\n');
fprintf(fid,'2 1 2\n');

% Close the file
fclose(fid);

% Display the contents of outfile to screen
type(outfile);
```

'''Method 2'''
```
clear;
outfile = '1.8.2.2_2.vtk';

% Create and open a file named outfile for writing ('w').
fid = fopen(outfile,'w');

fprintf(fid,'# vtk DataFile Version 3.0\n');
fprintf(fid,'A dataset with two lines and no attributes\n');
fprintf(fid,'ASCII\n');
fprintf(fid,'\n');
fprintf(fid,'DATASET POLYDATA\n');
fprintf(fid,'POINTS %d float\n',3);
fprintf(fid,'%f %f %f\n',0.0, 0.0, 0.0);
fprintf(fid,'%f %f %f\n',1.0, 0.0, 0.0);
fprintf(fid,'%f %f %f\n',1.0, 1.0, 0.0');
fprintf(fid,'LINES %d %d\n',2,6);
fprintf(fid,'%d %d %d\n',2, 0, 1);
fprintf(fid,'%d %d %d',2, 1, 2);
fclose(fid);
type(outfile);
```

'''Method 3'''
```
clear;

% Each row is a (x,y,z) value for a point.
P = [0.0, 0.0, 0.0; 1.0, 0.0, 0.0; 1.0, 1.0, 0.0];
Npts = size(P,1);

outfile = '1.8.2.2_3.vtk';

% Create and open a file named outfile for writing ('w').
fid = fopen(outfile,'w');

fprintf(fid,'# vtk DataFile Version 3.0\n');
fprintf(fid,'A dataset with two lines and no attributes\n');
fprintf(fid,'ASCII\n');
fprintf(fid,'\n');
fprintf(fid,'DATASET POLYDATA\n');
fprintf(fid,'POINTS %d float\n',Npts);
fprintf(fid,'%f %f %f\n',P(1,1),P(1,2),P(1,3));
fprintf(fid,'%f %f %f\n',P(2,1),P(2,2),P(2,3));
fprintf(fid,'%f %f %f\n',P(3,1),P(3,2),P(3,3));
fprintf(fid,'LINES %d %d\n',2,6);
fprintf(fid,'%d %d %d\n', 2, 0, 1);
fprintf(fid,'%d %d %d\n', 2, 1, 2);
fclose(fid);
type(outfile);
```

'''Method 4'''
```
clear;

% Each row is a (x,y,z) value for a point.
P = [0.0, 0.0, 0.0; 1.0, 0.0, 0.0; 1.0, 1.0, 0.0];
Npts = size(P,1);

outfile = '1.8.2.2_4.vtk';

% Create and open a file named outfile for writing ('w').
fid = fopen(outfile,'w');

fprintf(fid,'# vtk DataFile Version 3.0\n');
fprintf(fid,'A dataset with two lines and no attributes\n');
fprintf(fid,'ASCII\n');
fprintf(fid,'\n');
fprintf(fid,'DATASET POLYDATA\n');
fprintf(fid,'POINTS %d float\n',Npts);
for i = 1:Npts
  fprintf(fid,'%f %f %f\n',P(i,1),P(i,2),P(i,3));
end
fprintf(fid,'LINES %d %d\n',2,6);
fprintf(fid,'%d %d %d\n', 2, 0, 1);
fprintf(fid,'%d %d %d\n', 2, 1, 2);
fclose(fid);
type(outfile);
```
|}

#### Problem II 

Write three MATLAB programs, one for each of method 2, 3, and 4 described above, to create the following VTK file.

```
# vtk DataFile Version 3.0
Structured Points Example B
ASCII

DATASET STRUCTURED_POINTS
DIMENSIONS 2 2 2
ORIGIN 0.0 0.0 0.0
SPACING 1.0 1.0 1.0

POINT_DATA 8
VECTORS point_vectors float
0.0 0.0 1.0
0.0 0.0 1.0
0.0 0.0 1.0
0.0 0.0 1.0
1.0 0.0 0.0
1.0 0.0 0.0
1.0 0.0 0.0
1.0 0.0 0.0
```

{| class="wikitable collapsible collapsed"
! align="left" |&nbsp;Answer
|-
|
'''Method 2'''
```
clear;
outfile = '1.8.2.3_2.vtk';

% Create and open a file named outfile for writing ('w').
fid = fopen(outfile,'w');

fprintf(fid,'# vtk DataFile Version 3.0\n');
fprintf(fid,'Structured Points Example B\n');
fprintf(fid,'ASCII\n');
fprintf(fid,'\n');
fprintf(fid,'DATASET STRUCTURED_POINTS\n');
fprintf(fid,'DIMENSIONS %d %d %d\n',2,2,2);
fprintf(fid,'ORIGIN %f %f %f\n',0.0,0.0,0.0);
fprintf(fid,'SPACING %f %f %f\n',1.0,1.0,1.0);
fprintf(fid,'\n');
fprintf(fid,'POINT_DATA 8\n');
fprintf(fid,'VECTORS point_vectors float\n');
fprintf(fid,'%f %f %f\n',0.0, 0.0, 1.0);
fprintf(fid,'%f %f %f\n',0.0, 0.0, 1.0);
fprintf(fid,'%f %f %f\n',0.0, 0.0, 1.0);
fprintf(fid,'%f %f %f\n',0.0, 0.0, 1.0);
fprintf(fid,'%f %f %f\n',1.0, 0.0, 0.0);
fprintf(fid,'%f %f %f\n',1.0, 0.0, 0.0);
fprintf(fid,'%f %f %f\n',1.0, 0.0, 0.0);
fprintf(fid,'%f %f %f\n',1.0, 0.0, 0.0);
fclose(fid);
type(outfile);
```

'''Method 3'''
```
clear;

% Each row is a (x,y,z) value for a point.
P = [0.0, 0.0, 1.0; 0.0, 0.0, 1.0; 0.0, 0.0, 1.0; 0.0, 0.0, 1.0; ...
 1.0, 0.0, 0.0; 1.0, 0.0, 0.0; 1.0, 0.0, 0.0; 1.0, 0.0, 0.0];
Npts = size(P,1);

outfile = '1.8.2.3_3.vtk';

% Create and open a file named outfile for writing ('w').
fid = fopen(outfile,'w');

fprintf(fid,'# vtk DataFile Version 3.0\n');
fprintf(fid,'Structured Points Example B\n');
fprintf(fid,'ASCII\n');
fprintf(fid,'\n');
fprintf(fid,'DATASET STRUCTURED_POINTS\n');
fprintf(fid,'DIMENSIONS %d %d %d\n',2,2,2);
fprintf(fid,'ORIGIN %f %f %f\n',0.0,0.0,0.0);
fprintf(fid,'SPACING %f %f %f\n',1.0,1.0,1.0);
fprintf(fid,'\n');
fprintf(fid,'POINT_DATA %d\n',Npts);
fprintf(fid,'VECTORS point_vectors float\n');
fprintf(fid,'%f %f %f\n',P(1,1),P(1,2),P(1,3));
fprintf(fid,'%f %f %f\n',P(2,1),P(2,2),P(2,3));
fprintf(fid,'%f %f %f\n',P(3,1),P(3,2),P(3,3));
fprintf(fid,'%f %f %f\n',P(4,1),P(4,2),P(4,3));
fprintf(fid,'%f %f %f\n',P(5,1),P(5,2),P(5,3));
fprintf(fid,'%f %f %f\n',P(6,1),P(6,2),P(6,3));
fprintf(fid,'%f %f %f\n',P(7,1),P(7,2),P(7,3));
fprintf(fid,'%f %f %f\n',P(8,1),P(8,2),P(8,3));
fclose(fid);
type(outfile);
```

'''Method 4'''
```
clear;

% Each row is a (x,y,z) value for a point.
P = [0.0, 0.0, 1.0; 0.0, 0.0, 1.0; 0.0, 0.0, 1.0; 0.0, 0.0, 1.0; ...
 1.0, 0.0, 0.0; 1.0, 0.0, 0.0; 1.0, 0.0, 0.0; 1.0, 0.0, 0.0];
Npts = size(P,1);

outfile = '1.8.2.3_3.vtk';

% Create and open a file named outfile for writing ('w').
fid = fopen(outfile,'w');

fprintf(fid,'# vtk DataFile Version 3.0\n');
fprintf(fid,'Structured Points Example B\n');
fprintf(fid,'ASCII\n');
fprintf(fid,'\n');
fprintf(fid,'DATASET STRUCTURED_POINTS\n');
fprintf(fid,'DIMENSIONS %d %d %d\n',2,2,2);
fprintf(fid,'ORIGIN %f %f %f\n',0.0,0.0,0.0);
fprintf(fid,'SPACING %f %f %f\n',1.0,1.0,1.0);
fprintf(fid,'\n');
fprintf(fid,'POINT_DATA %d\n',Npts);
fprintf(fid,'VECTORS point_vectors float\n');
for i = 1:Npts
    fprintf(fid,'%f %f %f\n',P(i,1),P(i,2),P(i,3));
end
fclose(fid);
type(outfile);
```
|}

#### Problem III 

The wind velocity was measured at eight positions as indicated in the following table.

```
x    y   z    vx  vy  vz
0.0 0.0 0.0  1.0 1.0 0.0
1.0 0.0 0.0  1.0 2.0 0.0
1.0 1.0 0.0  2.0 2.0 0.0
0.0 1.0 0.0  2.0 2.0 0.0
0.0 0.0 1.0  1.0 1.0 0.0
1.0 0.0 1.0  1.0 2.0 0.0
1.0 1.0 1.0  2.0 2.0 0.0
0.0 1.0 1.0  2.0 2.0 0.0
```

Create a VTK file using MATLAB that represents the data in this table using Method 3 and Method 4.

{| class="wikitable collapsible collapsed"
! align="left" |&nbsp;Answer
|-
|
'''Method 3'''
```
clear;

% Each row is a (x,y,z) value for a point.
D = [1.0 1.0 0.0; 1.0 2.0 0.0; 2.0 2.0 0.0; 2.0 2.0 0.0; ...
1.0 1.0 0.0; 1.0 2.0 0.0; 2.0 2.0 0.0; 2.0 2.0 0.0]

Npts = size(D,1);

outfile = '1.8.2.4_3.vtk';

% Create and open a file named outfile for writing ('w').
fid = fopen(outfile,'w');

fprintf(fid,'# vtk DataFile Version 3.0\n');
fprintf(fid,'Wind velocity at unstructured points\n');
fprintf(fid,'ASCII\n');
fprintf(fid,'\n');
fprintf(fid,'DATASET STRUCTURED_POINTS\n');
fprintf(fid,'DIMENSIONS %d %d %d\n',2,2,2);
fprintf(fid,'ORIGIN %f %f %f\n',0.0,0.0,0.0);
fprintf(fid,'SPACING %f %f %f\n',1.0,1.0,1.0);
fprintf(fid,'\n');
fprintf(fid,'POINT_DATA %d\n',Npts);
fprintf(fid,'VECTORS point_vectors float\n');
fprintf(fid,'%f %f %f\n',D(1,1),D(1,2),D(1,3));
fprintf(fid,'%f %f %f\n',D(2,1),D(2,2),D(2,3));
fprintf(fid,'%f %f %f\n',D(3,1),D(3,2),D(3,3));
fprintf(fid,'%f %f %f\n',D(4,1),D(4,2),D(4,3));
fprintf(fid,'%f %f %f\n',D(5,1),D(5,2),D(5,3));
fprintf(fid,'%f %f %f\n',D(6,1),D(6,2),D(6,3));
fprintf(fid,'%f %f %f\n',D(7,1),D(7,2),D(7,3));
fprintf(fid,'%f %f %f\n',D(8,1),D(8,2),D(8,3));
fclose(fid);
type(outfile);
```

'''Method 4'''
```
clear;

% Each row is a (x,y,z) value for a point.
D = [1.0 1.0 0.0; 1.0 2.0 0.0; 2.0 2.0 0.0; 2.0 2.0 0.0; ...
1.0 1.0 0.0; 1.0 2.0 0.0; 2.0 2.0 0.0; 2.0 2.0 0.0]

Npts = size(D,1);

outfile = '1.8.2.4_4.vtk';

% Create and open a file named outfile for writing ('w').
fid = fopen(outfile,'w');

fprintf(fid,'# vtk DataFile Version 3.0\n');
fprintf(fid,'Wind velocity at unstructured points\n');
fprintf(fid,'ASCII\n');
fprintf(fid,'\n');
fprintf(fid,'DATASET STRUCTURED_POINTS\n');
fprintf(fid,'DIMENSIONS %d %d %d\n',2,2,2);
fprintf(fid,'ORIGIN %f %f %f\n',0.0,0.0,0.0);
fprintf(fid,'SPACING %f %f %f\n',1.0,1.0,1.0);
fprintf(fid,'\n');
fprintf(fid,'POINT_DATA %d\n',Npts);
fprintf(fid,'VECTORS point_vectors float\n');
for i = 1:Npts
    fprintf(fid,'%f %f %f\n',D(i,1),D(i,2),D(i,3));
end
fclose(fid);
type(outfile);
```

Alternative:

'''Method 3'''
```
clear;

% Each row is a (x,y,z) value for a point.
P = [0.0 0.0 0.0; 1.0 0.0 0.0; 1.0 1.0 0.0; 0.0 1.0 0.0; ...
0.0 0.0 1.0; 1.0 0.0 1.0; 1.0 1.0 1.0; 0.0 1.0 1.0];
D = [1.0 1.0 0.0; 1.0 2.0 0.0; 2.0 2.0 0.0; 2.0 2.0 0.0; ...
1.0 1.0 0.0; 1.0 2.0 0.0; 2.0 2.0 0.0; 2.0 2.0 0.0]

Npts = size(P,1);

outfile = '1.8.2.4_3.vtk';

% Create and open a file named outfile for writing ('w').
fid = fopen(outfile,'w');

fprintf(fid,'# vtk DataFile Version 3.0\n');
fprintf(fid,'Wind velocity at unstructured points\n');
fprintf(fid,'ASCII\n');
fprintf(fid,'\n');
fprintf(fid,'DATASET POLYDATA\n');
fprintf(fid,'POINTS %d float\n',Npts);
fprintf(fid,'%f %f %f\n',P(1,1),P(1,2),P(1,3));
fprintf(fid,'%f %f %f\n',P(2,1),P(2,2),P(2,3));
fprintf(fid,'%f %f %f\n',P(3,1),P(3,2),P(3,3));
fprintf(fid,'%f %f %f\n',P(4,1),P(4,2),P(4,3));
fprintf(fid,'%f %f %f\n',P(5,1),P(5,2),P(5,3));
fprintf(fid,'%f %f %f\n',P(6,1),P(6,2),P(6,3));
fprintf(fid,'%f %f %f\n',P(7,1),P(7,2),P(7,3));
fprintf(fid,'%f %f %f\n',P(8,1),P(8,2),P(8,3));
fprintf(fid,'\n');
fprintf(fid,'POINT_DATA %d\n',Npts);
fprintf(fid,'VECTORS point_vectors float\n');
fprintf(fid,'%f %f %f\n',D(1,1),D(1,2),D(1,3));
fprintf(fid,'%f %f %f\n',D(2,1),D(2,2),D(2,3));
fprintf(fid,'%f %f %f\n',D(3,1),D(3,2),D(3,3));
fprintf(fid,'%f %f %f\n',D(4,1),D(4,2),D(4,3));
fprintf(fid,'%f %f %f\n',D(5,1),D(5,2),D(5,3));
fprintf(fid,'%f %f %f\n',D(6,1),D(6,2),D(6,3));
fprintf(fid,'%f %f %f\n',D(7,1),D(7,2),D(7,3));
fprintf(fid,'%f %f %f\n',D(8,1),D(8,2),D(8,3));
fclose(fid);
type(outfile);
```

'''Method 4'''
```
clear;

% Each row is a (x,y,z) value for a point.
P = [0.0 0.0 0.0; 1.0 0.0 0.0; 1.0 1.0 0.0; 0.0 1.0 0.0; ...
0.0 0.0 1.0; 1.0 0.0 1.0; 1.0 1.0 1.0; 0.0 1.0 1.0];
D = [1.0 1.0 0.0; 1.0 2.0 0.0; 2.0 2.0 0.0; 2.0 2.0 0.0; ...
1.0 1.0 0.0; 1.0 2.0 0.0; 2.0 2.0 0.0; 2.0 2.0 0.0]

Npts = size(P,1);

outfile = '1.8.2.4_4.vtk';

% Create and open a file named outfile for writing ('w').
fid = fopen(outfile,'w');

fprintf(fid,'# vtk DataFile Version 3.0\n');
fprintf(fid,'Wind velocity at unstructured points\n');
fprintf(fid,'ASCII\n');
fprintf(fid,'\n');
fprintf(fid,'DATASET POLYDATA\n');
fprintf(fid,'POINTS %d float\n',Npts);
for i = 1:Npts
    fprintf(fid,'%f %f %f\n',P(i,1),P(i,2),P(i,3));
end
fprintf(fid,'\n');
fprintf(fid,'POINT_DATA %d\n',Npts);
fprintf(fid,'VECTORS point_vectors float\n');
for i = 1:Npts
    fprintf(fid,'%f %f %f\n',D(i,1),D(i,2),D(i,3));
end
fclose(fid);
type(outfile);
```
|}

## Problems - ParaView 



### Streamlines

The following file contains a cube with vectors associated with each of its corners.  Assume that the vectors represent measured velocities at the corners.

1. Use the Stream Tracer filter to estimate where a massless particle released at `(x,y,z) = (0,0,0)` would exit the cube region.  Document how your answer depends on the Integrator Type and Maximum Step Length (create a table that shows how the answer depends on these values).
2. Suppose that you wanted to have the massless particle released at `(x,y,z) = (0,0,0)` to exit through the center of one of the faces of the cube.  Modify the values for one or more of the vectors so this occurs.  

Note that if you do not see `Maximum Step Length`, click on the gear icon in the menu.

```
# vtk DataFile Version 3.0
Structured Points Example B
ASCII

DATASET STRUCTURED_POINTS
DIMENSIONS 2 2 2
ORIGIN 0.0 0.0 0.0
SPACING 1.0 1.0 1.0

POINT_DATA 8
VECTORS point_vectors float
0.0 0.0 1.0
0.0 0.0 1.0
0.0 0.0 1.0
0.0 0.0 1.0
1.0 0.0 0.0
1.0 0.0 0.0
1.0 0.0 0.0
1.0 0.0 0.0
```

## Streamlines II

The following file contains a cube with vectors associated with each of its corners.  Assume that the vectors represent measured velocities at the corners.

1. Use the Stream Tracer filter to estimate where a massless particle released at `(x,y,z) = (0,0,0)` would exit the cube region.  Document how your answer depends on the Integrator Type and Maximum Step Length (create a table that shows how the answer depends on these values).  Save your table in a file named `HW8a.txt`.
2. Suppose that you wanted to have the massless particle released at `(x,y,z) = (0,0,0)` to exit through the center of one of the faces of the cube.  Modify the values for one or more of the vectors so this occurs.  Save your file as `HW8a.vtk` and as the title, write the integrator type and maximum step length that you used.  (The title line in the file below is the one containing "Structured Points Example B".

Note that if you do not see `Maximum Step Length`, click on the gear icon in the menu.

```
# vtk DataFile Version 3.0
Structured Points Example B
ASCII

DATASET STRUCTURED_POINTS
DIMENSIONS 2 2 2
ORIGIN 0.0 0.0 0.0
SPACING 1.0 1.0 1.0

POINT_DATA 8
VECTORS point_vectors float
0.0 0.0 1.0
0.0 0.0 1.0
0.0 0.0 1.0
0.0 0.0 1.0
1.0 0.0 0.0
1.0 0.0 0.0
1.0 0.0 0.0
1.0 0.0 0.0
```


{| class="wikitable collapsible collapsed"
! align="left" |&nbsp;Answer
|-
|

This experiment was performed to

1. Determine how different integration methods performed against each other.
2. Compare how final answer depended on final step size.  In experiments like this you would plot final position versus dt and expect a curve that flattens as dt increases.
3. Have you get experience with understanding how something works or what something does in ParaView via experimentation.

Settings changed from default for StreamTracer
* Integration Direction: FORWARD
* Point: 0 0 0
* Maximum Steps: 10000
* Maximum Streamline Length: 10
* Number of Points: 1
* Radius: 0.0

Notes:
* Expected position of exit for x is 1.0.
* Assume last step position is exit points.  A better estimate of exit point would use the last two values of x and z to compute a straight line and then extrapolate straight line to find z exit at x = 1.0
* N varies as ~ 1.4/dt for all dt in RK 4-5.
* RK 4 gives identical results to RK 4-5 except for x value at dt = 0.08.
* All methods agree to within < 0.001 for dt = 0.01.

Last step positions x,y,z for various methods and maximum step lengths, dt. Number of steps is N.

```
RK 4-5
 dt      N      x       y       z
0.0001 14398 1.00012    0   0.841427
0.001   1439 0.999329   0   0.841279
0.01     143 0.990487   0   0.839601
0.02      71 0.980667   0   0.837711
0.04      35 0.961042   0   0.833856
0.08      17 0.921858   0   0.825822

RK 4
0.01    143  0.990487   0   0.839601
0.02     71  0.980667   0   0.837711
0.04     35  0.961042   0   0.833856
0.08     17  0.921857   0   0.825822

RK 2
0.01    143  0.990501   0   0.839594
0.02     71  0.980721   0   0.837686
0.04     35  0.961259   0   0.83375
0.08     17  0.922722   0   0.825373
```

# Activities 

## ParaView Tutorial

1. Select `Application->System Tools->Terminal` and type `/usr/local/paraview/bin/paraview &`
2. Download the tutorial data at http://www.paraview.org/Wiki/images/5/58/ParaViewTutorialData.zip and save on your desktop.  Double-click the zip file and extract to your desktop.  If successful, you should see a blue folder on your desktop with the title `ParaViewTutorialData`.
3. Work through the activities in chapter 2 of [http://www.paraview.org/Wiki/images/5/5d/ParaViewTutorial41.pdf]

## ParaView Basics

Create a VTK file and provide instructions for reading and selecting filters and menu items so that the following image is generated.  The spheres are located on a cube with sides of length 1.0 and all spheres have the same diameter.

[[Image:ms2.png|thumb|400px|left]]

{| class="wikitable collapsible collapsed"
! align="left" |&nbsp;Answer
|-
|

```
# vtk DataFile Version 3.0
Polydata + Polygons Example B with Cell Data
ASCII

DATASET POLYDATA
POINTS 8 float
0.0 0.0 0.0
0.0 1.0 0.0
1.0 1.0 0.0
1.0 0.0 0.0
0.0 0.0 1.0
0.0 1.0 1.0
1.0 1.0 1.0
1.0 0.0 1.0
```

After applying the VTK file, use the "Glyph" filter and change the Glyph Type under Glyph Source to "Sphere".
|}

## ParaView Basics

Create a VTK file and provide instructions for reading and selecting filters and menu items so that the following image is generated.  Create a VTK file that uses `CELL_DATA`.   When coloring by `cell_scalars` is selected, the following image should appear (after rotating the image).

<imgc>url=https://raw.githubusercontent.com/rweigel/cds301/master/vtk/PolydataPolygonsB_cell_data_and_point_data_scalar_default_view_2.png|width=400px|display=none|expire=1</imgc>

{| class="wikitable collapsible collapsed"
! align="left" |&nbsp;Answer
|-
|
```
# vtk DataFile Version 3.0
Polydata + Polygons Example B with Cell Data
ASCII

DATASET POLYDATA
POINTS 6 float
0.0 0.0 0.0
0.0 1.0 0.0
1.0 1.0 0.0
1.0 0.0 0.0
0.0 0.0 1.0
0.0 1.0 1.0

POLYGONS 2 10
4 0 1 2 3
4 0 4 5 1

CELL_DATA 2
SCALARS cell_scalars int 1
LOOKUP_TABLE default
1
2
```
|}

## Creating a Vector Field

### Part I 

In this problem, we would like to create a diagram similar to [http://bobweigel.net/cds301/images/raw.githubusercontent.com/rweigel/cds301/master/streamlines/vector_basics2quiver.png] using a VTK file.  To simplify things, we will only draw the 9 points in the lower-left corner.  The objective is to create a diagram with 9 dots and with vectors that are shown on the dots with appropriate lengths and directions.  Your diagram does not need to have the text annotations - only points and vectors are required.  (Do you need to specify a topology to do this, or is only a geometry needed?)

{| class="wikitable collapsible collapsed"
! align="left" |&nbsp;Answer
|-
|
```
# vtk DataFile Version 3.0
Wind velocity at unstructured points
ASCII

DATASET POLYDATA
POINTS 9 float
1.00 1.00 0.00
1.00 2.00 0.00
1.00 3.00 0.00
2.00 1.00 0.00
2.00 2.00 0.00
2.00 3.00 0.00
3.00 1.00 0.00
3.00 2.00 0.00
3.00 3.00 0.00

POINT_DATA 9
VECTORS point_vectors float
1.00 1.50 0.00
1.00 1.50 0.00
1.00 1.50 0.00
1.00 2.50 0.00
1.00 2.50 0.00
1.00 2.50 0.00
1.00 3.50 0.00
1.00 3.50 0.00
1.00 3.50 0.00
```

Alternatively, here's part of the vtk file for the 100 point version, as shown in the original image:

```
# vtk DataFile Version 3.0
Wind velocity at unstructured points
ASCII

DATASET POLYDATA
POINTS 100 float
1.00 1.00 0.00
1.00 2.00 0.00
1.00 3.00 0.00
...
10.00 8.00 0.00
10.00 9.00 0.00
10.00 10.00 0.00

POINT_DATA 100
VECTORS point_vectors float
1.00 1.50 0.00
1.00 1.50 0.00
1.00 1.50 0.00
...
1.00 6.00 0.00
1.00 6.00 0.00
1.00 6.00 0.00
```

The full vtk file was generated with the following matlab code:

```
clear;

outfile = '1.10.4.1.vtk';

% Create and open a file named outfile for writing ('w').
fid = fopen(outfile,'w');

fprintf(fid,'# vtk DataFile Version 3.0\n');
fprintf(fid,'Wind velocity at unstructured points\n');
fprintf(fid,'ASCII\n');
fprintf(fid,'\n');
fprintf(fid,'DATASET POLYDATA\n');
fprintf(fid,'POINTS 100 float\n');
for i = 1:10
    for j = 1:10
        fprintf(fid,'%.2f %.2f 0.00\n',i,j);
    end
end
fprintf(fid,'\n');
fprintf(fid,'POINT_DATA 100\n');
fprintf(fid,'VECTORS point_vectors float\n');
for i = 1:10
    for j = 1:10
        fprintf(fid,'%.2f %.2f 0.00\n',1,1+i*0.5);
    end
end
fclose(fid);
type(outfile);
```
|}

### Part II 

Modify the VTK file that you created in the previous problem so that it is three-dimensional.  The velocity in the z-direction should be equal to 3.0, and there should be a total of three layers in the z-direction (for a total of 27 points).

{| class="wikitable collapsible collapsed"
! align="left" |&nbsp;Answer
|-
|
```
# vtk DataFile Version 3.0
Wind velocity at unstructured points
ASCII

DATASET POLYDATA
POINTS 27 float
1.00 1.00 1.00
1.00 2.00 1.00
1.00 3.00 1.00
2.00 1.00 1.00
2.00 2.00 1.00
2.00 3.00 1.00
3.00 1.00 1.00
3.00 2.00 1.00
3.00 3.00 1.00
1.00 1.00 2.00
1.00 2.00 2.00
1.00 3.00 2.00
2.00 1.00 2.00
2.00 2.00 2.00
2.00 3.00 2.00
3.00 1.00 2.00
3.00 2.00 2.00
3.00 3.00 2.00
1.00 1.00 3.00
1.00 2.00 3.00
1.00 3.00 3.00
2.00 1.00 3.00
2.00 2.00 3.00
2.00 3.00 3.00
3.00 1.00 3.00
3.00 2.00 3.00
3.00 3.00 3.00

POINT_DATA 27
VECTORS point_vectors float
1.00 1.50 3.00
1.00 1.50 3.00
1.00 1.50 3.00
1.00 2.50 3.00
1.00 2.50 3.00
1.00 2.50 3.00
1.00 3.50 3.00
1.00 3.50 3.00
1.00 3.50 3.00
1.00 1.50 3.00
1.00 1.50 3.00
1.00 1.50 3.00
1.00 2.50 3.00
1.00 2.50 3.00
1.00 2.50 3.00
1.00 3.50 3.00
1.00 3.50 3.00
1.00 3.50 3.00
1.00 1.50 3.00
1.00 1.50 3.00
1.00 1.50 3.00
1.00 2.50 3.00
1.00 2.50 3.00
1.00 2.50 3.00
1.00 3.50 3.00
1.00 3.50 3.00
1.00 3.50 3.00
```
|}

### Part III 

Write a MATLAB program to create the VTK file.  See [[IO]] for information on creating files with MATLAB.  (I'll give more hints on how to do this on the actual HW assignment.)

{| class="wikitable collapsible collapsed"
! align="left" |&nbsp;Answer
|-
|
```
clear;

outfile = '1.10.4.3.vtk';

% Create and open a file named outfile for writing ('w').
fid = fopen(outfile,'w');

fprintf(fid,'# vtk DataFile Version 3.0\n');
fprintf(fid,'Wind velocity at unstructured points\n');
fprintf(fid,'ASCII\n');
fprintf(fid,'\n');
fprintf(fid,'DATASET POLYDATA\n');
fprintf(fid,'POINTS 27 float\n');
for i = 1:3
    for j = 1:3
        for k = 1:3
            fprintf(fid,'%.2f %.2f %.2f\n',i,j,k);
        end
    end
end
fprintf(fid,'\n');
fprintf(fid,'POINT_DATA 27\n');
fprintf(fid,'VECTORS point_vectors float\n');
for i = 1:3
    for j = 1:3
        for k = 1:3
            fprintf(fid,'%.2f %.2f 3.00\n',1,1+i*0.5);
        end
    end
end
fclose(fid);
type(outfile);
```
|}

## Creating VTK Files I

Create a VTK file such that when the following steps are followed, I see 1000 spheres with random colors between blue and red located at arbitrary 3-D positions.

1. Open file in ParaView
2. Select Apply
3. Select Representation of Points
4. Select `Filters > Alphabetical > Glyphs`
5. Select Sphere
6. Select Apply

Hint: See [[#POINT_DATA]].

{| class="wikitable collapsible collapsed"
! align="left" |&nbsp;Answer
|-
|
```
clear;

% Each row is a (x,y,z) value for a point.
P = rand(1000,3);
N = size(P,1);

fid = fopen('HW8_1.vtk','w');

fprintf(fid,'# vtk DataFile Version 3.0\n');
fprintf(fid,'HW 8_1\n');
fprintf(fid,'ASCII\n\n');
fprintf(fid,'DATASET POLYDATA\n');
fprintf(fid,'POINTS %d float\n',N);
%Avoid for loop using this technique
%PT = P';
%fprintf(fid,'%f %f %f\n',PT(:));
for i = 1:N
  fprintf(fid,'%f %f %f\n',P(i,1),P(i,2),P(i,3));
end
fprintf(fid,'\nPOINT_DATA %d\n',N);
fprintf(fid,'SCALARS point_scalars float\n');
fprintf(fid,'LOOKUP_TABLE default\n');
% Row in S is sum of squares of row in P.  Will result in points
% with colors that depend on distance from origin.
S = sum(P.^2,2);
%fprintf(fid,'%f\n',S);
for i = 1:N
  fprintf(fid,'%f\n',S(i));
end
fclose(fid);
```

## Creating VTK Files II

1. With a text editor, create a VTK file with only geometry for a circle approximated by sampling points every 45 degrees on a circle with radius 1.0.  Save your file as `circle_text1.vtk`.
2. Write a MATLAB program that creates a VTK file named `circle_matlab1.vtk` with the same content as above.
3. Add topology to `circle_text2.vtk` with a text editor.
4. Write a MATLAB program that creates a VTK file named `circle_matlab2.vtk` with the same content as above.

*Partial Answer*

```
clear;
theta = [0:45:360];
for i = 1:length(theta)
    x(i) = cosd(theta(i));
    y(i) = sind(theta(i));
end

fid = fopen('circle.vtk','w');
                                                                                               
fprintf(fid,'# vtk DataFile Version 3.0\n');                                                     
fprintf(fid,'Circle\n');                                                                   
fprintf(fid,'ASCII\n');                                                                            

fprintf(fid,'DATASET POLYDATA\n');
fprintf(fid,'POINTS %d float\n',length(x));                                                      
for i = 1:length(x)                                                                            
  fprintf(fid,'%f %f %f\n',x(i),y(i),0);                                                         
end 

% This will draw a circle, but no surface is
% shown when surface representation is selected.
%fprintf(fid,'LINES 1 10\n');
%fprintf(fid,'9 0 1 2 3 4 5 6 7 0\n');

fprintf(fid,'POLYGONS 1 9\n');
fprintf(fid,'8 0 1 2 3 4 5 6 7\n');
fclose(fid);
type circle.vtk
```

## Creating VTK Files III

With a text editor, create a VTK file with only geometry for a cylinder approximated as an object with 8 sides, radius of 1.0, and height of 2.0.

# ParaView Python Scripting

In this tutorial, no prior knowledge of Python is assumed.  You will be shown how to do basic operation in Python that modify objects in ParaView.  You do not need to download Python; it is distributed with ParaView.

## Getting Started 

This tutorial requires ParaView 4.4.

If you are using Linux in the classroom, first download ParaView 4.4 (4.0 is installed, so we need to install new version) by going to http://www.paraview.org/download/ and selecting the 64-bit Linux version.  A file named `ParaView-4.4.0-Qt4-Linux-64bit.tar.gz` will download into your `Downloads` folder.

Open a terminal and change to the Downloads directory using

    cd Downloads 

Then list its directory contents using

    ls

you should see `ParaView-4.4.0-Qt4-Linux-64bit.tar.gz`.  Then enter

    tar zxvf ParaView-4.4.0-Qt4-Linux-64bit.tar.gz

to unpack the file (a tar.gz file is like a zip file).

To start ParaView, enter

    ~/Downloads/ParaView-4.4.0-Qt4-Linux-64bit/bin/paraview

on the command line.

## Using the Python Shell 

Using a file browser, open the pdf file in `Downloads/ParaView-4.4.0-Qt4-Linux-64bit/doc` (or download [http://mag.gmu.edu/git-data/cds301/paraview/doc/TheParaViewGuide-CC-Edition.pdf]).  Work through through the examples given in section 1.5 (which starts on the bottom of page 11).  

To enter Python commands, start ParaView and select `Tools > Python Shell`.  This will open a new window with a Python command line (the command prompt is `>>>`).

Note that you do not need to enter `from paraview.simple import *` as indicated in the ParaView Guide; it is done for you when you open the Python Shell window.

## Writing a Script 

One option for saving Python commands is to save them in a file.  For example, using a text editor (gedit, Notepad, TextWangler) save

```
Sphere()
Show()
Render()
```

to a file named `sphere.py`.  In the Python Shell window, select `Run Script` and select this file.  This has the same effect as typing all of the above commands one-by-one on the Python Shell command line.

Note that an alternative to selecting the file as above is to enter
 execfile('/home/weigel/sphere.py')
on the Python Shell command line after replacing `/home/weigel/` with the appropriate path.

## Writing a Macro 

Commands can also be saved as a macro.  

Select `Macros > Add New Macro` and then select the file `sphere.py`.  On your toolbar, you will see the word `sphere`.  Clicking on it will execute the code in `sphere.py`.

<div style="background-color:yellow">
Important: To modify this macro, you must select `Macros > Edit` to edit the file.  The reason is that macros are copied to the directory: `~/.config/ParaView/Macros` and this editor opens the `sphere.py` file found in this directory.  (Your original script `sphere.py` will still exist, but it will not be executed when you select the word `sphere`.)

You can also edit the macro file with a text editor such as gedit by opening the `sphere.py` file located in `~/.config/ParaView/Macros`.  
</div>

To verify that you can edit and execute your macro, modify `sphere.py` using `Macros > Edit` so that it now reads

```
print "Executing Macro sphere.py"

Sphere()
Show()
Render()
```

save, and then click the word `sphere` in the menubar.  In the Python Shell window (open using `Tools > Python Shell`) you should see `Executing Macro sphere.py`, which was printed when you executed the macro.

Note that command `print "ABCD"` in Python is equivalent to the command `fprintf("ABCE\n")` in MATLAB.

If you execute the several times in a row, you will see additional spheres added to the pipeline.  To clear the pipeline before executing your macro, start your macros with the lines
```
for s in GetSources().values():
    Delete(s)
```

The extra block of code removes everything in the pipeline before any macro commands are executed. 

As a final test, enter the following in `sphere.py` macro:
```
for s in GetSources().values():
    Delete(s)

print "Executing Macro sphere.py"

Sphere()
SetDisplayProperties(Opacity=0.5)
Show()
Render()
```
and execute it.

The command `SetDisplayProperties(Opacity=0.5)` changes the opacity of the active source (in this case the sphere).  The `Render()` command is needed to explicitly tell ParaView to update the plot.

## Writing Scripts and Macros 

Determining the Python commands that one needs to execeute in ParaView to create a given display is not easy; there are many commands and options and there are not many examples on using them.

The `Python Trace` tool is often used as an alternative to documentation.  Typically one uses a trace to record a simple operation, such as creating a sphere and changing its opacity, and then code from the trace is extracted and placed in a larger program that is being built.

Note that the `Python Trace` tool will produce code that is more verbose than that in the previous example where a sphere with an opacity of 0.5 was created using

```
Sphere()
SetDisplayProperties(Opacity=0.5)
Render()
```

To see the code that the trace produces, select `Tools > Start Trace`, select the `Show Incremental Trace` option and click OK.  Then select `Sources > Sphere` and click Apply.  Then change the opacity of the sphere to 0.5.  To stop the trace, select `Tools > Stop Trace`.  The code produced is (note that `#` indicated a comment in Python):

```
#### import the simple module from the paraview
from paraview.simple import *
#### disable automatic camera reset on 'Show'
paraview.simple._DisableFirstRenderCameraReset()

# create a new 'Sphere'
sphere1 = Sphere()

# get active view
renderView1 = GetActiveViewOrCreate('RenderView')
# uncomment following to set a specific view size
# renderView1.ViewSize = [1014, 1102]

# show data in view
sphere1Display = Show(sphere1, renderView1)
# trace defaults for the display properties.
sphere1Display.ColorArrayName = [None, '']

# reset view to fit data
renderView1.ResetCamera()

# Properties modified on sphere1Display
sphere1Display.Opacity = 0.5

#### saving camera placements for all active views

# current camera placement for renderView1
renderView1.CameraPosition = [2.515544574485071, 2.515544574485071, 0.0]
renderView1.CameraViewUp = [0.0, 0.0, 1.0]
renderView1.CameraParallelScale = 0.9255186509230058

#### uncomment the following to render all views
# RenderAllViews()
# alternatively, if you want to write images, you can use SaveScreenshot(...).
```

Note that the command `RenderAllViews()` must be uncommented in order for the sphere to be displayed when the macro is executed.  You must remove the `# ` before the command `RenderAllViews()` (there should be no space before the start of the line and the `R` in Render).

The key parts of the two programs are shown below.  The key part of the trace output were determined by reading the code and a bit of trial-and-error to determine which parts were necessary for creating a sphere with an opacity of 0.5.  Understanding the other parts of the code will take trial-and-error, searches for documentation, and experimentation.

{| style="margin-bottom:1em" border="0" cellpadding="2" width="100%" align="left"
|-
|style="vertical-align:top" width="50%" |
```
Sphere()
SetDisplayProperties(Opacity=0.5)
Render()
Show()
```
|style="vertical-align:top"|                                                              
```
sphere1 = Sphere()
renderView1 = GetActiveVieworCreate('RenderView')
sphere1Display = Show(sphere1,renderView1)
sphere1Display.Opacity = 0.5
RenderAllViews()
```
|}

## Animation 

Note - there is an alternative animation method not covered here that uses ParaView's animation tool (see `View > Animation View`).

Previously we covered [[#Animation]] using MATLAB.  The main components of any animation are

1. A loop
2. Plotting
3. Saving the plot

The following MATLAB program draws a series of dots at `(x,y)`=`(1,1)`, `(2,2)`, etc. and then saves an image for each plot in the series.
```
for frame = 1:10
    fprintf('Frame %02d\n',frame)
    plot(frame,frame,'MarkerSize',20)
    fname = sprintf('animation_%02d',frame)
    print('-dpng',fname)
end
fprintf('Done\n')
```

To translate this into Python, we'll do it in steps.  First the loop

```
for frame = 1:10
    fprintf('Frame %02d\n',frame)
end
fprintf('Done\n')
```
in Python is
```
for frame in range(1,11):
    print "Frame %02d" % frame
print "Done"
```

<div style="background-color:yellow;padding:1em;margin:1em">
Note that there is no `end` statement!  In Python, indentation is required and is used to indicate the end of the loop.  The logic that Python uses to determine where to end the loop is along the lines of "if a given line is indented and the next line is not, the end of the loop is between these two lines".

In the above, the statement `print "Done"` is not indented, so it is not a part of the loop.  To see the importance of indentation, compare the output of the above with that from executing

```
for frame in range(1,11):
    print frame
    print "Done"
```

To test this code, copy the following to a text file named `test.py` and execute it by selecting `Tools > Python Shell` and choosing it after clicking `Run Script`.  (Copying muli-line statements onto the Python Shell command line does not seem to work.)

Because the `print` statement is indented the same amount as the first line after the `for` statement, it is assumed to be a part of the loop, equivalent to the MATLAB program

```
for frame = 1:10
    fprintf('Frame %02d\n',frame)
    fprintf('Done\n')
end
```
</div>

As another example of the importance of indentation, enter `  a=1` (include the leading spaces) on the Python Shell command line.  You should see an error message.  The correct way to assign a variable outside of a for loop is `a=1`.

The following program can be used as a basis for creating animations.  In most cases, you'll need to identify statements to place in the for loop that modify an object based on the frame number.  In this example the statement
 sphere1.Center = [frame/10.0,frame/10.0,0.0]
changes the center position of the sphere in each frame. 

Note that
 for frame in range(1,11)
has the same meaning as
 for frame = 1:10
in MATLAB and the Python expression
 fname = "animation_%02d.png" % frame
is equivalent to
 fname = sprintf('animation_%02d.png', frame)
in MATLAB.
```
for s in GetSources().values():
    Delete(s)

renderView1 = GetActiveViewOrCreate('RenderView')

# Create a sphere object
sphere1 = Sphere()
# Create a sphere display object
sphere1Display = Show(sphere1, renderView1)

for frame in range(1,11):
    # Set the x and y values of the center of the sphere based on the frame number
    sphere1.Center = [frame/10.0,frame/10.0,0.0]
    # Show the updated sphere
    RenderAllViews()
    # Create a filename based on the frame number
    fname = "animation_%02d.png" % frame
    # Save the frame as a png file
    WriteImage(fname)
```

## Problems 

## Creating Macros Using Python Trace I

Use `Python Trace` to figure out how to draw a cone by first, deleting everything in the pipeline.  Then select `Tools > Start Trace` and select `Show Incremental Trace` option and click OK.  Then select `Sources > Cone` and click Apply.  To stop the trace, select `Tools > Stop Trace`.  

You now have a script that will repeat the actions that you just executed between starting and stopping the trace.

Remove the `# ` before the command `RenderAllViews()` and then select `File > Save as Macro` and name the file `cone.py`.  You can now select the word `cone` in the menubar.

Copy `cone.py` into your repository and name it `HW9_1.py`.  When I select `Python Shell` and then select your file after clicking `Run Script`, I should see a cone on the screen.

{| class="wikitable collapsible collapsed"
! align="left" |&nbsp;Answer
|-
|
```
#### import the simple module from the paraview
from paraview.simple import *
#### disable automatic camera reset on 'Show'
paraview.simple._DisableFirstRenderCameraReset()

# create a new 'Cone'
cone1 = Cone()

# get active view
renderView1 = GetActiveViewOrCreate('RenderView')
# uncomment following to set a specific view size
# renderView1.ViewSize = [401, 526]

# show data in view
cone1Display = Show(cone1, renderView1)
# trace defaults for the display properties.
cone1Display.ColorArrayName = [None, '']
cone1Display.GlyphType = 'Arrow'
cone1Display.SetScaleArray = [None, '']
cone1Display.ScaleTransferFunction = 'PiecewiseFunction'
cone1Display.OpacityArray = [None, '']
cone1Display.OpacityTransferFunction = 'PiecewiseFunction'

# reset view to fit data
renderView1.ResetCamera()

#### saving camera placements for all active views

# current camera placement for renderView1
renderView1.CameraPosition = [0.0, 0.0, 3.2036135254332487]
renderView1.CameraFocalPoint = [0.0, 0.0, -1e-20]
renderView1.CameraParallelScale = 0.8291561935301535

#### uncomment the following to render all views
RenderAllViews()
# alternatively, if you want to write images, you can use SaveScreenshot(...).
```
|}

### Creating Macros Using Python Trace II

Create a macro named `cone2.py` that shows a blue cone. Your macro should start by deleting any existing sources in the pipeline.

Remember to remove the `# ` before the command `RenderAllViews()` (there should be no space before the start of the line and the `R` in Render) before saving `cone2.py` as a macro.

Copy `cone.py` into your repository and name it `HW9_2.py`.  When I select `Python Shell` and then select your file after clicking `Run Script`, I should see a blue cone on the screen.

{| class="wikitable collapsible collapsed"
! align="left" |&nbsp;Answer
|-
|
```
#### import the simple module from the paraview
from paraview.simple import *

# delete all sources
for s in GetSources().values():
    Delete(s)

#### disable automatic camera reset on 'Show'
paraview.simple._DisableFirstRenderCameraReset()

# create a new 'Cone'
cone1 = Cone()

# get active view
renderView1 = GetActiveViewOrCreate('RenderView')
# uncomment following to set a specific view size
# renderView1.ViewSize = [374, 526]

# show data in view
cone1Display = Show(cone1, renderView1)
# trace defaults for the display properties.
cone1Display.ColorArrayName = [None, '']
cone1Display.GlyphType = 'Arrow'
cone1Display.SetScaleArray = [None, '']
cone1Display.ScaleTransferFunction = 'PiecewiseFunction'
cone1Display.OpacityArray = [None, '']
cone1Display.OpacityTransferFunction = 'PiecewiseFunction'

# reset view to fit data
renderView1.ResetCamera()

# change solid color
cone1Display.DiffuseColor = [0.0, 0.0, 1.0]

#### saving camera placements for all active views

# current camera placement for renderView1
renderView1.CameraPosition = [0.0, 0.0, 4.43037269925063]
renderView1.CameraParallelScale = 1.166139459349895

#### uncomment the following to render all views
RenderAllViews()
# alternatively, if you want to write images, you can use SaveScreenshot(...).
```
|}

### Creating Macros Using Python Trace III

Create a macro that will generate a blue cylinder along the y-axis with a red sphere on top of it. 

Save your macro as `HW9_3.py`.  When I select `Python Shell` and then select your file after clicking `Run Script`, I should see a blue cylinder along the y-axis with a red sphere on top of it (not necessarily with the same orientation as the figure below). 

[[Image:person.png|thumb|500px|A view of a cylinder along the y-axis with a red sphere on top of it after executing the macro and rotating the image with the mouse.]]

<br style="clear:both"/>

### Extending a Macro

The following macro creates an axes with the same color scheme as the default orientation axes but with an origin of (x,y,z) = (0,0,0) and using cylinders instead of lines for for the axes. 

Using a text editor such as gedit, Notepad, or TextWrangler, copy the content below into a file named `axes.py`.  Then open this file in ParaView using the Python Shell window.  Remember that to modify this file, you now need to use `Macros > Edit` and then select `axes` from the list.

Then, add to this program so that

1. it has cones at the end of the cylinders so the axes look like arrows and
2. the axes are labeled X, Y, and Z.

Use the trace method described above to determine the code needed for each part.  Then add the needed code into the macro.

Save your macro as `HW9_4.py`. 

```
from paraview.simple import *
paraview.simple._DisableFirstRenderCameraReset()

[Delete(s) for s in GetSources().values()]

renderView1 = GetActiveViewOrCreate('RenderView')

# x
cylinderX        = Cylinder()
cylinderX.Radius = 0.02
cylinderX.Center = [0.0, 0.5, 0.0]
cylinderX.Height = 1.0

cylinderXDisplay = Show(cylinderX, renderView1)
cylinderXDisplay.ColorArrayName = [None, '']
cylinderXDisplay.DiffuseColor = [1.0, 0.0, 0.0]
# Default Cylinder is orientated along y-axis.
# To get x cylinder, rotate by -90 around z-axis.
cylinderXDisplay.Orientation = [0.0, 0.0, -90.0]

# y
cylinderY        = Cylinder()
cylinderY.Radius = 0.02
cylinderY.Center = [0.0, 0.5, 0.0]
cylinderY.Height = 1.0
cylinderYDisplay = Show(cylinderY, renderView1)
cylinderYDisplay.ColorArrayName = [None, '']
cylinderYDisplay.DiffuseColor = [1.0, 1.0, 0.5]
# Default Cylinder is orientated along y-axis, so 
# the following statement is not needed.
cylinderYDisplay.Orientation = [0.0, 0.0, 0.0]

# z
cylinderZ        = Cylinder()
cylinderZ.Radius = 0.02
cylinderZ.Center = [0.0, 0.5, 0.0]
cylinderZ.Height = 1.0
cylinderZDisplay = Show(cylinderZ, renderView1)
cylinderZDisplay.ColorArrayName = [None, '']
cylinderZDisplay.DiffuseColor = [0.0, 1.0, 0.0]
# Default Cylinder is orientated along y-axis.
# To get z cylinder, rotate by 90 around x-axis.
cylinderZDisplay.Orientation = [90.0, 0.0, 0.0]

Render()
```

{| class="wikitable collapsible collapsed"
! align="left" |&nbsp;Answer
|-
|
```
from paraview.simple import *
paraview.simple._DisableFirstRenderCameraReset()

[Delete(s) for s in GetSources().values()]

renderView1 = GetActiveViewOrCreate('RenderView')

# x
cylinderX        = Cylinder()
cylinderX.Radius = 0.02
cylinderX.Center = [0.0, 0.5, 0.0]
cylinderX.Height = 1.0

cylinderXDisplay = Show(cylinderX, renderView1)
cylinderXDisplay.ColorArrayName = [None, '']
cylinderXDisplay.DiffuseColor = [1.0, 0.0, 0.0]
# Default Cylinder is orientated along y-axis.
# To get x cylinder, rotate by -90 around z-axis.
cylinderXDisplay.Orientation = [0.0, 0.0, -90.0]

coneX        = Cone()
coneX.Radius = 0.05
coneX.Height = 0.1
coneX.Center = [1.0, 0.0, 0.0]

coneXDisplay = Show(coneX, renderView1)
coneXDisplay.ColorArrayName = [None, '']
coneXDisplay.GlyphType = 'Arrow'
coneXDisplay.SetScaleArray = [None, '']
coneXDisplay.ScaleTransferFunction = 'PiecewiseFunction'
coneXDisplay.OpacityArray = [None, '']
coneXDisplay.OpacityTransferFunction = 'PiecewiseFunction'
coneXDisplay.DiffuseColor = [1.0, 0.0, 0.0]

a3DText1 = a3DText()
a3DText1.Text = 'X'
linearExtrusion1 = LinearExtrusion(Input=a3DText1)
linearExtrusion1Display = Show(linearExtrusion1, renderView1)
linearExtrusion1Display.Scale = [0.1, 0.1, 0.1]
linearExtrusion1Display.Origin = [1,0,0]

# y
cylinderY        = Cylinder()
cylinderY.Radius = 0.02
cylinderY.Center = [0.0, 0.5, 0.0]
cylinderY.Height = 1.0
cylinderYDisplay = Show(cylinderY, renderView1)
cylinderYDisplay.ColorArrayName = [None, '']
cylinderYDisplay.DiffuseColor = [1.0, 1.0, 0.5]
# Default Cylinder is orientated along y-axis, so 
# the following statement is not needed.
cylinderYDisplay.Orientation = [0.0, 0.0, 0.0]

coneY        = Cone()
coneY.Radius = 0.05
coneY.Height = 0.1
coneY.Center = [0.0, 1.0, 0.0]
coneY.Direction = [0.0, 1.0, 0.0]

coneYDisplay = Show(coneY, renderView1)
coneYDisplay.ColorArrayName = [None, '']
coneYDisplay.GlyphType = 'Arrow'
coneYDisplay.SetScaleArray = [None, '']
coneYDisplay.ScaleTransferFunction = 'PiecewiseFunction'
coneYDisplay.OpacityArray = [None, '']
coneYDisplay.OpacityTransferFunction = 'PiecewiseFunction'
coneYDisplay.DiffuseColor = [1.0, 1.0, 0.0]

a3DText1 = a3DText()
a3DText1.Text = 'Y'
linearExtrusion1 = LinearExtrusion(Input=a3DText1)
linearExtrusion1Display = Show(linearExtrusion1, renderView1)
linearExtrusion1Display.Scale = [0.1, 0.1, 0.1]
linearExtrusion1Display.Origin = [0,1,0]

# z
cylinderZ        = Cylinder()
cylinderZ.Radius = 0.02
cylinderZ.Center = [0.0, 0.5, 0.0]
cylinderZ.Height = 1.0
cylinderZDisplay = Show(cylinderZ, renderView1)
cylinderZDisplay.ColorArrayName = [None, '']
cylinderZDisplay.DiffuseColor = [0.0, 1.0, 0.0]
# Default Cylinder is orientated along y-axis.
# To get z cylinder, rotate by 90 around x-axis.
cylinderZDisplay.Orientation = [90.0, 0.0, 0.0]

coneZ        = Cone()
coneZ.Radius = 0.05
coneZ.Height = 0.1
coneZ.Center = [0.0, 0.0, 1.0]
coneZ.Direction = [0.0, 0.0, 1.0]

coneZDisplay = Show(coneZ, renderView1)
coneZDisplay.ColorArrayName = [None, '']
coneZDisplay.GlyphType = 'Arrow'
coneZDisplay.SetScaleArray = [None, '']
coneZDisplay.ScaleTransferFunction = 'PiecewiseFunction'
coneZDisplay.OpacityArray = [None, '']
coneZDisplay.OpacityTransferFunction = 'PiecewiseFunction'
coneZDisplay.DiffuseColor = [0.0, 1.0, 0.0]

a3DText1 = a3DText()
a3DText1.Text = 'Z'
linearExtrusion1 = LinearExtrusion(Input=a3DText1)
linearExtrusion1Display = Show(linearExtrusion1, renderView1)
linearExtrusion1Display.Scale = [0.1, 0.1, 0.1]
linearExtrusion1Display.Origin = [0,0,1]

Render()
```
|}

### Animation

<span style="background-color:yellow">In ParaView 5.0, this does not work</span>

Create an animation of a sphere moving in the z direction in 10 steps.

Save your macro as `HW9_5.py`.  When I execute this script, I should see a sphere moving on the screen and 10 png files written on my disk.

{| class="wikitable collapsible collapsed"
! align="left" |&nbsp;Answer
|-
|
```
for s in GetSources().values():
    Delete(s)

renderView1 = GetActiveViewOrCreate('RenderView')
sphere1 = Sphere()
sphere1Display = Show(sphere1, renderView1)

for frame in range(1,11):
    sphere1.Center = [0.0,0.0,frame/10.0]
    RenderAllViews()
    fname = "animation_%02d.png" % frame
    WriteImage(fname)
```
|}

### Animation

<span style="background-color:yellow">In ParaView 5.0, this does not work</span>

Create an animation of a sphere with a color that changes from the default color to increasing amounts of red over 10 frames.  In the final frame the sphere should be red.

Save your macro as `HW9_6.py`.  When I execute this script, I should see a sphere with a changing color on the screen and 10 png files written on my disk.

{| class="wikitable collapsible collapsed"
! align="left" |&nbsp;Answer
|-
|
```
for s in GetSources().values():
    Delete(s)

renderView1 = GetActiveViewOrCreate('RenderView')
sphere1 = Sphere()
sphere1Display = Show(sphere1, renderView1)

for frame in range(1,11):
    sphere1Display.DiffuseColor = [1.0,(frame-10.0)/-9.0,(frame-10.0)/-9.0] 
    RenderAllViews()
    fname = "animation_%02d.png" % frame
    WriteImage(fname)
```
|}

### The Camera

The following macro creates x ,y, and z axes centered (x,y,z) = (0,0,0) and a sphere of radius 0.1 at the origin.

The goal of this problem is to determine the meaning of the terms `CameraPosition`, `CameraViewUp` `CameraFocalPoint`, and `CameraParallelScale` does, if anything.

After experimenting with this program, write your own explanation of each of these terms that could be used in place of the empty space after these terms that appear if you enter on the Python Shell command line

 r1 = GetActiveViewOrCreate('RenderView')
 help(r1)

Save your explanation as `HW8_EC.txt`.

```
from paraview.simple import *
paraview.simple._DisableFirstRenderCameraReset()

for s in GetSources().values():
    Delete(s)

renderView1 = GetActiveViewOrCreate('RenderView')

# x
cylinderX        = Cylinder()
cylinderX.Radius = 0.02
cylinderX.Center = [0.0, 0.5, 0.0]
cylinderX.Height = 1.0

cylinderXDisplay = Show(cylinderX, renderView1)
cylinderXDisplay.ColorArrayName = [None, '']
cylinderXDisplay.Orientation = [0.0, 0.0, -90.0]
cylinderXDisplay.DiffuseColor = [1.0, 0.0, 0.0]

# y
cylinderY        = Cylinder()
cylinderY.Radius = 0.02
cylinderY.Center = [0.0, 0.5, 0.0]
cylinderY.Height = 1.0

cylinderYDisplay = Show(cylinderY, renderView1)
cylinderYDisplay.ColorArrayName = [None, '']
cylinderYDisplay.Orientation = [0.0, 0.0, 0.0]
cylinderYDisplay.DiffuseColor = [1.0, 1.0, 0.5]

# z
cylinderZ        = Cylinder()
cylinderZ.Radius = 0.02
cylinderZ.Center = [0.0, 0.5, 0.0]
cylinderZ.Height = 1.0
cylinderZDisplay = Show(cylinderZ, renderView1)
cylinderZDisplay.ColorArrayName = [None, '']
cylinderZDisplay.Orientation = [90.0, 0.0, 0.0]
cylinderZDisplay.DiffuseColor = [0.0, 1.0, 0.0]

sphere1 = Sphere()
sphere1.Radius = 0.1
sphere1Display = Show(sphere1, renderView1)
sphere1Display.DiffuseColor = [0.0, 0.0, 1.0]


renderView1.CameraPosition = [1.0, 1.0, 0.0]
renderView1.CameraViewUp = [0.0, 0.0, 1.0]
renderView1.CameraFocalPoint = [0.0, 0.0, 0.0]
renderView1.CameraParallelScale = 1.0

Render()
```

## Notes  

### See also

* http://www.paraview.org/Wiki/ParaView/Simple_ParaView_3_Python_Filters
* http://www.paraview.org/Wiki/Python_Programmable_Filter
* http://www.paraview.org/Wiki/Python_Pictures_and_Movies
* http://www.paraview.org/Wiki/Python_Calculator

### Reading Files

 import os
 os.chdir('/path/to/files')
 reader = OpenDataFile('filename.vtk')
 Show()
 Render()

If you are editing a VTK file and want to avoid having to delete the pipeline and select the file with `File > Open` + Apply, use the following in the Python Shell:
 [Delete(s) for s in GetSources().values()];reader=OpenDataFile('filename.vtk');Show();Render()
and then use up-arrow and enter each time you want to test the file.  Or save this as a macro.

### Selective Deletion

Delete source with name `Sphere1`:

```
for s in GetSources():
    if (s[0]  'Sphere1'):
        print "Deleting" + s[0]
        Delete(FindSource(s[0]))
```

### Home Directory
Instead of using an absolute path, use [http://stackoverflow.com/questions/4028904/how-to-get-the-home-directory-in-python]
 from os.path import expanduser
 home = expanduser("~")
 execfile(home+'/sphere.py')

### Current Directory

```
 import os
 os.path.realpath('.')
```
or

```
 os.getcwd()
```

See also [http://stackoverflow.com/questions/2632199/how-do-i-get-the-path-of-the-current-executed-file-in-python?lq=1]

### Module and Scripts

Executing script from python command line
```
>>> execfile('sphere.py')
```

as an alternative to `python sphere.py`

From [http://stackoverflow.com/questions/1186789/what-is-the-best-way-to-call-a-python-script-from-another-python-script]:

If you want test1.py to remain executable with the same functionality as when it's called inside service.py, then do something like:

test1.py
```
def main():
    print "I am a test"
    print "see! I do nothing productive."

if __name__  "__main__":
    main()
```

service.py
```
import test1
test1.main() # do whatever is in test1.py
```

----

Another way:
```
File test1.py:
 print "test1.py"

File service.py:
 import subprocess
 subprocess.call("test1.py", shell=True)
```
