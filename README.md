Fei Yang

Mar. 2023

# Introduction 

This article introduces an experimental feature of the open-source project [Three.V8]( https://github.com/fynv/Three.V8), namely Mixed-resolution Grid of Light Probes (or LODProbeGrid which is used for coding).  Three.V8 is a 3D rendering engine and application framework that integrates Google's V8 JavaScript engine, so that games and applications can be coded in JavaScript, while the 3D rendering part is based on the full-featured, native graphics APIs. The LODProbeGrid is a global illumination component of Three.V8, which is evolved from Uniform Grid of Light Probes, the latter is a simple and widely used GI structure. Comparing the uniform grid, the proposed mixed-resolution gird is more compact in storage. It maintains less number of light probes while still meeting the same requirement of probe density.

The technique mainly tackles with is issue that the requirement of probe density is often varying across the scene, because of the uneven distribution of geometry. An uniform grid of probes can be inefficient when such a requirement is present.  An uniform grid only have a fixed density everywhere, therefore, it often ends up either containing much more probes than necessary (oversampling), or use a lower density than necessary (undersampling). In view of such deficiency of an uniform grid, we propose a mixed-resolution, non-uniform grid for probe placement. We will introduce an algorithm to decide the level of subdivision for different areas of space, which will construct the structure of the grid automatically according to the geometry distribution of the scene. At the same time, because of the change to the probe grid structure, we also have to adopt a modified interpolation and weighting scheme during rendering.

It is clear that the technique can increase the efficiency of storage and network transfer of pre-computed GI information, because of the reduced amount of data. It also reduces the time needed for GI data preparation, because of the reduced total number of probes. However, it is also observed that there is some marginal performance loss at runtime on some devices, due to the increased complexity of data structure. Therefore, for performance sensitive uses, we offer an option to convert a mixed-resolution grid of light probes to an uniform grid, which can be done any time before applying it to a scene.

While the technique could potentially have more use-cases, such as using it together with GPU accelerated ray-tracing for dynamic global illumination, we are currently unable to evaluate these scenarios because of the setup of Three.V8, which is mobile targeted and doesn't contain a ray-tracing system. 

# Related Techniques

## Spherical Harmonics

Spherical harmonics are ideal for representing a low-frequency spherical function like irradiance information[[3]](#3). They are extremely compact. Order 3 SH only consists of 9 spherical harmonics basis functions. The spherical harmonics basis functions are linear. When we need to interpolate the irradiance function at some spatial point, we can just calculate a linear combination of the SH coefficients.

We use spherical harmonics to approximate the irradiance function at different spatial locations. While it would have been much easier to do it using ray-traced samples, in this work, we are using a 2 step approach. First rendering to a cube-map, then convert the cube-map to SH coefficients. We use the methods described in [[4]](#4) and [[5]](#5) for the 2nd step.

## Uniform Grid of Light Probes

The simplest form of GI could be using pre-computed irradiance probes distributed in a uniform grid. It is the easiest for shaders to look up the nearest probe around a spatial point, and perform trilinear interpolation. 

Uniform grids of probes are used in NVIDIA's DDGI[[1]](#1) and related works[[2]](#2). The interpolation and weighting scheme proposed in these works, including the visibility test using distance information, has largely resolved the artifacts seen in earlier uniform grid systems, and made it robust to a large range of different environments. In this work, we derive our probe interpolation and weighting scheme based on the tricks used in DDGI for our mixed resolution grid, while restricting to precomputed GI only.

## Image MIP-mapping

When using a uniform grid of irradiance probes represented in SH form, it is easy to consider it as a 3D volumetric image with 9xRGB channels. The trilinear interpolation we use for the probes is also similar to what we do with images. Therefore, it is natural to consider to use techniques developed for images to handle the problem of probe grids.

The issue we currently have with a probe grid is the varying requirement of resolution because of the uneven distribution of geometry, which is basically a level-of-detail problem. For images, we have mip-maps to handle it. For image mip-mapping, we normally down-sample a high-resolution image to half of its dimensions each time, forming a series of images with decreasing sizes. Vertically looking, it can also be seen as a quadtree(for 2D) or an octree(for 3D) hierarchical structure. 

When there is a varying requirement of resolution, 2x2 blocks of pixels from different resolution levels can be compactly packed together, forming an image of mixed resolution. When we are sampling a continuous function, the forming of a mixed resolution grid can be done reversely. We first divide the space with a low resolution "base" grid, so we can sample at the center of each cell. Then by selectively subdividing some of the cells, we get sampling locations of a higher level of resolution, which replaces the low resolution samples. This is what we are going to do with the probe grid.


## Voxelization

Voxelization techniques extract information from a 3D-scene composed of meshes into a volumetric structure. The most efficient way to do voxelization is through rasterization and random texture write operations. While it is possible to develop global illumination techniques directly based on voxelization, in this work, we are using it only for acquiring geometry distribution information, which can help us decide which probe grid cells should be sub-divided.


# The Mixed Resolution Grid

Our mixed resolution grid of probes consists of probes from a mixed levels of resolution. The base level grid is specified by user interactively. Then some of the grid cells are selectively subdivided, and each one of the corresponding low-resolution probes is replaced by 8 probes of a higher resolution level. Voxelization is used in order to decide which cells to be subdivided. A constructed mixed resolution grid can be sampled by shaders using a scheme similar to the one used in DDGI with modifications in order to adapt to the difference of the grid structure.

We first overview what the structure of a constructed mixed resolution grid looks like. Then we go into details how the grid is constructed. Finally, we explain how to sample the irradiance value from a constructed grid.


## The Grid Structure



## Constructing the Grid

## Sampling the Grid

# Results


# Conclusion



# References
<a id="1">[1]</a> 
Majercik Z., Guertin J.P., Nowrouzezahrai D. AND McGuire M. 2019. Dynamic Diffuse Global Illumination with Ray-Traced Irradiance Fields. Journal of Computer Graphics Techniques.

<a id="2">[2]</a> 
McGuire M., Mara M., Nowrouzezahrai D. AND Luebke D. 2017. Realtime
global illumination using precomputed light field probes. In Proceedings of the 21st
ACM SIGGRAPH Symposium on Interactive 3D Graphics and Games.

<a id="3">[3]</a> 
Ramamoorthi R., Hanrahan P. 2001. An efficient representation for irradiance environment maps. SIGGRAPH '01: Proceedings of the 28th annual conference on Computer graphics and interactive techniques.

<a id="4">[4]</a> 
Sloan P. P. 2008. Stupid Spherical Harmonics (SH) Tricks. GDC Lecture.

<a id="5">[5]</a> 
Sloan P. P. 2013. Efficient Spherical Harmonic Evaluation. Journal of Computer Graphics Techniques.



