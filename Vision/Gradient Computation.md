
The **gradient** of an image measures how quickly pixel intensity changes in different directions.

For a grayscale image $I(x,y)$, the gradient is a vector:

$∇I=(∂I∂x,∂I∂y)\nabla I = \left( \frac{\partial I}{\partial x}, \frac{\partial I}{\partial y} \right)∇I=(∂x∂I​,∂y∂I​)$

- $\frac{\partial I}{\partial x}$​: change in horizontal direction
- $\frac{\partial I}{\partial y}$​: change in vertical direction


#### Sobel

- Most popular method
- Uses smoothing + differentiation

Kernels:

$G_x \begin{bmatrix} -1 & 0 & 1 \\ -2 & 0 & 2 \\ -1 & 0 & 1 \end{bmatrix} \quad G_y = \begin{bmatrix} -1 & -2 & -1 \\ 0 & 0 & 0 \\ 1 & 2 & 1 \end{bmatrix}$

$G=\sqrt{Gx2​+Gy2​​}$
$θ=arctan2(Gy​,Gx​)$

Where:
- $G$ → strength of change → likely an edge
- $\theta$ → orientation of edge → used in thinning