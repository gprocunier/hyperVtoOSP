# VM Migration from Hyper-V to OpenStack Guide

This guide outlines the process for migrating a virtual machine (VM) from Microsoft Hyper-V to an OpenStack environment.

## Prerequisites
- [Certified Guest Operating Systems in Red Hat OpenStack Platform](https://access.redhat.com/articles/973163)
- [virtio-win drivers installation guide](https://github.com/)
- [Cloudbase cloud-init for Windows page](https://github.com/cloudbase/cloudbase-init)
- [Windows Instances Cloud Init Guide](https://github.com/gprocunier/dcib/blob/main/openstack/misc/windows-instances-cloud-init.md)
---

- Access to the Hyper-V environment and the OpenStack environment.
- Installation of `virtio-win` drivers on the Windows VM for KVM compatibility.
- Tools for VM conversion (like `qemu-img`).

## Process Overview

1. **Prep-work for the VM in Hyper-V**
2. **Export the VM Disk from Hyper-V**
3. **Convert the VM Disk to a Compatible Format**
4. **Upload the Converted Image to OpenStack**
5. **Create a New Instance in OpenStack Using the Uploaded Image**

## Detailed Steps

### Windows Team - Preparation
### 1. Download the virtio-win driver iso and run the install MSI in the VM being migrated.
- ![Splash Screen](https://github.com/gprocunier/hyperVtoOSP/blob/main/images/virtio-win-install.png?raw=true)
- Select the default options for the driver components and select YES when the unsigned driver prompt appears.  Please see the unsigned public repo information available here: https://github.com/virtio-win/virtio-win-pkg-scripts/blob/master/README.md#virtio-win-driver-signatures
- ![Unsigned install](https://github.com/gprocunier/hyperVtoOSP/blob/main/images/virtio-win-unsigned.png?raw=true)

### 2. Download the cloudbase-init install msi and run it in the VM being migrated.
- ![Splash Screen](https://github.com/gprocunier/hyperVtoOSP/blob/main/images/cloudbase-install.png?raw=true)
- ![Service Configuration](https://github.com/gprocunier/hyperVtoOSP/blob/main/images/cloudbase-config.png?raw=true)
- This configures the user that cloudbase-init will install and use.  The purpose of this account is to create control linkage between OpenStack and the windows instance to provide first-time-boot configuration "as required".
- ![Installation Finish](https://github.com/gprocunier/hyperVtoOSP/blob/main/images/cloudbase-finish.png?raw=true)
- If the VM is going to be used as a general purpose template for subsequent VMs then both sysprep and shutdown after installation should be selected.  If the vm is just being migrated as-is then both can be left unchecked.

### 3. Record the VM NIC MAC / IP / VLANs, denote which one is the primary nic

### Windows Team - Steps to export the VM Disk
### 1. Export the VM Disk from Hyper-V
- In Hyper-V Manager, right-click the VM and select "Export."
- Choose a location to save the exported files.
    - Provide Unix team the source.vhdx for conversion
- Alternatively:
  - To export VM via PowerShell, type the command below:
```bash
Export-VM -Name \<vm name\> -Path \<path\>
```

### Unix Team - 1. Convert the VM Disk to a Compatible Format

- Use `qemu-img` or similar tool to convert the VM disk to a RAW format.

  ```bash
  qemu-img convert -p -f vhdx -O raw source.vhdx destination.raw
  ```

### Unix Team - 2. Upload the Converted Image to OpenStack

- Use the OpenStack CLI or Horizon dashboard to upload the QCOW2 image.

  ```bash
  openstack image create "VM_Name" --file destination.qcow2 --disk-format raw --container-format bare --public
  ```

### Unix Team - 3. Create a New Instance in OpenStack Using the Uploaded Image

- Launch a new instance in OpenStack using the uploaded image.

  ```bash
  openstack server create --image VM_Name --flavor b1.medium --network private_network VM_Instance_Name
  ```

### Additional Resources for Windows Instances and Cloud Init

- For details on Windows guest bootstrap and cloud-init, see the [Windows Instances Cloud Init Guide](https://github.com/gprocunier/dcib/blob/main/openstack/misc/windows-instances-cloud-init.md).

## Post-Migration Checks

- After migration, verify the functionality of the VM in the OpenStack environment.

## Troubleshooting

- Ensure `virtio` drivers and cloudbase-init are correctly installed for proper functionality.
