# Reprovisionable Lab
## Vars
* DOMAIN = mjsx.io
* NET-PROV = 172.24.0.0/24
* NET-ADMIN = 172.24.16.0/24
* NET-PUBLIC = 172.24.32.0/24
* {VAULTPW}
* {TEMPPW}

## Equipment
* (Optional) One temporary laptop with sufficient memory for 2 VMs
* 3x Compute Nodes
* 1x (or more) Control Node(s)
* 1x provisioning subnet
	* Non-routable
	* Used for PXE boot and provisioning
	* BMC interfaces
* 1x admin subnet
	* SSH access to servers
	* IPA traffic
	* Accessible from admin workstations
* 1x (or more) public subnets
	* Access to applications

## Control Plane (re)build
### End state
* control-0.{DOMAIN}: hosts control plane (Foreman+Katello, IPA)
	* bare-metal
	* CentOS Steam 9
	* libvirt
	* Integrated with IPA
	* Managed by Foreman+Katello on ops-1
* ops-1.{DOMAIN}: VM on control-0.{DOMAIN} with Foreman + Katello
	* CentOS Steam 9
	* Foreman + Katello
	* Integrated with IPA
		* SSO login into Foreman
		* Provision new servers auto-joined to IPA
		* DNS managed by IPA
	* Self-Managed by Foreman+Katello
* ipa-1.{DOMAIN}: VM on control-0.{DOMAIN} Node with IPA for {DOMAIN}
	* CentOS Steam 9
	* Self-Integrated with IPA
	* Managed by Foreman+Katello on ops-1
	* Authoritative DNS for {DOMAIN} and for reverse domain {NET-PROV}
* (Optional) ipa-2, ipa-3, ipa-X: VM(s) on control-X.{DOMAIN}
	* CentOS Steam 9
	* Self-Integrated with IPA
	* Managed by Foreman+Katello on ops-1
* All instances managed by Foreman + Katello on ops-1, integrated with IPA for SSO and PBAC/RBAC
* No root passwords, all access by IPA creds and ssh keys, privilege escalation by sudo
* Any passwords stored in a Vault, protected by {VAULTPW}
* Can rebuild entire control plane from Ansible playbooks, using a temporary bootstrap node and single control plane server

### Bootstrap Node => Control Node ###
_Bootstrap Node could be a compute node, a laptop, etc, that will be unused after installation_

#### Manual Steps
1. Prep the Bootstrap Node
	* FQDN unimportant, as long as reachable via SSH
	* Must have libvirt
	* Must have Ansible
	* copy (git clone) these ansible playbooks

#### Automatic Ansible
1. Deploy ipa-0 as a VM on bootstrap
	* ipa-0.{DOMAIN}
	* Bridged NIC
	* Minimal CentOS Stream 9
	* Static IP on {NET-PROV}
	* default disk layout
	* Root password = {TEMPPW}
	* allow root access via SSH
	* Install FreeIPA for {DOMAIN}
	* DNS Forward Lookup {DOMAIN}
	* DNS Reverse Lookup {NET-PROV}
2. Deploy ops-0 as a VM on bootstrap
	* ops-0.{DOMAIN}
	* Bridged NIC
	* Install CentOS Stream 9
	* Static IP on {NET-PROV}
	* default disk layout
	* Root password = {TEMPPW}
	* allow root access via SSH
	* Ansible
	* Install Foreman+Katello on ops-0
	* Sync CentOS Stream 9 content
	* Integrate Foreman with IPA
4. Provision Foreman ops-1 as VM on bootstrap
	* Static IP on {NET-PROV}
	* Integrated with IPA
	* Managed by ops-1
	* Install Foreman+Katello
	* Integrate Foreman with IPA
	* Sync CentOS Steam 9 content
5. Re-register ops-1 with self
	* Managed by ops-1
6. Provision control-0.{DOMAIN} from ops-1
	* Integrated with IPA
	* Managed by ops-1
7. Migrate ops-1 to control
8. Provision ipa-1 VM on control
	* Integrated with IPA
	* Managed by ops-1
9. Remove ipa-0
10. Eliminate bootstrap
11. Provision ipa-2
12. Establish IP addresses for ipa-1 and ipa-2 as DNS servers in Foreman DHCP

## Lab Operation
* Log in to ops.{DOMAIN} to provision Compute Nodes

# ToDo
* [Application Centric Deployments](https://docs.theforeman.org/nightly/Deploying_Hosts_AppCentric/index-katello.html)
	* `foreman-installer --enable-foreman-plugin-acd --enable-foreman-proxy-plugin-acd --enable-foreman-proxy-plugin-acd`
