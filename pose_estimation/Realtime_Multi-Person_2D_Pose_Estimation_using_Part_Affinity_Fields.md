# Open pose

Overview of pose estimation:

[https://beyondminds.ai/blog/an-overview-of-human-pose-estimation-with-deep-learning/](https://beyondminds.ai/blog/an-overview-of-human-pose-estimation-with-deep-learning/)

Awesome pose estimation paper:

[cbsudux/awesome-human-pose-estimation](https://github.com/cbsudux/awesome-human-pose-estimation)

# Realtime Multi-Person 2D Pose Estimation using Part Affinity Fields

**Note:** We will review the author’s [journal article](https://arxiv.org/abs/1812.08008), published in 2018 at arXiv with better accuracy and faster speed than the [CVPR 2017 version.](https://arxiv.org/abs/1611.08050)

## Introduction:

In this work, the author provides a real-time method for Multi-Person 2D Pose Estimation based on bottom-up approach instead of detection-based approach in other works. The proposed method uses a nonparametric representation, which was referred to as Part Affinity Fields (PAFs), to learn to associate body parts with individuals in the image.

## Method:

**Openpose Pipeline:**

![](https://i.imgur.com/SwelzIn.png)

The above figure is the overall pipeline of OpenPose. There are a few steps:

- First, the image is passed through a baseline network to extract feature maps. In the paper, the author uses the first 10 layers of VGG-19 model.
- Then, the feature maps are processed with multiple stages CNN to generate: **1) a set of Part Confidence Maps** and **2) a set of Part Affinity Fields (PAFs)**
- **Part Confidence Maps**: a set of 2D confidence maps **S** for body part locations. Each joint location has a map.
- **Part Affinity Fields (PAFs)**: a set of 2D vector fields **L** which encodes the degree of association between parts.
- Finally, the **Confidence Maps** and **Part Affinity Fields** are processed by a greedy algorithm to obtain the poses for each person in the image.


**Parts and Pairs**
![](https://i.imgur.com/jVVYTNF.png)

- A body part is an element of the body, like neck, left shoulder or right hip.
- A pair is a couple of parts. A connection between parts. Also, we are going to deal with connections between ears and shoulders that do not exist in real life but needed to improve the accuracy for crowded images.

### **Confidence Maps**

**A Confidence Map** is a 2D representation of the belief that a particular body part can be located in any given pixel.

Let J be the number of body part locations (joints). Then, **Confidence Maps** are:

![](https://i.imgur.com/ki5YpYL.png)

![](https://i.imgur.com/8EEV0Ar.png)

In summary, each map is correspond for a joint and has the same size as the input image.

### **Part Affinity Fields (PAFs)**

**A Part Affinity Field (PAF)** is a set of flow fields that encodes unstructured pairwise relationships between body parts.

Each pair of body parts has a **(PAF)**, i.e neck, nose, elbow, etc,.

Let CC be the number of pairs of body parts. Then, **Part Affinity Fields (PAFs)** are:

![](https://i.imgur.com/6MgIeG0.png)

![](https://i.imgur.com/74Xo2vl.png)

If a pixel is on a limb (body part), the value in Lc at that pixel is a 2D unit vector from the start joint to the end joint.


### Multi-stage CNN
![](https://i.imgur.com/LOxCPHP.png)

![](https://i.imgur.com/yM7E1Fg.png)
![](https://i.imgur.com/orZdAGp.png)


In previous work, PAFs and body part location estimation were refined simultaneously across training stages. In this paper, it shows that a PAF-only refinement rather than both PAF and body part location refinement results in a substantial increase in both runtime performance and accuracy.


![](https://i.imgur.com/meMc2qo.png)
At every stage, we will compute the loss and backpropogate to prevent the vanishing gradient problem.

### Bipartite graph
![](https://i.imgur.com/uBhGRmV.png)

It computes the line integral along the segment connecting each couple of part candidates, over the corresponding PAFs (x and y) for that pair.

### Assignment

They use the Hungarian algorithm to deal with the assignment problem in weighted bipartite graph.

### Merging

1. Sort all pairwise possible connections by their PAF score.
2. If a connection tries to connect 2 body parts which have already been assigned to different people, the algorithm recognizes that this would contradict a PAF connection with a higher confidence, and the current connection is subsequently ignored.

## Results
![](https://i.imgur.com/CvPbs1l.png)
For the entire MPII testing set, our method
without scale search already outperforms previous stateof-the-art methods by a large margin, i.e., 13% absolute
increase on mAP.
> Note: scale search is a method that scale the image to multiple size and average the heat map and PAFs.


**The Demo video:**
https://www.youtube.com/watch?v=pW6nZXeWlGM
![](https://i.imgur.com/YmwXw03.jpg)

## Discusssion

![](https://i.imgur.com/jBN0ZM1.png)

In highly crowded images where people are overlapping, the approach tends to merge annotations from different people, while missing others, due to the overlapping PAFs that make the greedy multi-person parsing fail. 

Reference:
1. https://medium.com/ai-academy-taiwan/openpose-%E8%BF%91%E5%B9%B4%E4%BE%86%E6%9C%80%E5%BC%B7%E5%A4%A7%E7%9A%84%E5%A4%9A%E4%BA%BA%E4%BA%BA%E9%AB%94%E5%A7%BF%E6%85%8B%E8%BE%A8%E8%AD%98%E6%A8%A1%E5%9E%8B-8ee62ad7142a
2. https://towardsdatascience.com/cvpr-2017-openpose-realtime-multi-person-2d-pose-estimation-using-part-affinity-fields-f2ce18d720e8
3. https://arvrjourney.com/human-pose-estimation-using-openpose-with-tensorflow-part-2-e78ab9104fc8
4. https://zhuanlan.zhihu.com/p/143220168
