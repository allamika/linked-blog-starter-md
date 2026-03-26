The idea:  If you slightly move a window around a point, how much does the image change?

#### The local Energy function for a patch (x,y):
$$E(u,v) = \sum_{x,y}\left[I(x+u,y+v) - I(x,y)\right]^2$$
With the taylor expansion
$$I(x+u,y+v)≈I(x,y)+Ix​u+Iy​v$$
$$E(u,v) \approx \sum (I_x u + I_y v)^2$$
#### Matrix form:
$$  
E(u,v) \approx \sum_{x,y} w(x,y) \left( I_x^2 u^2 + 2 I_x I_y u v + I_y^2 v^2 \right)  
$$
$$  
E(u,v) =  
\begin{bmatrix}  
u & v  
\end{bmatrix}  
M  
\begin{bmatrix}  
u \\  
v  
\end{bmatrix}  
$$
$$  
M = \sum \\ \begin{bmatrix} 
I_x^2 & I_x I_y \\I_x I_y & I_y^2 \\ 
\end{bmatrix}  
$$

This measures how much the image changes when shifted by $(u,v)$
Matrix $M$ captures **gradient distribution** in a neighborhood.

#### Harris Response Function
Its eigenvalues λ1,λ2\lambda_1, \lambda_2λ1​,λ2​ tell us everything:
- Flat : small, small
- Edge : large, small
- Corner : large, large

$\det(M) = \lambda_1 \lambda_2$
$\text{tr}(M) = \lambda_1 + \lambda_2$

Response:
$R=det(M)−k⋅tr(M)^2$