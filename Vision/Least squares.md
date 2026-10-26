
#### Matrix form
We are trying to find the solution that minimise the error $\varepsilon$ of $y = X\beta + \varepsilon$
$$ X = \begin{pmatrix} x_1 & 1 \\ x_2 & 1 \\ \vdots & \vdots \\ x_n & 1 \end{pmatrix}, \quad \beta = \begin{pmatrix} a \\ b \end{pmatrix}, \quad y = \begin{pmatrix} y_1 \\ \vdots \\ y_n \end{pmatrix} $$
We are tying to minimize $\|\mathbf{y} - \mathbf{X}\boldsymbol{\beta}\|^2$
$S(\beta) = \|\mathbf{y} - \mathbf{X}\boldsymbol{\beta}\|^2 = (\mathbf{y} - \mathbf{X}\boldsymbol{\beta})^\top(\mathbf{y} - \mathbf{X}\boldsymbol{\beta})$
C'est une **forme quadratique convexe** en $\boldsymbol{\beta}$  elle ressemble à une parabole dans un espace de dimension pp p. Elle admet donc un minimum global unique (si $\mathbf{X}^\top \mathbf{X}$ est inversible).

$\nabla_{\beta} S = -2\mathbf{X}^\top \mathbf{y} + 2\mathbf{X}^\top \mathbf{X}\boldsymbol{\beta} = 0$
$-2\mathbf{X}^\top \mathbf{y} + 2\mathbf{X}^\top \mathbf{X}\hat{\boldsymbol{\beta}} = 0$

**Gauss Solution:**

$\hat{\boldsymbol{\beta}} = (\mathbf{X}^\top \mathbf{X})^{-1} \mathbf{X}^\top \mathbf{y}$
