Before using any of this operator it is strongly advise to use a Gaussian smoothing to reduce the sensibility of this operator to noise. As linear operator the gradient and the laplacian can be combine with the Gaussian smoothing to create one operator (GoG and LoG)

#### Gradient
$\nabla I = \left( \frac{\partial I}{\partial x}, \frac{\partial I}{\partial y} \right)$
The gradient operator can be compute with the sobel operator: [[Gradient Computation]]
- gradient magnitude = edge strength
	- $|\nabla I| = \sqrt{ \left( \frac{\partial I}{\partial x} \right)^2 +  \left( \frac{\partial I}{\partial y} \right)^2 }$
- gradient direction = edge orientation
	- $\theta = \tan^{-1}\left( \frac{\frac{\partial I}{\partial y}}{\frac{\partial I}{\partial x}} \right)$
	- the edge is perpendicular to this direction
Then a threshold need to be use to detect if a pixel is an edge or not

#### Laplacian
The laplacian is the some of the curvature (second derivative) along the axis ([[Laplacian Computation]]):
$\nabla^2 I = \frac{\partial^2 I}{\partial x^2} + \frac{\partial^2 I}{\partial y^2}$
Because the laplacian represent the magnitude of the second derivative and edge would be marked as a zero crossing