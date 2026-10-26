We write the system in matrix form:  
  
$$  
A x = b  
$$  
  
- $A \in \mathbb{R}^{n \times m}$: coefficient matrix  
- $x \in \mathbb{R}^{m}$: unknown vector  
- $b \in \mathbb{R}^{n}$: observation vector

#### Overdetermined System ($n > m$)

There is generally no exact solution. We solve:

$$
\min_{x} \|Ax - b\|^2
$$

This leads to the normal equations:

$$
A^T A x = A^T b
$$

Solution (if $A^T A$ is invertible):

$$
x = (A^T A)^{-1} A^T b
$$