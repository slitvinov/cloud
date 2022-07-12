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
Created [https://www.googleapis.com/compute/v1/projects/aphros-sim/zones/europe-west6-a/instances/aphros].
NAME    ZONE            MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP   STATUS
aphros  europe-west6-a  c2-standard-4               10.172.0.2   34.65.237.51  RUNNING
```

```
gcloud compute ssh aphros
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
