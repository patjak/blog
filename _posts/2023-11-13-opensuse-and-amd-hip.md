---
title: openSUSE and AMD HIP
date: 2023-11-13
tags: opensuse amd rocm hip gpu compute
published: false
---
In this post I'll describe what I did to get started writing HIP applications on openSUSE Tumbleweed. Since Tumbleweed is a rolling release distribution this information might get outdated rather quickly so I'll describe how to set this up in openSUSE Leap 15.5 as well.

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

# Installing HIP
HIP requires the ROCm stack which consists of quite a few components. I have made a some attempts to build ROCm with the openSUSE build system but have not been able to package everything needed to get to the point where HIP applications can be built. More on this later. Luckily AMD provides prebuilt packages for SUSE that also works on Leap and Tumbleweed.

## Prerequisites

### Perl packages for Leap 15.5
```
$ zypper addrepo https://download.opensuse.org/repositories/devel:/languages:/perl/15.5/devel:languages:perl.repo
```

### Perl packages for Tumbleweed
```
$ zypper addrepo https://download.opensuse.org/repositories/devel:/languages:/perl/openSUSE_Tumbleweed/devel:languages:perl.repo
```
## Adding the ROCm repo
AMD provides an out-of-tree AMDGPU kernel driver but the openSUSE kernel should work just fine so we can skip that step. Instead we continue by adding the ROCm repository. Replace $VER with whatever version of ROCm you would like to install. ROCm supports installing multiple versions side-by-side so you can go back and repeat this step whenever a new version is released.
```
$ export VER=5.7.1

$ tee --append /etc/zypp/repos.d/rocm.repo <<EOF
[ROCm-$VER]
name=ROCm$VER
name=rocm
baseurl=https://repo.radeon.com/rocm/zyp/$VER/main
enabled=1
gpgcheck=1
gpgkey=https://repo.radeon.com/rocm/rocm.gpg.key
EOF

$ zypper refresh
```

## Installing the SDK
All you need to do now is to install the rocm-hip-sdk package that will pull in all the dependencies in the ROCm stack. Tumbleweed currently complains about missing libffi.so.7 for openmp-extras-runtime but since we are not interested in openmp we can just ignore the dependency.
```
$ zypper install rocm-hip-sdk
```
Or if you need a specific version for which you have also added the repo in the previous step
```
$ zypper install rocm-hip-sdk5.6.1
```
