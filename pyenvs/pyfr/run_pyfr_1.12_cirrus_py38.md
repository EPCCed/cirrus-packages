Instructions for running PyFR 1.12 on Cirrus
============================================

These instructions are for running PyFR 1.12 on Cirrus (SGI ICE XA, Intel Xeon Broadwell (CPU) and Cascade Lake (GPU)) using Python 3.8.

The instructions take the form of two Slurm submission scripts one using `srun` the other using `mpirun`.
The two scripts are very similar, the main difference being the setting of two Slurm environment variables in the `mpirun` script.

Remember to change the setting for `PRFX` to a path appropriate for your Cirrus project. The submission scripts below assume a locally installed
Miniconda3 virtual environment containing mpi4py, pyfr and supporting packages, see [build instructions](build_pyfr_1.12_cirrus_py38.md) for further details.


Launch a PyFR job (via srun) that uses 16 GPUs across 4 Cascade Lake GPU nodes
------------------------------------------------------------------------------

```bash
#!/bin/bash
 
#SBATCH --job-name=pyfr
#SBATCH --time=4-00:00:00
#SBATCH --partition=gpu-cascade
#SBATCH --nodes=4
#SBATCH --exclusive
#SBATCH --qos=gpu
#SBATCH --gres=gpu:4
#SBATCH --account=[budget code]

NGPUS=16
NGPUS_PER_NODE=4
CPUS_PER_TASK=10

CUDA_VERSION=10.2
OPENMPI_VERSION=4.1.0
MPI4PY_VERSION=3.0.3
MINICONDA3_VERSION=4.9.2

PRFX=/path/to/work
MINICONDA3_ROOT=${PRFX}/miniconda3/${MINICONDA3_VERSION}
MPI4PY_ROOT=${PRFX}/mpi4py/${MPI4PY_VERSION}-ompi-${OPENMPI_VERSION}

INPUTDIR=${PRFX}/input
MESH=${INPUTDIR}/meshes/${NGPUS}GPU_3D_11deg_endwalls_z33.pyfrm
INIT=${INPUTDIR}/tri_airfoil_Re3000_M015_3D.ini


module load nvidia/cuda-${CUDA_VERSION}
module load nvidia/mathlibs-${CUDA_VERSION}
module load openmpi/${OPENMPI_VERSION}-cuda-${CUDA_VERSION}

. ${MINICONDA3_ROOT}/activate.sh
. ${MPI4PY_ROOT}/env.sh

export UCX_MEMTYPE_CACHE=n
export OMPI_MCA_mca_base_component_show_load_errors=0

srun --ntasks=${NGPUS} --tasks-per-node=${NGPUS_PER_NODE} --cpus-per-task=${CPUS_PER_TASK} pyfr run -b cuda -p ${MESH} ${INIT}

conda deactivate
```


Launch a PyFR job (via mpirun) that uses 16 GPUs across 4 Cascade Lake GPU nodes
--------------------------------------------------------------------------------

```bash
#!/bin/bash
 
#SBATCH --job-name=pyfr
#SBATCH --time=4-00:00:00
#SBATCH --partition=gpu-cascade
#SBATCH --nodes=4
#SBATCH --exclusive
#SBATCH --qos=gpu
#SBATCH --gres=gpu:4
#SBATCH --account=[budget code]

NGPUS=16
NGPUS_PER_NODE=4
CPUS_PER_TASK=10

CUDA_VERSION=10.2
OPENMPI_VERSION=4.1.0
MPI4PY_VERSION=3.0.3
MINICONDA3_VERSION=4.9.2

PRFX=/path/to/work
MINICONDA3_ROOT=${PRFX}/miniconda3/${MINICONDA3_VERSION}
MPI4PY_ROOT=${PRFX}/mpi4py/${MPI4PY_VERSION}-ompi-${OPENMPI_VERSION}

INPUTDIR=${PRFX}/input
MESH=${INPUTDIR}/meshes/${NGPUS}GPU_3D_11deg_endwalls_z33.pyfrm
INIT=${INPUTDIR}/tri_airfoil_Re3000_M015_3D.ini


module load nvidia/cuda-${CUDA_VERSION}
module load nvidia/mathlibs-${CUDA_VERSION}
module load openmpi/${OPENMPI_VERSION}-cuda-${CUDA_VERSION}

. ${MINICONDA3_ROOT}/activate.sh
. ${MPI4PY_ROOT}/env.sh


export SLURM_NTASKS_PER_NODE=${NGPUS_PER_NODE}
export SLURM_TASKS_PER_NODE='${NGPUS_PER_NODE}(x${SLURM_NNODES})'

export UCX_MEMTYPE_CACHE=n
export OMPI_MCA_mca_base_component_show_load_errors=0

mpirun -n ${NGPUS} -N ${NGPUS_PER_NODE} pyfr run -b cuda -p ${MESH} ${INIT}

conda deactivate
```


Loading the `openmpi/4.1.0-cuda-11.2` module sets a collection of OpenMPI MCA environment variables
---------------------------------------------------------------------------------------------------

```bash
OMPI_MCA_opal_common_ucx_opal_mem_hooks=1
OMPI_MCA_UCX_MEM_MMAP_HOOK_MODE=none
OMPI_MCA_btl_openib_allow_ib=1
OMPI_MCA_btl_openib_if_include=mlx5_0:1,mlx5_1:1,mlx5_2:1,mlx5_3:1
OMPI_MCA_pml=ucx
OMPI_MCA_btl_openib_warn_default_gid_prefix=0
```

MCA stands for Modular Component Architecture. The settings above ensure that the openib Byte Transfer Layer (BTL)
is active and that the UCX API is used for the Point-to-point Management Layer (PML).

The Cirrus GPU nodes each have four Mellanox Infiniband interfaces, labelled `mlx5_0` through to `mlx5_3`.
The setting for the `btl_openib_if_include` variable ensures that those interfaces are used for the openib BTL
(the number after the colon indicates the port).

Note, the openib BTL uses the OpenFabrics Alliance's (OFA) verbs API stack to support InfiniBand devices, see
https://www.open-mpi.org/faq/?category=openfabrics for further details.