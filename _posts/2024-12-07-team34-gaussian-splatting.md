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

-   Problem Statement
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
3D Gaussian Splatting consists of a three step process
1. Scene representation with 3D Gaussians
2. 3D Gaussian optimization
3. Real-time rendering algorithm

Fig X. 3D Gaussian Splatting Pipeline

### 3D Gaussians
3D Gaussians are initialized by a set of 3D points from SfM and characterized by 4 parameters:
- Mean 𝜇 (3D point)
- Covariance matrix Σ
- Opacity 𝛼
- Color c (spherical harmonics coefficients)

![GuassianSplattingPipeline]({{ '/assets/images/34/GaussianSplattingPipeline.png' | relative_url }})
{: style="width: 400px; max-width: 100%;"}
_Fig 4. GaussianSplattingPipeline_.

Fig X. Effect of a 3D gaussian on a 3D point p
![GuassianPoint]({{ '/assets/images/34/3DGaussianPoint.png' | relative_url }})
{: style="width: 400px; max-width: 100%;"}
_Fig 4. 3D Gaussian Point_.

Fig X. Equation definition of a 3D 
![GuassianSplattingEquation]({{ '/assets/images/34/GaussianEquation.png' | relative_url }})
{: style="width: 400px; max-width: 100%;"}
_Fig 4. Gaussian Splatting Equation_.

Gaussian. x is the 3d point/mean, and Σ is the covariance matrix.

### 3D Gaussian Optimization
Gaussian optimization is the optimization process that gradually refines the rendering of the 3D scene through Iterations of rendering and comparison with training views. Fine-grained Adaptive control of Gaussians can be achieved by focusing on regions with missing geometric features and regions where Gaussians cover large regions. To do this, two basic operations are performed:
- Small Gaussians in under reconstructed regions are cloned
- Large Gaussians in regions of high variance split up into two new Gaussians 

Fig X. Algorithm for under/over-reconstructed regions during gaussian optimization.
![Clone and Split]({{ '/assets/images/34/CloneAndSplit.png' | relative_url }})
{: style="width: 400px; max-width: 100%;"}
_Fig 4. Clone and Split_.

Comparisons with the rendered and training images use a combination of L1 Loss and D-SSIM Loss, defined as (𝜆 is a hyperparameter; it is set to 0.2 in the original paper):
Fig X. Loss term for 3DGS.
![3DGS Loss]({{ '/assets/images/34/3DGSLoss.png' | relative_url }})
{: style="width: 400px; max-width: 100%;"}
_Fig 4. 3D GS Loss_.

### 3D Gaussian Rasterizer
Finally, the 3D Gaussian Rasterizer is what is responsible for actually rendering the gaussians in the 3D scene. As opposed to traditional geometric primitives/shapes like triangles and polygons, the 3D Gaussian Rasterizer renders gaussian blots. To create a rendered view, the rasterizer projects 3D Gaussian distributions into a 2D image plane using a perspective or orthogonal projection, mimicking how a camera sees a 2D plane. To achieve blending and accumulation, the Rasterizer blends overlapping Gaussians by accumulating their contributions to pixel values in the rendered image using alpha blending or similar techniques to simulate transparency and lighting effects. To achieve depth ordering and occlusion handling, the Rasterizer makes it so that closer Gaussians occlude farther ones which might involve sorting Gaussians or using depth-buffer-mechanisms. Finally, to achieve light and shading effects, the Rasterizer incorporates shading effects by adjusting intensity and color of gaussians based on lightning conditions.

### Results
Fig X. 3DGS results
![3DGS Results]({{ '/assets/images/34/3DGS Results.png' | relative_url }})
{: style="width: 400px; max-width: 100%;"}
_Fig 4. 3DGS Results_.

