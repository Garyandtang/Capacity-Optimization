# Capacity-Optimization

The implementation is divided into the following steps:
Step 1 to Step 8 are for a large network. After being fully trained, the large network is used to teach a small network.
1. Train the large SM to segment out all person parts as one category. This is much simpler than 7 class prediction and the network can roughly segment out all people.
2. Also use LM to predict the skeletons of all people.
3. Based on the results from Step 1 and Step 2, we can roughly segment out the regions containing people and set the backgrounds to zeros. This can reduce the workload of CNNs. 
4. Enlarge the region if interested regions are too small and re-use LM to generate skeleton on the enlarged regions.
5. Train and test on the simplified images.
6. Based on the similarity of skeletons, choose some people from training images to simulate those from test images. Especially simulate the cases where people in test images are small, or of hard poses.
7. Data augmentation: Scaling, Rotation, Color(Add a value to the hue channel of the HSV representation) Also we need to random the backgrounds and foregrounds.
8. Use several networks to deal with different cases (scales), then use the results of them to teach one network.
9. Use the large SM to teach the small SM. 

Techniques for data augmentation:
The first way of data augmentation is through creating examples which have similar poses to those from the test set. Besides the CNN for segmentation, a model for human pose analysis is trained. Skeleton detection is a more simple task than person part segmentation and there are more training data available. As a result, the predictions from the model for pose analysis tend to be more generalizable. In the proposed method, each image is firstly divided into regions each of which contains one person. For each person in the training set and test set, a vector describing the pose is predicted. Then we try to find the person with the most similar gesture from target regions in test data for each person in the source region from training data. Upon matching each pair of people, we conduct homography transformation on the source regions to make them more similar to the target regions. The ground truth masks for source region are transformed in the same way. The transformations produce fake test images which are similar to test images and with ground truth labels. Some examples are shown in Fig. 1. People are segmented out for clear demonstration.

           Pair 1                      Pair 2                   Pair 3                 Pair 4
               
Pair 5                                               Pair 6
       
Fig. 1. Examples showing 6 pairs of images containing people with similar poses. Within each pair, the left image is from test set, the right one is from training set. We’ve searched the training set to find the person with the most similar pose to each person from test images and conducted homography transformations on the selected regions from training data.
As can be seen from Fig. 1, we’ve generated a fake test set with labels based on training data. We’ve conducted homography transformations to map fake images to real ones based on the coordinates of critical joints. The gap between the fake test set and real test set is surely lower than that between training set and real test set. Experiments will show that the involvement of fake test set during training can improve the performance of segmentation.
To evaluate the similarity in poses, a feature descriptor is developed in our work. The 14 joints are nose, neck, left shoulder, left elbow, left wrist, right shoulder, right elbow, right wrist, left hip, left knee, left ankle, right hip, right knee and right ankle. A center point can be computed by averaging the coordinates of the detected joints and is shown by the red dot in Fig. 2 (a). 
Fig. 2 describes the proposed descriptor, two histograms are used to describe the pose of each person. The sum of Euclidean distances between two pairs of histogram vectors evaluates the similarity in pose between two people. We’ve divided one image into regions each of which contains one person. As a result, multiple images from training data can be used to create one fake test image that is similar to a real test image, as is shown in Pair 6 in Fig. 1. 
Moreover, the existence of other variances, such as rotations, scaling or changes in illumination may reduce the similarity between fake test images and real ones. 
   
(a)	(b)
Fig. 2. The proposed descriptor for describing poses. (a) The indices of joints. (b) The offsets of different joints from the center. The offsets are normalized with respect to the maximal distance between pairs of joints. The top histogram describes the normalized offsets in the vertical direction while the bottom one describes those in the horizontal direction.
