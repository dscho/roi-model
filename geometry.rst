Geometric shape primitives
==========================

Overview
--------

This section specifies how shapes are described in the model.  For
some shapes, there are several alternative ways of specifying them;
which are worth supporting needs further discussion.  One point to
consider is that the different ways preserve the intent behind the
original measurement and what is in the original metadata where this
makes sense, even if this does mean some redundancy; this won't impact
on the actual drawing/analysis code, which can deal with each shape in
a canonical form.  This records how the measurement was made by the
user, which may have implications in further analysis and/or
verification that the measurement was correct.

While some shapes have been included here for completeness, it's quite
possible that not all are needed, particularly in all dimensions.

If anyone wants to check the maths behind the geometry, that would be
much appreciated, because I'm firstly not an expert in this area, and
it's also quite possible I've made some typos.  The naming of the
shapes is probably also wanting some improvement.

Alternative shape representations
---------------------------------

Using the current ROI model is that there is only one way to describe
each shape.  e.g. a polyline can only be described as a series of
points; it might in some cases be more natural to specify one as a
starting point and a series of vectors; while either are fine just to
draw the ROI, it would desirable to store what was measured, since
converting it to a canonical representation is lossy, and removes the
original measurements taken, and hence the intent of the original
annotation.  This applies to other shapes as well.  For example, a
circle or ellipse can be described by a bounding box (which may itself
be a point and one or two vectors, or a set of points), or by a point
and radius or half-axes, or by the Mahalanobis distance (typically for
computing from a normal distribution of points).  For a cylinder/cone,
we can specify this in multiple ways also from a circle/ellipse plus
length, or point plus vector (length and direction) plus radius (or
half-axes).

The current model is focussed on drawing shapes, while making
measurements involves drawing only for visualisation; the important
parts are the values for making the measurement, and of course the
results.  Some programs (e.g. AxioVision) have separate sets of
objects for drawing (annotation) and measurement.  These are a largely
overlapping set, but the former are not used for any
length/area/volume/pixel measurements.  Objects such as scale bars and
labels are for drawing only.

.. todo::
    Common methods for all primitives:
    Bounding box [AlignedCuboid3D]
    Rotation centre [Vertex2D/Vertex3D]
    Control points [may use points and vertex to describe position and movement path]
    Conversion to 2D (slab through); equivalent to intersection with cuboid.  Should
    all primitives support a minimum of intersection with AlignedCuboid3D?  Or Mesh3D
    for non-square images.
    Can 2D methods use alternative axes to project in xz/yz?  Default
    to xy.  If all 2D shapes must be represented by 3D forms (i.e. are
    just proxies), then the equivalent 3D can be used quite simply.
    Get greymap/bitmap.
    Get 2D/3D mesh.
    Intersect (only for cuboid?)  Need to clip to image volume
    (optionally).  Also useful to reduce to 2D (which can be a cuboid
    for a single plane).
    Non-aligned shapes inherit/implement the aligned forms.
    Shrink and grow: move polygons along surface normals for meshes.
    For other shapes, this will require recalculation of the geometry.

    Add triangle as special case of polygon, which can be a special case of mesh?

    Meshes: Need to be able to triangulate if higher order polygons are possible.

    Add representation number to start of number list; this will allow
    shapes to be embedded in other shapes and be self-describing.
    e.g. all circle types may be used to specify a circular cylinder
    end.  This will simplify the specification of more complex shapes
    by limiting the number of variants.

Shape serialisation
-------------------

Key considerations:

- A shape exists in a set of dimensions e.g. xy, xyz, xyt.  The shape
  must define the number of dimensions it exists in, and their identity.
- A shape must be identifiable unambiguously
- A shape must be versioned (to permit correction of any
  design/analysis bugs without altering any data retrospectively);
  this permits the replacement of the buggy implementation while not
  removing it.
- In order to allow code reuse and flexible use of shapes, shapes may
  include other shapes as part of their primitive specification.

