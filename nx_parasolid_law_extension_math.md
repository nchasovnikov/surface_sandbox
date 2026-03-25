# Mathematical Foundations of Extension Tools in Siemens NX

This document summarizes the mathematical explanations of: 1. The
**Extension Tool** in Siemens NX 2. The **Parasolid kernel extension
algorithms** 3. The **Law Extension tool with spine curve definition**

------------------------------------------------------------------------

# 1. Parametric Representation of Geometry

Most CAD systems represent curves and surfaces parametrically.

## Curves

A curve is defined as:

$$
\mathbf{C}(u) = (x(u), y(u), z(u)), \quad u \in [u_0, u_1]
$$

For a **NURBS curve**:

$$
\mathbf{C}(u) =
\frac{\sum_{i=0}^{n} N_{i,p}(u)\, w_i\, \mathbf{P}_i}
{\sum_{i=0}^{n} N_{i,p}(u)\, w_i}
$$

Where:

-   $P_i$ = control points\
-   $w_i$ = weights\
-   $N_{i,p}(u)$ = B-spline basis functions\
-   $p$ = polynomial degree

Extension occurs when evaluating the curve outside its domain:

$$
u > u_1 \quad \text{or} \quad u < u_0
$$

------------------------------------------------------------------------

# 2. Linear (Tangent) Extension

At the endpoint:

$$
\mathbf{C}(u_1)
$$

Tangent vector:

$$
\mathbf{T} = \frac{d\mathbf{C}}{du}\Big|_{u=u_1}
$$

The extension line is:

$$
\mathbf{E}(s) =
\mathbf{C}(u_1) + s\,\mathbf{T}
$$

where

$$
s \ge 0
$$

This preserves **G1 continuity**.

------------------------------------------------------------------------

# 3. Polynomial / Natural Spline Extension

If the last segment of the spline is:

$$
\mathbf{C}(u) =
a_0 + a_1 u + a_2 u^2 + a_3 u^3
$$

Extension evaluates the same polynomial for:

$$
u > u_1
$$

This typically preserves curvature trends.

------------------------------------------------------------------------

# 4. Surface Representation

Surfaces use two parameters:

$$
\mathbf{S}(u,v)
$$

For NURBS surfaces:

$$
S(u,v)=
\frac{\sum_{i=0}^{n}\sum_{j=0}^{m} N_{i,p}(u)M_{j,q}(v) w_{ij} P_{ij}}
{\sum_{i=0}^{n}\sum_{j=0}^{m} N_{i,p}(u)M_{j,q}(v) w_{ij}}
$$

Domain:

$$
u \in [u_0,u_1], \quad v \in [v_0,v_1]
$$

Surface extension modifies this domain.

------------------------------------------------------------------------

# 5. Parasolid Kernel Extension Algorithm

Parasolid typically performs three steps:

1.  Boundary derivative evaluation\
2.  Control net extrapolation\
3.  Rebuilding a valid NURBS surface

## Boundary derivatives

Position:

$$
P_0 = S(u_1,v)
$$

First derivative:

$$
T = \frac{\partial S}{\partial u}(u_1,v)
$$

Second derivative:

$$
K = \frac{\partial^2 S}{\partial u^2}(u_1,v)
$$

------------------------------------------------------------------------

## Control Net Extrapolation

If the last control points are:

$$
P_{n-1}, P_n
$$

Next control point:

$$
P_{n+1} = P_n + (P_n - P_{n-1})
$$

Higher-order extrapolation:

$$
P_{n+1}=P_n + (P_n-P_{n-1}) + \alpha(P_n-2P_{n-1}+P_{n-2})
$$

------------------------------------------------------------------------

## Knot Vector Extension

Original knot vector:

$$
U = [u_0, u_1, ..., u_m]
$$

Extended knot vector:

$$
U_{new} =
[u_0,...,u_m, u_m+\Delta]
$$

------------------------------------------------------------------------

# 6. Newton Solver Used in Kernels

Many geometric operations solve nonlinear systems using
**Newton--Raphson iteration**.

$$
x_{k+1} = x_k - J^{-1}F(x_k)
$$

Used for:

-   curve-surface intersections
-   reparameterization
-   trimming updates

------------------------------------------------------------------------

# 7. Law Extension in Siemens NX

Law Extension uses:

-   boundary curve
-   spine curve
-   law function
-   direction field

------------------------------------------------------------------------

## Boundary Curve

For surface boundary at $u = u_1$:

$$
C(v) = S(u_1,v)
$$

------------------------------------------------------------------------

## Spine Curve

Defined as:

$$
\Gamma(s) = (x(s),y(s),z(s)), \quad s \in [0,1]
$$

Parameter mapping:

$$
s = \frac{v-v_0}{v_1-v_0}
$$

------------------------------------------------------------------------

## Law Function

Defines extension magnitude:

$$
L(s)
$$

Example polynomial:

$$
L(s) = a_0 + a_1 s + a_2 s^2
$$

------------------------------------------------------------------------

## Surface Normal

$$
N(v) =
\frac{\partial S}{\partial u} \times
\frac{\partial S}{\partial v}
$$

Normalized.

------------------------------------------------------------------------

## Extension Point

Boundary point:

$$
P(v) = S(u_1,v)
$$

Extended point:

$$
P_{ext}(v) =
P(v) + L(s) D(v)
$$

Where $D(v)$ is the direction vector.

------------------------------------------------------------------------

# 8. Extension Surface

A ruled surface form:

$$
S_{ext}(t,v) =
(1-t)C(v) + tC_{ext}(v)
$$

with

$$
t \in [0,1]
$$

------------------------------------------------------------------------

# 9. Hermite Surface Form (G1/G2 Continuity)

General Hermite surface:

$$
S_{ext}(t,v) =
h_0(t)C(v)
+ h_1(t)C_{ext}(v)
+ h_2(t)T(v)
+ h_3(t)T_{ext}(v)
$$

Where:

-   $h_i(t)$ = Hermite basis functions
-   $T(v)$ = tangent vectors

------------------------------------------------------------------------

# 10. Displacement Field Interpretation

Law Extension can be interpreted as a displacement field:

$$
\mathbf{F}(v) = L(s(v)) D(v)
$$

This field generates a new surface from the boundary curve.

------------------------------------------------------------------------

# Summary

The extension operations in Siemens NX and Parasolid rely on:

-   NURBS mathematics
-   control net extrapolation
-   derivative continuity
-   displacement fields guided by law functions and spine curves.

These techniques allow smooth geometric continuation while maintaining
CAD model robustness.
