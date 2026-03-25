# Advanced Mathematical Notes on Surface Extension in Siemens NX and Parasolid

Author: Generated technical summary

------------------------------------------------------------------------

# 1. Overview

Modern CAD systems such as **Siemens NX** rely on the **Parasolid
geometric kernel** for B-rep modeling. Surface extension operations ---
including **Extend**, **Law Extension**, and advanced surfacing tools
--- are implemented using:

-   NURBS mathematics
-   Differential geometry of surfaces
-   Boundary value interpolation
-   Numerical solvers

This document presents the deeper mathematical structures used in these
algorithms.

------------------------------------------------------------------------

# 2. NURBS Curve Mathematics

A NURBS curve is defined as

$$
C(u)=\frac{\sum_{i=0}^{n} N_{i,p}(u) w_i P_i}{\sum_{i=0}^{n} N_{i,p}(u) w_i}
$$

Where

-   $P_i$ : control points\
-   $w_i$ : weights\
-   $N_{i,p}$ : B‑spline basis functions of degree $p$

The basis functions follow the Cox--de Boor recursion:

$$
N_{i,0}(u)=
\begin{cases}
1 & u_i \le u < u_{i+1}\\
0 & otherwise
\end{cases}
$$

$$
N_{i,p}(u)=
\frac{u-u_i}{u_{i+p}-u_i}N_{i,p-1}(u)
+
\frac{u_{i+p+1}-u}{u_{i+p+1}-u_{i+1}}N_{i+1,p-1}(u)
$$

------------------------------------------------------------------------

# 3. NURBS Surface Representation

A tensor‑product surface:

$$
S(u,v)=
\frac{\sum_{i=0}^{n}\sum_{j=0}^{m}
N_{i,p}(u)M_{j,q}(v) w_{ij} P_{ij}}
{\sum_{i=0}^{n}\sum_{j=0}^{m}
N_{i,p}(u)M_{j,q}(v) w_{ij}}
$$

Domain:

$$
u \in [u_0,u_1], \quad v \in [v_0,v_1]
$$

Surface extension modifies the parameter domain while preserving
continuity.

------------------------------------------------------------------------

# 4. Differential Geometry of Surfaces

First derivatives:

$$
S_u=\frac{\partial S}{\partial u}, \quad
S_v=\frac{\partial S}{\partial v}
$$

Surface normal:

$$
N=\frac{S_u \times S_v}{||S_u \times S_v||}
$$

Second derivatives:

$$
S_{uu},\quad S_{uv},\quad S_{vv}
$$

These derivatives define curvature behavior.

------------------------------------------------------------------------

# 5. First Fundamental Form

The metric tensor of the surface:

$$
E = S_u \cdot S_u
$$

$$
F = S_u \cdot S_v
$$

$$
G = S_v \cdot S_v
$$

Matrix form:

$$
I =
\begin{bmatrix}
E & F \\
F & G
\end{bmatrix}
$$

This describes local surface stretching.

------------------------------------------------------------------------

# 6. Second Fundamental Form

Curvature terms:

$$
e = S_{uu} \cdot N
$$

$$
f = S_{uv} \cdot N
$$

$$
g = S_{vv} \cdot N
$$

Matrix:

$$
II =
\begin{bmatrix}
e & f \\
f & g
\end{bmatrix}
$$

------------------------------------------------------------------------

# 7. Principal Curvatures

Principal curvatures solve:

$$
det(II - kI)=0
$$

Leading to

$$
k_1, k_2
$$

Mean curvature:

$$
H = \frac{k_1 + k_2}{2}
$$

Gaussian curvature:

$$
K = k_1 k_2
$$

These curvature properties strongly influence high‑quality surface
extension.

------------------------------------------------------------------------

# 8. Continuity Constraints

Surface joins require continuity levels.

### G0

Position continuity

$$
S_1 = S_2
$$

### G1

Tangent continuity

$$
S_{1u} \parallel S_{2u}
$$

### G2

Curvature continuity

$$
S_{1uu} = S_{2uu}
$$