## Applications
3D gaussian splatting can be used for many different applications. We will cover a few here:
Geometry Editing
Definition:
Geometry editing refers to modifying the spatial arrangement, structure, or properties of 3D blobs to define the underlying surface shape or characteristics of a 3D object
Examples:
Rigid Transformations: Moving, rotating, or scaling the object or parts of it.
Non-Rigid Deformations: Stretching, bending, or warping parts of the object for more complex shape changes.
Topology Adjustments: Adding or removing Gaussians to represent changes in the model's topology (e.g., splitting a face or creating holes).
Advancements:
Utilizing surface priors and explicit deformation methods to optimize Gaussian parameters and number
Frosting Layer
Techniques:
Mesh-Based Editing: Combining Gaussian Splatting with conventional meshes to use mesh vertices for parameterizing Gaussian positions. This enables real-time edits but is limited in flexibility for significant topology changes.
Regularization and Surface Priors: Using priors like surface normals and gradients from explicit deformation methods to maintain consistency during edits.
Topology Optimization: Advanced methods like face splitting or adding Gaussians to dynamically adjust the model's topology.
Challenges:
Large-Scale Deformations: Significant changes in shape can be difficult because the relationship between Gaussians and the object surface might become inconsistent.
Topology Modifications: Traditional GS struggles with modifying the connectivity or structure of the object, as Gaussians are often tied to a fixed mesh or surface.
Appearance Editing
Definition:
Focuses on visual attributes of 3D objects, such as their color, texture, shading, and material properties rather than geometric properties.
Gaussians in GS can store additional attributes like color, opacity, and texture information.
Examples:
Texture Painting: Adding or altering textures on the object's surface.
Material Editing: Changing reflectivity, transparency, or other material properties.
Lighting and Shadows: Adjusting how light affects the object, enabling relighting.
Object Inpainting: Filling in missing or occluded regions to restore or create new details.
Techniques:
Language-Guided Editing: Using diffusion models (e.g., GaussianEditor) with natural language input to make targeted changes in appearance.
Disentanglement: Separating geometry from appearance to allow independent modifications of texture and lighting (e.g., TextureGS and 3DGM).
Mask-Based Updates: Applying segmentation models to isolate regions for editing while preserving surrounding details.
Depth and Cross-Attention: Techniques like GaussCtrl and Wang et al. use depth maps or multi-view cross-attention to ensure consistency across perspectives.
Challenges:
Consistency: Ensuring that changes to appearance align with the object's geometry across different views.
Interdependence: Appearance attributes like texture and lighting are intertwined with geometry, making disentanglement and independent editing complex.
Complexity: High-quality edits require accounting for multi-view consistency and realistic rendering constraints.
Physical Simulation
Definition:
Supports realistic simulations for fluid dynamics, motion, and collisions.
Physical properties like mass, velocity, and surface normals are encoded into Gaussian parameters for simulation purposes.
Applications:
Solid and Fluid Dynamics: Simulating interactions between solids (e.g., deformable objects) and fluids (e.g., water, smoke).
Physically-Based Rendering: Incorporating realistic reflections, refractions, and lighting effects based on physical interactions.
Dynamic Behavior: Modeling real-world behaviors like elasticity, collisions, and deformation using particle-based dynamics.
Techniques:
Particle-Based Dynamics (PBD): Gaussian Splashing integrates 3DGS with PBD to simulate cohesive dynamics, including solid-fluid interactions and dynamic rendering.
Continuum Deformation: PhysGaussian applies deformation models to Gaussian kernels, simulating physically realistic changes in shape and rendering.
Spring-Mass Models: Spring-Gaus employs spring-mass systems to simulate dynamic properties, extracting parameters like mass and velocity from real-world inputs like videos.
Surface Alignment: Normal-based alignment ensures that Gaussians conform to realistic surface orientations, improving physical and visual consistency.
Challenges:
Realism: Achieving physically accurate results while maintaining computational efficiency.
Integration: Combining simulation with rendering and other Gaussian attributes like appearance or geometry.
Dynamic Adaptation: Adjusting Gaussian parameters dynamically to reflect changing physical states (e.g., deformation, splitting).

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

DreamGaussian is a 3D content generation framework that adapts 3D Gaussian splatting for image-to-3D mesh generation. Previous iterations of 3D content generation frameworks utilized Neural Radiance Fields (NeRFs), but these approaches typically require hour-long optimizations that limit their use for real-world applications. DreamGaussian is inspired by the contributions of 3D Gaussian Splatting and Zero-1-to-3, a NeRF-based diffusion framework that synthesizes novel views of an object given a single input view [10].

DreamGaussian leverages the efficiency of 3D Gaussian Splatting by representing the 3D scene as a collection of small, localized Gaussian functions. 3D Gaussians are initialized randomly and then iteratively optimized to reconstruct the input, refining position, scale, and orientation of each Gaussian. A Score Distillation Sampling (SDS) loss distills 3D geometry and appearance from 2D diffusion models [11], minimizing the difference between the rendered images and the text prompt or, in our case, input images. An overview of the pipeline is as follows:

![DreamGaussian]({{ '/assets/images/34/dream.png' | relative_url }})
{: style="width: 800px; max-width: 100%;"}
_Fig 4. DreamGaussian framework for generation and mesh extraction_

1. Score Distillation Sampling is used to sample from the Zero-1-to-3 XL diffusion prior and densify the Gaussians
2. A local density query algorithm is performed to extract mesh geometry. The Marching Cubes algorithm is adapted with culled Gaussians to reduce queries in blocks for efficient rasterization, and the weighted opacity of each 3D Gaussian is summed.
3. The rendered RGB image is back-projected to the mesh surface and baked as the texture by unwrapping the mesh's UV coordinates and sampling 9 azimuths and 3 elevations to render. This texture image serves as an initialization for texture fine-tuning.
4. fff

![DreamGaussian Results]({{ '/assets/images/34/dreamgaussian_results.png' | relative_url }})
{: style="width: 800px; max-width: 100%;"}
_Fig 4. Comparison of generation speed and mesh quality between DreamGaussian and previous works_

