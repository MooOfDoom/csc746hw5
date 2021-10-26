# sobel-filter-cpu-gpu

This directory contains the code for three different implementations of a Sobel edge detection
filter. The three implementations are:

* CPU only in C++, with OpenMP parallelism
* GPU only in CUDA
* CPU in C++ with OpenMP Device Offload for running on the GPU

# Build instructions - general

This assignment requires use of some very specific compilers and environmental settings, as follows:

% module purge  
% module load cmake  
% module load cgpu  
% module load PrgEnv-llvm/13_rc3  

Then, if you're using bash:  

% export CXX=clang++  

Or if you're using csh/tcsh:  

% setenv CXX clang++  

As an alternative,  these module commands are packaged up in a shell file in this distribution that you can 
just source:  

% source scripts/modules_for_cgpu.sh

Then, once your environment is set up, then:

% mkdir build  
% cd build  
% cmake ../  

It is OK to do builds on the login node once you have set up the environment above.

# Comments about all the codes

In all three cases -- CPU only, GPU, and OpenMP device offload -- the programs will all read raw bytes
of an image from a hard-coded location and of a hard-coded sizes. You are free to modify the source
of data and image size if you like.

The image provided with the distribution, located in the data subdirectory, is of a sufficient size , 7112x5146, to
cause the CPU and GPU to do a non-trivial amount of work. More information about these data are located below.

# Testing and verifying your computations

For all programs, when you run the code, it reads from a hard-coded data file, and writes to a hard-coded output file, also located in the data subdirectory. Each of the 3 different codes writes to a different named file, e.g., processed-raw-int8-4x-cpu.dat, processed-raw-int8-4x-gpu.dat, etc.

You can verify your result by displaying the image with the python script provided. For example,

% python scripts/imshow.py processed-raw-int8-4x-gpu.dat 7112 5146

This will display the results of the "correct results" of the sobel filter applied to the default input dataset, zebra-gray-int8-4x. 

Note: if you're running this script from Cori, please be sure that you:
* ssh -Y user@cori.nersc.gov when you connect so that X connections are tunneled through ssh, and the image will actually display remotely, and
* do a %module load python    otherwise you will be accessing an outdated version of python.

# Information about data files

Zebra file dimensions 
* Original: 3556 2573
* 4x Augmented: 7112 5146

* zebra-gray-int8.dat - raw 8-bit grayscale pixel values from the Zebra_July_2008-1.jpg image
* zebra-gray-int8-4x.dat - raw 8-bit grayscale pixel values from the Zebra_July_2008-1.jpg image but 
augmented 2x in each direction

Source file:  Zebra_July_2008-1.jpg, obtained from Wikimedia commons, https://commons.wikimedia.org/wiki/File:Zebra_July_2008-1.jpg

# python display script

imshow.py - a python script to display the raw 8-bit pixel values in grayscale. 

Usage:  
    python imshow.py filename-of-raw-8bit-bytes int-cols-width int-rows-height

# Running the CPU code

In order to run on a KNL node on Cori, it seems one must do a module purge before salloc'ing an interactive KNL node. Once on the node, you can set up the environment again as in the build instructions above. Navigate to the build directory and type:

% export OMP_PLACES=threads
% export OMP_PROC_BIND=spread
% export OMP_NUM_THREADS=16
% ./sobel_cpu

To run with a different number of threads, just do exportOMP_NUM_THREADS=x before running again, where x is the desired number of threads.

# Running the CUDA code

In the build directory, type

% ./sobel_gpu

This will run the GPU code with the default configuration of 256 threads per block and 1 block. In order to specify a different configuration, two optional command line arguments may be given, which specify the number of threads per block and the number of blocks, respectively. For instance, to run 64 threads per block with 256 blocks, the command is

% ./sobel_gpu 64 256

In order to gather performance data for the run, you can type

% export LIBOMPTARGET_INFO=4

Then use nvprof like so:

% nvprof ./sobel_gpu 64 256

To collect SM efficiency data, you can run

% nvprof -m sm_efficiency ./sobel_gpu 64 256

In order to facilitate the collection of this data for all 42 mandated configurations, you can run the script

% sbatch ../job-gpu

This outputs the profiling information in job-gpu.eXXXXXXXX, where XXXXXXXX is the job number assigned to the sbatch. Similarly, for the SM efficiency data, run

% sbatch ../job-gpu-sm

# Running the OpenMP-offload code

From the build directory type

% ./sobel_cpu_omp_offload

Profiling information can be gathered from nvprof analogously to the CUDA code, above, including the batch scripts ../job-offload and ../job-offload-sm.