In the following shape descriptions, all shapes are identified by a
Shape ID and Representation ID.  The shape specifies the geometric
shape type.  The representation specifies both the primitives required
for serialisation, and can also be used for versioning the
shape--i.e. it also specifies the behaviour for conversion to greymaps
and bitmaps.

.. index::
    Shape

Shape
-----

An abstract description of a shape.

Representation:

==== ======== =================
Name Type     Description
==== ======== =================
S1   ShapeID  Shape
R1   RepID    Representation
==== ======== =================

Concrete implementations of shapes provide further elements in their
representation.  The above are only sufficient to describe the shape
and its representation.  The combination of shape and representation
specifies the data required to construct the shape.

Note that one disadvantage of this method is that a reader will be
required to understand how to deserialise all shape types; it's not
possible to skip unknown shapes due to not knowing their lengths
(which may be variable).  However, this would be an issue for a purely
XML-based implementation as well, so may not be a problem in practice.

When a shape embeds a specific shape, it may skip the ShapeID and/or
RepID header if one or both of these are fixed.  If both headers are
skipped, this is indicated with a '*', or if only the ShapeID header
is skipped, this is indicated with '&':

======= ========================================
Type    Description
======= ========================================
Cube3D  Shape contains a 3D cube
Cube3D* Shape contains a 3D cube with no header
Cube3D& Shape contains a 3D cube with RepID only
======= ========================================


Alignment
Aligned shape variants are aligned at right-angles to the x and y (2D) or x, y and z (3D) axes.


Circle2D
^^^^^^^^

.. todo::
   Specify using bounding square.  Inherits all Square2D and AlignedSquare2D representations.
    Specify using diameter (two points)

.. index:: Sphere3D

Sphere3D
^^^^^^^^

.. todo::
   Specify using bounding cube .  Inherits all Cube3D and AlignedCube3D representations.
    Specify using diameter (two points)


.. todo::
    Specify using reversed radius (vector3D to centre)
    Specify using diameter (two points)
    Specify using 4 points around surface (->radius and centre)


AlignedEllipse2D
^^^^^^^^^^^^^^^^


Representation 2: Bounding rectangle.  Inherits all AlignedRectangle2D
representations.


Ellipse2D
^^^^^^^^^

Representation 2: Bounding rectangle: Inherits all Rectangle2D and
AlignedRectangle2D representations.

Representation 3: Mahalanbobis distance used to draw an ellipse using the mean
coordinates (P1) and 2 × 2 covariance matrix (COV1)

==== ========= =======================
Name Type      Description
==== ========= =======================
S1   ShapeID   Shape
R1   RepID     Representation
P1   Vertex2D  Centre point (mean)
COV1 double[4] 2 × 2 covariance matrix
==== ========= =======================

.. index::
    AlignedEllipsoid3D

AlignedEllipsoid3D
^^^^^^^^^^^^^^^^^^

Aligned at right angles to xyz axes.

Representation 1: Centre and half axes

==== ======== =================
Name Type     Description
==== ======== =================
S1   ShapeID  Shape
R1   RepID    Representation
P1   Vertex3D Centre point
V1   Vector3D Half axes (x,y,z)
==== ======== =================

Representation 2: Centre and half axes (specified separately).

==== ======== ==============
Name Type     Description
==== ======== ==============
S1   ShapeID  Shape
R1   RepID    Representation
P1   Vertex3D Centre point
V1   Vector3D Half axis (x)
V2   Vector3D Half axis (y)
V3   Vector3D Half axis (z)
==== ======== ==============

Representation 3: Bounding cuboid: Inherits all AlignedCuboid3D representations.

.. index::
    Ellipsoid3D

Ellipsoid3D
^^^^^^^^^^^

May be rotated; not aligned at right angles to xyz axes.

Representation 1: Centre and half axes; V2 and V3 are at right-angles
to V1 and each other, so have reduced dimensions.

==== ======== ===============
Name Type     Description
==== ======== ===============
S1   ShapeID  Shape
R1   RepID    Representation
P1   Vertex3D Centre point
V1   Vector3D Half axes (xyz)
V2   Vector2D Half axes (xy)
V3   Vector1D Half axes (x)
==== ======== ===============

