# Deploy vSphere cluster on FlashArray with Ansible

A simple although complete example to deploy a vSphere cluster from the ground, totally automated by Ansible.

The collection of playbooks and roles allows to deploy a vSphere cluster of ESXi hosts with the dedicated vcenter and backed by a FlashArray connected to the hosts by a FC or iSCSI SAN.

## What is needed

Single cluster: A minimum of two physical hosts.

Management cluster and workload clusters: A minimum of two physical hosts for the management cluster and a further set of two/three hosts for each additional workload cluster.

Storage: one FlashArray connected to the servers to be installed through a FC or iSCSI fabric.

Networking: A minimum of an access/service LAN connecting the hosts on which the clusters have to be installed. A real scenario requires redundant networks.

A Linux server for the bootstrap/installation node. Ansible will have to be installed on the node, together with the necessary Galaxy collections. The node must be connected to the management LAN of each vSphere cluster to be installed. A DHCP server must exists on those LANs and the boostrap server can be configured for that purpose. 

## Prepare bootstrap/management server

Install Ansible. Refer to the official documentation and choose your preferred method for installing Ansible.
Install the geerlingguy docker and pip roles from the Ansible Galaxy

ansible-galaxy install geerlingguy.docker
ansible-galaxy install geerlingguy.pip


Install the VMware Galaxy Community modules. At present the modules that include specific operations, like vVols datastore management, VMFS datastore importing and resignaturing, and registering virtual machines from a datastore are not yet available from the official Galaxy repository, therefore you may want to instal those from the GitHub repository of the PR:

ansible-galaxy install git+https://github.com:genegr/community.vmware.git,bef2b3ce601b318931e3615648390519cc7a7bba

If you prefer the current official release, install it with the following instead.

ansible-galaxy install community.vmware

Install the necessary dependecies

pip install -r .ansible/roles/community.vmware/community.vmware/requirements.txt

Install Pure Storage FlashArray Galaxy modules

ansible-galaxy collection install purestorage.flasharray

Complete the installation with the required Python libraries

pip install -r 

If you get the error

ERROR: Could not find a version that satisfies the requirement python>=3.6 (from versions: none)
ERROR: No matching distribution found for python>=3.6

find the line python>="3.6" in the file ~/.ansible/collections/ansible_collections/purestorage/flasharray/requirements.txt 

and replace it with python_version>="3.6

Install community.docker collection

ansible-galaxy collection install community.docker




The boostrap host uses containerized images of the dnsmasq DHCP/Boot server and the nginx http install server, therefore you need to deploy docker on the host. Run the simple docker.yaml playbook for that:

ansible-playbook -i inventory_wkld01.yaml docker.yaml

If you use Ansible vault to protect your sensitive information in the inventory file(s) (and you should), then run

ansible-playbook --ask-vault-pass  -i inventory_wkld01.yaml docker.yaml

The installation process of the ESXi hosts uses PXE/iPXE booting, therefore it is necessary to download both the base PXE firmware and the iPXE firmware (which has to be built from sources).
Download the latest syslinux tarball from kernel.org
https://mirrors.edge.kernel.org/pub/linux/utils/boot/syslinux. The current release can be found here https://mirrors.edge.kernel.org/pub/linux/utils/boot/syslinux/6.xx/syslinux-6.03.tar.gz. Place the tarball in a folder of your choice and set the corresponding variable in the ansible inventory file

syslinux_tarball: /path/to/syslinux-tarball



Build iPXE

Set the desination directory for the iPXE binaries in the inventory file

ipxe_compile_dir: /path/to/ipxe-build-dir

then use the provided ansible playbook to build the necessary iPXE firmware

ansible-playbook -i inventory ipxe-build.yaml


Download the necessary ISO images for the ESXi and VCSA and place those in a directory of your choice, the set the related variables in the inventory file according to those locations.

os_iso: /path/of/VMware-VMvisor-Installer-version.iso
vcsa_iso: /path/of/VMware-VCSA-all-version.iso


Prepare ESXi installation files
This step prepares the ESXi boot environment, essentially creating the files for the DHCP and HTTP server from the previously downloaded ISO DVD images and the startup firmware, using the PXE installation procedure.

Configure the following variables in your inventory file


