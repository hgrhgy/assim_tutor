# GEOSCHEM 安装教程

## 1. 配置计算环境

## 1.1 安装Spack
为了更好的安装后面的依赖，先安装Spack，可以方便管理环境的各类依赖。modules是用在hpc环境中，管理计算依赖的包/库管理软件。

### 1.1.1 安装spack
```shell
## 下载源码，源码直接就可以使用
$ git clone https://github.com/spack/spack.git
```
把以下配置放入到`.bashrc`中
```shell
# 源码存放的目录
export SPACK_PREFIX=/path/to/spack/src 
export PATH=$SPACK_PREFIX/bin:$PATH
```
创建spack环境并激活
```shell
$ spack env create geoschem 
$ spack env activate geoschem
```



## 1.2 安装配置编译环境

### 1.2.1 配置gcc
一般来说linux的开发环境是成套的，定义了gcc，g++和fortran编译器都配置好了
```shell
$ spack install gcc@9.2.0
$ spack compiler find
$ spack load gcc@9.2.0 #会配置环境变量FC,CC,CXX
```
> NOTE:
> 如果执行`spack install gcc`出现`Curl failed with error 28`的超时错误，需要在spack安装目录下`etc/config.yaml`中修改链接超时的时间。

## 1.3 安装/配置库环境

### 1.3.1 安装依赖库文件到spack
```shell
$ spack install netcdf-c%gcc@9.2.0 parallel-netcdf=true
$ spack install netcdf-fortran%gcc^netcdf-c+parallel-netcdf@9.2.0
$ spack install git%gcc@9.2.0
$ spack install flex%gcc@9.2.0              
$ spack install cmake%gcc@9.2.0        
$ spack install gdb%gcc@9.2.0     
```

### 1.3.2 配置加载依赖库文件
```shell
$ spack load netcdf-c%gcc@9.2.0
$ spack load netcdf-fortran%gcc@9.2.0
$ spack load git%gcc@9.2.0
$ spack load flex%gcc@9.2.0                
$ spack load cmake%gcc@9.2.0        
$ spack load gdb%gcc@9.2.0     
```

### 1.3.3 配置环境变量
```sh
# netcdf-c
export NETCDF_HOME=`spack location -i netcdf%gcc@9.2.0`
export GC_BIN=$NETCDF_HOME/bin
export GC_INCLUDE=$NETCDF_HOME/include
export GC_LIB=$NETCDF_HOME/lib
# netcdf-gfortran
export NETCDF_FORTRAN_HOME=`spack location -i netcdf-fortran%gcc@9.2.0`
export GC_F_BIN=$NETCDF_FORTRAN_HOME/bin
export GC_F_INCLUDE=$NETCDF_FORTRAN_HOME/include
export GC_F_LIB=$NETCDF_FORTRAN_HOME/lib
```


## 1.4 配置并行相关环境变量
```sh
# 线程数量
export OMP_NUM_THREADS=8
# 堆栈大小
ulimit -s unlimited
export OMP_STACKSIZE=500m
```
# 2. 配置GEOSCHEM

## 2.1 下载GEOSCHEM源码和测试用例

```shell
## 源码
$ git clone https://github.com/hgrhgy/geos-chem Code.12.8.1
$ cd UT.12.8.1
$ git checkout -b GC_12.8.1 12.8.1
## 测试用例
$ git clone https://github.com/hgrhgy/geos-chem-unittest UT.12.8.1
$ cd UT.12.8.1
$ git checkout -b GC_12.8.1 12.8.1
```

## 2.2 下载GEOSCHEM测试用例所需数据

```sh
wget -r -np -nH -N -R "*.html" "http://geoschemdata.computecanada.ca/ExtData/GEOSCHEM_RESTARTS"
wget -r -np -nH -N -R "*.html" "http://geoschemdata.computecanada.ca/ExtData/GEOS_4x5"
# wget -r -np -nH -N -R "*.html" "http://geoschemdata.computecanada.ca/ExtData/${文件夹路径(path/to/data)}
```
> NOTE：
> 数据有很多，根据HEMCO.log出错信息下全数据

## 2.3 创建rundirs
可以利用测试用例创建执行目录，创建之前需要对测试用例的配置进行更改

### 2.3.1 修改配置目录

```shell
$ cd ${GC_HOME}UT/UT.12.8.0/perl
$ vim CopyRunDirs.input
```

配置文件内容(需要配置的见注释)：


