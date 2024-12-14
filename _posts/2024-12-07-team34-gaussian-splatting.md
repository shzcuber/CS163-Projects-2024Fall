---
layout: post
comments: true
title: Gaussian Splatting
author: Shawn Zhuang, Allen Luo, William Smith, Howard Huang
date: 2024-12-07
---

> Gaussian Splatting is a novel 3D reconstruction algorithm, radically improving the generation time compared to NeRFs using 3D Gaussians. In this project, we review its implementation as well as several applications of the more capable 3D reconstruction algorithm.

<!--more-->

{: class="table-of-content"}

- Problem Statement
  {:problem-statement}

## Problem Statement

Our problem statement is the following: given a set of sparse images, how can we construct a corresponding 3D scene? [1]

![3D Reconstruction]({{ '/assets/images/34/3dreconstruction.png' | relative_url }})
{: style="width: 400px; max-width: 100%;"}
_Fig 1. Problem Statement image_ [1].

## Background

### Structure From Motion

Structure from Motion (SfM) was an early solution to this problem. It transforms inputs of 2D images to a sparse point cloud.
It does this through 2D image aligment, in which distinctive features are matched in each image, the camera
pose is estimated, and the 3D coordinates are calculated.
**COLMAP** is a package and CLI that implements SfM, allowing a user to easily generate the sparse point clouds through SfM.

![Structure from Motion]({{ '/assets/images/34/sfm.png' | relative_url }})
{: style="width: 800px; max-width: 100%;"}
_Fig 2. SfM_

### Neural Radiance Fields (NeRFs)

Similary to SfM, given set of 2D images + camera poses, NeRFs also generates 3D scene.
However, it revolutionzized 3D reconstruction by representing 3D scenes with neural networks rather than mesh based representations.
This allows us to represent fine scene geometry, using a raycasting algorithm.
In addition, since it's implicitly represented, only requiring the MLP weights, the storage costs are small.
However, this implicit representation makes rendering times large. In addition, downstream tasks such as scene editing and mesh generation are more difficult.

![NeRFs]({{ '/assets/images/34/nerfs.png' | relative_url }})
{: style="width: 800px; max-width: 100%;"}
_Fig 3. NeRFs_

## 3D Gaussian Splatting

### 3D Gaussians

### 3D Gaussian Optimization

### 3D Gaussian Rasterizer

### Results

## Applications

### Mesh Extraction (SuGaR)

### Deformable 3DGS

As mentioned, NeRFs struggle due to their slow speed, which makes it inefficient and ineffective. It's constrained by ray-casting. This makes it impractical to generate dynamic scenes, where the objects are moving.
However, 3DGS, by itself, also cannot effectively generate dynamic scenes. Inherently, it doesn't understand the spatiotemporality of motion. This is shown by simply introducing a time-conditioned learnable parameter to 3DGS.
Therefore, to make it compatible with dynamic scenes, we introduce a deformation field, which is essentially an MLP network mapping from each coordinate of 3D Gaussians and time to position, rotation, and scaling. Which we jointly learn this with the 3D Gaussians. This decouples motion and geometry structure, which introduces the geometric priors of the scene, associating changes in 3D Gaussians with both time and coordinates. The pipeline is as follows:

![Deformable 3DGS]({{ '/assets/images/34/deformable.png' | relative_url }})
{: style="width: 800px; max-width: 100%;"}
_Fig 3. Deformable 3DGS_

1. From the SfM point clouds, we create our 3D Gaussians $$G(x, r, s, \sigma)$$ (center position $$x$$, opacity $$\sigma$$, 3D covariance matrix from quaternion $$r$$ and scaling $$s$$. Spherical harmonics model the view-dependent appearance of each 3D Gaussian (viewing angle, changing light etc)
2. We decouple the 3D Gaussians and the deformation field to model the 3D Gaussians that change over time. The deformation field takes in the positions of the 3D Gaussians with time $$t$$, and outputs $$\delta x$$, $$\delta r$$, $$\delta s$$
3. The deformed 3D Gaussians ($$G(x + \delta x, r+\delta r, s + \delta s, \sigma)$$) is rasterized using differential Gaussian Rasterization Pipeline, a tile-based rasterizer that allows $$\alpha$$-blending of anisotropic splats). The tile based approach ensures only tiles overlapping Gaussian’s footprint are rendered. In addition, the $$\alpha$$-blending combines multiple overlapping layers to create smooth, translucent effects. It determines pixel colors by blending Gaussian’s contribution with background based on opacity — this **avoids sharp edges** between overlapping Gaussians
4. 3D Gaussians and deformation network can then be optimized jointly through fast backward pass.