Representation 2: Bounding cuboid: Inherits all Cuboid3D and
AlignedCuboid3D representations.

Representation 3: Mahalanbobis distance used to draw an ellipse using the mean
coordinates (P1) and 3 × 3 covariance matrix (COV1)

==== ========= =======================
Name Type      Description
==== ========= =======================
S1   ShapeID   Shape
R1   RepID     Representation
P1   Vertex3D  Centre point (mean)
COV1 double[9] 3 × 3 covariance matrix
==== ========= =======================

.. index::
    Polyline Splines

Polyline Splines
----------------

.. index::
    PolylineSpline2D

PolylineSpline2D
^^^^^^^^^^^^^^^^

Representation:

======= ======== ================
Name    Type     Description
======= ======== ================
S1      ShapeID  Shape
R1      RepID    Representation
NPOINTS Count    Number of points
P1      Vertex2D Line start
…       Vertex2D Further points
Pn      Vertex2D Line end
======= ======== ================

.. index::
    PolylineSpline3D

PolylineSpline3D
^^^^^^^^^^^^^^^^

Representation:

======= ======== ================
Name    Type     Description
======= ======== ================
S1      ShapeID  Shape
R1      RepID    Representation
NPOINTS Count    Number of points
P1      Vertex3D Line start
…       Vertex3D Further points
Pn      Vertex3D Line end
======= ======== ================

.. index::
    Polygon splines

Polygon splines
---------------

.. index::
    PolygonSpline2D

PolygonSpline2D
^^^^^^^^^^^^^^^

Representation:

======= ======== ================
Name    Type     Description
======= ======== ================
S1      ShapeID  Shape
R1      RepID    Representation
NPOINTS Count    Number of points
P1      Vertex2D Line start
…       Vertex2D Further points
Pn      Vertex2D Line end
======= ======== ================

.. index::
    PolygonSpline3D

PolygonSpline3D
^^^^^^^^^^^^^^^

Representation:

======= ======== ================
Name    Type     Description
======= ======== ================
S1      ShapeID  Shape
R1      RepID    Representation
NPOINTS Count    Number of points
P1      Vertex3D Line start
…       Vertex3D Further points
Pn      Vertex3D Line end
======= ======== ================

.. index::
    Cylinders

Cylinders
---------

.. index::
    AlignedCircularCylinder3D

AlignedCircularCylinder3D
^^^^^^^^^^^^^^^^^^^^^^^^^

Aligned 

.. index::
    CircularCylinder3D

CircularCylinder3D
^^^^^^^^^^^^^^^^^^

Representation 1: Start and endpoint, plus radius.

==== ======== =====================
Name Type     Description
==== ======== =====================
S1   ShapeID  Shape
R1   RepID    Representation
P1   Vertex3D Centre of first face
P2   Vertex3D Centre of second face
V1   Vector1D Radius
==== ======== =====================

Representation 2: Start point, distance to endpoint, plus radius

==== ======== =================================
Name Type     Description
==== ======== =================================
S1   ShapeID  Shape
R1   RepID    Representation
P1   Vertex3D Centre of first face
V1   Vector3D Distance to centre of second face
V2   Vector1D Radius
==== ======== =================================

Representation 3: Start and endpoint, plus vectors to define radius
(V1) and angle of start face, and unit vector defining angle of end
face.  Face angles other than right-angles let chains of cyclinders be
used for tubular structures without gaps at the joins.

.. note::
    Should V2 only allow angle, assuming radius from V1, or also allow
    a second radius to represent a conical section?

==== ======== ==============================
Name Type     Description
==== ======== ==============================
S1   ShapeID  Shape
R1   RepID    Representation
P1   Vertex3D Centre of first face
P2   Vertex3D Centre of second face
V1   Vector3D Radius and angle of first face
V2   Vector3D Angle of second face
==== ======== ==============================

Representation 4: Start point, distance to endpoint, plus vectors to
define radius (V2) and angle of start face, and unit vector defining
angle of end face (V3).  Face angles other than right-angles let
chains of cyclinders be used for tubular structures without gaps at
the joins.

