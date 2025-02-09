# Matterport3D 360° RGBD Dataset

This extension to the original [Matterport3D dataset](https://github.com/niessner/Matterport) contains data aimed for high-resolution 360° monocular depth estimation. In particular, we provide the original ‘skybox’ images in equirectangular projection with their ground-truth depth maps, both at 2048×1024 resolution. Depth has been rendered from the original meshes, representing **metric** depth. 

## Data

Out of the initial 10,800 skybox images, we provide camera poses and aligned depth maps for 9,684 (~89.6%). Data is structured in 90 folders (one per building) and there are 4 files per sample:
- ```<panorama_uuid>_rgb.png``` — equirectangular colour image
- ```<panorama_uuid>_depth.dpt``` — 32-bit floating-point metric ground-truth depth map
- ```<panorama_uuid>_vis.png``` — visualisation of the above depth map
- ```<panorama_uuid>_pose.txt``` — camera pose of the ‘skybox’ image

```<panorama_uuid>``` is a unique string.
 
```pose``` contains the extrinsic camera parameters representing the camera-to-world [```R^t C```] transformation. The world coordinate system is the same as the original Matterport3D (X right, Y forward, Z up). For the camera, we use Blender’s right-handed camera coordinate systems (X right, Y up, Z backward). The first three stored values are the camera centre ```C = -R^t t```, and the last four are the quaternion ```xyzw``` representing the rotation of the camera with respect to the world, i.e. ```R^t```.

### Pose estimation approach

We estimated the poses for the equirectangular skybox images relative to the Matterport3D meshes using 360° structure-from-motion. Specifically, we use [OpenMVG](https://github.com/openMVG/openMVG)'s 360° SfM for each skybox image, i.e. a total of 10,800 SfM instances. We dertermined empirically that the camera centre for skybox images is approximately in the same location as the tripod camera centre provided in Matterport3D. With this assumption, for each SfM instance, we render 18 synthetic images with known poses placed on three concentric circles around the tripod centre. Next, we solve for the unknown pose of the skybox image. With our approach, we succesfully estimated 9,864 poses. A pose outlier rejection step helps removing the poses whose camera center is away more than 2 standard deviations from the per-building mean difference of skybox camera centre and tripod camera centre. Finally, we use the estimated poses and the Matterport3D meshes to render ground-truth depth maps that are aligned to each skybox image.

## Code

Minimal example for reading and using ```pose``` files:

```
from scipy.spatial.transform import Rotation as R
import numpy as np

def read_pose(filename):
    with open(filename, "r") as f:
        pose = f.read().splitlines()[0].split(" ")

    pose = [float(p) for p in pose]
    C = np.array(pose[:3])
    rot = R.from_quat(pose[3:7]).as_matrix()
    return C, rot

# world2cam -> Point x=[0,0,0] in world cs to camera cs
x = np.array([0, 0, 0])
x_prime = rot.T @ (x - C)

# cam2world -> Point x=[0,0,0] in camera cs to world cs
x = np.array([0, 0, 0])
x_prime = rot @ x + C
```

## Citation
If you use this data, please cite the original [Matterport3D dataset](https://github.com/niessner/Matterport) and [360MonoDepth](https://manurare.github.io/360monodepth):

```
@inproceedings{reyarea2021360monodepth,
    title={{360MonoDepth}: High-Resolution 360{\deg} Monocular Depth Estimation},
    author={Manuel Rey-Area and Mingze Yuan and Christian Richardt},
    booktitle={CVPR},
    year={2022}
}
```
