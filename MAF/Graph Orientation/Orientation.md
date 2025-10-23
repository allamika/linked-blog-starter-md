# Orientation LinePack V3

## Cavity detection and position

### Detection

The orientation uses YOLO models to detect the position of the stem and the fruit on the image. Our goal is to estimate the position of the stem on the fruit to estimate the next movement to do.

A detection system runs in parallel for all orientors using batches and produces bounding boxes of the fruit and cavity in the image. Then, we calculate the position of the center of the cavity box in the fruit bounding box and normalize it $(x_i, y_i)$.
### To sperical coordinates

We use the assumption that the fruit is a sphere and project the image $(xi,yi)$ on it $(x,y,z)$.
$x = xi$
$y = yi$
$z = 1 - (x^2 - y^2)$

This Cartesian coordinates are in a coordinate space where:
- $\vec{z}$ points to the camera
- $\vec{x}$ and $\vec{y}$ are the same as in the image coordinate system

But we want our final coordinates to match the coordinate system of the orientors:
- $\vec{x}$ following the belt
- $\vec{y}$ from the fruit supply to the belt
- $\vec{z}$ from bottom to top  

We know the vertical and horizontal angle of the camera, $\theta$ and $\phi$. So two rotations are applied:
- the first of $\theta$ around the $\vec{x}$ axis to put the $\vec{z}$ axis in the right position
- the second of $\phi$ around the $\vec{z}$ axis to put the $\vec{y}$ and $\vec{x}$ axis in the right position

Then we can move to the spherical coordinate system:

$r = \sqrt{x^2 + y^2 + z^2}$
$\theta = \arccos{z}$
$\phi = \arctan{(y/x)}$

In the final coordinate system $\theta=0$ correspond to the top position and $\phi=0$ correspond to the position maximizing $x$

---
## Orientation algorithm

### Architecture

The algoritm is a feedback loop that detects => estimates cavity position on fruit => estimates next movement => commands the motors => detects...\

This algorithm uses steps and jump between then depending on the completion of the current step or change of state of the system.\

During each loop, the thread goes through the orientation function to compute the next mouvement to the motors depending on the current step of the orientation and the position of the cavity then move to the next step is needed.
### Movement function

The orientor are able to do multiple movements : Forward, Backward, Clockwise, CouterClockwise and diagonal mouvement (combinaision of the four first).

Multiple function are available to create an orientation:
- OrientationStemResearchPeach: alternates forward and clockwise movement to find the cavity. Returns true and goes to the next step if the cavity is found.
- SetCavityToThetaAndPhi(theta, phi): computes a motor mouvement to approach the cavity to the input position. Returns true and goes to the next step if the cavity is close enough to the input position.
- SetCavityToTheta(theta) : moves forward until the cavity position reaches theta. If the cavity is lost in the process, a timeout depending on the last position of the stem is launched. Returns true if the cavity reach the input theta position or if the time out is reached
- SetCavityToTheta(phi) : moves forward until the cavity position reaches theta. If the cavity is lost in the process, a timeout depending on the last position of the stem is launched. Returns true if the cavity reach the input phi position or if the time out is reached

#### Fonction with target

Some of the orientation functions use a target position($T\phi, T\theta$) and a tolerance angle ($Tol\phi, Tol\theta$) for the stem final position. This tolerance produces an area where the stem is judge as being in a valid position, red for $\phi$, yellow for $\theta$ and orange when both conditions are met.
But due to the instability of phi around the pole the valid area becomes very small and hard to use. To avoid this issue we choose to modulate $Tol\phi$ near the pole. The factor $p$ can be used to amplify or diminish this effect.
$PTol\phi= Tol\phi / \sin^p{\theta}$

A geogebra file is available to visualize the tolerance area with a given $Tol\phi$, $Tol\theta$ and $T\theta$

### Cavity in Plane Orientation

### Cavity Bottom Orientation

### Cavity Top Orientation

### Color Orientation