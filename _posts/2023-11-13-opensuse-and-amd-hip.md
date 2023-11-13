---
title: openSUSE and AMD HIP
date: 2023-11-13
tags: opensuse amd rocm hip gpu compute
published: false
---
In this post I'll describe what I did to get started writing HIP applications on openSUSE Tumbleweed. Since Tumbleweed is a rolling release distribution this information might get outdated rather quickly so I'll also describe how to set this up in openSUSE Leap 15.5.

# What is HIP?
Here's how AMD describes HIP:

"HIP is a C++ Runtime API and Kernel Language that allows developers to create portable applications for AMD and NVIDIA GPUs from single source code."

Here's a very simplified example of HIP code:

## main.cpp
```
#include <hip/hip_runtime.h>

__global__ void device_code(args...) {
  // GPU code goes here
}

int main(int argc, char **argp) {
  ...
  hipLaunchKernelGGL(device_code, blocks, threads, 0, 0, args...);
  ...
}
```
And you would compile it as a normal c++ program but with hipcc instead of g++ or clang++
```
$ hipcc main.cpp -o hip-test
```

The resulting binary
