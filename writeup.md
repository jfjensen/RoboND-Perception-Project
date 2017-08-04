# Project: Perception Pick & Place

![demo-1](https://user-images.githubusercontent.com/20687560/28748231-46b5b912-7467-11e7-8778-3095172b7b19.png)


## [Rubric](https://review.udacity.com/#!/rubrics/1067/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
## Exercise 1, 2 and 3 pipeline implemented
#### 1. Pipeline for filtering and RANSAC plane fitting implemented.
The RGB-D camera is producing a lot of points, so a Voxel grid down-sampling filter is applied to the point cloud (this is done in `project.py`). Then 2 pass-through filters are applied:

* Pass-through filter in z-axis which does a cut-off at top and bottom (removes most of the table).
* Pass-through filter in y-axis which does cut-off left and right (removes sides of the drop-boxes).

Afterwards a RANSAC plane segmentation algorithm is applied to the point cloud for determining remaining parts of the table. These points are referred to as inliers. These are not needed, rather it is the outliers which are relevant. This point cloud contains the objects on the table.

#### 2. Pipeline including clustering for segmentation implemented.  
A KD-tree is used to keep the point cloud and allows the clustering algorithm to do quick searches. Then, the clustering algorithm is used to find points in the point cloud which are close together, i.e. form a cluster. Parameters are set in order for the algorithm to know maximum and minimum size of any given cluster. To finish, each discovered cluster is given a color. Each cluster represents an object in the scene.


#### 3. Features extracted and SVM trained.  Object recognition implemented.

##### SVM Training
The sensor stick from the exercises is used as a camera for taking samples (snapshots) of each the 8 objects (this is done in `capture_features.py`). Each object is sampled from 20 different angles. Then for each sample, histograms of the colors and the normal vectors are calculated in `features.py`.

A histogram is made for each HSV color channel and for each axis (x-y-z) of the normal vectors. All histograms consist of 64 bins. The color histograms and the normal vector histograms are concatenated and normalized separately. The result is two histograms: one for HSV color and one for the normal vectors.

These are then concatenated to form one feature vector. So the final result is thus one feature vector per sample per object ( 8 x 20 = 160 features ) . All the feature vectors and corresponding labels (object names) are output in a file `training_set.sav`.

The `training_set.sav` is then used as training data for the SVM classifier. The SVM is set to use an RBF kernel and is trained using 5-fold cross validation. An accuracy of 77% is achieved. This is sufficient for classifying the objects in the scene, while at the same time avoiding overfitting. The learned model is saved in the file `model.sav`.

##### Object Recognition
The SVM is initialized by loading the `model.sav` created above.

First histograms are created based on the colors and the normal vectors of the objects which were found by clustering. For each object these histograms are concatenated and scaled. Then a prediction is made by the trained SVM. The result is the predicted label (name of the object).

The predicted labels published to ROS, such that the simulator in RViz can access them and add them to the scene. 

![topview](/images/topview.png)


## Pick and Place Setup

#### 1. For all three tabletop setups (`test*.world`), perform object recognition, then read in respective pick list (`pick_list_*.yaml`). Next construct the messages that would comprise a valid `PickPlace` request output them to `.yaml` format.

For each of the 3 scenes, object recognition is done. Below are screenshots showing the labels in the 3 scenes. It is only in scene 3, where there is one object which is unlabeled. That object is the 'glue'. It is not labeled due to the fact that is obscured by the book in front.

![scene1](/images/scene1.png)

![scene2](/images/scene2.png)

![scene3](/images/scene3.png)

In addition a YAML file is created for each of the 3 scenes. Each YAML file contains the label of the objects (the name) in the scene, the arm which should grab which object, the position where each object is to be picked up and where it is to be placed.

