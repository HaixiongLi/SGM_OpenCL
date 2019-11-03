
## Theoretical Background

A heterogeneous and fully parallel stereo matching algorithm for depth estimation. Stereo Matching is based on the disparity estimation algorithm, an algorithm designed to calculate 3D depth information about a scene from a pair of 2D images captured by a stereoscopic camera. The algorithm contains the following stages:

* Cost Volume Construction - weighted absolute difference of colours and gradients function.
* Cost Volume Filtering - Adaptive Support Weight (ADSW) Guided Image Filter (GIF) function.  
* Disparity Selection - Winner-Takes-All (WTA) minimum cost selection.  
* Post Processing - left-right occlusion check, invalid pixel replacement and weight-median filtering.  

<p align="center">
<img src="docs/de_bd.png" alt="Disparity estimation process block diagram" width=80%>
</p>

## Implementation Details

* All stages of the algorithm have been developed in both C++ and OpenCL.  
	* C++ parallelism is introduced via the POSIX threads (pthreads) library. Disparity level parallelism is supported, enabling up to 64 concurrent threads.  
	* OpenCL parallelism is inherent through the concurrent execution of kernels on an OpenCL-compatible device. The optimum level of parallelism will be bounded by the platform & devices.  
* Support for live video disparity estimation using the OpenCV VideoCapture interface as well as static image computation.
* Additional integration of the OpenCV Semi-Global Block Matching (SGBM) algorithm.

## Installation

### Prerequisites
* Hardware:
	* Development Platform - preferably including devices supporting OpenCL
	* Stereo Camera - to use the algorithm in video mode - the [ZED Stereo Camera](https://www.stereolabs.com/) is used in our experimentation.
* Software Libraries:
	* OpenCV 3.0 or later - [Installation in Linux instructions](https://docs.opencv.org/3.4.0/d7/d9f/tutorial_linux_install.html)
	* pthread library for non-OpenCL execution on the CPU
	* OpenCL Library for execution on the GPU
	* cmake v3.2, git

### Compilation 
* Clone repo to the platform: `git clone https://github.com/PRiME-project/PRiMEStereoMatch.git`
* Enter the base directory: `cd PRiMEStereoMatch/`.
* Create a build directory: `mkdir build`
* Enter the build directory: `cd build`
* Invoke cmake to build the compilation files: `cmake ..` (Two dots are required in order to reference the base directory)
* Compile the project with the generated makefile: `make -jN`. 
	* Set N to the number of simultaneous threads supported on your compilation platform, e.g. `make -j8`.

### Deployment
* Run the application from the build dir: `./PRiMEStereoMatch <program arguments>`
* The program mode is selected with git-style commands. Valid commands include:
	* video
		* [optional] When specifying the video mode, the following arguments can be included:
			* --recal - recalculate the intrinsic and extrinsic parameters of the stereo camera. Previously captured chessboard images must be supplied if the RECAPTURE flag is not also set.
			* --recap - record chessboard image pairs in preparation for calibration. A chessboard image must be presented in front of the stereo camera and in full view of both cameras. Press the R key to capture a frame. The last frame captured is shown beneath the video stream.
	* image
		* [optional] When specifying the image mode, the following arguments can be included:
			* -l [i]left *image filename>* -r *right image filename*
			* -gt *ground truth filename*
* A set of global options also exist, which must be specified for all modes:
	* -a (--alg=) - Set the default matching algorithm to run. It has options {STEREO_GIF, STEREO_SGBM}. This can also be toggled during executions.

* For example, to run using a stereo camera, specify:
	* `./PRiMEStereoMatch video`
* To run with calibration and capture beforehand, specify:
	* `./PRiMEStereoMatch video --recal --recap`
* Image disparity estimation is achieved using:
	* `./PRiMEStereoMatch image -l left_img.png -r right_img.png`

* The first time the application is deployed using a stereo camera, the --recal and --recap flags must be set in order to capture chessboard image to calculate the intrinsic and extrinsic parameters.
* This process only needs to be repeated if the relative orientations of the left and right cameras are changed or a different resolution is specified.
* Once the intrinsic and extrinsic parameters have been calucalted and saved to .yml files, the application can be re-run with the same camera without needing to recalibrate as the parameters will be loaded from these files. The files can be found in the data directory.

### Interactivity

* Press h to display a help menu on the command line. This shows input and control options for the program which change the way the algorithm behaves for the next frame.
* Control Options:
	* Matching Algorithm (a): STEREO_GIF or STEREO_SGBM
	* STEREO_GIF:
		* Numbers 1 - 8: (CPU only) change the number of simultaneous pthreads created
		* m: switch the computational mode between OpenCL (GPU) and pthreads (CPU)
		* t: switch the data type use for processing between 32-bit float and 8-bit char
	* STEREO_SGBM:
		* m: switch the computational mode between MODE_SGBM, MODE_HH and MODE_SGDM_3WAY

## Directory Structure

```
folders:
	assets			- OpenCL kernel files
	data			- program data including input images, stereo camera parameters, calibration images
	docs			- images for the readme & wiki
	include			- Project header files (h/hpp)
	src			- Project source files (c/cpp)
	
files:
	CMakeLists.txt		- cmake project compilation file
	LICENCE.txt			- license file
	README.md			- this readme file
```

## References

### Code

Some components of the application are based on source code from the following locations:

 [rookiepig/CrossScaleStereo](https://github.com/rookiepig/CrossScaleStereo) - The basis for some C++ functions (GNU Public License)

 [atilimcetin/guided-filter](https://github.com/atilimcetin/guided-filter) - CPU-based GIF implementation using the Fast Guided Filter (MIT License)

### Literature

The algorithm in this work is based in parts on those presented in the following publications:  

<a name="Hosni2011CVPR">[Hosni2011CVPR]</a>: C. Rhemann, A. Hosni, M. Bleyer, C. Rother, and M. Gelautz. Fast cost-volume filtering for visual correspondence and beyond. In CVPR, 2011

<a name="Hosni2011ICME">[Hosni2011ICME]</a>: A. Hosni, M. Bleyer, C. Rhemann, M. Gelautz and C. Rother, Real-time local stereo matching using guided image filtering, in Multimedia and Expo (ICME), 2011 IEEE International Conference on, Barcelona, 2011. 

<a name="Ttofis2014">[Ttofis2014]</a>: C. Ttofis and T. Theocharides, High-quality real-time hardware stereo matching based on guided image filtering, in Design, Automation and Test in Europe Conference and Exhibition (DATE), Dresden, 2014. 

<a name="He2012">[He2012]</a>: K. He, J. Sun and X. Tang, Guided Image Filtering, Pattern Analysis and Machine Intelligence, IEEE Transactions on, pp. 1397-1409, 02 October 2012. 
