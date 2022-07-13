# cloud

- https://cloud.google.com/compute/docs/instances/create-hpc-vm
- https://www.intel.com/content/www/us/en/high-performance-computing/hpc-platform-specification.html
- Rahn, Tobias. Noise Estimation in HPC Cloud Networks. BS thesis. ETH Zurich, 2021.
  https://www.research-collection.ethz.ch/handle/20.500.11850/513171
- https://www.intel.com/content/www/us/en/develop/documentation/installation-guide-for-intel-oneapi-toolkits-linux/top/installation/install-using-package-managers/yum-dnf-zypper.html

Enable Compute Engine API
- https://console.developers.google.com/apis/api/compute.googleapis.com/overview?project=400748927902

```
gcloud compute instances create aphros \
        --boot-disk-size=40GB \
        --image-family=hpc-centos-7 \
        --image-project=cloud-hpc-image-public \
        --machine-type=c2-standard-4 \
        --maintenance-policy=TERMINATE \
        --metadata=google_install_mpi=--intel_mpi \
        --zone=europe-west6-a \
#
```

```
gcloud compute instances delete aphros
```

```
gcloud compute ssh aphros --zone=europe-west6-a
```

```
cat >/tmp/oneAPI.repo << '!'
[oneAPI]
name=Intel® oneAPI repository
baseurl=https://yum.repos.intel.com/oneapi
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://yum.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB
!
sudo mv /tmp/oneAPI.repo /etc/yum.repos.d
sudo yum install intel-basekit intel-hpckit -y
. /opt/intel/oneapi/setvars.sh
```

```
git clone https://github.com/cselab/aphros.git --branch icc
cd aphros/src
../make/bootstrap
make -j4 -k -f Makefile_legacy APHROS_PREFIX=$HOME/.local USE_MPI=1 USE_HDF=0 USE_OPENCL=0 USE_AVX=1 USE_OPENMP=1 CXX=mpiicpc CC=icc
```

```
cd aphros/deploy
./install_setenv $HOME/.aphros
. $HOME/.local/bin/ap.setenv
mkdir build
cd build
cmake .. -DUSE_HYPRE=0
make -j4
make install
```

```
cd aphros/src
mkdir build
cd build
cmake .. -DUSE_HDF=0 -DUSE_BACKEND_CUBISM=1 -DUSE_BACKEND_LOCAL=1 -DUSE_BACKEND_NATIVE=1 -DUSE_HYPRE=0 -DMPI_C_COMPILER=mpiicc -DMPI_CXX_COMPILER=mpiicpc
make -j 4
make install
make test
```

```
$ cat .ssh/config
Host gc
     HostName 34.65.237.51
     IdentityFile ~/.ssh/google_compute_engine
     UserKnownHostsFile=/dev/null
     StrictHostKeyChecking=no
```

Install cmake
```
wget https://github.com/Kitware/CMake/releases/download/v3.24.0-rc3/cmake-3.24.0-rc3.tar.gz
tar zxf cmake-3.24.0-rc3.tar.gz
cd cmake-3.24.0-rc3
./bootstrap -- -DCMAKE_USE_OPENSSL=OFF
make -j4
sudo make install
```
