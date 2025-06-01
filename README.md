# project6
## Introduction
These days there is a mad rush to be able to analyze massive data sets. In this project, you will perform a quadratic regression on a 4M = 4,194,304 x-y pair dataset. The quadratic regression will produce the equation of a parabola that best fits the data points. The parabola will be of the form: y = Qx2 + Lx + C. Your job will be to run multiple tests to characterize the performance, and to determine what the values of Q, L, and C are.

## Some Help
- Here is proj06.cpp, a skeleton C++ program.
- Here is proj06.cl, a skeleton OpenCL kernel program.

## Requirements:
- You are to read the DATASIZE x-y pairs in the data file and store them in two host (CPU) arrays. The file is here: p6.data
Left-click to look at it, right-click to download it.
To be sure you have the entire file, the size of the file should be 79691776 bytes. The first 5 lines should be:
```
    0.68     11.44
    0.57     13.25
    0.82     11.08
   -0.33      9.62
   -0.44      8.06
```
and the last 5 lines should be:
```
   -0.40      5.45
    0.36     12.42
   -0.71      9.33
   -0.58      7.53
    0.45     12.70
```
- You will then create two DATASIZE-sized device (GPU) arrays and copy all the x-y pairs into them.
- To perform a quadratic regression, one needs to produce 7 summations:
    1. Sum of all x4 = Σx4 = sumx4
    2. Sum of all x3 = Σx3 = sumx3
    3. Sum of all x2 = Σx2 = sumx2
    4. Sum of all x = Σx = sumx
    5. Sum of all (x2*y) = Σ(x2y) = sumx2y
    6. Sum of all (x*y) = Σ(xy) = sunxy
    7. Sum of all y = Σy = sumy

Trust me on this for now. I'd be happy to go over the derivation of why we need these sometime if you'd like.

- Create DATASIZE-sized host and device arrays for all seven of these. You will fill them on the device (GPU) side then - transfer them to the host (CPU) side to sum each array.
- In the proj06.cl file, compute these quantities for each x-y indexed with the OpenCL variable gid.
- Use different combinations of DATASIZE and LOCALSIZE and measure the performances.
- Use performance units that make sense. Joe Parallel used "MegaXYsProcessedPerSecond".
- Bring all seven of these quantities back and place them in the previously-created host arrays.
- Add up the values in the host arrays to produce these CPU floats: sumx4, sumx3, sumx2, sumx, sumx2y, sumxy, and sumy.
- Use the Solve3( ) function to turn these seven quantities into the Q, L, and C line parameters (see below).
- Make two graphs:
    1. Performance versus DATASIZE, with a series of colored Constant-Local-Size curves
    2. Performance versus Local Size (LOCALSIZE), with a series of colored Constant-DATASIZE curves
- Your commentary PDF should tell:
    1. What machine you ran this on
    2. Show the table and graphs
    3. What patterns are you seeing in the performance curves? What difference does the amount of data you process make? What difference does the size of each work-group make?
    4. Why do you think the patterns look this way?
    5. What did you get for Q, L, and C? (Hint: the actual values either end in .0 or .5 -- you don't need to report any more than one decimal digit.)

## Determining the Q, L, and C for the Parabola Equation
To complete the quadratic regression, you need to determine the optimal values of Q, L, and C from the parabola equation: y = Q*x2 + L*x + C. To do this, you need to solve a three-equations-three-unknowns system. Fortunately, you don't need to figure out how to do this yourself. Call the Solve3( ) function (it's in the skeleton code) like this:
```
float Q, L, C;
Solve( sumx4, sumx3, sumx2, sumx, sumx2y, sumxy, sumy, DATASIZE,   &Q, &L, &C );
```

For those new to C/C++, the ampersand (&) means "address of". It gives a function a way to fill values that you want returned. When this function is done executing, Q, L, and C will have the computed values in them, ready for you to include in your report.

## Developing this Project in Linux
You will need the following files:
1. cl.h
2. cl_platform.h

If you are on rabbit, compile, link, and run like this:
``` 
g++  -o  proj06  proj06.cpp  /usr/local/apps/cuda/10.1/lib64/libOpenCL.so.1  -lm  -fopenmp
./proj06
```

If you are on rabbit, put lines in a bash file like this:
```
#!/bin/bash
for s in 4096 16384 65536 262144 1048576 4194304
do
        for b in 8 32 64 128 256
        do
                g++ -DDATASIZE=$s -DLOCALSIZE=$b -o proj06 proj06.cpp /usr/local/apps/cuda/10.1/lib64/libOpenCL.so.1  -lm -fopenmp
                ./proj06
        done
done
```

DATASIZE needs to be no larger than 4194304, since that is how many x-y pairs there are in the file.

If you are on the DGX System, put these lines in a bash file:
```
#!/bin/bash
#SBATCH  -J  Proj06
#SBATCH  -A  cs475-575
#SBATCH  -p  classgputest
#SBATCH  --constraint=v100
#SBATCH  --gres=gpu:1
#SBATCH  -o  proj06.out
#SBATCH  -e  proj06.err
#SBATCH  --mail-type=BEGIN,END,FAIL
#SBATCH  --mail-user=joeparallel@oregonstate.edu

for s in 4096 16384 65536 262144 1048576 4194304
do
        for b in 8 32 64 128 256
        do
                g++ -DDATASIZE=$s -DLOCALSIZE=$b -o proj06 proj06.cpp /usr/local/apps/cuda/11.7/lib64/libOpenCL.so.1  -lm -fopenmp
                ./proj06
        done
done
```

and then submit that file using the sbatch slurm command.

## Getting the right platform and device number:
OpenCL is capable of running on multiple parts of the system you are on (CPU, GPUs, etc.). So, you must decide which one to use.

The skeleton code contains a function called SelectOpenclDevice( ) which selects what it thinks is the best Platform and Device to run your OpenCL code on. Call it from your main program. (That call is already in the skeleton code.) It will print out what it has decided to do. Feel free to comment out those print statements later so they don't interfere with producing an output file.

On rabbit, it produced:
**I have selected Platform #0, Device #0: Vendor = NVIDIA, Type = CL_DEVICE_TYPE_GPU**

## Grading:
Feature | Points
-|-
Performance table | 10
Graph of Performance versus DATASIZE | 25
Graph of Performance versus LOCALSIZE work-group size | 25
Determine the correct Q, L, and C values | 15
Commentary | 25
Potential Total | 100

Note: the graph of performance versus DATASIZE needs to have colored curves of constant LOCALSIZE

Note: the graph of performance versus LOCALSIZE size needs to have colored curves of constant DATASIZE
