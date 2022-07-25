# cloud

- https://cloud.google.com/solutions/hpc
- https://cloud.google.com/compute/docs/instances/create-hpc-vm
- https://cloud.google.com/architecture/parallel-file-systems-for-hpc
- https://cloud.google.com/hpc-toolkit/docs/quickstarts/slurm-cluster
- https://www.intel.com/content/www/us/en/high-performance-computing/hpc-platform-specification.html
- Rahn, Tobias. Noise Estimation in HPC Cloud Networks. BS thesis. ETH Zurich, 2021.
  https://www.research-collection.ethz.ch/handle/20.500.11850/513171
- https://www.intel.com/content/www/us/en/develop/documentation/installation-guide-for-intel-oneapi-toolkits-linux/top/installation/install-using-package-managers/yum-dnf-zypper.html
- https://github.com/GoogleCloudPlatform/hpc-toolkit.git

Enable Compute Engine API
- https://console.developers.google.com/apis/api/compute.googleapis.com/overview?project=400748927902

Enable Cloud Filestore API
https://console.cloud.google.com/apis/library/file.googleapis.com?project=aphros-sim

Create a single node instanse
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

Work with an instance
```
gcloud compute instances list
gcloud compute instances delete aphros
gcloud compute instances stop aphros
gcloud compute instances start aphros
```

After login to aphros instance
```
. ~/.local/bin/ap.setenv
. /opt/intel/oneapi/setvars.sh
```

ssh to an instance
```
gcloud compute ssh aphros --zone=europe-west6-a
```

or

```
$ cat .ssh/config
Host gc
     HostName 34.65.212.137
     ForwardX11Trusted yes
     ForwardX11 yes
     IdentityFile ~/.ssh/google_compute_engine
     StrictHostKeyChecking=no
     UserKnownHostsFile=/dev/null
```

Intall intell MPI
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

Compile aphros without cmake
```
git clone https://github.com/cselab/aphros.git --branch icc
cd aphros/src
../make/bootstrap
make -j4 -k -f Makefile_legacy APHROS_PREFIX=$HOME/.local USE_MPI=1 USE_HDF=0 USE_OPENCL=0 USE_AVX=1 USE_OPENMP=1 CXX=mpiicpc CC=icc
```

Compile aphros with cmake
Install cmake
```
wget https://github.com/Kitware/CMake/releases/download/v3.24.0-rc3/cmake-3.24.0-rc3.tar.gz
tar zxf cmake-3.24.0-rc3.tar.gz
cd cmake-3.24.0-rc3
sudo yum install openssl-devel -y
./bootstrap
make -j4
sudo make install
```

```
cd aphros/deploy
./install_setenv $HOME/.aphros
. $HOME/.local/bin/ap.setenv
mkdir build
cd build
cmake ..
make -j4
make install
```

```
cd aphros/src
mkdir build
cd build
cmake .. -DUSE_HDF=0 -DUSE_BACKEND_CUBISM=1 -DUSE_BACKEND_LOCAL=1 -DUSE_BACKEND_NATIVE=1 -DMPI_C_COMPILER=mpiicc -DMPI_CXX_COMPILER=mpiicpc
make -j 4
make install
CC=icc CXX=icpc make test
```

Install `ghpc`
- https://cloud.google.com/hpc-toolkit/docs/setup/install-dependencies

```
wget -O - https://apt.releases.hashicorp.com/gpg | gpg --dearmor > /tmp/x
sudo mv /tmp/x /usr/share/keyrings/hashicorp-archive-keyring.gp
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update
sudo apt install terraform
sudo apt install packer
```

```
git clone git@github.com:GoogleCloudPlatform/hpc-toolkit.git
```

```
ghpc create aphros.yaml -w
terraform -chdir=aphros/primary init
terraform -chdir=aphros/primary validate
terraform -chdir=aphros/primary plan -no-color
terraform -chdir=aphros/primary apply -no-color
```

Destroy the cluster
```
terraform -chdir=aphros/primary destroy -no-color
```