==== ======== =================================
Name Type     Description
==== ======== =================================
S1   ShapeID  Shape
R1   RepID    Representation
P1   Vertex3D Centre of first face
V1   Vector3D Distance to centre of second face
V2   Vector3D Radius and angle of first face
V3   Vector3D Angle of second face
==== ======== =================================

.. note::
    Should V3 only allow angle, assuming radius from V2, or also allow
    a second radius to represent a conical section?

.. index::
    AlignedEllipticCylinder3D

AlignedEllipticCylinder3D
^^^^^^^^^^^^^^^^^^^^^^^^^

.. todo::
    Inherits from AlignedEllipse.

.. index::
    EllipticCylinder3D

EllipticCylinder3D
^^^^^^^^^^^^^^^^^^

Representations 1 and 2 describe basic elliptic cylinders with faces
at right angles; the following representations permit faces at
arbitrary angles.  Face angles other than right-angles let chains of
cyclinders be used for tubular structures without gaps at the joins.

Representation 1: Start and endpoint, plus half axes.

==== ======== =====================
Name Type     Description
==== ======== =====================
S1   ShapeID  Shape
R1   RepID    Representation
P1   Vertex3D Centre of first face
P2   Vertex3D Centre of second face
V1   Vector2D Half axes (xy)
V2   Vector1D Half axes (x)
==== ======== =====================

.. note::
   Is the dimensionality of the half axes correct here?

Representation 2: Start point, distance to endpoint, plus half axes

==== ======== =======================
Name Type     Description
==== ======== =======================
S1   ShapeID  Shape
R1   RepID    Representation
P1   Vertex3D Centre of first face
V1   Vector3D Distance to second face
V2   Vector3D Half axes (xy)
V3   Vector2D Half axes (x)
==== ======== =======================

.. note::
   Is the dimensionality of the half axes correct here?

.. todo::
    Should half axes and angle be specified in same vector or separately?

 3: Start and endpoint, plus vectors to define half axes (V1 and V2)
    and angle of start face, and unit vector defining angle of end
    face (V3).

==== ======== =============================
Name Type     Description
==== ======== =============================
S1   ShapeID  Shape
R1   RepID    Representation
P1   Vertex3D Centre of first face
P2   Vertex3D Centre of second face
V1   Vector3D Half axes of first face (xyz)
V2   Vector2D Half axes of first face (xy)
V3   Vector3D Angle of second face
==== ======== =============================

 3: Start and endpoint, plus vectors to define half axes (V1 and V2)
    and angle of start face, and unit vector defining angle of end
    face (V3).

==== ======== =======================
Name Type     Description
==== ======== =======================
S1   ShapeID  Shape
R1   RepID    Representation
P1   Vertex3D Centre of first face
V1   Vector3D Distance to second face
V2   Vector3D Half axes (xyz)
V3   Vector2D Half axes (xy)
V4   Vector3D Angle of second face
==== ======== =======================

Representation 4: Bounding cuboid: Inherits all Cube3D and Cuboid3D
representations; first face is the base.

.. index::
    Arcs

Arcs
----

.. index::
    Arc2D

Arc2D
^^^^^

Representation 1:

==== ======== ==============
Name Type     Description
==== ======== ==============
S1   ShapeID  Shape
R1   RepID    Representation
P1   Vertex2D Centre point
P2   Vertex2D Arc start
V1   Vector2D Arc end
==== ======== ==============

Representation 2:

==== ======== ==============
Name Type     Description
==== ======== ==============
S1   ShapeID  Shape
R1   RepID    Representation
P1   Vertex2D Centre point
V2   Vector2D Arc start
V1   Vector2D Arc end
==== ======== ==============

.. index::
    Arc3D

Arc3D
^^^^^

Representation 1:

==== ======== ==============
Name Type     Description
==== ======== ==============
S1   ShapeID  Shape
R1   RepID    Representation
P1   Vertex3D Centre point
P2   Vertex3D Arc start
V1   Vector3D Arc end
==== ======== ==============

