Procedure to restrict CoreOS root partition from growing to consume the entire drive. 
#Procedure to restrict CoreOS root partition from growing to consume the entire drive

Just create a new partition during installation with a MachineConfig YAML manifest. 
LVMStorage can use this empty partition after the installation is complete to dynamically allocate storage to VMs and Pods/containers.

Instructions:
Create a Butane template and render it into a MachineConfig YAML manifest. Include the custom MachineConfig YAML manifest during the installation process (see Section 3.12 step 14)

update the device: value
update the start_mib: value - This controls how large the root "/" filesystem will be
update the size_mib: value - This controls how large your LVMStorage partition will be

more info about CoreOS partitioning at https://docs.openshift.com/container-platform/4.15/installing/installing_bare_metal/installing-bare-metal.html#installation-user-infra-machines-advanced_disk_installing-bare-metal 

more info about Butane syntax at https://coreos.github.io/butane/config-openshift-v4_14/ 

Convert to MachineConfig yaml with butane create-a-partition-for-lvmstorage.bu -o 98-create-a-partition-for-lvmstorage.yaml 


# This is the Butane template file that gets turned into the MachineConfig YAML manifest
120GiB is all that should be necessary for RHOCP & CoreOS itself, at least up to 4.15 in April 2024
we use here /dev/nvme0n1, so you need to make sure that's the same storage device you'll use on every node!
```
---
variant: openshift
version: 4.14.0
metadata:
  labels:
    machineconfiguration.openshift.io/role: master
  name: 98-create-a-partition-for-lvmstorage
storage:
  disks:
  - device: /dev/nvme0n1 
    partitions:
    - label: lvmstorage
      start_mib: 120000  # let CoreOS use the first 120GB (minimum 25000 MiB, recommend 120000 MiB or more)
      size_mib:  0       # any size, or "0" to use the rest of the disk
```
# change the label from "worker" to "master" for SNO (single node OpenShift) or 2 or 3 node clusters that are combo master/worker
# COPY THIS FINAL CODE BLOCK INTO THE ASSISTED INSTALLER
```
---
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  labels:
    machineconfiguration.openshift.io/role: master
  name: 98-create-a-partition-for-lvmstorage
spec:
  config:
    ignition:
      version: 3.4.0
    storage:
      disks:
        - device: /dev/nvme0n1
          partitions:
            - label: lvmstorage
              sizeMiB: 0
              startMiB: 120000
```

