Instructions for building PyFR 1.12 on Cirrus
=============================================

These instructions are for building PyFR 1.12 on Cirrus (SGI ICE XA, Intel Xeon Broadwell (CPU) and Cascade Lake (GPU)) using Python 3.8.


Setup initial environment
-------------------------

```bash
PRFX=/path/to/work
cd ${PRFX}
```

Remember to change the setting for `PRFX` to a path appropriate for your Cirrus project.


Create and setup a Miniconda3 virtual environment
-------------------------------------------------

```bash
PYTHON_LABEL=py38

MINICONDA_TAG=miniconda
MINICONDA_LABEL=${MINICONDA_TAG}3
MINICONDA_TITLE=${MINICONDA_LABEL^}
MINICONDA_VERSION=4.9.2
MINICONDA_ROOT=${PRFX}/${MINICONDA_LABEL}/${MINICONDA_VERSION}-${PYTHON_LABEL}
MINICONDA_BASH_SCRIPT=${MINICONDA_TITLE}-${PYTHON_LABEL}_${MINICONDA_VERSION}-Linux-x86_64.sh

mkdir -p ${MINICONDA_LABEL}
cd ${MINICONDA_LABEL}

wget https://repo.anaconda.com/${MINICONDA_TAG}/${MINICONDA_BASH_SCRIPT}
chmod 700 ${MINICONDA_BASH_SCRIPT}
unset PYTHONPATH
bash ${MINICONDA_BASH_SCRIPT} -b -f -p ${MINICONDA_ROOT}
rm ${MINICONDA_BASH_SCRIPT}
cd ${MINICONDA_ROOT}

PATH=${MINICONDA_ROOT}/bin:${PATH}
conda init --dry-run --verbose > activate.sh
conda_env_start=`grep -n "# >>> conda initialize >>>" activate.sh | cut -d':' -f 1`
conda_env_stop=`grep -n "# <<< conda initialize <<<" activate.sh | cut -d':' -f 1`

echo "sed -n '${conda_env_start},${conda_env_stop}p' activate.sh > activate2.sh" > sed.sh
echo "sed 's/^.//' activate2.sh > activate.sh" >> sed.sh
echo "rm activate2.sh" >> sed.sh
. ./sed.sh
rm ./sed.sh

. ${MINICONDA_ROOT}/activate.sh
conda update -y -n root --all

conda deactivate
```


Build and install mpi4py using OpenMPI 4.1.0
--------------------------------------------

```bash
cd ${PRFX}

PYTHON_LABEL=py38
PYTHON_LABEL_LONG=python${PYTHON_LABEL:2:1}.${PYTHON_LABEL:3:1}

MINICONDA_VERSION=4.9.2
CUDA_VERSION=11.2
OPENMPI_VERSION=4.1.0

module load openmpi/${OPENMPI_VERSION}-cuda-${CUDA_VERSION}

MINICONDA_ROOT=${PRFX}/miniconda3/${MINICONDA_VERSION}-${PYTHON_LABEL}
. ${MINICONDA_ROOT}/activate.sh

MPI4PY_LABEL=mpi4py
MPI4PY_VERSION=3.0.3
MPI4PY_NAME=${MPI4PY_LABEL}-${MPI4PY_VERSION}

mkdir -p ${MPI4PY_LABEL}
cd ${MPI4PY_LABEL}

wget https://github.com/${MPI4PY_LABEL}/${MPI4PY_LABEL}/archive/${MPI4PY_VERSION}.tar.gz
tar -xvzf ${MPI4PY_VERSION}.tar.gz
rm ${MPI4PY_VERSION}.tar.gz

cd ${MPI4PY_NAME}

MPI4PY_ROOT=${PRFX}/${MPI4PY_LABEL}/${MPI4PY_VERSION}-ompi-${OPENMPI_VERSION}
python setup.py build
python setup.py install --prefix=${MPI4PY_ROOT}
python setup.py clean --all

echo "ROOT=${MPI4PY_ROOT}" > ${MPI4PY_ROOT}/env.sh
echo "export LIBRARY_PATH=\${ROOT}/lib:\${LIBRARY_PATH}" >> ${MPI4PY_ROOT}/env.sh
echo "export LD_LIBRARY_PATH=\${ROOT}/lib:\${LD_LIBRARY_PATH}" >> ${MPI4PY_ROOT}/env.sh
echo "export PYTHONPATH=\${ROOT}/lib/${PYTHON_LABEL_LONG}/site-packages:\${PYTHONPATH}" >> ${MPI4PY_ROOT}/env.sh
. ${MPI4PY_ROOT}/env.sh

conda deactivate
```


Install PyFR and supporting packages
------------------------------------

```bash
cd ${PRFX}

PYTHON_LABEL=py38

MINICONDA_VERSION=4.9.2
CUDA_VERSION=11.2
OPENMPI_VERSION=4.1.0
MPI4PY_VERSION=3.0.3

module load nvidia/cuda-${CUDA_VERSION}
module load nvidia/mathlibs-${CUDA_VERSION}
module load openmpi/${OPENMPI_VERSION}-cuda-${CUDA_VERSION}

MINICONDA_ROOT=${PRFX}/miniconda3/${MINICONDA_VERSION}-${PYTHON_LABEL}
. ${MINICONDA_ROOT}/activate.sh

MPI4PY_VERSION=3.0.3
MPI4PY_ROOT=${PRFX}/mpi4py/${MPI4PY_VERSION}-ompi-${OPENMPI_VERSION}
if [[ ${LD_LIBRARY_PATH} != *"${MPI4PY_ROOT}"* ]]; then
  . ${MPI4PY_ROOT}/env.sh
fi

pip install appdirs
pip install gimmik
pip install h5py
pip install mako
pip install numpy
pip install pytools

pip install pyfr==1.12.1

conda deactivate
```