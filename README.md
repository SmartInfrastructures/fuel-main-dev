# OPS-Deploy

This is the code repository for the OPS-Deploy, the management and deployment tool for FIWARE Lab nodes.
This project is part of FIWARE [1]. 

OPS-Deploy is an open source project, based on Fuel by Mirantis [2] and closely developed to the OpenStack community. It provides a web UI through that a cloud administrator can intuitively deploy and manage an OpenStack environment. 

OPS-Deploy is used in FIWARE project in order to deploy a more coherent and tested installation within the FIWARE Lab  federation guaranteeing as well as a better deployment and a more manageable issues resolution.

For any feedbacks or bug reports, please use the the github issues tool.

## Overall description
OPS-Deploy is a complex software composed by a set of Puppet [3] scripts, a task orchestrator (Nailgun [4]), a task executor (Astute [5]) and a UI. Its goal is to provide a user friendly tool for deploying a new FIWARE Lab node based on OpenStack. The tool has a double advantage: support a cloud infrastructure owner to set up a new node more quickly than a manual installation and as well as building a more coherent and tested node within the FIWARE Lab federation.
As said previously, OPS-Deploy is based on Fuel by Mirantis and obviously its architecture reflects the original structure. An high level architecture diagram is provided below. For any detailed information, please refer to the Fuel official documentation [6].

