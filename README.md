# FlexPod Converged Infrastructure setup using Ansible for FlexPod Datacenter with Cisco UCS M7 IMM, VMware vSphere 8.0, and NetApp ONTAP 9.16.1 for SAP HANA TDI implementations 

Note that the scripts in this repository have now been successfully tested with NetApp ONTAP 9.16.1, Cisco Nexus NXOS 10.4(4), and Cisco UCS 4.3(5).

This repository for FlexPod contains Ansible playbooks to configure Cisco Nexus, Cisco UCS Intersight, and NetApp ONTAP. This repository can be used for setting up Cisco devices and NetApp ONTAP Storage in preparation for SAP HANA Tailored Datacenter Integration (TDI) based implementation as covered in the following Cisco Validated Design (CVD): [https://www.cisco.com/c/en/us/td/docs/unified_computing/ucs/UCS_CVDs/flexpod_saphanatdi_iac_deploy.html.](https://www.cisco.com/c/en/us/td/docs/unified_computing/ucs/UCS_CVDs/flexpod_sap_hana_tdi_m7.html)

The CVD lays out the complete process for configuring the FlexPod OpenShift tenant using Ansible. As a prerequisite to this installation FlexPod Base should be installed on the FlexPod using https://www.cisco.com/c/en/us/td/docs/unified_computing/ucs/UCS_CVDs/flexpod_base_imm_m7_iac.html. Since these playbooks are intended to save time in setting up a working FlexPod, a complete FlexPod as shown below is needed to execute the playbooks. The Cisco FC and NFS solution based FlexPod as shown below was used to validate these Ansible scripts but FlexPods with NFS only setup are also supported.

![block-diagram](https://github.com/ucs-compute-solutions/FlexPod-M7-SAPHANA/blob/main/ReadmePics/Main-Topology.jpg)

# Set up the execution environment

To execute various ansible playbooks, a linux based system will need to be setup as described in the CVD with the packages listed at the following pages:

- Cisco UCS Intersight: https://galaxy.ansible.com/ui/repo/published/cisco/intersight/
- Cisco NxOS: https://galaxy.ansible.com/ui/repo/published/cisco/nxos/
- NetApp ONTAP: https://galaxy.ansible.com/ui/repo/published/netapp/ontap/
- VMware: https://galaxy.ansible.com/ui/repo/published/community/vmware/

# How to execute these playbooks?

![block-diagram](https://github.com/ucs-compute-solutions/FlexPod-M7-SAPHANA/blob/main/ReadmePics/Ansible-Order.jpg)

The Ansible playbooks address preparation of Intel M7 based Intersight Managed servers to host both single-host and mulit-host SAP HANA systems either virtualized or bare metal installed. Supports server booting either via FC or iSCSI from SAN. 
Because a number of manual tasks need to be executed between running the Ansible playbooks, the CVD document should be used as a guide for running the playbooks. Commentary is included in the variable files to guide filling in those values.

The steps for setting up a FlexPod with FC, NF and iSCSI (booting-only) storage protocols are:

1.  Create a directory and clone the repository from Github with "git clone https://github.com/ucs-compute-solutions/FlexPod-M7-SAPHANA.git".
2.  Fill in the variable files according to the CVD.
3.  Execute the Nexus playbook with "ansible-playbook ./Setup_Nexus.yml -i inventory" to setup the Nexus switches.
4.  Execute the NetApp storage playbook with "ansible-playbook -i inventory Setup_ONTAP.yml -t ontap_config_part_1" to set the base Infra-SVM config.
5.  Query the Infra-SVM iSCSI IQN and FC LIFs and add to the "all.yml" file.
6.  Follow the manual steps in the CVD to create an Intersight Organization and Resource Group.
7.  Execute the IMM playbooks with "ansible-playbook ./Setup_IMM_Pools.yml", "ansible-playbook ./Setup_IMM_Server_Policies.yml", and "ansible-playbook ./Setup_IMM_Server_Profile_Templates.yml" to setup the Cisco UCS Server Profile Pools, Policies, and Templates to address VMware virtualized SAP HANA implementation.
8.  Additionally/OR - Execute the IMM playbooks with, "ansible-playbook ./Setup_IMM_Server_Policies_Baremetal.yml", and "ansible-playbook ./Setup_IMM_Server_Profile_Templates_Baremetal.yml" to setup the Cisco UCS Server Profile Policies and Templated to address bare metal SAP HANA implementation.
9.  Follow the manual steps in the CVD to create UCS IMM server profiles VMware virtualized or bare metal HANA nodes.
10. Query the ESXi/bare metal host IQNs or WWPNs from the server profiles and add to the "all.yml" file.
11. If configuring Fibre Channel, execute the MDS playbook with "ansible-playbook ./Setup_MDS.yml -i inventory".
12. Execute the NetApp storage playbook with "ansible-playbook -i inventory Setup_ONTAP.yml -t ontap_config_part_2" to create and map the ESXi/bare metal boot LUNs from the Infra-SVM.
13. If implementing virtualized SAP HANA, follow the manual steps in the CVD to install VMware ESXi and assign IPs on the three (or more) host servers using Cisco Intersight OS Install.
14. If implementing virtualized SAP HANA, execute the ESXi playbook with "ansible-playbook ./Setup_ESXi.yml -i inventory" to setup the ESXi hosts.
15. If implementing virtualized SAP HANA, bring a vCenter into the environment by either installing vCenter on the first ESXi host according to the CVD, copying it in, or establishing L3 routing to it.
16. If implementing virtualized SAP HANA, setup the vCenter and the configured ESXi hosts to it by executing "ansible-playbook ./Setup_vCenter.yml -i inventory".
17. If implementing virtualized SAP HANA, follow the manual steps in the CVD to complete setting up vCenter and the ESXi hosts and subsequent SAP HANA node VM preparation steps - appropriate OS and post configuration pre-requisites for SAP HANA installation.
18. If implementing bare metal SAP HANA, follow the manual steps in the CVD to install bare metal nodes with appropriate OS and post configuration pre-requisites for SAP HANA installation.
19. Execute the NetApp storage playbook for SAP HANA specific HANA-SVM configuration with "ansible-playbook -i inventory Setup_ONTAP.yml" to create HANA-SVM with all required elements for SAP HANA persistance storage configuration.
11. If configuring Fibre Channel, execute the tenant specifc MDS playbook with "ansible-playbook ./Setup_MDS.yml -i inventory" to create the HANA database specific persistence zoning to enable its presentation to the intended nodes.
20. Follow the manual steps in the CVD to prepared the nodes to host SAP HANA persistence partitions.
21. Follow the manual steps in the CVD to install SAP HANA.
