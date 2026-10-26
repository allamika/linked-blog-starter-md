
#### Preprocessing
- Edge detection 
- Thresholding 
- Shrink and expand
- Thinning 

### Fitting Lines to Edges
##### Naive formulation: Distance on Y axis
Minimizing the square of the distance on the y axis
- Forme : 
$$y=ax+b$$
- Residual error: 
$$S(a,b) = \sum_{i=1}^{n} e_i^2 = \sum_{i=1}^{n} (y_i - ax_i - b)^2$$
- Minimisation: We look for the partial derivative equal to 0
$$\frac{\partial S}{\partial a} = -2\sum_{i=1}^{n} x_i(y_i - ax_i - b) = 0$$$$\frac{\partial S}{\partial b} = -2\sum_{i=1}^{n} (y_i - ax_i - b) = 0$$
- We obtain the system:
$$\begin{cases} a\sum x_i^2 + b\sum x_i = \sum x_i y_i \\ a\sum x_i + nb = \sum y_i \end{cases}$$
- The solution is:
$$a = \frac{n\sum x_i y_i - \sum x_i \sum y_i}{n\sum x_i^2 - (\sum x_i)^2}, \quad b = \bar{y} - a\bar{x}$$
- Problem: It's not the minimisation we are really looking for 
 ![[Curve and line detection fig1.png]]
 
##### Better formulation: Perpendicular distance
- Forme
$$x \sin\theta - y \cos \theta + \rho = 0$$
- Residual error
$$S(\theta, \rho) = \frac{1}{N} \sum_{i} (x_{i} \sin \theta - y_{i} \cos \theta + \rho)^2$$
- Minimisation: We look for the partial derivative equal to 0
	- Minimize over $\rho$ :
$$\frac{\partial S}{\partial \rho} = \frac{N}{2} \sum_{i} (x_i \sin\theta - y_i \cos\theta + \rho) = 0$$
	- Which gives :
$$\rho^* = \bar{y}\cos\theta - \bar{x}\sin\theta$$
$$with: \bar{x} = \frac{1}{N}\sum x_i \quad and \quad \bar{y} = \frac{1}{N}\sum y_i$$
	- Substitute $\rho^*$ :
$$S(\theta) = \frac{1}{N}\sum_{i} \left[(x_i - \bar{x})\sin\theta - (y_i - \bar{y})\cos\theta\right]^2$$
$$S(\theta) = A\sin^2\theta - B\sin\theta\cos\theta + C\cos^2\theta$$$$with : A = \frac{1}{N}\sum \tilde{x}_i^2 \quad and \quad B = \frac{2}{N}\sum \tilde{x}_i\tilde{y}_i \quad and \quad C = \frac{1}{N}\sum \tilde{y}_i^2$$
	- Minimize over $\theta$ :
$$\frac{\partial S}{\partial \theta} = (A-C) \sin 2\theta = B \cos 2\theta$$
	- Which gives : 
$$\tan 2\theta^* = \frac{B}{A - C} = \frac{\frac{2}{N}\sum \tilde{x}_i\tilde{y}_i}{\frac{1}{N}\sum \tilde{x}_i^2 - \frac{1}{N}\sum \tilde{y}_i^2}$$

![[Curve and line detection fig2.png]]

### Curves
Find a polynomial that fits the points :
$y = f(x) = (ax^3 + bx^2 + cx + d)^2$

- Minimize $E = \frac{1}{N} \sum_i(y_i - ax_i^3 - bx_i^2 - cx_i - d)^2$
- Least squares fit : 
$$\frac{\partial E}{\partial a} = 0 \quad \frac{\partial E}{\partial b} = 0 \quad \frac{\partial E}{\partial c} = 0 \quad \frac{\partial E}{\partial d } = 0 \quad$$
- Solving as a Linear System:
$$Xa = y$$
$$$$