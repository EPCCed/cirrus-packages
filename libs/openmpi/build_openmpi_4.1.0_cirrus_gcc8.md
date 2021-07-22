Instructions for building OpenMPI 4.1.0 on Cirrus
=================================================

These instructions are for building OpenMPI 4.1.0 on Cirrus (SGI ICE XA, Intel Xeon Broadwell (CPU) and Cascade Lake (GPU)) using gcc 8.2.0.


Setup initial environment
-------------------------

```bash
PRFX=/path/to/work
OPENMPI_LABEL=openmpi
OPENMPI_VERSION=4.1.0
OPENMPI_NAME=${OPENMPI_LABEL}-${OPENMPI_VERSION}

mkdir -p ${PRFX}/${OPENMPI_LABEL}
cd ${PRFX}/${OPENMPI_LABEL}

wget https://download.open-mpi.org/release/open-mpi/v${OPENMPI_VERSION}/${OPENMPI_NAME}.tar.gz
tar xzf ${OPENMPI_NAME}.tar.gz
rm ${OPENMPI_NAME}.tar.gz
cd ${OPENMPI_NAME}
```

Remember to change the setting for `PRFX` to a path appropriate for your Cirrus project.


Build and install OpenMPI for CPU
---------------------------------

```bash
module load zlib/1.2.11
module load gcc/8.2.0

./configure CC=gcc CXX=g++ FC=gfortran \
  CFLAGS="-I/lustre/sw/pmi2/include" \
  LDFLAGS="-L/lustre/sw/pmi2/lib" \
  --enable-mpi1-compatibility --enable-mpi-fortran \
  --enable-mpi-interface-warning --enable-mpirun-prefix-by-default \
  --with-slurm \
  --with-ucx=/lustre/sw/ucx/1.9.0 \
  --with-pmi=/lustre/sw/pmi2 --with-pmi-libdir=/lustre/sw/pmi2/lib \
  --with-libevent=/lustre/sw/libevent/2.1.12 \
  --prefix=/lustre/sw/${OPENMPI_LABEL}/${OPENMPI_VERSION}

make
make install
make clean
```


Build and install OpenMPI for GPU (CUDA 10.1)
---------------------------------------------

```bash
module unload gcc
module load nvidia/cuda-11.2
module load nvidia/mathlibs-11.2
module swap gcc gcc/8.2.0

./configure CC=gcc CXX=g++ FC=gfortran \
  CFLAGS="-I/lustre/sw/pmi2/include" \
  LDFLAGS="-L/lustre/sw/pmi2/lib" \
  --enable-mpi1-compatibility --enable-mpi-fortran \
  --enable-mpi-interface-warning --enable-mpirun-prefix-by-default \
  --with-slurm \
  --with-ucx=/lustre/sw/ucx/1.9.0-cuda-11.2 \
  --with-pmi=/lustre/sw/pmi2 --with-pmi-libdir=/lustre/sw/pmi2/lib \
  --with-cuda=${CUDAROOT} \
  --with-libevent=/lustre/sw/libevent/2.1.12 \
  --prefix=/lustre/sw/${OPENMPI_LABEL}/${OPENMPI_VERSION}-cuda-11.2

make
make install
make clean
```