### G3 (Class‑A surfacing)

Curvature rate continuity

$$
\frac{d^3 S}{du^3}
$$

Automotive surfacing often targets G3.

------------------------------------------------------------------------

# 9. Parasolid Surface Extension Strategy

General pipeline:

1.  Extract boundary curve
2.  Compute derivatives
3.  Construct continuation surface
4.  Rebuild NURBS representation
5.  Update B‑rep topology

Boundary point:

$$
P(v)=S(u_1,v)
$$

Derivatives:

$$
T(v)=S_u(u_1,v)
$$

$$
K(v)=S_{uu}(u_1,v)
$$

------------------------------------------------------------------------

# 10. Control Net Extrapolation

If final control points are

$$
P_{n-2},P_{n-1},P_n
$$

Linear extrapolation:

$$
P_{n+1}=P_n+(P_n-P_{n-1})
$$

Second‑order extrapolation:

$$
P_{n+1}=
P_n + (P_n-P_{n-1})
+ \alpha(P_n-2P_{n-1}+P_{n-2})
$$

This preserves curvature trends.

------------------------------------------------------------------------

# 11. Knot Vector Extension

Given knot vector

$$
U=[u_0,...,u_m]
$$

Extension:

$$
U_{new}=[u_0,...,u_m,u_m+\Delta]
$$

Basis functions automatically expand.

------------------------------------------------------------------------

# 12. Hermite Surface Construction

Boundary conditions:

Position

$$
S(0,v)=C(v)
$$

Tangent

$$
\frac{\partial S}{\partial t}(0,v)=T(v)
$$

Hermite surface form:

$$
S(t,v)=
h_0(t)C(v)+
h_1(t)C_{ext}(v)+
h_2(t)T(v)+
h_3(t)T_{ext}(v)
$$

Basis functions:

$$
h_0=2t^3-3t^2+1
$$

$$
h_1=-2t^3+3t^2
$$

$$
h_2=t^3-2t^2+t
$$

$$
h_3=t^3-t^2
$$

------------------------------------------------------------------------

# 13. Law Extension Mathematics

Boundary curve

$$
C(v)=S(u_1,v)
$$

Spine curve

$$
\Gamma(s)=(x(s),y(s),z(s))
$$

Parameter mapping

$$
s=\frac{v-v_0}{v_1-v_0}
$$

Law function

$$
L(s)
$$

Direction vector

$$
D(v)=\frac{\Gamma'(s)}{||\Gamma'(s)||}
$$

Displacement field

$$
F(v)=L(s(v))D(v)
$$

New boundary curve

$$
C_{ext}(v)=C(v)+F(v)
$$

------------------------------------------------------------------------

# 14. Ruled Surface Approximation

Simplest extension surface:

$$
S(t,v)=(1-t)C(v)+tC_{ext}(v)
$$

with

$$
t\in[0,1]
$$

------------------------------------------------------------------------

# 15. Numerical Solvers

CAD kernels solve many nonlinear systems.

Newton iteration:

$$
x_{k+1}=x_k-J^{-1}F(x_k)
$$

Where

-   $J$ = Jacobian matrix

Applications:

-   surface intersection
-   closest point computation
-   trimming curve updates

------------------------------------------------------------------------

# 16. B‑Rep Topology Updates

After geometric creation:

1.  create new face
2.  extend edge curves
3.  update vertices
4.  re‑trim surfaces

The boundary representation maintains model consistency.

------------------------------------------------------------------------

# 17. Curvature‑Driven Extension (Advanced)

In high‑quality surfacing, extension direction may follow **principal
curvature directions**.

Eigenvectors of the shape operator give directions:

$$
S^{-1}II
$$

These vectors define surface flow directions used in Class‑A surface
generation.

------------------------------------------------------------------------

# 18. Summary

Surface extension algorithms combine:

-   NURBS algebra
-   differential geometry
-   boundary interpolation
-   numerical optimization
-   B‑rep topology management

These mathematical tools allow CAD kernels such as Parasolid to produce
stable, smooth surface continuations used in engineering and industrial
design.
