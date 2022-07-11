# cloud

- https://cloud.google.com/compute/docs/instances/create-hpc-vm
- https://www.intel.com/content/www/us/en/high-performance-computing/hpc-platform-specification.html

Enable Compute Engine API
- https://console.developers.google.com/apis/api/compute.googleapis.com/overview?project=400748927902

```
gcloud compute instances create aphros \
        --zone=europe-west6-a \
        --image-family=hpc-centos-7 \
        --image-project=cloud-hpc-image-public \
        --maintenance-policy=TERMINATE \
        --machine-type=c2-standard-4
```

```
Created [https://www.googleapis.com/compute/v1/projects/aphros-sim/zones/europe-west6-a/instances/aphros].
NAME    ZONE            MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP   STATUS
aphros  europe-west6-a  c2-standard-4               10.172.0.2   34.65.237.51  RUNNING
```