> ```sh
> ...
> #------------------------------------------------------------------------------
> #
> # !INPUTS:
> #
> # %%% ID tags %%%
> #
>    VERSION        : 12.8.0
>    DESCRIPTION    : Create run directory from UnitTest
> #
> # %%% Data path and HEMCO settings %%%
> #
> #  *** 配置的数据根目录 ***
>    GCGRID_ROOT    : /share/nas1_share2/ExtData
>    DATA_ROOT      : {GCGRIDROOT}/
>    VERBOSE        : 0
>    WARNINGS       : 1
> #
> # %%% Code and queue settings %%%
> #
> #  *** 配置的代码目录， 其中HOME为用户目录 ***
>    CODE_DIR       : {HOME}/geoschem/GC/Code.12.8.0
> #
> # %%% Unit tester path names %%%
> #
> #  *** 配置的测试用例目录 ***
>    UNIT_TEST_ROOT : {HOME}/geoschem/UT/UT.12.8.0
>    RUN_ROOT       : {UTROOT}/runs
>    RUN_DIR        : {RUNROOT}/{RUNDIR}
>    PERL_DIR       : {UTROOT}/perl
> #
> # %%% Target directory and copy command %%%
> # 
> #  *** 配置的执行目录生成地址 ***
>    COPY_PATH      : {HOME}/geoschem/GC/rundirs
>    COPY_CMD       : cp -rfL
> #
> # !RUNS:
> #  Specify the runs directories that you want to copy below.
> #  Here we provide a few examples, but you may copy additional entries from
> #  UnitTest.input and modify the dates as needed. You can deactivate copying
> #  run certain directories by commenting them out with "#".
> #
> # *** 这些是测试用例的选项，打开注释可以生成对应的用例
> #--------|-----------|------|----------------|------------|--------------|-----|
> # MET    | GRID      | NEST | SIMULATION     | START DATE | END DATE     |EXTRA|
> #--------|-----------|------|----------------|------------|--------------|-----|
> # ======= Standard ===========================================================
>   geosfp   4x5         -      standard         2016070100   2016080100     -
> # merra2   4x5         -      standard         2016070100   2016080100     -
> # geosfp   2x25        -      standard         2016070100   2016080100     -
> # merra2   2x25        -      standard         2016070100   2016080100     -
> # ======= GEOS-Chem benchmark ======================================= =========
> ...
> ```

### 2.3.2 生成执行目录

```shell
$ ./gcCopyRunDirs
```

> NOET: 
> gcCopyRunDirs会用到部分的数据，主要是RESTART的数据；
> 如果出现各种warning，多数是因为数据没有下载完全，根据提示完成数据准备；

### 2.3.3 生成测试用例
下面脚本命令行执行后，会在geosfp_4x5_standard生成可执行文件geos

```shell
$ cd ${HOME}/geoschem/GC/rundirs
$ cd geosfp_4x5_standard
$ mkdir gcbuild
$ cd gcbuild
$ cmake ../CodeDir
$ make -j4 install
```

### 2.3.4 执行测试用例
```shell
$ cd ..
$ ./geos
```

## 3. 形成的配置文件
geos.env， 通过`source geos.env`使用
```sh
#!/bin/bash
#==============================================================================
# %%%%% Clear existing environment variables %%%%%
#==============================================================================
unset PERL_HOME
unset IDL_HOME
unset EMACS_HOME
unset CC
unset CXX
unset FC
unset F77
unset F90
unset NETCDF_BIN
unset NETCDF_HOME
unset NETCDF_INCLUDE
unset NETCDF_LIB
unset NETCDF_FORTRAN_BIN
unset NETCDF_FORTRAN_HOME
unset NETCDF_FORTRAN_INCLUDE
unset NETCDF_FORTRAN_LIB
unset GC_BIN
unset GC_INCLUDE
unset GC_LIB
unset GC_F_BIN
unset GC_F_INCLUDE
unset GC_F_LIB
unset OMP_NUM_THREADS
unset OMP_STACKSIZE
 
#==============================================================================
# %%%%% Load modules for GNU Fortran 7.1 %%%%%
#
# NOTE: Your module load commands may be different than these.
# Ask your IT staff for more information.
#==============================================================================
echo "activate python env geoschem"
conda activate geoschem

echo "activate spack env geoschem"
spack env activate geoschem

echo "spack set compiler gcc@9.2.0"
spack load gcc@9.2.0

echo "loading dependencies ..."
spack load netcdf-c%gcc@9.2.0
spack load netcdf-fortran%gcc@9.2.0
spack load git%gcc@9.2.0
spack load flex%gcc@9.2.0       
spack load cmake%gcc@9.2.0        
spack load gdb%gcc@9.2.0     

# Define F90 and F77 environment variables (may be needed by some software)
export F90=$FC
export F77=$FC

#==============================================================================
# %%%%% Settings for OpenMP parallelization %%%%%
#==============================================================================

# Max out the stack memory for OpenMP
# Asking for a huge number will just give you the max availble
export OMP_STACKSIZE=500m

# By default, set the number of threads for OpenMP parallelization to 1
export OMP_NUM_THREADS=24

# Redefine number threads for OpenMP parallelization
# (a) If in a SLURM partition, set OMP_NUM_THREADS = SLURM_CPUS_PER_TASK
# (b) Or, set OMP_NUM_THREADS to the optional first argument that is passed
if [[ -n "${SLURM_CPUS_PER_TASK+1}" ]]; then
  export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
elif [[ "$#" -eq 1 ]]; then
  if [[ "x$1" != "xignoreeof" ]]; then
     export OMP_NUM_THREADS=$1
  fi
fi
echo "Number of OpenMP threads: $OMP_NUM_THREADS"

#==============================================================================
# %%%%% Define relevant environment variables %%%%%
#==============================================================================

# Machine architecture
export ARCH=`uname -s`

# netcdf-c
export NETCDF_HOME=`spack location -i netcdf-c%gcc@9.2.0`
export GC_BIN=$NETCDF_HOME/bin
export GC_INCLUDE=$NETCDF_HOME/include
export GC_LIB=$NETCDF_HOME/lib

# netcdf-fortran
export NETCDF_FORTRAN_HOME=`spack location -i netcdf-fortran%gcc@9.2.0`
export GC_F_BIN=$NETCDF_FORTRAN_HOME/bin
export GC_F_INCLUDE=$NETCDF_FORTRAN_HOME/include
export GC_F_LIB=$NETCDF_FORTRAN_HOME/lib
```
