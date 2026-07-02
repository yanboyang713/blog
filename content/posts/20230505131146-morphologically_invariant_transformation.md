---
title: "morphologically invariant transformation"
draft: false
---

[image processing]({{< relref "20230407142839-image_processing.md" >}})

Morphologically invariant transformations represent a crucial concept in the field of computer vision, particularly for enhancing robustness against significant illumination changes. These transformations are characterized by their extraordinary ability to remain consistent under any global, monotonically increasing rescaling of the input signal. This characteristic is referred to as morphological invariance. Such invariance ensures that the transformed signal maintains its essential features regardless of variations in intensity, thereby enabling algorithms to perform consistently in diverse lighting conditions. Morphologically invariant transformations are designed to encapsulate the maximum amount of information possible, which is pivotal for accurate and reliable matching of structures in computer vision applications. Among the algorithms that leverage this property, the [rank transform]({{< relref "2023-12-07-124607-rank_transform.md" >}}) and [census transform]({{< relref "2023-12-07-124626-census_transform.md" >}}) are the most prominent. Both algorithms utilize the principle of morphological invariance to efficiently handle variations in illumination, thus playing a significant role in the development of robust computer vision systems.


## Common User Case {#common-user-case}

-   [stereo camera match]({{< relref "20230505133448-stereo_camera.md" >}})
-   [optical flow]({{< relref "20230505131942-optical_flow.md" >}})


## Reference List {#reference-list}

1.  Demetz, O., Hafner, D., &amp; Weickert, J. (2015). Morphologically invariant matching of structures with the complete rank transform. International Journal of Computer Vision, 113, 220-232.
2.  <https://github.com/bill2239/ranktransform>
