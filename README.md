# Yellow-Grid Removal and ArUco Marker Recovery

This repository is a fork of [`vortex-image-filtering`](https://github.com/vortexntnu/vortex-image-filtering), a ROS 2 image-filtering package developed by Vortex.

My contribution is the `RemoveGrid` filter, a computer-vision algorithm designed to remove a yellow grid that partially obscures an ArUco marker. The remaining filters, framework, and supporting code in this repository originate from the Vortex project unless otherwise stated.

## Problem

When a yellow grid partially obscures an ArUco marker, the standard ArUco detector may no longer be able to identify the marker.

## Approach

The `RemoveGrid` filter processes the image in the following stages:

1. The image is cropped and, if required, rotated according to parameters specified by the user. Processing a smaller region reduces the computational cost of the later inpainting operation.
2. The image is converted from RGB to the HSV colour space. Testing showed that HSV provided more reliable separation of the yellow grid from the rest of the image.
3. Colour thresholding is used to identify the yellow grid and create a binary mask.
4. The mask is dilated to cover the grid edges and any fragmented areas that were not fully captured by the thresholding.
5. OpenCV's `inpaint` function estimates the missing image regions using information from the surrounding pixels. This removes the detected grid and reconstructs the obscured parts of the ArUco marker.
6. When required, the processed region is warped and rotated back to its original position and inserted into the full-sized image.

The reconstructed marker can then be passed to an ArUco detector.

## Code

The filter is implemented in:

[`include/image_filtering/lib/filters/remove_grid.hpp`](include/image_filtering/lib/filters/remove_grid.hpp)

## Testing and Parameter Tuning

The HSV threshold parameters were tuned using a random-search procedure. The filter can be tested in the Stonefish simulator by following the instructions provided in the [Vortex GitHub repository](https://github.com/vortexntnu/vortex-stonefish-sim).

---

# Image Filtering Node
[![Industrial CI](https://github.com/vortexntnu/vortex-image-filtering/actions/workflows/industrial-ci.yml/badge.svg)](https://github.com/vortexntnu/vortex-image-filtering/actions/workflows/industrial-ci.yml)
[![pre-commit.ci status](https://results.pre-commit.ci/badge/github/vortexntnu/vortex-image-filtering/main.svg)](https://results.pre-commit.ci/latest/github/vortexntnu/vortex-image-filtering/main)
[![codecov](https://codecov.io/github/vortexntnu/vortex-image-filtering/graph/badge.svg?token=6XHprkpUsR)](https://codecov.io/github/vortexntnu/vortex-image-filtering)

The `image_filtering_node` is a ROS 2 node developed in the `vortex::image_filters` namespace. It is designed to subscribe to image topics, apply various image filters using OpenCV, and publish the filtered images back to ROS.

## Features

- **Multiple Filters**: Supports sharpening, unsharpening, eroding, dilating, white balancing, and a custom "ebus" filter.
- **Dynamic Reconfiguration**: Allows for runtime changes to filter parameters and subscribed image topics via ROS 2 parameters.
- **Parameter-Driven Configuration**: Configures filter types and specific attributes through ROS 2 parameters.

## Configuration

Configure the node using ROS 2 parameters:

- **`image_topic`**: Topic from which the node will subscribe to image messages.
- **`filter_params.filter_type`**: Specifies the filter to apply. Current options include:
  - `nofilter`
  - `sharpening`
  - `unsharpening`
  - `eroding`
  - `dilating`
  - `white_balancing`
  - `ebus`
- **Other filter-specific parameters** such as `blur_size`, `size`, `contrast_percentage`, can be configured through parameter `filter_params`.{filter_name}.{param_name}

Parameters can be set through a YAML file or dynamically adjusted at runtime.