Representation 2:

==== ======== ==============
Name Type     Description
==== ======== ==============
S1   ShapeID  Shape
R1   RepID    Representation
P1   Vertex3D Centre point
V2   Vector3D Arc start
V1   Vector3D Arc end
==== ======== ==============

.. index::
    Masks

Masks
-----

Masks may be either grey masks (double or integer) or bitmasks.

For all of the following masks, DATA should be stored outside the ROI
specification either as BinData or (better) in an IFD for OME-TIFF.
It could be stored as part of the double array, but this would be
quite inefficient.

.. note::
   Masks are applied to the bounding rectangle, and so a 1:1
   correspondance between mask and image pixel data is not required.
   In this case, a new greymask should be computed which is aligned
   with the pixel data, and then (if required) thresholded to a
   bitmask.

.. index::
    GreyMask2D

GreyMask2D
^^^^^^^^^^

Representation:

The mask is applied to the bounding rectangle.  Dimensions specify the
x and y size of the mask.  DATA is the mask pixel data.

==== =========== =================================
Name Type        Description
==== =========== =================================
S1   ShapeID     Shape
R1   RepID       Representation
P1   Vertex2D    Start point of bounding rectangle
P2   Vertex2D    End point of bounding rectangle
DIM1 Vector2D    Mask dimensions (x,y)
DATA double[x,y] Mask data
==== =========== =================================

.. index::
    BitMask2D

BitMask2D
^^^^^^^^^

Representation:

The mask is applied to the bounding rectangle.  Dimensions specify the
x and y size of the mask.  DATA is the mask pixel data.

==== =========== =================================
Name Type        Description
==== =========== =================================
S1   ShapeID     Shape
R1   RepID       Representation
P1   Vertex2D    Start point of bounding rectangle
P2   Vertex2D    End point of bounding rectangle
DIM1 Vector2D    Mask dimensions (x,y)
DATA bool[x,y]   Mask data
==== =========== =================================

.. index::
    GreyMask3D

GreyMask3D
^^^^^^^^^^

Representation:

The mask is applied to the bounding cuboid.  Dimensions specify the
x, y and z size of the mask.  DATA is the mask pixel data.

==== ============= =================================
Name Type          Description
==== ============= =================================
S1   ShapeID       Shape
R1   RepID         Representation
P1   Vertex3D      Start point of bounding rectangle
P2   Vertex3D      End point of bounding rectangle
DIM1 Vector3D      Mask dimensions (x,y)
DATA double[x,y,z] Mask data
==== ============= =================================

.. index::
    BitMask3D

BitMask3D
^^^^^^^^^

Representation:

The mask is applied to the bounding cuboid.  Dimensions specify the
x, y and z size of the mask.  DATA is the mask pixel data.

==== =========== =================================
Name Type        Description
==== =========== =================================
S1   ShapeID     Shape
R1   RepID       Representation
P1   Vertex3D    Start point of bounding rectangle
P2   Vertex3D    End point of bounding rectangle
DIM1 Vector3D    Mask dimensions (x,y)
DATA bool[x,y,z] Mask data
==== =========== =================================

.. index::
    Meshes

Meshes
------


Mesh representation depends upon the mesh format.  In the examples
below, face-vertex meshes are used.

.. index::
    Mesh2D

Mesh2D
^^^^^^

Representation:

===== ================ ====================================================
Name  Type             Description
===== ================ ====================================================
S1    ShapeID          Shape
R1    RepID            Representation
NFACE Count            Number of faces
VREF  double[NFACE][3] Vertex references per face, counterclockwise winding
NVERT Count            Number of vertices
VERTS Vertex2D[NVERT]  Vertex coordinates
===== ================ ====================================================

Vertex references are indexes into the VERTS array.  Vertex-face
mapping is implied, and will require the implementor to construct the
mapping.

.. index::
    Mesh3D

Mesh3D
^^^^^^

Representation:

