# Orientation LinePack V3

## Motor rotation direction convention
The Linepack as two type of orientor used depending on the fruit and the orientation needed
### Bicone
The Bicone orientor is used for handling apples. It provides three available movements: forward, backward and rotation of the suction cup. The direction of these movements can be configured in Setting => Test => Bic. rotat/Vent rotat
### Band
The Band orientor is used for peaches, plums, oranges, and clementines. It consists of two parallel belts that can move independently. By combining their motions, the system can produce forward movement, backward movement, clockwise rotation, counterclockwise rotation, or a combination of these actions. The direction of these movements can be configured in Setting => Test => Bic Rotat / Motor Posit
## Cavity detection and position
### Detection

The orientation uses YOLO models to detect the position of the stem and the fruit on the image. Our goal is to estimate the position of the stem on the fruit to estimate the next movement to do.

A detection system runs in parallel for all orientors using batches and produces bounding boxes of the fruit and cavity in the image. Then, we calculate the position of the center of the cavity box in the fruit bounding box and normalize it $(x_i, y_i)$.

### To sperical coordinates

We use the assumption that the fruit is a unit-sphere and reproject the image point $(xi,yi)$ on its $(x,y,z)$-coordinates of the unit-sphere:

$x = xi $\
$y = yi $\
$z = 1 - (x^2 - y^2) $ 

This Cartesian coordinates are in a coordinate space where:
- $\vec{z}$ points to the camera.
- $\vec{x}$ and $\vec{y}$ are the same as in the image coordinate system.

But we want our final coordinates to match the coordinate system of the orientors:
- $\vec{x}$ following the belt 
- $\vec{y}$ from the fruit supply to the belt 
- $\vec{z}$ from bottom to top

![orientationV3Graph](https://github.maf-roda.com/user-attachments/assets/6104b902-a519-4136-817a-f3ca9eda18e8)

We know the vertical and horizontal angle of the camera, $\theta$ and $\phi$. So two rotations are applied:
- the first of $\theta$ around the $\vec{x}$ axis to put the $\vec{z}$ axis in the right position 
- the second of $\phi$ around the $\vec{z}$ axis to put the $\vec{y}$ and $\vec{x}$ axis in the right position

Then we can move to the spherical coordinate system:

$r = \sqrt{x^2 + y^2 + z^2}$\
$\theta = \arccos{z} $\
$\phi = \arctan{(y/x)}$

In the final coordinate system $\theta=0$ correspond to the top position and $\phi=0$ correspond to the position maximizing $x$

---

## Orientation algorithm
### Architecture 
The algoritm is a feedback loop that detects => estimates cavity position on fruit => estimates next movement => commands the motors => detects...\
This algorithm uses steps and jump between then depending on the completion of the current step or change of state of the system.\
During each loop, the thread goes through the orientation function to compute the next mouvement to the motors depending on the current step of the orientation and the position of the cavity then move to the next step is needed.

Most orientation processes follow the same step structure.
- **WaitingForFruit**: the system waits a few hundred milliseconds to allow the fruit to stabilize inside the orientor.
- **OStemResearch**: the system alternates between forward and clockwise movements in order to search for the stem.
- Orientation movement step (series of step) : specific movement sequence is executed to position the cavity in the desired location. This step depends on the particular orientation being performed.
- **CFin**: motors are stopped and orientor status is updated
### Movement function
The orientor are able to do multiple movements : Forward, Backward, Clockwise, CouterClockwise and diagonal mouvement (combinaision of the four first).

Multiple function are available to create an orientation:
- _OrientationStemResearchPeach_: alternates forward and clockwise movement to find the cavity. Returns true and goes to the next step if the cavity is found.
- _SetCavityToThetaAndPhi(theta, phi)_: computes a motor mouvement to approach the cavity to the input position. Returns true and goes to the next step if the cavity is close enough to the input position.
- _SetCavityToTheta(theta)_ : moves forward until the cavity position reaches _theta_. If the cavity is lost in the process, a timeout depending on the last position of the stem is launched. Returns true if the cavity reach the input _theta_ position or if the time out is reached.
- _SetCavityToPhi(phi)_ : moves forward until the cavity position reaches _phi_. If the cavity is lost in the process, a timeout depending on the last position of the stem is launched. Returns true if the cavity reach the input _phi_ position or if the time out is reached.

#### Fonction with target

Some of the orientation functions use a target position($T\phi, T\theta$) and a tolerance angle ($Tol\phi, Tol\theta$) for the stem final position. This tolerance produces an area where the stem is judge as being in a valid position, red for $\phi$, yellow for $\theta$ and orange when both conditions are met.

Base system            |  Base system at pole  |  Modulation at pole
:-------------------------:|:-------------------------:|:-------------------------:
![geo1](https://github.maf-roda.com/user-attachments/assets/315de97d-b233-42f8-acd1-e1a1243fb87f)  |  ![geo2](https://github.maf-roda.com/user-attachments/assets/9adec1b3-8c30-48d5-a160-3b7582f9f04a)  |  ![geo3](https://github.maf-roda.com/user-attachments/assets/8aee3508-11ff-41e0-ac11-66ab4fa02a2b)



But due to the instability of $\phi$ around the pole the valid area becomes very small and hard to use. To avoid this issue we choose to modulate $Tol\phi$ near the pole. The factor $p$ can be used to amplify or diminish this effect.

$MTol\phi= Tol\phi / \sin^p{\theta}$

A geogebra file is available to visualize the tolerance area with a given $Tol\phi$, $Tol\theta$ and $T\theta$

### Cavity in Plane Orientation (Band)
During the "Orientation movement" use the function _SetCavityToThetaAndPhi(90, 90)_ to put the cavity on the equatorial plane and the axis.
### Cavity Bottom Orientation (Band)
During the "Orientation movement" use the function _SetCavityToThetaAndPhi(90, 90)_ to put the cavity on the equatorial plane and the axis. Then use pre-timed mouvement to put the cavity under the fruit.
### Cavity Top Orientation (Band)
During the "Orientation movement" use the function _SetCavityToThetaAndPhi(0, _)_ to put the cavity on the top pole of the fruit.
### Cavity in the Plane (Bicone)
During the "Orientation movement" use forward or backward movement to put the cavity on the equatorial plane (closest side). Then raise the suction cup and rotate it to place the cavity with the desired angle on the plane.
### Color Orientation (Bicone)
During the "Orientation movement" use forward or backward movement to put the cavity on the equatorial plane (closest side). Then raise the suction cup and rotate it to place the cavity with the side (phi =0). Next step move forward until finding the best color (red or green) and finish by raising the suction cup and rotating it to the desired angle on the plane. 