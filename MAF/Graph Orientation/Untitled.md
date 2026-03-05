The Linepack manages many cameras of different brands for multiple purposes.  
The available brands are:
- Teli
- IDS

Each camera can have different and multiple usages:
- Orientor: detection in orientors to guide orientation
- Loader: detection in loaders and orientors to guide the loading process
- FruitCounting: detection and counting of fruits to send to Orphea
- TrayScanner: detection of holes to validate the correspondence with the selected tray
- PatternScanner: detection of fruits to validate whether the pattern has been placed correctly

Each camera has a name that indicates the associated line, robot, robot position, and functions. The name must follow this regex:  
"^L{numLine}_ROBOT(\d+)_CAM(\d+)_(.\*)".
The element "(.\*)" corresponds to the list of camera functionalities following the pattern:  
"function?(-function)_"

Example:  
L1_ROBOT0_CAM0_Orientor-Loader-FruitCounting is the first camera of robot 0 in line 1, with the functions Orientor, Loader, and FruitCounting.

The camera can be loaded from the network using the button "Load from Network" in the "Cameras" tab.

## Camera Configuration
For each camera, two elements need to be configured outside of the Linepack: the camera name and the AOI.

### Teli
In the case of Teli cameras, the field corresponding to the name is **DeviceUserID**.

![[Pasted image 20260226094622.png]]![[Pasted image 20260226094508.png]]

The AOI can be modified in **Image Format**:

![[Pasted image 20260226100726.png]]

### IDS
In the case of IDS cameras, the field corresponding to the name is **EEPROM** in **Camera Information**.

![[Pasted image 20260226102458.png]]

The AOI can be set by launching the viewer in expert mode and using the AOI selector.

![[Pasted image 20260226103009.png]]![[Pasted image 20260226103143.png]]

To be able to use this configuration in the Linepack, it must be exported using:  
"File => Save Parameters => To File",

which will produce a ".ini" file. Then move this file to:  
LinePack/Configuration/Camera.

## Code Structuration

![[Pasted image 20260226144018.png]]

