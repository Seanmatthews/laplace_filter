# laplace_filter

A sample Morphological Laplacian operator (Laplacian of Gaussian) applied to an image using CUDA and OpenCV.

It works by accepting a YUV420p image as input, converting it to RGB, forming a 1D Laplacian of Gaussian kernel, and then performing convolution of that kernel, and its transpose, over all three channels of the image. The results of the two different convolutions are added and the result is displayed. 

Rather than looping over the channels of the image, all channels are considered side-by-side during convolution (actually, OpenCV stores the channels interlaced). This offers a considerable speedup.

Typically, with Gaussian kernels, you would perform convolution with a 2D kernel over an image by separating the 2D kernel into Nx1 and 1xN vectors (for separate horizontal and vertical passes). You can do this with any kernel which is separable. However, the Laplacian kernel is not separable and the convolution cannot be performed in this manner. Though you can effectively "separate" (it's not a real separation) the 2D kernel by adding the resuls of the convolution of the 1D kernel and its transpose. Adding these results is possible because the Laplacian kernel is a zero sum oeprator.


## Usage

`./laplacefilter <image path> <width> <height> [gaussian sigma]`

## Requirements

* NVidia CUDA (recommended 8.0)
* OpenCV (recommended 3.1)

## Performance results

* Tested on GeForce GTX 780 Ti
* 1920x1080 image with 3 channels
* byte count: 6220800
* grid: (360, 90) block: (16, 12)
* Kernel runtime: 0.356928 ms
* Bandwidth: 34.8575 GB/s
* 5809.58 MPixels/s
* Command: `laplacefilter tree.yuv 1920 1080 0.5`

*No warmup period built in

Original
![Alt text](tree.jpg?raw=true "Original")

All channels
![Alt text](allChannels.jpg?raw=true "All channels")

Red channel
![Alt text](redChannel.jpg?raw=true "Red channel")

Blue channel
![Alt text](blueChannel.jpg?raw=true "Blue channel")

Green channel
![Alt text](greenChannel.jpg?raw=true "Green channel")

## Assumptions

### Kernel size
A 5x5 cross-shaped kernel is used for convolution. The reasons for this include:

* A non-configurable kernel size makes the loop unrolling code much simpler. Loop unrolling provides a significant speed up, and simple code is, well, easy to read/write.
* A 2D Laplacian kernel may be approximated by adding the results of horizontal and vertical 1D Laplacian kernel convolutions. Performing the convolution with the cross formed by two 1D kernels, offers considerable speed up due to fewer arithmetic operations.

### No weird input sizes

The methods used for grid and block size calculation may fail on uncommon input sizes (for example, 512x511). This is an edge case I did not get to.

### Implicitly thresholded result

The Laplacian kernel, while a zero sum operator, can produce negative values in the reult matrix. Typically, you would search for the zero-crossings in this matrix and set those locations to 1 (or 255, etc). Since this is for demonstration purposes, I implicitly clip the negative values to 0 and I don't scale the result to the [0, 255] range. I do this because 1) OpenCV does not display negative values, 2) you essentially get a zero crossing "plus" with this method-- the edges between negative and positive values are shown, but so are any large areas of positive values, and 3) the result looks nice! Regardless, the program demonstrates the speedy application of a Laplacian operator on an image.

### Only accepts YUV420p image

The sign of the data, as well as the bit length representation, are specific to this format. To create a test image from, say, a jpeg, use the following ffmpeg command with your own image's names and size:
`ffmpeg -i yourjpeg.jpg -s 1920x1080 -pix_fmt yuv420p yourjpeg.yuv`