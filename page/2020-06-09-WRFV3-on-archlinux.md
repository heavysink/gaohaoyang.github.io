---
layout: post
title:  "如何在Archlinux下安装WRF V3.x 适用GCC/GFORTRAN 10.x"
categories: WRF
tags: WRF 
author: heavysink
---

* content
{:toc}

## 起因

用于提交学校超算之前检验namelist.input，以及看看我瞎JB改的code能不能跑起来。如果提交超算的配置里面有错的话，学校超算会让我先排几个小时的队，然后再告诉我配置错了... 而且学校超算经常preempt我提交的任务，所以一些轻活（比如验证之类的）干脆就让我那破双路2660先干了吧！

## 过程

首先，把这几个包装上：
``` bash
pacman -S tcsh libtirpc inetutils time netcdf-fortran-openmpi
```

然后设定如下环境变量（推荐放进rc里面）
``` bash
export HDF5=/usr
export PHDF5=/usr
export NETCDF=/usr
```

进入WRFV3代码文件夹，./configure 如果要跑serial（2D理想模型），选32. GNU(serial); 如果要并行，选34. GNU(dmpar)。据说OpenMP目前还不完善...

接着把share/landread.c里面rpc/types.h改成tirpc/rpc/types.h，同理rpc/xdr.h --> tirpc/rpc/xdr.h

接着修改生成的configure.wrf，有如下几个变量需要修改
```bash
SFC             =       gfortran -fallow-argument-mismatch -fallow-invalid-boz
DM_FC           =       mpif90 -DMPI2_SUPPORT -fallow-argument-mismatch -fallow-invalid-boz
DM_CC           =       mpicc -DMPI2_SUPPORT -I/usr/include/tirpc
CPP             =      /usr/bin/cpp -P -nostdinc
LIB_EXTERNAL    = \
                       -L$(WRF_SRC_ROOT_DIR)/external/io_netcdf -lwrfio_nf -L/usr/lib -lnetcdff -lnetcdf -L$(WRF_SRC_ROOT_DIR)/external/            io_pnetcdf -lwrfio_pnf -L/usr/lib -lpnetcdf   -L/usr/lib -lhdf5_fortran -lhdf5 -lm -lz -ltirpc
```
然后照常./compile <需要的算例> 即可。
