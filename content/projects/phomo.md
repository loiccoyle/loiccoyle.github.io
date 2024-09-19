---
author:
  - Loïc Coyle
title: "Phomo: the photo mosaic python library"
date: "2024-09-04"
description: Don't FOMO on Phomo
summary: Don't FOMO On Phomo
categories:
  - Projects
tags:
  - Python
  - Art
ShowToc: true
TocOpen: false
cover:
  image: /images/projects/phomo/cover.png
  alt: Photo mosaic of the word "Phomo" made of faces
---

An introduction to [{{< ico bootstrap github >}} loiccoyle/phomo](https://github.com/loiccoyle/phomo).

## The shoulders on which I stand

[Photo mosaics](https://en.wikipedia.org/wiki/Photographic_mosaic) are images composed of many smaller images arranged to best reconstruct the master image. The details of the smaller images are visible up close but blur and blend into the master image as the magnification decreases.

> I'll refer the big image as the **master image** and small images as the **tile images**.

{{< figure src="/images/projects/phomo/zoom_out.gif" title="Photo mosaic zoom out" align=center width="500px" >}}

There already exists a couple great photo mosaic python libraries:

- [{{< ico bootstrap github >}} danielballan/photomosaic](https://github.com/danielballan/photomosaic)
- [{{< ico bootstrap github >}} john2x/photomosaic](https://github.com/john2x/photomosaic)

Both these projects arrange the tile images by dividing the master image into smaller regions, computing the average colours of the tiles and regions and then assigning the tile with the closest average colour to the region.

While this method is very computationally efficient, by using the average colour, it loses a lot of the information contained in the tiles and master regions. This can lead loss of detail and suboptimal assignments.

With this said, the goal of `phomo` is to extend on the current photo mosaic libraries mainly improving the tile assignment algorithm and squeezing out more performance.

## Features

The main features of `phomo` are:

- **improved tile to master region metric**
- **optimal tile assignment algorithm**
- **master region grid visualization and subdivision**
- **colour palette matching**
- **concurrent computation & GPU acceleration**

### Tile to master region loss metric

Instead of using the average colour differences as the deciding metric in the tile assignment algorithm, why not use the difference of the colour distribution of the tile images and the master regions.

To illustrate the advantages of this method compared to the average colour metric, let us consider a single high contract master region.

{{< figure src="/images/projects/phomo/starry_night_region.jpg" title="High contract region of The Starry Night" align=center width="500px" >}}

And let us assume that we want to decide which of the following two tiles to assign to the master region.

{{< figure src="/images/projects/phomo/tiles.png" title="" align=center width="500px" >}}

The tile of the left contains the average colour of the master region, and the tile of the right is a picture of a cypress tree in front of a blue sky.

When using the average colour difference metric, the assignment algorithm would favour the tile of the left which would lose the contrast of the master region. By using the full colour distribution difference, the assignment algorithm would favour the tile of the right, which does a better job at preserving the contrast of the master region.

> `phomo` actually implements a [few different metrics](https://github.com/loiccoyle/phomo/blob/main/phomo/metrics.py) which all make use of the spatial distribution of the colours within the tiles and master regions.

### Tile assignment algorithm

Great, so we have a nice way of measuring the compatibility of the tiles with the master regions, but now comes the question of using this metric to figure out which tile goes where.
Let us assume we have computed the loss metric between every tile and master region and stored the results in a matrix.

> This matrix is referred to as the **distance matrix**.

The goal is the assign each tile to a master region while minimizing the loss over the entire reconstructed mosaic.

This is a well known problem, referred to as the [assignment problem](https://en.wikipedia.org/wiki/Assignment_problem), which luckily for me has a well known solution: the [Hungarian algorithm](https://en.wikipedia.org/wiki/Hungarian_algorithm), a variant of which is implemented in `scipy` as [`scipy.optimize.linear_sum_assignment`](https://docs.scipy.org/doc/scipy/reference/generated/scipy.optimize.linear_sum_assignment.html).

### Master region grid

When exploring existing libraries, I quite liked the grid visualization and subdivision features of the [{{< ico bootstrap github >}} danielballan/photomosaic](https://github.com/danielballan/photomosaic) package, showcased in their [docs](http://danielballan.github.io/photomosaic/docs/tutorial.html#partition-tiles). The basic idea is to subdivide master regions with high contrast into smaller regions, effectively increasing the resolution in these high contrast regions.

{{< figure src="/images/projects/phomo/photomosaic_grid.png" title="Grid snippet from the photomosaic docs" align=center width="600px" >}}

As such this feature was implemented in `phomo` and an example of its usage is shown in the [example `jupyter` notebooks provided](https://github.com/loiccoyle/phomo/blob/main/examples/faces.ipynb).

{{< figure src="/images/projects/phomo/phomo_grid.png" title="Grid snippet from the phomo examples" align=center width="400px" >}}

### Colour palette matching

> This feature is inspired by the [palette adjustment](http://danielballan.github.io/photomosaic/docs/palette.html) feature in [{{< ico bootstrap github >}} danielballan/photomosaic](https://github.com/danielballan/photomosaic).

To create good looking photo mosaics, it helps to have similar colour distributions between the master regions and the tiles.

> Assume we have a master image with a blue colour palette and our tiles are mostly reddish, then we obviously won't obtain the most accurate photomosaic.

`phomo` provides three methods to match the tile and master image colour palettes.

- **palette normalisation:** This method stretches out the colour distributions of both the tiles and the master images to cover the full colour space, this will lead to more appealing photo mosaics but will sacrifice the fidelity of both the master and tile images.
- **match the tile images to the master image**: This method uses the [Reinhard colour transfer algorithm](https://www.semanticscholar.org/paper/Color-Transfer-between-Images-Reinhard-Ashikhmin/f3a11158e9d8bdfdf07dca756335c084fce0123e) to transfer the colour distribution of the master image to the tiles. This will leave the master image unchanged but will alter the tile images.
- **match the master image to the tile images**: Similar to the previous method, this uses the same colour transfer algorithm to transger the colour distribution of the tiles to the master image. This will leave the tile images unchanged but will alter the master image.

> These three methods are showcased in the [example colour matching notebook](https://github.com/loiccoyle/phomo/blob/main/examples/matching.ipynb).

### Concurrent computation

`phomo` has a few tricks to improve the computation speed of the **distance matrix**. The simplest of which is to leverage the user's multiple cores parallelize the computation. Each worker is given a chunk of the tile images and computes the loss metric between its tiles and the master regions. The workers then combine their results to produce the distance matrix.

An alternative method to speed up the distance metric computation is to use the GPU to accelerate the computation. `phomo` provides a `CUDA` kernel which computes the loss metric on the GPU. This requires the user to have a compatible GPU as well as enough VRAM to store the master regions and tiles. `phomo` uses the [`numba` library](https://numba.readthedocs.io/en/stable/cuda/overview.html) to compile `python` code to `CUDA` kernels. This method significantly speeds up the distance matrix computation.

> An example showcasing these two methods is provided in the [example performance notebook](https://github.com/loiccoyle/phomo/blob/main/examples/performance.ipynb). **On my system, GPU acceleration leads to a 30x speed increase.**
