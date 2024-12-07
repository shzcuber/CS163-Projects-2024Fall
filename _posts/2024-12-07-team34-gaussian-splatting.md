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

SfM outputs a sparse point cloud given several 2D images
2D image alignment by matching key points across images
COLMAP

## NeRFs

Given set of images + camera poses, generates 3D scene
Generates rays for each pixel and viewing direction
Samples points along each ray, accumulating color and density
Leverages COLMAP structure-from-motion for pose estimation
Implemented using a multi-layer perceptron (MLP) architecture

Pros:
Can capture fine scene geometry using raycasting algorithm
Small storage costs due to implicit representation (only requires MLP weights)
Cons:
Very slow rendering times (~1–2 days in original implementation)
Implicit representation makes downstream tasks more difficult
Scene editing, mesh generation, etc.

# 3D Gaussian Splatting

## 3D Gaussians

## 3D Gaussian Optimization

## 3D Gaussian Rasterizer

## Results

## Applications

### Mesh Extraction (SuGaR)

### Deformable 3DGS

3DGS for dynamic scenes
Jointly learning a deformation field
along with the 3D Gaussians
Learning the geometric prior information about the scene
Deformation field is an MLP network mapping from
each coordinate of 3D Gaussians + time to position, rotation,
and scaling
Using annealing smooth training to reduce
jitter in interpolated frames

### DreamGaussian

Adapts 3D Gaussian splatting for image-to-3D mesh generation
Utilizes diffusion prior (Zero-1-to-3 XL)
Apply local density queries and color back-projection for mesh optimization
Adopts 3DGS Gaussian culling technique with Marching Cubes for efficient rasterization

### Instant Splat
