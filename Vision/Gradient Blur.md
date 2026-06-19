Instead of treating all nearby pixels equally, Gaussian blur gives **more importance to pixels closer to the center** and less to those farther away. This creates a natural-looking blur.

$$  
G(x, y) = \frac{1}{2\pi\sigma^2} \, e^{-\frac{x^2 + y^2}{2\sigma^2}}  
$$
- x , y: distance from the center pixel
- $\sigma$ (sigma): controls how strong the blur is
    - Small sigma → slight blur
    - Large sigma → heavy blur