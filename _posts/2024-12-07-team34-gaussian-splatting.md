---
layout: post
comments: true
title: Gaussian Splatting
author: Shawn Zhuang, Allen Luo, William Smith, Howard Huang
date: 2024-12-07
---

> Gaussian Splatting is a 3D reconstruction algorithm.

<!--more-->

{: class="table-of-content"}

- TOC
  {:toc}

## Problem Statement

Given a set of sparse images, how can we construct a corresponding 3D scene? [1]

![3D Reconstruction]({{ '/assets/images/34/3dreconstruction.png' | relative_url }})
{: style="width: 400px; max-width: 100%;"}
_Fig 1. Problem Statement image_ [1].

# Background

## Structure From Motion

Structure from Motion (SfM) was an early solution to this problem. It akes in the 2D images and outputs a sparse point cloud.
It does this through 2D image aligment, in which distinctive features are matched in each image, the camera
pose is estimated, and the 3D coordinates are calculated.
**COLMAP** is a package and CLI that implements SfM, allowing a user to easily generate the sparse point clouds through SfM.

![Structure from Motion]({{ '/assets/images/34/sfm.png' | relative_url }})
{: style="width: 800px; max-width: 100%;"}
_Fig 2. SfM_

## Neural Radiance Fields (NeRFs)

Similary to SfM, given set of 2D images + camera poses, NeRFs also generates 3D scene.
However, it revolutionzized 3D reconstruction by representing 3D scenes with neural networks rather than mesh based representations.
This allows us to represent fine scene geometry, using a raycasting algorithm.
In addition, since it's implicitly represented, only requiring the MLP weights, the storage costs are small.
However, this implicit representation makes rendering times large. In addition, downstream tasks such as scene editing and mesh generation are more difficult.

![NeRFs]({{ '/assets/images/34/nerfs.png' | relative_url }})
{: style="width: 800px; max-width: 100%;"}
_Fig 2. NeRFs_

# 3D Gaussian Splatting

## 3D Gaussians

## 3D Gaussian Optimization

## 3D Gaussian Rasterizer

## Results

## Applications

### Mesh Extraction (SuGaR)

### Deformable 3DGS

As mentioned, NeRFs struggle due to their slow speed, which makes it inefficient and ineffective. It's constrained by ray-casting. This makes it impractical to generate dynamic scenes, where the objects are moving.
However, 3DGS, by itself, also cannot effectively generate dynamic scenes. Inherently, it doesn't understand the spatiotemporality of motion. This is shown by simply introducing a time-conditioned learnable parameter to 3DGS.
Therefore, to make it compatible with dynamic scenes, we introduce a deformation field, which is essentially an MLP network mapping from each coordinate of 3D Gaussians and time to position, rotation, and scaling. Which we jointly learn this with the 3D Gaussians. This decouples motion and geometry structure, which introduces the geometric priors of the scene, associating changes in 3D Gaussians with both time and coordinates.

![Deformable 3DGS]({{ '/assets/images/34/deformable.png' | relative_url }})
{: style="width: 800px; max-width: 100%;"}
_Fig 2. Deformable 3DGS_

### DreamGaussian

DreamGaussian adapts 3D Gaussian splatting for image-to-3D mesh generation.
Utilizes diffusion prior (Zero-1-to-3 XL)
Apply local density queries and color back-projection for mesh optimization
Adopts 3DGS Gaussian culling technique with Marching Cubes for efficient rasterization

### Instant Splat
