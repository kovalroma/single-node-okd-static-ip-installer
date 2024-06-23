# What's this
This is an Ansible playbook that builds an *.iso file for creating a single-node OKD cluster with a static IP. By default, such an installation requires DHCP in the local network.

The automation steps are created based on the following manuals:

1. [Installing OpenShift on a single node](https://docs.okd.io/latest/installing/installing_sno/install-sno-installing-sno.html)
2. [Single Node OpenShift deployment with Static IP](https://ibm.github.io/waiops-tech-jam/blog/single-node-openshift-deployment-with-static-ip/)


## Minimum resource requirements

| vCPU         | Memory | Storage |
| ------------ | ------ | ------- |
| 6 vCPU cores | 16G    | 120G    |


# How to use

1. Install Ansible on your local machine using your favorite method.
2. Install Git.
3. Clone the repository.
4. Open `create_sno_okd_iso_file.yml` and set up variables.
5. Run Ansible with the command:
    ```
    ansible-playbook ./create_sno_okd_iso_file.yml
    ```
6. Copy the *.iso file to your hypervisor and start the machine.

7. Post boot actions. After SNO vm boot via the customized ISO file, It will become a bootstrap node in the first time, no further action to take during this period, Once bootstrap process completed, the VM will automatically reboot and become a master node. This time the master node will have the static IP setting as we've already manipulated the master.ign file. but the hostname information won't be copied. Hence the master node will started with hostname localhost. We need correct it. Log in via SSH and use the following command 
    ```
    hostnamectl set-hostname sno-test
    ```
8. Try to open your OKD web interface using the following URL path: [https://console-openshift-console.apps.<cluster_name>.<base_domain>](https://console-openshift-console.apps.<cluster_name>.<base_domain>)


# Playbook variables
Open the playbook `create_sno_okd_iso_file.yml` and edit the variables in the section:
```yaml
vars:
    okd_version: "4.15.0-0.okd-2024-03-10-010116"
    arch: "x86_64"
    ...
    ssh_key: "ssh-rsa"
```

Or you can create a file `external_vars.yml` in the main repo folder and save you variables there. In this case variables in the file will have more priority.

## Example content of external_vars.yml
```
vim external_vars.yml
```

Copy paste the following content into the file in the vim editor. 
```yaml
---
# in the above example, this would be external_vars.yml
okd_version: "4.15.0-0.okd-2024-03-10-010116"
arch: "x86_64"
base_folder: "/home/user/okd_iso_temp"
base_folder_prod: "/home/user/okd_iso"
# Cluster configuration
cluster_base_domain: "cn.ibm.com"
cluster_name: "sno-test"
cluster_cluster_networks_cidr: "10.128.0.0/14"
cluster_cluster_networks_hostprefix: "23"
cluster_service_network: "172.30.0.0/16"
# IP adress virtual machine for OKD single node
cluster_vm_machine_ip: "192.168.0.1"
# Virtual machine network adress with mask
cluster_machine_network: "192.168.0.0/24"
# Gateway ip adress virtual machine for OKD single node
cluster_vm_machine_gateway: "192.168.0.254"
# IP adress mask virtual machine for OKD single node
cluster_vm_machine_ip_mask: "255.255.255.0"
# Virtual machine interface name
cluster_vm_machine_interface_name: "eth0"
# DNS IP adress virtual machine for OKD single node
cluster_vm_machine_ip_dns: "8.8.8.8"
# Virtual machine disk path
cluster_installationdisk: "/dev/sda"
# Dummy pull secret for OKD. Fix bypass for pull-secret https://github.com/okd-project/okd/issues/182
pull_secret: '{"auths":{"fake":{"auth":"aWQ6cGFzcwo="}}}'
# SSH public key for virtual machine
ssh_key: "ssh-rsa"
is_debug: false
```

The `external_vars.yml` file, used for storing sensitive or environment-specific settings, is included in `.gitignore` to prevent it from being committed to the repository. This ensures security and allows developers to make local changes without affecting the shared codebase. It is recommended that each developer creates their own `external_vars.yml` file locally with the necessary configuration variables. Reference this file in your code as needed, but be aware it will not be tracked by Git.  

## Variable description

### OKD version for installation
`okd_version: "4.15.0-0.okd-2024-03-10-010116"` 

### OKD architecture
`arch: "x86_64"`

### Folder for bootstrapping
`base_folder: "/home/user/okd_iso_temp"`

### Folder for saving ISO and credentials 
`base_folder_prod: "/home/user/okd_iso"`

### Cluster domain name
`cluster_base_domain: "cn.ibm.com"`

### Cluster name    
`cluster_name: "sno-test"`

### Cluster network settings    
`cluster_cluster_networks_cidr: "10.128.0.0/14"`  
`cluster_cluster_networks_hostprefix: "23"`  
`cluster_service_network: "172.30.0.0/16"`
    
### IP address of the virtual machine for the OKD single node
`cluster_vm_machine_ip: "192.168.0.1"`

### Gateway IP address of the virtual machine for the OKD single node
`cluster_vm_machine_gateway: "192.168.0.254"`

### IP address mask of the virtual machine for the OKD single node
`cluster_vm_machine_ip_mask: "255.255.255.0"`

### Virtual machine interface name
`cluster_vm_machine_interface_name: "eth0"`

### DNS IP address of the virtual machine for the OKD single node
`cluster_vm_machine_ip_dns: "8.8.8.8"`

### Virtual machine disk path 
`cluster_installationdisk: "/dev/sda"`

### Dummy pull secret for OKD. Fix bypass for pull-secret 
`pull_secret: '{"auths":{"fake":{"auth":"aWQ6cGFzcwo="}}}'`

More info on [GitHub](https://github.com/okd-project/okd/issues/182)

### SSH public key for the virtual machine
`ssh_key: "ssh-rsa "`

# Debug
Log in to the running server via SSH and run
```
journalctl -b -f -u release-image.service -u bootkube.service
```

Check running containers
```
crictl ps
```


# Known Issues




# Credits
1. [Installing OpenShift on a single node](https://docs.okd.io/latest/installing/installing_sno/install-sno-installing-sno.html)
2. [Single Node OpenShift deployment with Static IP](https://ibm.github.io/waiops-tech-jam/blog/single-node-openshift-deployment-with-static-ip/)
3. Reddit user [triplewho](https://www.reddit.com/user/triplewho/) for his [comment](https://www.reddit.com/r/openshift/comments/1ddccym/comment/l85vinh/)