### Instant Splat
Task
The goal of InstantSplat is to address the shortcomings of COLMAP when its randomly initialized Gaussians or sparse SfM points fail with sparse-view data or data with too few images and lacks an error correction mechanism which causes errors to build up over time. 
3 Main Contributions: 
The overall architecture of Instant splat consists of a pipeline that integrates Multi-View Stereo (MVS) with 3D-GS to address sparse-view reconstruction in the reliance on SfM and training efficiency. 

InstantSplat also introduces an efficient, confidence-aware point downsampler address scene overparameterization. 

Finally, InstantSplat proposes a self-correction mechanism in the form of a gradient-based joint optimization framework using photometric loss to align the Gaussians and camera parameters in a self-supervised manner 

## Running Existing Codebases

### Vanilla 3D Gaussian Splatting

We first ran vanilla 3D Gaussian Splatting as described in the foundation paper to reconstruct scenes. We obtained the following results with the provided example images:

![3DGS Train]({{ '/assets/images/34/gaussian_splatting_train.gif' | relative_url }})
{: style="width: 800px; max-width: 100%;"}
_Fig 4. Vanilla 3D Gaussian Splatting train scene_

![3DGS Truck]({{ '/assets/images/34/gaussian_splatting_truck.gif' | relative_url }})
{: style="width: 800px; max-width: 100%;"}
_Fig 4. Vanilla 3D Gaussian Splatting truck scene_

There are noticeble artifacts present in the final splats, as well as low-resolution surfaces with rough edges due to a shorter training time (~1 hour). Generated scenes would be of higher quality if trained for 48 hours as in the original 3DGS implementation, or by using an optimized method like InstantSplat. We attempted to run the scenes with the online [InstantSplat demo](https://huggingface.co/spaces/kairunwen/InstantSplat), but it seems to not be currently maintained.

### DreamGaussian

We ran DreamGaussian on a single view of a T-Rex from the D-NeRF synthetic dataset, which was used to benchmark Deformable 3D Gaussian Splatting. We obtained the following result, placed alongside the Deformable 3DGS render for comparison. For context, the original scene contains 220 training views for training and validation.

![Deformable 3DGS T-Rex]({{ '/assets/images/34/deformable_trex.gif' | relative_url }})
{: style="width: 400px; max-width: 100%; margin: 0 auto;"}

_Fig 4. Render of T-Rex with Deformable 3DGS_

![DreamGaussian T-Rex]({{ '/assets/images/34/dreamgaussian_trex.gif' | relative_url }})
{: style="width: 800px; max-width: 100%;"}
_Fig 4. Render of T-Rex with DreamGaussian (single input view)_

Multi-view Gaussian splatting methods still significantly outperform single-view methods in terms of reconstruction quality. However, these generative methods can serve as a solid baseline for efficient 3D content creation, especially given that it requires just a single view and the model outputs a solid mesh that can be easily modified. Single-view applications of 3D Gaussian Splatting are still a very active area of research.

## References

[1] Kerbl, Bernhard, Kopanas, Georgios, Leimkühler, Thomas, and Drettakis, George. [_3D Gaussian Splatting for Real-Time Radiance Field Rendering_](https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/). ACM Transactions on Graphics, 42(4), July 2023.

[2] Yang, Ziyi, et al. "Deformable 3d gaussians for high-fidelity monocular dynamic scene reconstruction." Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition. 2024.

[3] Tang, Jiaxiang, Ren, Jiawei, Zhou, Hang, Liu, Ziwei, and Zeng, Gang. [_DreamGaussian: Generative Gaussian Splatting for Efficient 3D Content Creation_](https://arxiv.org/abs/2309.16653). _arXiv preprint arXiv:2309.16653_, 2023.

[4] Guédon, Antoine, and Vincent Lepetit. "Sugar: Surface-aligned gaussian splatting for efficient 3d mesh reconstruction and high-quality mesh rendering." Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition. 2024.

[5] Fan, Zhiwen, Cong, Wenyan, Wen, Kairun, Wang, Kevin, Zhang, Jian, Ding, Xinghao, Xu, Danfei, Ivanovic, Boris, Pavone, Marco, Pavlakos, Georgios, Wang, Zhangyang, and Wang, Yue. [_InstantSplat: Unbounded Sparse-view Pose-free Gaussian Splatting in 40 Seconds_](https://arxiv.org/abs/2403.20309). _arXiv preprint arXiv:2403.20309_, 2024.

[10] Liu, Ruoshi, et al. Zero-1-to-3: Zero-Shot One Image to 3D Object. arXiv:2303.11328, arXiv, 20 Mar. 2023. arXiv.org, https://doi.org/10.48550/arXiv.2303.11328.

[11] Poole, Ben, et al. DreamFusion: Text-to-3D Using 2D Diffusion. arXiv:2209.14988, arXiv, 29 Sept. 2022. arXiv.org, https://doi.org/10.48550/arXiv.2209.14988.
