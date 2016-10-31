# laplace_filter

A sample Morphological Laplacian operator (Laplacian of Gaussian) applied to an image using CUDA and OpenCV.

## Assumptions

### Kernel size
A 5x5 cross-shaped kernel is used for convolution. The reasons for this include:

* A non-configurable kernel size makes the loop unrolling code much simpler. Loop unrolling provides a significant speed up, and simple code is, well, easy to read/write.
* A 2D Laplacian kernel may be approximated by adding the results of horizontal and vertical 1D Laplacian kernel convolutions. Performing the convolution with the cross formed by two 1D kernels, offers considerable speed up due to fewer arithmetic operations.

### No weird input sizes

The methods used for grid and block size calculation may fail on uncommon input sizes (for example, 511x512). This is an edge case I did not get to.