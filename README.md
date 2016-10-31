# laplace_filter

A sample Morphological Laplacian operator (Laplacian of Gaussian) applied to an image using CUDA and OpenCV.

## Usage

`./laplacefilter <image path> <width> <height> [gaussian sigma]`

## Requirements

* NVidia CUDA (recommended 8.0)
* OpenCV (recommended 3.1)

## Assumptions

### Kernel size
A 5x5 cross-shaped kernel is used for convolution. The reasons for this include:

* A non-configurable kernel size makes the loop unrolling code much simpler. Loop unrolling provides a significant speed up, and simple code is, well, easy to read/write.
* A 2D Laplacian kernel may be approximated by adding the results of horizontal and vertical 1D Laplacian kernel convolutions. Performing the convolution with the cross formed by two 1D kernels, offers considerable speed up due to fewer arithmetic operations.

### No weird input sizes

The methods used for grid and block size calculation may fail on uncommon input sizes (for example, 512x511). This is an edge case I did not get to.

### Implicitly thresholded result

The Laplacian kernel, while a zero sum operator, can produce negative values in the reult matrix. Typically, you would search for the zero-crossings in this matrix and set those locations to 1 (or 255, etc). Since this is for demonstration purposes, I implicitly clip the negative values to 0 and I don't scale the result to the [0, 255] range. I do this because 1) OpenCV does not display negative values, 2) you essentially get a zero crossing "plus" with this method-- the edges between negative and positive values are shown, but so are any large areas of positive values, and 3) the result looks nice! Regardless, the program demonstrates the speedy application of a Laplacian operator on an image.