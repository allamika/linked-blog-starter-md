 The Linepack manage a lot of cameras different brands for multiple purpose. 
The available brands are :
- Teli
- IDS
This camera can have different and multiple usages:
- Orientor : detection in orientors to guide the orientations
- Loader : detection in loaders and orientors to guide the loading process
- FruitCounting : detection and count of fruits to send to Orphea 
- TrayScanner : detection of hole to validate the correspondance between the selected tray
- PatternScanner : detection of fruits to validate if the pattern as been placed correctly

Each camera as a name that give the associated line, robot, position in robot and functions.The name need to follow this regex : "^L{numLine}\_ROBOT(\d+)\_CAM(\d+)\_(.\*)". The element "(.\*)" corresponds to the list of functionality of the camera following the pattern : "function?(-function)\*"
Ex : L1_ROBOT0_CAM0_Orientor-Loader-FruitCounting is the first camera of the robot 1 in line 1 with the functions Orientor, Loader and FruitCounting

The camera can be loaded from the network using the button "Load from Network" in the "Cameras" tab
## Camera Configuration
For each camera two elements needs to be configured outside of the linepack : the camera name and the AOI
### Teli
In the case of Teli cameras the field corresponding to the name is DeviceUserID.

  ![[Pasted image 20260226094622.png]]![[Pasted image 20260226094508.png]]
The AOI can be modified in Image Format :
![[Pasted image 20260226100726.png]]

### IDS
 In the case of Teli cameras the field corresponding to the name is EEPROM in Camera information.
![[Pasted image 20260226102458.png]]
For the AOI can be set by launching the viewer in expert mode and using the aoi selecteur 
![[Pasted image 20260226103009.png]]![[Pasted image 20260226103143.png]]

To be able to use this configuration in the linepack, it needs to be exported with "Fichier => Enregistrer paramètres => vers fichier" wich will produce a '.ini'. Then move this file in the LinePack to Configuration/Camera.

## Code Structure


The BaseCamera class is an abstract and serializable class whose configuration is saved through serialization. It serves as the foundational class for all camera types in the system. The BaseCamera class contains a dictionary of type `<CameraFunction, CameraFunctionality>`, which is used to store and manage the different camera functionalities associated with a given camera instance. Each CameraFunctionality object maintains its own list of Regions of Interest (ROIs), allowing specific areas to be defined and managed independently for each functionality.

The CameraIDS and CameraTeli classes both inherit from the BaseCamera class and extend its behavior according to their specific camera implementations. 
Be careful, every attribut of the class inheriting BaseCamera will have its attributs serialized if this need to be avoid use \[JsonIgnore\] 

The CameraManager class acts as an interface between the camera objects and the rest of the application, ensuring proper communication and coordination between the camera layer and other system components.


![[Pasted image 20260226145220.png]]