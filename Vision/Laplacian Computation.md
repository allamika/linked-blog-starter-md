
The laplacian is the some of the curvature (second derivative) along the axis:
$\nabla^2 I = \frac{\partial^2 I}{\partial x^2} + \frac{\partial^2 I}{\partial y^2}$
The discrete approximation is 
$\frac{\partial^2 I}{\partial x^2} \approx I(x+1, y) - 2I(x, y) + I(x-1, y)$
$\frac{\partial^2 I}{\partial y^2} \approx I(x, y+1) - 2I(x, y) + I(x, y-1)$
The laplacian kernels:
4 neighbor :
$$  
\begin{bmatrix}  
0 & 1 & 0 \\  
1 & -4 & 1 \\  
0 & 1 & 0  
\end{bmatrix}  
$$
8 neighbor :
$$  
\begin{bmatrix}  
1 & 1 & 1 \\  
1 & -8 & 1 \\  
1 & 1 & 1  
\end{bmatrix}  
$$