### DreamGaussian

DreamGaussian is a 3D content generation framework that adapts 3D Gaussian splatting for image-to-3D mesh generation. Previous iterations of 3D content generation frameworks utilized NeRFs, but using 3DGS, speed and efficiency has been drastically improved. The implementation includes alleviating blurriness in directly generated 3D Gaussians by designing an efficient mesh extraction algorithm from 3D Gaussians using local density querying. In addition a generative UV-space refinement stage is proposed to enhance the texture details.
Notably as well, score distillation sampling (SDS) is used to calculate the loss -- this addresses the 3D data limitation, distilling 3D geometry and appearance from powerful 2D diffusion models. Lastly, it adopts a 3DGS Gaussian culling technique with Marching Cubes for efficient rasterization.

![DreamGaussian]({{ '/assets/images/34/dream.png' | relative_url }})
{: style="width: 800px; max-width: 100%;"}
_Fig 4. DreamGaussian_

### Instant Splat

## Running Existing Codebases

### Vanilla 3D Gaussian Splatting

We first ran vanilla 3D Gaussian Splatting as described in the foundation paper to reconstruct scenes. We obtained the following results with the provided example images:

![3DGS Train]({{ '/assets/images/34/gaussian_splatting_train.gif' | relative_url }})
{: style="width: 800px; max-width: 100%;"}
_Fig 4. DreamGaussian_
![3DGS Truck]({{ '/assets/images/34/gaussian_splatting_truck.gif' | relative_url }})
{: style="width: 800px; max-width: 100%;"}
_Fig 4. DreamGaussian_

There are noticeable artifacts present in the final splats, as well as low-resolution surfaces with rough edges due to a shorter training time (~1 hour). Generated scenes would be of higher quality if trained for 48 hours as in the original 3DGS implementation, or by using an optimized method like InstantSplat. We attempted to run the scenes with the online [InstantSplat demo](https://huggingface.co/spaces/kairunwen/InstantSplat), but it seems to not be currently maintained.

### DreamGaussian

We ran DreamGaussian on a single view of a T-Rex from the D-NeRF synthetic dataset, which was used to benchmark Deformable 3D Gaussian Splatting. We obtained the following result, placed alongside the Deformable 3DGS render for comparison. For context, the original scene contains 220 training views for training and validation.

![Deformable 3DGS T-Rex]({{ '/assets/images/34/deformable_trex.gif' | relative_url }})
{: style="width: 800px; max-width: 100%;"}
_Fig 4. Deformable 3DGS T-Rex_
![DreamGaussian T-Rex]({{ '/assets/images/34/dreamgaussian_trex.gif' | relative_url }})
{: style="width: 800px; max-width: 100%;"}
_Fig 4. DreamGaussian T-Rex_

Multi-view Gaussian splatting methods still significantly outperform single-view methods in terms of reconstruction quality. However, these generative methods can serve as a solid baseline for efficient 3D content creation, especially given that it requires just a single view and the model outputs a solid mesh that can be easily modified. Single-view applications of 3D Gaussian Splatting are still a very active area of research.

## References

[1] Kerbl, Bernhard, Kopanas, Georgios, Leimkühler, Thomas, and Drettakis, George. [_3D Gaussian Splatting for Real-Time Radiance Field Rendering_](https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/). ACM Transactions on Graphics, 42(4), July 2023.

[2] Yang, Ziyi, et al. "Deformable 3d gaussians for high-fidelity monocular dynamic scene reconstruction." Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition. 2024.

[3] Tang, Jiaxiang, Ren, Jiawei, Zhou, Hang, Liu, Ziwei, and Zeng, Gang. [_DreamGaussian: Generative Gaussian Splatting for Efficient 3D Content Creation_](https://arxiv.org/abs/2309.16653). _arXiv preprint arXiv:2309.16653_, 2023.

[4] Guédon, Antoine, and Vincent Lepetit. "Sugar: Surface-aligned gaussian splatting for efficient 3d mesh reconstruction and high-quality mesh rendering." Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition. 2024.

[5] Fan, Zhiwen, Cong, Wenyan, Wen, Kairun, Wang, Kevin, Zhang, Jian, Ding, Xinghao, Xu, Danfei, Ivanovic, Boris, Pavone, Marco, Pavlakos, Georgios, Wang, Zhangyang, and Wang, Yue. [_InstantSplat: Unbounded Sparse-view Pose-free Gaussian Splatting in 40 Seconds_](https://arxiv.org/abs/2403.20309). _arXiv preprint arXiv:2403.20309_, 2024.