===== ================ ====================================================
Name  Type             Description
===== ================ ====================================================
S1    ShapeID          Shape
R1    RepID            Representation
NFACE Count            Number of faces
VREF  double[NFACE][3] Vertex references per face, counterclockwise winding
NVERT Count            Number of vertices
VERTS Vertex3D[NVERT]  Vertex coordinates
===== ================ ====================================================

Vertex references are indexes into the VERTS array.  Vertex-face
mapping is implied, and will require the implementor to construct the
mapping.

.. index::
    Labels

Labels
------


Text placement and alignment
----------------------------

In order to annotate text next to measurements, it would be ideal if
it were possible to control text placement and orientation.  Currently
the coordinate of the first letter is required.  However, it would be
nicer if the text could be also placed to the right of the point or
centred on the point.  And additionally, to the top, middle or bottom
for vertical placement.  Rotation would also be useful, though it's
probably achievable indirectly via the transformation matrix, i.e. you
would effectively have these anchors for placement, where 1 is the
current behaviour.

::

   7      8      9
   4Text h5ere...6
   1      2      3

This is needed to e.g. align text along measurement lines.  Having a
rotation angle specified directly would also save the need for complex
calculations to work out the rotation origin and transform every time
you want to just place a label along a line.  It also makes it
possible to place text in the centre of a shape.


.. index::
    Text2D

Text2D
^^^^^^

Representation 1: Text aligned relative to a point.  Inherits all
Point2D and Point3D representations.

Representation 2: Text aligned relative to a line.  Inherits all
Line2D and Line3D, Direction2D and Direction3D representations.
    
Representation 3: Text aligned and flowed inside a rectangle.
Inherits all AlignedSquare2D, Square2D, AlignedRectangle2D and
Rectangle2D representations.

.. index::
    Scale bars

Scale bars
----------

.. index::
    Scale2D

Scale2D
^^^^^^^

Representation 1: Scale bar between two points.  Inherits all Line2D representations.

Representation 1: Scale bar described by vector.  Inherits all Distance2D representations.

.. index::
    Scale3D

Scale3D
^^^^^^^

Representation 1: Scale bar between two points.  Inherits all Line3D representations

Representation 1: Scale bar described by vector.  Inherits all Distance3D representations.

.. note::
    A 3D scale may need to be a 3D grid to allow visualisation of
    perspective, in which case the representation will define the grid
    bounding cuboid; inherit AlignedCuboid3D representations.  Permit
    scale rotation with Cuboid3D?  Allow specification of grid size
    and only allow sizing in discrete units?

Additional primitives
---------------------

3D spline surfaces
  Natural cubic spline (Catmull-Rom)

The axiovision curve type is most likely a natural cubic spline, the
curve passing smoothly through all points, but without local control.
It is simply represented as a list of points through which the curve
must pass; there are no additional control points.  Depending upon if
they are doing any custom stuff, it might not be possible to represent
with pixel-perfect accuracy.

Curves might be more generally applicable to other formats, and useful
in their own right.  It might be worth considering adding a spline
type with local control where the curve passes straight through the
control points such as Catmull-Rom splines.  This would make it very
simple for non-experts to fit smooth lines while annotating their
images.

Shape combination
-----------------

Logical combination
^^^^^^^^^^^^^^^^^^^

Representation:

Logical combination of two shapes using the formula S2 B1 S3, e.g S2 ∧
S3.  Shapes must have compatible dimensions.

==== ======== ================================
Name Type     Description
==== ======== ================================
S1   ShapeID  Shape
R1   RepID    Representation
B1   BLogic   Bitwise binary logical operator
S2   ShapeID  Shape1
S3   ShapeID  Shape2
==== ======== ================================

Concatenation
^^^^^^^^^^^^^

Logical combination of shapes.  Shapes must not share any dimensions,
i.e. this is a means of adding additional dimensions to shapes.  The
purpose is to enable combination of 2D or 3D geometry with additional
nD shapes.

Representation:

======= ================ =================
Name    Type             Description
======= ================ =================
S1      ShapeID          Shape
R1      RepID            Representation
NSHAPES Count            Number of shapes
SHAPES  ShapeID[NSHAPES] Shapes to combine
======= ================ =================