![OPD-Deploy Architecture](https://github.com/SmartInfrastructures/fuel-main-dev/blob/si/2.0/doc/source/_static/OPS-Deploy_Architecture.jpg)

In OPS-Deploy several third-party components like Cobbler, Puppet, Mcollective live together to Fuel specific components (e.g. Astute) and FIWARE’s elements (e.g. monitoring GEs ).
The original project has required some customizations or enhancements as adapt the GUI to FIWARE style guide or create the UI elements for enabling the monitoring components installation as well as to develop the installation scripts for each FIWARE component integrated.

The user is able to interact with OPS-Deploy using both GUI and CLI. They interact with Nailgun which implements REST API as well as deployment data management. It manages disk volumes configuration data, networks configuration data and any other environment specific data which are necessary for successful deployment. Astute can be viewed as composed by Nailgun's workers. Each of them runs certain actions according to the instructions provided from Nailgun. Nailgun uses SQL database to store its data and AMQP service to interact with workers.

Cobbler is used as operating system provisioning service and DHCP service provider.

Finally, Puppet is the deployment service and through MCollective agents are performed specific tasks like hard drives clearing, network connectivity probing on the discovered nodes.


### Features available
The version 2.1 of OPS-Deploy is based on the stable branch of Fuel by Mirantis version 5.1 [7]. It installs the Icehouse 2014.1.3 release of OpenStack on Ubuntu 12.04.4 and the following monitoring modules:

- Nagios 3.5.1
- Context Broker 0.13
- NGSI Adapter 1.1.1
- Event Broker 1.3.1
- OpenStackDataCollector

The monitoring node is installed whether in Multi-Node mode or in HA mode on a separate node.

#### New features
- Add nagios check for ceph
- Add nagios check for mongodb
- Resolved bugs
- Check for nova-conductor instead of nova-network when appropriate

#### Resolved bugs
-  Check for nova-conductor instead of nova-network when appropriate

For any further information, please refer to the Fuel release plan [8].

## Installation 
You can download the OPS-Deploy installer from https://github.com/SmartInfrastructures/fuel-main-dev/releases. It is distributed as an ISO image, that can be installed  using a virtualization software package, such as VirtualBox, or on a bare-metal server.
The first option is suggested only for testing scopes, whereas the second one is suggested for production environment.
When installation is completed the system will be booted. Please pay attention to remove the installation media from the master node. Finally, by the browser you can visit the page http://10.20.0.2:8000 and log in using the admin credentials (by default they are admin/admin), whereas the default admin credentials for logging in the master node are root/r00tme. It is highly recommended to change the password after you log in (using the passwd command). 

For any further information about the installation procedure,  please refer to the Fuel User Guide [9].

### Prerequisites 

For testing scope, the suggested minimum hardware requirements are:
- Dual-core CPU
- 2+ GB RAM
- 1 gigabit network port
- HDD 80 GB with dynamic disk expansion

For a production environment, the suggested minimum hardware requirements are:
- Quad-core CPU
- 4+ GB RAM
- 1 gigabit network port
- HDD 256+ GB

### Network setup
On the OPS-Deploy node (also named master node), the eth0 network interface is configured to reply to PXE requests. The default network is 10.20.0.2/24 and the gateway 10.20.0.1.
After the OPS-Deploy Master Node is installed and booted, the user can power on all slave nodes (where the user is going to install OpenStack). First of all, ensure that slave nodes are physically installed in the same network as the Master. After that, the user can boot each node in PXE boot mode (the user should enable it, modifying the BIOS boot order).

Each node sends out DHCP discovery requests and gets the response from the OPS-Deploy node that runs the DHCP server (provided by Cobbler).
When a node receives the response from the OPS-Deploy node, it fetches the pxelinux bootloader and then the bootstrap image (CentOS based Linux in memory) from the OPS-Deploy node's TFTP server and boots into it.
When this image is loaded, it reports the node's readiness and configuration to the master node. This could take a few minutes.

### Installation verification

In order to verify the correct installation of the OPS-Deploy, the user can use the following command:
*fuel --os-username admin --os-password admin release*

The answer should be as follows:

id | name                       | state     | operating_system | version
---|----------------------------|-----------|------------------|-------------
1  | Icehouse on CentOS 6.5     | available | CentOS           | 2014.1.1-5.1
2  | Icehouse on Ubuntu 12.04.4 | available | Ubuntu           | 2014.1.1-5.1


## Known issues
OPS-Deploy inherits some issues from Fuel 5.1.1. The main of them, are summarized below.

### OPS-Deploy requires a pingable default gateway in order to deploy
OPS-Deploy must be able to ping the default gateway in order to deploy the environment. If your configuration does not
include a pingable default gateway, you can work around it by specifying the Fuel Master node (or any other
pingable host) as the default gateway.
Alternatively, you can apply  [Patch 138448](https://review.openstack.org/#/c/138448) to disable the requirement to ping the default gateway. After applying this patch, you need to enable it with following sequence of steps [8].

### Deassociate floating IP button may disappear from Horizon menu

The Deassociate floating IP button may disappear from the Horizon menu when using Neutron network
topologies. You can, however, still use the Horizon UI to deassocciate IP addresses: navigate to the Project page,
then open Access&Security -> Floating IPs and deassociate the IP addresses here. See [Patch 1325575] (https://bugs.launchpad.net/bugs/1325575) .

## User manual
The user manual is available in the doc folder at https://github.com/SmartInfrastructures/fuel-main-dev/tree/si/2.1/doc.

## License
Apache License, Version 2.0, January 2004

## References

[1] FIWARE: http://www.fiware.org/

[2] Fuel by Mirantis: http://fuel.mirantis.com/

[3] Puppet: https://puppetlabs.com/

[4] Nailgun: https://docs.fuel-infra.org/fuel-dev/develop/env.html#nailgun

[5] Astute: https://docs.fuel-infra.org/fuel-dev/develop/env.html#astute

[6] Fuel Architecture: https://docs.fuel-infra.org/fuel-dev/develop/architecture.html

[7] Fuel by Mirantis 5.1: https://docs.mirantis.com/openstack/fuel/fuel-5.1/

[8] Fuel 5.1.1 release notes: https://docs.mirantis.com/openstack/fuel/fuel-5.1/release-notes.html#release-notes

[9] Fuel 5.1.1 User guide: https://docs.mirantis.com/openstack/fuel/fuel-5.1/user-guide.html
